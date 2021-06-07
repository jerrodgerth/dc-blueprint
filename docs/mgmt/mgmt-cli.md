# Fabric Management

## CLI

### Overview

SR Linux provides a transaction-based CLI written in Python.
The CLI reads directly from the YANG data models and provides the same information available via gNMI or JSON-RPC.
Being transaction-based, the CLI allows for a number of changes to be made to the configuration before requiring an explicit commit from the operator before applying it.
The default location for the configuration file is `/etc/opt/srlinux/config.json`.
Upon boot up this config file is loaded by SR Linux `mgmt_server` when it starts and publishes content to IDB for other applications to consume.

!!!warning
    The CLI should not be the primary method for automation or scripting.  Rather it is suggested to use either the gNMI or JSON-RPC interfaces.

If there is no configuration file present, a basic configuration file is auto-generated with the following values:

* Creation of a management network instance
* Management interface is added to the mgmt network instance
* DHCP v4/v6 is enabled on mgmt interface
* A set of default of logs are created, most in memory – one catch all in a file called messages
* SSH server is enabled
* Some default IPv4/v6 CPM filters

#### Datastores

SR Linux has three datastores that are exposed as modes within the CLI.

* **Running:** the default mode when logging in and displays the intended configuration currently active on the device
* **State:** the running configuration plus the addition of any dynamically added data.  Some examples of state specific data are operational state of various elements, counters and statistics, BGP auto-discovered peer, LLDP peer information, etc.
* **Candidate:** without any modification this datastore is equivalent to the running datastore. Any changes made to this datastore require a `commit` before the changes go through YANG and application validation and then are applied to the device. This datastore has two modes currently:
    * *Shared Candidate:* this is the default mode when entering the candidate mode in the CLI.  This allows multiple users on the device to modify the same configuration concurrently and therefor when any of the users perform a commit it applies ALL changes from all users in the shared candidate mode.
    * *Exclusive Candidate:* this mode must be explicitly specified when entering the candidate datastore.  If an operator chooses to start an exclusive candidate session no other users may enter the candidate datastore to make changes.
        * If there is an active shared candidate (uncommitted change in shared candidate) then the user is not allowed to enter in an exclusive mode.  Likewise if there is an active exclusive candidate.
        * A user can clear an active session using a ‘tools’ command.

!!!note
    gNMI & JSON-RPC both use an exclusive candidate and an implicit commit when making a configuration change on the device.

#### Validation

When a configuration is committed from any of the configuration modes, the configuration goes through three sets of validation:

1. YANG validation.
This validation confirms parent/child syntax is present and that any references exist.
YANG validation is not state or resource aware and therefore limited, however, it does provide a first level of validation (it is also the most performant type of validation – so is used in all cases where available).
Any validation failure will result in the commit not being accepted and no configuration will be applied.
2. Application validation.
Each application receives a copy of the intended configuration and performs a test apply of the configuration.
Each application will do its own validation and will not impact current system operation.
If application validation fails, the commit fails and no configuration will be applied.	
3. Forwarding plane validation.
If the forwarding plane is unable to accept the configuration then a graceful roll-back is done immediately.
Examples are exhaustion of resources or the hardware cannot be programmed – this will result in a commit failure.

#### Checkpoints

A configuration *checkpoint* can be created to save state at a particular time.
It is then possible to revert back to a given checkpoint if needed.
Checkpoints are created two different ways.
First, using `tools system configuration generate-checkpoint` or second, using `save checkpoint` from candidate mode.
It is also possible to set `/system/configuration/auto-checkpoint` to *true* which will generate a checkpoint whenever configuration is committed.
Configuration can be reverted or rolled back to a previous checkpoing using either a `tools` or `load` command.
Here are some additional notes regarding the checkpoint functionality:

* SRLinux allows for 10 checkpoints to be created by default but may be configured to save more up to 255 checkpoints. 
    * If the `max-checkpoints` value is changed to anything lower than the default 10 and there are existing 10+ checkpoints saved, the system will delete the older checkpoints required to meet new `max-checkpoints` value.
* When an operator creates a checkpoint, they may specify the following:
    * Name of the checkpoint
    * Comment to be added to a checkpoint
* When a checkpoint is created the following fields are also stored above and beyond the configured parameters:
    * Created date
    * Release running when created
    * Username of user who created it
    * Size of the checkpoint
    * ID: checkpoints are 0 indexed with 0 being the newest, any new checkpoint created increases the index of all other checkpoints by 1


### Features

* **Output Modifiers.** Advanced Linux output modifiers `grep`, `more`, `wc`, `head`, and `tail` are exposed directly through the SR Linux CLI.
* **Suggestions & List Completions.** As commands are typed suggestions are provided.  Tab can be used to list options available.
* **Output Format.** When displaying info from a given datastore, the output can be formatted in one of three ways:
    * *Text:* this is the default out, it is JSON-like but not quite JSON.
    * *JSON:* the output will be in JSON format.
    * *Table:* The CLI will try to format the output in a table, this doesn’t work for all data but can be very useful.
* **Aliases.** An alias is used to map a CLI command to a shorter easier to remember command.  For example, if a command is built to retrieve specific information from the state datastore and filter on specific fields while formatting the output as a table the CLI command could get quite long.  An alias could be configured so that a shorter string of text could be used to execute that long CLI command.  Alias can be further enhanced to be dynamic which makes them extremely powerful because they are not limited to static CLI commands.


### Plugins

add a section or reference to plugins?
