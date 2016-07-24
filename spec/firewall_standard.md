# Firewall Shell Standard

#### Version 1.0.0


## Introduction
The Firewall Shell Standard is a project used to define a standard for all Firewall Shells that integrate with CloudShell.
The standard defines the Shell’s data model, commands and a set of guidelines that should be followed in Firewall Shells development and customization.



## Revision History

Version | Date | Notes
--- | --- | ---
1.0.0 | 2016-06-23 | First release of the Firewall Shell Standard


## Definitions
### Granularity
A firewall Shell should support all firewall devices with the same Vendor and OS. For example a correct shell granularity will be “Cisco ASA Shell” and not “Cisco Nexus 5510 Shell”.

### Specific Models Certification
Each released Shell should have a list of certified models. Model certification can be done only by Quali’s engineering, and validates that all the Shell’s capabilities are working for a specific model.
The Shell should also work for non-certified models, and in case some gaps are found a new version of the Shell will be released with the gaps addressed and the model certified.

### Generic Data Model
All firewall Shells share the same generic data model, except the model of the root level which is different per each Shell. The data model shouldn’t be modified.
The attributes associated with those generic models are also shared by all firewall Shells and their values are populated by the driver. It is allowed to add custom attributes only to the root level model, and it isn’t allowed to remove attributes from any level.

### Versioning
The firewall Shell version follows Semantic Versioning Guidelines (see details in http://semver.org). In short, the version structure is Major.Minor.Patch, for example “1.0.2”. A Path version is promoted when making backward-compatibility bug fixes, a Minor version is promoted when adding functionality in a backwards-compatible manner and a  Major version is promoted when making a backwards incompatible changes.

### Dependencies
In case the firewall shell is written in Python and has dependencies to Python packages (that follow Semantic Versioning Guidelines) the dependency should be to a range of Patch versions, for example to “cloudshell-firewall-cisco-asa 2.1.X”.
The dependency to cloudshell-automation-api will be to the latest Patch version (cloudshell-automation-api package version is of the format “CloudShell_Version.X”, for example 7.0.X”).



## Data Model
### Families & Models
The firewall shell standard supports Firewall devices.

#### Firewall Data Model
- Firewall
 - Chassis
    - Module
       - Port
       - Sub Module
          - Port
    - Port
    - Power Port
 - Port Channel



##### Example:

- Family: Firewall, Model: Cisco ASA Firewall
 - Family: Chassis, Model: Generic Chassis
    - Family: Module, Model: Generic Module
       - Family: Port, Model: Generic Port
       - Family: Sub Module, Model: Generic Sub Module
          - Family: Port, Model: Generic Port
    - Family: Port, Model: Generic Port
    - Family: Power Port, Model: Generic Power Port
 - Family: Port Channel, Model: Generic Port Channel


#### Family Rules

Family | Rules
--- | ---
Firewall | Searchable
Chassis | Searchable
Module | Searchable
Sub Module | Searchable
Port | Searchable, Connectable, Locked By Default
Port Channel | Searchable, Connectable, Locked By Default
Power Port | Searchable, Connectable, Locked By Default


#### Port Channel Usage

The Port Channel is a logical entity that allows grouping of several physical ports to create one logical link.
In CloudShell, all the ports configured to the port channel shouldn’t be “physically connected” and instead the port channel resource will be “physically connected”. The names of all the ports configured to the port channel will appear in the “Associated Ports” attribute on the port channel.
Addition or removal of ports from the port channel will require execution of Autoload to update the resource representation in CloudShell and a manual update of the physical connections in CloudShell by the administrator.

#### Resource Name and Address
Family | Model | Resource Name | Resource Address
--- | --- | --- | ---
Firewall | [Vendor] [OS] Firewall | (user defined) | (user defined - IP)
Chassis | Generic Chassis | Chassis[ID] | [ID]
Module | Generic Module | Module[ID] | [ID]
Sub Module | Generic Sub Module | SubModule[ID] | [ID]
Port | Generic Port | The name of the interface as appears in the device. Any “/” character is replaced with “-“, spaces trimmed.] | [ID]
Port Channel | Generic Port Channel | The name of the interface as appears in the device. Any “/” character is replaced with “-“, spaces trimmed. | PC[ID]
Power Port | Generic Power Port | PP[ContainerID][ID] | PP[ContainerID][ID]

Note: The [ID] for each sub-resource is taken from the device itself (corresponds to the names defined in the device).


### Attributes
#### Guidelines
- Attributes which aren’t relevant to a devices won’t be populated by the driver.
- All attributes which aren't user-input are "read only"
- The attribute rules are as follows - all attributes which are user input should have the rule "Configuration" enabled, all attributes which aren't user input should have the rules "Settings" and "Available For Abstract Resources" enabled.
- It is possible to customize the attribute rules selection after importing the Shell to CloudShell.
- Attributes shouldn’t be removed.
- Custom attributes should be added only to the root level model.
- All attributes are of type String unless mentioned otherwise

##### [Vendor] [OS] Firewall

Attribute Name | Details | User input?
--- | --- | ---
User | User with administrative privileges | Yes
Password | Attribute of type Password | Yes
Enable Password | Attribute of type Password, The Enable password is required by some CLI protocols such as Telnet | Yes
System Name | A unique identifier for the device, if exists in the device terminal/os | No
Contact Name | | No
OS Version | | No
Vendor | | No
Location | The device physical location identifier. For example: Lab1/Floor2/Row5/Slot4 | No
Model | The device model. This information is typically used for abstract resource filtering. | No
SNMP Read Community | | Yes
SNMP Write Community | | Yes
SNMP V3 User | | Yes
SNMP V3 Password | | Yes
SNMP V3 Private Key | | Yes
SNMP Version | Possible values – v1, v2c, v3 | Yes
Console Server IP Address | | Yes
Console User | | Yes
Console Port | Attributes of type Numeric | Yes
Console Password | Attribute of type Password | Yes
CLI Connection Type | Attribute of type Lookup. Possible values – Auto, Console, SSH, Telnet, TCP | Yes
Power Management | Attribute of type Boolean. Possible values – True, False, Used by the power management orchestration, if enabled, to determine whether to automatically manage the device power status | Yes
Backup Location | Used by the save/restore orchestration to determine where backups should be saved | Yes
Sessions Concurrency Limit | Attributes of type Numeric. Default is 1 (no concurrency) | Yes



#####  Generic Chassis

Attribute Name | Details | User input?
--- | --- | ---
Model | | No
Serial Number | | No


#####  Generic Module

Attribute Name | Details | User input?
--- | --- | ---
Model | | No
Version | | No
Serial Number | | No


##### Generic Sub Module

Attribute Name | Details | User input?
--- | --- | ---
Model | | No
Version | | No
Serial Number | | No


##### Generic Port

Attribute Name | Details | User input?
--- | --- | ---
MAC Address | | No
L2 Protocol Type | Such as POS, Serial | No
IPv4 Address | | No
IPv6 Address | | No
Port Description | | No
Bandwidth | Attributes of type Numeric | No
MTU | Attributes of type Numeric | No
Duplex | Attributes of type Lookup. Half or Full | No
Adjacent | System or Port | No
Protocol Type | Default values is Transparent (=”0”) | No
Auto Negotiation | True or False | No


#####  Generic Port Channel

Attribute Name | Details | User input?
--- | --- | ---
Associated Ports | value in the form “[portResourceName],…”, for example “GE0-0-0-1,GE0-0-0-2” | No
IPv4 Address | | No
IPv6 Address | | No
Port Description | | No
Protocol Type | Default values is Transparent (=”0”) | No




#####  Generic Power Port

Attribute Name | Details | User input?
--- | --- | ---
Model | | No
Serial Number || No
Version | | No
Port Description | | No



## Commands
Below is a list of all the commands that will be part of the standard Shell, their names and interfaces. Each firewall Shell that will be released by Quali’s engineering will include implementation for all those commands.
When creating a new shell according to the standard it is OK not to implement all commands and/or implement additional command, but a command with a functionality that fits one of the predefined list commands should be implemented according to the standard.

Command outputs: On failure an exception containing the error will be thrown and the command will be shown as failed. A failure is defined as any scenario in which the command didn’t complete its expected behavior, regardless if the issue originates from the command’s input, device or the command infrastructure itself. On success the command will just return as passed with no output. The “Autoload” command has a special output on success that CloudShell reads when building the resource hierarchy. The “Save” command will return an output on success with the file name (exact syntax below).


### Autoload
Queries the devices and loads the structure and attribute values into CloudShell.
  - SNMP Based



### Save
Creates a configuration file.
  - Inputs
      - Configuration Type – optional, if empty the default value will be taken. Possible values – StartUp or Running Default value – Running
      - Folder Path – the path in which the configuration file will be saved. Won’t include the name of the file but only the folder. This input is optional and in case this input is empty the value will be taken from the “Backup Location” attribute on the root resource. The path should include the protocol type (for example “tftp://asdf”)

  - Output: <FullFileName>
   - The configuration file name should be “[ResourceName]-[ConfigurationType]-[DDMMYY]-[HHMMSS]”


### Restore
Restores a configuration file.
   - Inputs
       - Path – the path to the configuration file, including the configuration file name. The path should include the protocol type (for example “tftp://asdf”). This input is mandatory.
       - Restore Method – optional, if empty the default value will be taken. Possible values – Append or Override Default value – Override
       - Configuration Type - mandatory, no default. Possible values - StartUp or Running


### Load Firmware
Loads a firmware onto the device
   - CLI based
   - Applies to the whole device, also in case of multi-chassis device
   - Inputs:
     - File Path
     - Remote Host


### Run Custom Command
Executes any custom command entered by the user on the device
  - Inputs:
    - Custom command – the command itself. Note that commands that require a response aren’t supported


### Run Custom Config Command
Executes any custom config command entered by the user on the device.
  - Inputs:
    - Custom command – the command itself. Note that commands that require a response aren’t supported
    - This command will be hidden from the UI and accessible only via API.


### Shutdown
Sends a graceful shutdown to the device
  - Inputs:
    - Note that not all devices support a shutdown command. In such cases the command just wouldn’t be implemented
