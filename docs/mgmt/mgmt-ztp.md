# Fabric Management

## Zero Touch Provisioning

### Overview

Zero Touch Provisioning (ZTP) is a mechanism/process used to initialize and configure a device without the need for administrator intervention.
A device undergoing ZTP has the ability to be powered on, become operational and remotely configurable without the need for any administrator to pre-provision or  pre-configure the device.​

In SR Linux, the ZTP process is handled by a ZTP application which is started as a service by the Linux OS.​
The ZTP application has a set of actions that can be manually executed via a ZTP CLI by an administrator or executed automatically at boot based on configurable ZTP options.​
The option which facilitates the traditional “zero touch” aspect of ZTP in SR Linux is called Autoboot.
Autoboot is enabled by default from the factory.​
Lastly, the ZTP application is also responsible for starting the `srlinux` service which will start `app_mgr` and in turn start SR Linux applications.​


### ZTP Process

This is the ZTP process based on factory-enabled options:​

1. The DUT’s grub bootloader loads the Linux OS from the internal SD card
2. ZTP application is started as service during boot of Linux OS
3. DHCPv4/v6 started on the mgmt interface of the active CPM (chassis serial # sent to DHCP server to identify the DUT)
4. DHCP server provides an IP address as well as the URL of the python provisioning script
5. DUT executes the provisioning script, the script may upgrade the DUT, apply a config, install packages etc.
6. Upon successful completion of provisioning script ZTP starts the srlinux service
7. SR Linux `app_mgr` starts and begins the application boot sequence


### Required Components

* DHCP server (IPv4 or IPv6) – To support the assignment of IP addresses through DHCP requests and offers.
* File server – for staging and transfer of RPMs, configurations, images, and scripts. HTTP, HTTPS, FTP and TFTP are supported.
(For HTTPS, the default Mozilla certificate should be used.)
* DHCP relay – required if the server is outside the management interface broadcast domain.


### Support Deployment Models

* Nodes, HTTP file servers, and DHCP server in the same subnet (**Figure 4-1**)

<figure>
  <img src="/_images/fig-04-01.png" width="600" />
  <figcaption>Figure 4-1</figcaption>
</figure>

* HTTP file servers and DHCP server in the same subnet, separate from the nodes (**Figure 4-2**)

<figure>
  <img src="/_images/fig-04-02.png" width="600" />
  <figcaption>Figure 4-2</figcaption>
</figure>

* Nodes, HTTP file servers, and DHCP server in different subnets (**Figure 4-3**)

<figure>
  <img src="/_images/fig-04-03.png" width="600" />
  <figcaption>Figure 4-3</figcaption>
</figure>


### Sample Python Provisioning Script 


```python
import errno
import os
import sys
import signal
import subprocess
from subprocess import Popen, PIPE
import threading
import commands
import time

folder = '21.3.1-410'
version = '21.3.1-410'

srlinux_image_url = 'http://192.168.1.10/load/{}/srlinux-{}.bin'.format(folder, version)
srlinux_image_md5_url = 'http://192.168.1.10/load/{}/srlinux-{}.bin.md5'.format(folder, version)
srlinux_config_url = 'http://192.168.1.10/configs/ztp/generic_mgmt.json'

intf = 'mgmt0'

class ProcessError(Exception):
    def __init__(self, msg, errno=-1):
        Exception.__init__(self, msg)
        self.errno = errno

class ProcessOpen(Popen):
    def __init__(self, cmd, cwd=None, env=None, flags=None, stdin=None,
        stdout=None, stderr=None, universal_newlines=True,):
        self.__use_killpg = False
        shell = False
        if not isinstance(cmd, (list, tuple)):
            shell = True
        # Set flags to 0, subprocess raises an exception otherwise.
        flags = 0
        # Set a preexec function, this will make the sub-process create it's
        # own session and process group - bug 80651, bug 85693.
        preexec_fn = os.setsid
        self.__cmd = cmd
        self.__retval = None
        self.__hasTerminated = threading.Condition()
        Popen.__init__(self, cmd, cwd=cwd, env=env, shell=shell, stdin=stdin,
           stdout=PIPE, stderr=PIPE, close_fds=True, universal_newlines=universal_newlines,
           creationflags=flags,)
        print("Process [{}] pid [{}]".format(cmd, self.pid))

    def _getReturncode(self):
        return self.__returncode

    def __finalize(self):
        # Any finalize actions
        pass

    def _setReturncode(self, value):
        self.__returncode = value
        if value is not None:
            # Notify that the process is done.
            self.__hasTerminated.acquire()
            self.__hasTerminated.notifyAll()
            self.__hasTerminated.release()

    returncode = property(fget=_getReturncode, fset=_setReturncode)

    def _getRetval(self):
        # Ensure the returncode is set by subprocess if the process is finished.
        self.poll()
        return self.returncode

    retval = property(fget=_getRetval)

    def wait_for(self, timeout=None):
        if timeout is None or timeout < 0:
            # Use the parent call.
            try:
                out, err = self.communicate()
                self.__finalize()
                return self.returncode, out, err
            except OSError as ex:
                # If the process has already ended, that is fine. This is
                # possible when wait is called from a different thread.
                if ex.errno != 10:  # No child process
                    raise
                return self.returncode, "", ""
        try:
            out, err = self.communicate(timeout=timeout)
            self.__finalize()
            return self.returncode, out, err
        except subprocess.TimeoutExpired:
            self.__finalize()
            raise ProcessError(
                "Process timeout: waited %d seconds, "
                "process not yet finished." % (timeout)
            )

    def kill(self, exitCode=-1, sig=None):
        if sig is None:
            sig = signal.SIGKILL
        try:
            if self.__use_killpg:
                os.killpg(self.pid, sig)
            else:
                os.kill(self.pid, sig)
        except OSError as ex:
            self.__finalize()
            if ex.errno != 3:
                # Ignore:   OSError: [Errno 3] No such process
                raise
        self.returncode = exitCode
        self.__finalize()

    def commandline(self):
        """returns string of command line"""
        if isinstance(self.__cmd, six.string):
            return self.__cmd
        return subprocess.list2cmdline(self.__cmd)

    __str__ = commandline

def execute_and_out(command, timeout=None):
    print("Executing command: {}".format(command))
    process = ProcessOpen(command)
    try:
        #logger.trace("Timeout = {}".format(timeout))
        ret, out, err = process.wait_for(timeout=timeout)
        return ret, out, err
    except ProcessError:
        print("{} command timeout".format(command))
        process.kill()
        return errno.ETIMEDOUT, "", ""

def execute(command, timeout=None):
    ret, _, _ = execute_and_out(command, timeout=timeout)
    return ret

def srlinux():
    set_downgrade()
    nos_install()

def set_downgrade():
    cmd = 'ztp option downgrade --status enable'
    ret,out,err = execute_and_out(cmd)

def post_tasks():
    nos_configure()

def nos_install():
    cmd = 'ztp image upgrade --imageurl {} --md5url {}'.format(srlinux_image_url, srlinux_image_md5_url)
    ret,out,err = execute_and_out(cmd)

def nos_configure():
    cmd = 'ztp configure-nos --configurl {}'.format(srlinux_config_url)
    ret,out,err = execute_and_out(cmd)

def main():
    srlinux()
    # post_tasks()

if __name__ == '__main__':
    main()
```
