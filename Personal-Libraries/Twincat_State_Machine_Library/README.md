# TwinCAT State Machine Library

A comprehensive and robust state machine library for TwinCAT 3 automation projects, designed for industrial process control and sequential operations.

## 🎯 Overview

This library provides a sophisticated state machine framework that combines state management, step sequencing, and permissive handling in a single, cohesive package. It's designed to be the foundation for complex automation sequences with built-in safety interlocks, fault handling, and operator interfaces.

## 📋 Features

### Core Components
- **State Machine Management**: Complete state lifecycle with 12 distinct states
- **Sequential Step Control**: Robust step-by-step process execution with fault handling
- **Permissive System**: Configurable safety interlocks with bypass capabilities
- **HMI Integration**: Ready-to-use HMI data structures for operator interfaces
- **Auto-Run Capability**: Automatic cycling for continuous operations
- **Fault Recovery**: Built-in retry mechanisms and fault isolation

### Key Capabilities
- ✅ **Multi-State Operation**: Idle, Running, Stopping, Homing, Aborting, and more
- ✅ **Permission Groups**: Home, Start, Proceed, and Auto-Interlock permissions
- ✅ **Step Timeout Handling**: Configurable timeouts with automatic fault detection
- ✅ **Bypass Functionality**: Individual permissive bypass for maintenance/testing
- ✅ **Pause/Resume**: Process interruption and continuation capabilities
- ✅ **Status Monitoring**: Comprehensive status reporting for diagnostics

## 🏗️ Architecture

### Function Blocks

#### `FB_StateMachine`
The main state machine controller that manages:
- **State Transitions**: Handles all 12 operational states
- **Command Processing**: Processes Start, Stop, Home, Abort, Pause, Proceed commands
- **Permission Evaluation**: Manages multiple permission groups
- **Auto-Start Logic**: Handles automatic operation requests
- **Timer Management**: Step timing and timeout detection

**Key Inputs:**
- `EnableIn`: Master enable for the state machine
- `Commands`: Grouped command structure (Start, Stop, Home, etc.)
- `NextStep`: Target step for transitions
- `cfgAutoRun`: Enable automatic cycling
- Permission inputs for different operational phases

**Key Outputs:**
- `Status`: Comprehensive status structure with all state flags
- `Permissions`: Current permission status for all groups
- `StepInfo`: Active step information and timing
- `ConditionStatus`: Current condition evaluations

#### `FB_SequenceStep`
Individual step controller for sequential operations:
- **Step Activation**: Manages when a step becomes active
- **Permissive Checking**: Evaluates step-specific conditions
- **Timeout Management**: Configurable step timeouts
- **Fault Handling**: Step-level fault detection and recovery
- **Next Step Logic**: Determines sequence progression

**Key Features:**
- One-shot activation signals
- Configurable timeout values
- Fault step handling (cfgStep + 1)
- Retry capability for fault recovery

#### `FB_Permissives`
Flexible permissive evaluation system:
- **Bit-Level Control**: Handles up to 16 individual permissives
- **Bypass Capability**: Individual permissive bypass functionality
- **Status Reporting**: Detailed permissive status feedback
- **Configuration Driven**: Runtime configurable descriptions and settings

### Data Structures

#### State Management
- `E_SM_State`: Enumeration of all possible states
- `ST_SM_Commands`: Grouped command inputs
- `ST_SM_Status`: Comprehensive status outputs
- `ST_SM_Permissions`: Permission group status
- `ST_SM_StepInfo`: Step timing and information

#### Permissive System
- `ST_PermIntlk_cfg`: Permissive configuration with names and descriptions
- `ST_PermIntlk_HMI`: HMI interface for permissive management
- `ST_PermIntlk_sts`: Permissive status reporting

#### Step Control
- `ActiveStep`: Current step information with fault status and permissions

#### HMI Integration
- `SM_HMI`: Complete HMI interface structure
- `SM_HMI_cfg`: HMI configuration parameters
- `SM_HMI_ocmd`: HMI operator commands
- `SM_HMI_sts`: HMI status display

## 🚀 Quick Start

### Basic Implementation

```st
PROGRAM MAIN
VAR
    // State Machine Instance
    MyStateMachine : FB_StateMachine;
    
    // Step Instances
    Step_10_Initialize : FB_SequenceStep;
    Step_20_Process : FB_SequenceStep;
    Step_30_Complete : FB_SequenceStep;
    
    // Data Structures
    CurrentStep : ActiveStep;
    HMI_Interface : SM_HMI;
    Commands : ST_SM_Commands;
    
    // Step Variables
    NextStepNumber : DINT;
    StepTimer : TON;
END_VAR

// Execute State Machine
MyStateMachine(
    EnableIn := TRUE,
    Commands := Commands,
    NextStep := NextStepNumber,
    cfgAutoRun := FALSE,
    HMI := HMI_Interface,
    CurrentStep := CurrentStep
);

// Execute Step Sequence
Step_10_Initialize(
    inpStep := CurrentStep.Step,
    cfgStep := 10,
    cfgNextStep := 20,
    inpStepTimerSeconds := DINT_TO_INT(StepTimer.ET / T#1S),
    cfgStepTimeout := 30,
    ActiveStep := CurrentStep
);

Step_20_Process(
    inpStep := CurrentStep.Step,
    cfgStep := 20,
    cfgNextStep := 30,
    inpStepTimerSeconds := DINT_TO_INT(StepTimer.ET / T#1S),
    cfgStepTimeout := 60,
    ActiveStep := CurrentStep
);

// Determine next step
IF Step_10_Initialize.outNextStep <> CurrentStep.Step THEN
    NextStepNumber := Step_10_Initialize.outNextStep;
ELSIF Step_20_Process.outNextStep <> CurrentStep.Step THEN
    NextStepNumber := Step_20_Process.outNextStep;
END_IF;

// Step Timer Management
StepTimer(IN := MyStateMachine.Status.Running, PT := T#1H);
IF MyStateMachine.Status.Running = FALSE THEN
    StepTimer(IN := FALSE);
END_IF;
```

### Permission Configuration

```st
VAR
    HomePerm_Config : ST_PermIntlk_cfg := (
        Name := 'Home Permissions',
        Desc := ['Safety OK', 'E-Stop Clear', 'Door Closed', '', '', '', '', '', '', '', '', '', '', '', '', ''],
        BypassPermissives := 0  // No bypasses initially
    );
    
    StartPerm_Config : ST_PermIntlk_cfg := (
        Name := 'Start Permissions',
        Desc := ['Material Present', 'Recipe Loaded', 'System Ready', '', '', '', '', '', '', '', '', '', '', '', '', ''],
        BypassPermissives := 0
    );
END_VAR
```

## 🔧 Configuration

### State Machine States

| State | Value | Description |
|-------|-------|-------------|
| IDLE | 0 | Initial state, ready for commands |
| RUNNING | 1 | Actively executing sequence |
| STOPPING | 2 | Controlled stop in progress |
| HOMING | 4 | Homing operation active |
| ABORTING | 8 | Emergency abort in progress |
| HOMED | 16 | Successfully homed, ready to start |
| STOPPED | 32 | Safely stopped |
| ABORTED | 64 | Emergency abort completed |
| COMPLETE | 128 | Sequence completed successfully |
| PAUSING | 256 | Pause operation in progress |
| PAUSED | 512 | Sequence paused |
| PROCEEDING | 1024 | Resume from pause in progress |

### Permission Groups

1. **Home Permissions** (`inpHomePerm`): Required for homing operations
2. **Start Permissions** (`inpStartPerm`): Required to start sequence
3. **Proceed Permissions** (`inpProceedPerm`): Required to resume from pause
4. **Auto Interlock** (`inpAutoIntlk`): General safety interlocks

## 📖 Usage Examples

### Advanced Step Configuration

```st
// Step with complex permissives and timeout
Step_50_CriticalProcess(
    inpStep := CurrentStep.Step,
    cfgStep := 50,
    cfgNextStep := 60,
    inpStepTimerSeconds := DINT_TO_INT(StepTimer.ET / T#1S),
    inpRetry := Commands.Retry,
    inpStepPermissives := ProcessPermissives_Word,  // 16-bit permissive word
    cfgStepTimeout := 120,  // 2 minute timeout
    inpFaultStep := ExternalFaultCondition,
    ActiveStep := CurrentStep
);
```

### HMI Integration

```st
// HMI Command Processing
Commands.Start := HMI_Interface.ocmd.Start;
Commands.Stop := HMI_Interface.ocmd.Stop;
Commands.Home := HMI_Interface.ocmd.Home;
Commands.Abort := HMI_Interface.ocmd.Abort;
Commands.Pause := HMI_Interface.ocmd.Pause;
Commands.Proceed := HMI_Interface.ocmd.Proceed;
Commands.Retry := HMI_Interface.ocmd.Retry;

// HMI Status Updates
HMI_Interface.sts.Running := MyStateMachine.Status.Running;
HMI_Interface.sts.Stopped := MyStateMachine.Status.Stopped;
HMI_Interface.sts.Faulted := MyStateMachine.Status.Faulted;
HMI_Interface.sts.CurrentStep := CurrentStep.Step;
```

## 🔒 Safety Features

- **Multiple Permission Levels**: Separate permissions for different operations
- **Individual Bypass Control**: Each permissive can be individually bypassed
- **Fault Isolation**: Step-level fault detection and handling
- **Emergency Abort**: Immediate abort capability from any state
- **Interlock Monitoring**: Continuous safety interlock evaluation

## 📊 Dependencies

### Required TwinCAT Libraries
- **Tc2_Standard** (v3.4.5.0): Standard function blocks and data types
- **Tc2_System** (v3.6.4.0): System functions and utilities
- **Tc3_Module** (v3.4.5.0): Module management functions

### TwinCAT Version
- **TwinCAT 3**: Version 3.1.4024.13 or later
- **Development Environment**: TwinCAT XAE Shell

## 🛠️ Development

This library is actively under development. Planned additions include:
- Additional specialized function blocks for common automation tasks
- Extended HMI templates and visualization components
- Recipe management integration
- Advanced diagnostic and logging capabilities
- Network communication interfaces

## 📝 Version History

### v1.0.0 (Current)
- Initial release with core state machine functionality
- Basic step sequencing with permissive handling
- HMI integration structures
- Comprehensive fault handling and recovery

## 🤝 Contributing

This is a personal automation library under active development. Future enhancements will focus on:
- Expanding function block library
- Adding specialized industrial control blocks
- Improving HMI integration templates
- Enhanced diagnostic capabilities

## 📄 License

This library is developed for personal and professional automation projects. Please ensure compliance with your organization's software policies when using in commercial applications.

---

**Author**: Ernest  
**Created**: July 2025  
**TwinCAT Version**: 3.1.4024.13  
**Library Type**: Industrial Automation Framework
