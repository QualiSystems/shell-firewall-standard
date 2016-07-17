# Networking Shell Standard

#### Version 3.2.0


## Introduction
The Networking Shell Standard is a project used to define a standard for all networking Shells (L2 and L3) that integrate with CloudShell.
The standard defines the Shell’s data model, commands and a set of guidelines that should be followed in networking Shells development and customization.



## Revision History

Version | Date | Notes
--- | --- | ---
3.2.0 | 2016-07-14 | Added a new attribute named "VRF Management Name" on the root model. The attribute will be filled in by the administrator (optional). Save and Restore commands will use the value in this attribute in case no such input was passed to the command.
3.1.0 | 2016-06-23 | Added a new data model for Wireless Controller (in addition to Switch and Router).
3.0.0 | 2016-06-09 | The name and address of the Power Port has changed from "PP[ID]" to "PP[ContainerID][ID]" in order to support devices with power ports that have the same ID but are under different containers. **This change isn't backwards compatible.**
2.1.0 | 2016-05-29 | 1) Added an optional input parameter (VRF Management Name) to the Save and Restore commands; used to share same/overlapping sub-net on the same core. 2) Added support for concurrent sessions to the device (concurrent execution of commands), along with an attribute (Sessions Concurrency Limit) that allows the admin to set the maximal no. of concurrent sessions the shell can use.
2.0.1 | 2016-03-02 | 1) Fixed the root model name (removed the "Generic" from the root model name). 2) The attributes rules definition has been clarified - all attributes which are user input should have the rule "Configuration" enabled, all attributes which aren't user input should have the rules "Settings" and "Available For Abstract Resources" enabled. 3) Updated the Add_VLAN and Remove_VLAN commands Q-in-Q inputs to be compatible with 6.4 VLAN Shell.
2.0.0 | 2016-02-25 | 1) Replaced the bulk Connect and Disconnect commands to one ApplyConnectivityChanges command in order to be compatible with CloudShell 7.0 connectivity 2) A new “configuration type” input for the Restore command. 3) The commands output has been standardize. Exception on fail, Pass with no output on Success. Only Autoload and Save commands has an output. 4) The “Console Port”, “MTU” and “Bandwidth” attribute were changed from String to Numeric. **The 4th item isn't backwards compatible.**
1.0.0 | 2016-02-15 | First release of the Networking Shell Standard


## Definitions
### Granularity
A networking Shell should support all networking devices with the same Vendor and OS. For example a correct shell granularity will be “Cisco NXOS Shell” and not “Cisco Nexus 5K Shell”.

### Specific Models Certification
Each released Shell should have a list of certified models. Model certification can be done only by Quali’s engineering, and validates that all the Shell’s capabilities are working for a specific model.
The Shell should also work for non-certified models, and in case some gaps are found a new version of the Shell will be released with the gaps addressed and the model certified.

### Generic Data Model
All networking Shells share the same generic data model, except the model of the root level which is different per each Shell. The data model shouldn’t be modified.
The attributes associated with those generic models are also shared by all networking Shells and their values are populated by the driver. It is allowed to add custom attributes only to the root level model, and it isn’t allowed to remove attributes from any level.

### Versioning
The networking Shell version follows Semantic Versioning Guidelines (see details in http://semver.org). In short, the version structure is Major.Minor.Patch, for example “1.0.2”. A Path version is promoted when making backward-compatibility bug fixes, a Minor version is promoted when adding functionality in a backwards-compatible manner and a  Major version is promoted when making a backwards incompatible changes.

### Dependencies
In case the networking shell is written in Python and has dependencies to Python packages (that follow Semantic Versioning Guidelines) the dependency should be to a range of Patch versions, for example to “cloudshell-networking-cisco-nxos 2.1.X”.
The dependency to cloudshell-automation-api will be to the latest Patch version (cloudshell-automation-api package version is of the format “CloudShell_Version.X”, for example 7.0.X”).



## Data Model
### Families & Models
The networking shell standard supports both L2 (Switch), L3 (Router) and Wireless Controller.

** Switch Data Model **
- Switch
 - Chassis
    - Module
       - Port
       - Sub Module
          - Port
    - Port
    - Power Port
 - Port Channel


 ** Router Data Model **
 - Router
  - Chassis
     - Module
        - Port
        - Sub Module
           - Port
     - Port
     - Power Port
  - Port Channel


  ** Wireless Controller Data Model **
  - Router
   - Chassis
      - Module
         - Port
         - Sub Module
            - Port
      - Port
      - Power Port
   - Port Channel

** Example **
- Family: Switch, Model: Cisco NXOS Switch
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
Switch | Searchable
Router | Searchable
Wireless Controller | Searchable
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
Switch | [Vendor] [OS] Switch | (user defined) | (user defined - IP)
Router | [Vendor] [OS] Router | (user defined) | (user defined - IP)
Wireless Controller | [Vendor] [OS] Wireless Controller | (user defined) | (user defined - IP)
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

##### [Vendor] [OS] Switch or [Vendor] [OS] Router or [Vendor] [OS] Wireless Controller

Attribute Name | Details | User input?
--- | --- | ---
User | User with administrative privileges | Yes
Password | Attribute of type Password | Yes
Enable Password | Attribute of type Password | Yes
System Name | | No
Contact Name | | No
OS Version | | No
Vendor | | No
Location | | No
Model | | No
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
Power Management | Attribute of type Boolean. Possible values – True, False | Yes
Backup Location | | Yes
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



### commands
Below is a list of all the commands that will be part of the standard Shell, their names and interfaces. Each networking Shell that will be released by Quali’s engineering will include implementation for all those commands.

When creating a new shell according to the standard it is OK not to implement all commands and/or implement additional command, but a command with a functionality that fits one of the predefined list commands should be implemented according to the standard.

Command outputs: On failure an exception containing the error will be thrown and the command will be shown as failed. A failure is defined as any scenario in which the command didn’t complete its expected behavior, regardless if the issue originates from the command’s input, device or the command infrastructure itself. On success the command will just return as passed with no output. The “Autoload” command has a special output on success that CloudShell reads when building the resource hierarchy. The “Save” command will return an output on success with the file name (exact syntax below).

Note that the connectivity ApplyConnectivityChanges command behaves differently between Switches and Routers. In Switches, the ApplyConnectivityChanges command can configure VLAN Access or VLAN Trunk on a port, or clear the VLAN configuration from the port. In Routers, the ApplyConnectivityChanges command creates a sub-interface (which isn't modeled in CloudShell) on the port which allows traffic with the specified VLAN tags (both outer and inner) to pass via this port. Clearing the VLAN configuration from a port in a Router translates to removal of the sub-interfaces. This means that a device which is directly connected to a router can be connected only to VLAN service of type Trunk in CloudShell.


- ** Autoload ** – queries the devices and loads the structure and attribute values into CloudShell.
  - SNMP Based



  - ** Save ** – creates a configuration file.
    - Inputs
        - Configuration Type – optional, if empty the default value will be taken. Possible values – StartUp or Running Default value – Running
        - Folder Path – the path in which the configuration file will be saved. Won’t include the name of the file but only the folder. This input is optional and in case this input is empty the value will be taken from the “Backup Location” attribute on the root resource. The path should include the protocol type (for example “tftp://asdf”)
        - VRF Management Name - optional, no default. VRF (Virtual routing and Forwarding) is used to share same/overlapping sub-net on the same core. Service Providers use it to share their backbone with multiple customers and also assign a management VRF which they use to manage the devices. In case no value is passed in this input the command will use the value in the "VRF Management Name" attribute on the root model (which can be empty).

   - Output: "<FullFileName>,"
   - The configuration file name should be “[ResourceName]-[ConfigurationType]-[DDMMYY]-[HHMMSS]”


   - ** Restore ** – restores a configuration file.
     - Inputs
         - Path – the path to the configuration file, including the configuration file name. The path should include the protocol type (for example “tftp://asdf”). This input is mandatory.
         - Restore Method – optional, if empty the default value will be taken. Possible values – Append or Override Default value – Override
         - Configuration Type - mandatory, no default. Possible values - StartUp or Running
         - VRF Management Name - optional, no default. VRF (Virtual routing and Forwarding) is used to share same/overlapping sub-net on the same core. Service Providers use it to share their backbone with multiple customers and also assign a management VRF which they use to manage the devices. In case no value is passed in this input the command will use the value in the "VRF Management Name" attribute on the root model (which can be empty).


 - ** Load Firmware ** – loads a firmware onto the device
     - CLI based
     - Applies to the whole device, also in case of multi-chassis device
     - Inputs:
       - File Path
       - Remote Host


   - ** Add_VLAN ** – configures VLAN on a port / port-channel
       - Inputs:
         - VLAN_Ranges - string input, Possible values – support a specific VLAN ID or VLAN range (in the format of “a-b,c”)
         - VLAN_Mode, Possible values – access or trunk
         - port – the full address of the port
         - Additional_Info
       - This command should be tagged as “connected command”
       - This command should be hidden from the UI
         - If Additional_Info = QNQ --> configuration mode is Q-in-Q.
If Additional_Info = QNQ,<ID/Range> --> configuration mode is Selective Q-in-Q according to the ID/Range (the syntax for the ID/Range is "a,b-c", for example "100-200,340,1000-2000" or just "200")
If Additional_Info = empty --> configures VLAN in regular mode.

 Note - the "Add_VLAN" command won't exist in Shells which support CloudShell version 7.0 and above. It is replaced by the ApplyConnectivityChanges Command.


 - ** Remove_VLAN ** – clears VLAN configuration from a port / port-channel
     - Inputs:
       - VLAN_Ranges - string input - Possible values – support a specific VLAN ID or VLAN range (in the format of “a-b,c”)
       - VLAN_Mode - Possible values – access or trunk
       - port – the full address of the port
       - Additional_Info
     - This command should be tagged as “connected command”
     - This command should be hidden from the UI

Note - the "Remove_VLAN" command won't exist in Shells which support CloudShell version 7.0 and above. It is replaced by the ApplyConnectivityChanges Command.


- ** Run Custom Command ** – executes any custom command entered by the user on the device
    - Inputs:
      - Custom command – the command itself. Note that commands that require a response aren’t supported


  - ** Run Custom Config Command ** – executes any custom config command entered by the user on the device.
      - Inputs:
        - Custom command – the command itself. Note that commands that require a response aren’t supported
        - This command will be hidden from the UI and accessible only via API.


  - ** Shutdown ** – sends a graceful shutdown to the device
      - Inputs:
        - Note that not all devices support a shutdown command. In such cases the command just wouldn’t be implemented

  - ** ApplyConnectivityChanges ** – configures VLANs on multiple ports or port-channels
      - Inputs:
        - Request – a JSON with bulk “add_vlan” or “remove_vlan” request. The request includes the list of ports and VLANs that should be configured or cleared on those ports. See separate article with the JSON schema and an example.
      - Output: a JSON with the command’s response, should include success/fail per each connection request.
      - This command is compatible with the connectivity in CloudShell 7.0 version and above.


Notes: (1) The ApplyConnectivityChanges command will be available in the Shell only when applicable in the device; (2) The standard doesn’t support different VLAN request to the same Switch/Router/Wireless-Controller port at the same time. For example connecting the same port to multiple VLAN services each with a different VLAN ID/range. When configuring VLAN ID/range on a port the assumption is that there is no other VLAN configured on it.         
