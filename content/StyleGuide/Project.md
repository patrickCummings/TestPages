---
title: "Automation Studio Project"
date: 2018-11-09T14:32:45-08:00
draft: true
---

### File Structure
The packages in the logical view are traditionally arranged in order of importance as below with corresponding descriptions. Order of packages may differ between projects.

* _Documentation_ - Project documentation and build tracking(see [Tracking Machine Builds](#tracking-machine-builds)).
* _FirstInitProg_ - A task used for initializing file devices and permanent variables. This is the first task run on startup.
* _HMI_ - User facing variables, tasks, and resources.
* _DataHandling_ - Handles database management, permanent memory, some file management, and recording data.
* _MotionControl_ - Motor control, CNC, Robotics, Synchro, and anything motion related.
* _ProcessControl_ - Low level control of process tasks such as temperature control, metrology data, and control of peripheral systems.
* _Communication_ - Communication between machine components.
* _Machine Control_ - Top level control, safety programs, and machine commands and statuses.
* _Diagnostics_ - Error management and logging.
* _Utilities_ - Any auxiliary helper functions.
* _Mapp_ - - Mapp components.
* _Test_ - Tasks used for testing machine functionality.
* _Libraries_ - Holds any libraries needed for the project.
* _GmcIpConfig_ - Settings for the GMC Interpreter.
* _GmcIpUserConfig_ - User settings for the GMC Interpreter.

### Configuration Variables

Each task can contain its own configuration as a task local variable named "Configuration". Any global configuration is kept in a global structure named "gConfiguration". These configuration variables are maintained in permanent memory, and can be backed up and restored to and from a file, configuration.csv.

### Permanent Memory

Permanent Memory is handled by the Persist Library. Using this library gives us more flexibility in how the permanent memory is handled. We can easily backup and restore permanent memory. It also gives us the ability to save local variables and parts of structures. For example: the axis parameter structures are saved directly in permanent memory so that user changes can be saved through power cycles. This is currently not possible as a built in function since only entire global structures can be saved to permanent memory.

Persist is given multiple data chunks so that different parts of permanent memory can be backed up and restored separately. For example, the homing data can be backed up and restored separate from the Machine configuration.

Some unused space in the buffer is a good idea to allow changes to be made to the data structures without having to reconfigure permanent memory.

### Tracking Machine Builds

It is important to know what software and configuration is shipped on each machine. After a machine configuration is changed, it should be backed up to file and saved in the repository for future reference. The folders on the User partition should be saved in repository in the Documents folder, contained in a folder with their Serial number, or other standard identifier.

The important folders are Recipe and Configuration. Before a machine ships, and before a machine is updated or worked on, these folders MUST be backed up to ensure that the machine can be restored to it's running state without reconfiguration.

### Subsystems

Each subsystem should be able to function on its own. It should contain the commands required to request something happen and give proper responses. 
Structures should generally be camel case and abbreviated, except internal. Members should be camel case and descriptive. Do not denote the type unless it is for IO (see below for examples). 
This is usually a structure that has the following components hierarchy:

```
//Commands that this subsystem can respond to
int.in.cmd.myCommands..
subsystem.in.cmd.acknowledgeError//Acknowledge Error Command to clear persistent errors

//Configuration variables
//May or may not be user changeable.
//Can be saved through power cycle and backed up to file
subsystem.in.cfg.staticBootValues..

//Runtime parameters
//These are not typically saved through power cycle	
//These are parameters that change through use of the subsystem
subsystem.in.par.dynamicApplicationValues..

//Any IO that is specific to a subsystem can be included under IO
//The prefix should indicate the direction and type of IO.
subsystem.io.diMyDigitalInput
subsystem.io.doMyDigitalOutput
subsystem.io.aiMyAnalogInput
subsystem.io.aoMyAnalogOutput

//Output statuses from this system
subsystem.out.statusOutputs..
subsystem.out.busy
subsystem.out.done

//Error Information integrates with ErrorCollector
subsystem.out.error//Error Bit - Indicates an active error that needs to be cleared
subsystem.out.errorID//Error ID - Can be used to look up language specific text
subsystem.out.errorString//Error Text string to display

//Variables that should only be used by this subsystem
//Function blocks should be in FUB
subsystem.internal.fub.internalFunctionBlocks..
//Other internal variables/structures can be included directly in .Internal
subsystem.internal.internalVariables..
```



### Function block states

 ```
//States with function blocks
//With Error, Busy, Done bits
IF myFub.error THEN
	//Handle error
ELSIF myFub.done THEN
	//Do next thing
ELSIF NOT myFub.busy THEN
	//Execute FUB
	myFub.enable:= TRUE;
END_IF
```

### System Integration <a name="system-integration"></a>

Integration of subsystems into the main machine logic should be done using Piper’s state/response structure. Piper is the main mechanism that is used to handle synchronization of many subsystems 

### Functionality (Piper?)

The subsystem should be self contained and typically in its own task. Most integration with the rest of the system should be done using Piper’s state/substate/response structure. If the subsystem needs tighter integration with other components, it may be necessary to make the subsystem structure global or make a separate global interface to for other systems to use.

Machine state changes should be requested using:

gMachine.in.cmd..

### Motion functions

ServoMgr task contains all the axis function that apply to all axes, including real and virtual motors. This includes status, error handling, position feedback etc. New functionality that should be applied to all axes, such as tuning or torque monitoring, should be added to the ServoMgrs AxisBasicFn_cyclic. 

For other, process specific functions, such as probing routines that require PLCopen (as opposed to G-Code) should be contained in a task that is specific to that functionality. The task for the process should use its own PLCopen function blocks, as opposed to using AxisBasicFn commands. This allows the proper use of function blocks ‘Done’, ‘Busy’, ‘Error’ and ‘CommandAborted’.

For process function that use G-Code - the general case would be to populate gCNCMgr with the required data, then command motion with gMachine.IN.CMD.Start. This is important due to administrative functions that need to occur before starting a CNC program; such as setting limits based on the operating mode.

### Synchro

Synchro is designed to be a modular backbone for a kinematic system. Synchro can be added to any project to convert it from a pile of axes into a working system with frames, tools, compensations, kinematics and geometric offsets.

Generally the rest of the application should only need to interface to Synchro through a defined application interface for enabling the synchro chain, changing between frames, and setting operating modes.

## Source Control

