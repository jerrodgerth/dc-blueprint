# Fabric Management

## Logging

### Overview

Logging capabilities within SR Linux are built on `rsyslogd` which is a well known and highly utilized Linux logging system which allows for log filtering as well as saving logs to file, memory or sending logs to remote servers.
The primary configuration file is `/etc/rsyslog.conf` and other Linux applications can extend the configuration file by providing their own configuration files in `/etc/rsyslog.d/`.

SR Linux creates a minimal primary config which enables default Linux logs.
SR Linux also creates its own configuration file within `/etc/rsyslog.d/` and this file is modified via the SR Linux YANG models (via CLI, gNMI etc).
Operators may create their own filters and actions as they see fit however if an operator modifies sections of this file owned by SR Linux, SR Linux will overwrite them with the YANG configs.

### Log Destinations

SRLinux supports four log destinations: buffer, console, file and remote server.

#### Buffer

* In order to reduce disk I/O, a file like log destination can be used called **buffer**
* When SR Linux boots it creates a non swapable `tmpfs` located at `/var/log/srlinux/buffer`
    * This tmpfs resides entirely in memory and is hard coded to 512MB of RAM
* When a buffer log destination is configured and committed:
    * SR Linux will calculate the available space based on other buffer retention policies
    * If not, enough space is available the commit will fail
* Because this is a `tmpfs` in memory, if the node is rebooted the buffered logs will be lost
* Buffers can be configured to be persistent:
    * When the `persist` leaf is enabled the buffered log can be copied to disk
    * The `persist` value in seconds defines how frequently the log is copied to the disk
    * When a buffer rolls over, it is automatically copied over to the disk

#### Files

* As the name suggests log a file log destination will save logs to the disk as a file
* By default, if no file location is explicitly specified the default file location is `/var/log/srlinux/file/<filename>`
    * If the default location is changed, it is up the operator to ensure there is enough capacity for the file.
* Rotate: Number of files to keep in rotation when a maximum file size is reached.
* Size: Number of bytes an individual output file cannot exceed.
* Input: Subsystem or facility.
* Format: Text format of the output syslog messages, in legacy syslog $template style
* Directory: Fully qualified path of a directory where the log file(s) shall be maintained.

#### Console

* Logs will be sent to `/dev/console` which is usually the console port on the CPM
* Input: Subsystem or facility
* Format: Text format of the output syslog messages, in legacy syslog $template style

#### Remote server

* Logs can be sent to a remote syslog server
    * A single network-instance can be specified under the `/system/logging` node to choose which ip-vrf should be used to reach the logging servers
* Input: Subsystem or facility
* Remote-port: Transport port for syslog to use for messages sent to a remote server (default syslog port is 514)
* Transport: Transport protocol for syslog to use for messages sent to a remote server (UDP or TCP)

### Tracing

SR Linux supports the concept of enabling protocol specific traces.
Traces are used to debug a specific protocol, typically on demand during a troubleshooting session.
When enabled under a given protocol, traces logs will be available as an input under the corresponding subsystem with a priority of `debug`.
A destination or output of the logs must then be configured to capture the trace logs, setting the input as the corresponding subsystem matching exact debug (for example).
As a trace is nothing more than a specific input, all destinations previously discussed are suitable outputs.

