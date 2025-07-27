# TwinCAT State Machine Library - AI Agent Instructions

## Architecture Overview

This is a **TwinCAT 3 industrial automation library** implementing a comprehensive state machine framework. The library follows a **three-layer architecture**:

1. **State Machine Core** (`FB_StateMachine`) - Orchestrates 12 operational states with command processing
2. **Step Sequencing** (`FB_SequenceStep`) - Manages individual process steps with fault handling  
3. **Permissive System** (`FB_Permissives`) - Handles 16-bit safety interlocks with bypass capability

## Key Design Patterns

### State Machine Pattern
- **12 enumerated states** in `E_SM_State`: IDLE→HOMING→HOMED→RUNNING→COMPLETE cycle
- **State transitions** are command-driven with permission validation
- **Fault states** use odd step numbers (step+1 indicates fault condition)
- **Pause/Resume** preserves previous state in `ReturnState` and `ReturnNextStep`

### Permissive System Pattern
```st
// Each permission group uses 16-bit evaluation with bypass capability
FB_Permissives(
    inPermissives := hardware_bits,     // Real sensor inputs
    cfg.BypassPermissives := bypass_mask // Individual bit bypasses
);
// Result: stsOK = TRUE when all required bits OR bypassed
```

### Step Sequencing Pattern
```st
// Steps define: current step, next step, timeout, and permissives
FB_SequenceStep(
    cfgStep := 10,           // This step's ID
    cfgNextStep := 20,       // Normal progression
    cfgStepTimeout := 30,    // Seconds before fault
    inpStepPermissives := permissions_word
);
// Fault handling: automatically transitions to cfgStep + 1 on fault
```

## Critical File Structure

### Data Types (`DUTs/`)
- **State Machine/**: Core state management structures
  - `E_SM_State` - State enumeration (power-of-2 values for bitwise operations)
  - `ST_SM_Commands` - 8 command inputs (Start, Stop, Home, Abort, etc.)
  - `ST_SM_Status` - 25+ status outputs for comprehensive monitoring
- **Permissives/**: Safety interlock system  
  - `ST_PermIntlk_cfg` - 16 string descriptions + bypass mask
  - `ST_PermIntlk_sts` - Bitwise evaluation result + overall OK flag
- **Steps/**: Sequential operation management
  - `ActiveStep` - Current step state with fault tracking

### Function Blocks (`POUs/`)
- `FB_StateMachine` - 743 lines, handles all state transitions and command processing
- `FB_SequenceStep` - Step-level control with timeout and fault detection  
- `FB_Permissives` - 16-bit permissive evaluation with bypass logic

## Development Conventions

### Naming Patterns
- **Commands**: Imperative verbs (`Start`, `Stop`, `Home`, `Abort`)
- **Status**: Present participle (`Running`, `Stopping`, `Homing`) or past participle (`Stopped`, `Homed`)
- **Permissions**: Descriptive + "OK" suffix (`HomePermOK`, `StartPermOK`)
- **HMI structures**: `<Type>_HMI` suffix for operator interface data

### State Machine Logic
- **Command prioritization**: Abort > Stop > Other commands (mutually exclusive)
- **Permission checking**: Each state transition validates required permission groups
- **Timer management**: `TimerReset` flag triggers on step changes, managed in state machine core
- **Auto-run capability**: `cfgAutoRun` enables continuous cycling when Complete→Start conditions met

### Integration Points
- **Library reference**: Use "State Machine Twincat" in TwinCAT project references
- **Namespace**: `State_Machine_Twincat` for importing function blocks
- **HMI binding**: `SM_HMI` structure provides complete operator interface
- **I/O integration**: Permission inputs typically mapped to hardware interlocks

## Usage Examples

### Basic Implementation
```st
// Declare instances
fbStateMachine : FB_StateMachine;
currentStep : ActiveStep;
commands : ST_SM_Commands;
hmi : SM_HMI;

// Execute state machine  
fbStateMachine(
    EnableIn := TRUE,
    Commands := commands,
    CurrentStep := currentStep,
    HMI := hmi,
    inpHomePerm := safety_interlocks,
    inpStartPerm := process_ready_bits
);
```

### Step Sequence Pattern
```st
// Define multiple steps with progression logic
Step_10(cfgStep := 10, cfgNextStep := 20, ActiveStep := currentStep);
Step_20(cfgStep := 20, cfgNextStep := 30, ActiveStep := currentStep);

// Determine next step from any active step
IF Step_10.outNextStep <> currentStep.Step THEN
    NextStepNumber := Step_10.outNextStep;
ELSIF Step_20.outNextStep <> currentStep.Step THEN  
    NextStepNumber := Step_20.outNextStep;
END_IF;
```

## Testing & Training Project Integration

The **Training HMI** project demonstrates library usage:
- Basic state machine instantiation in `MAIN.TcPOU`
- Custom equipment control (`FB_CameraControl`) showing state pattern application
- I/O mapping through `IO_Mapping.TcGVL` for Cognex vision system integration
- HMI command/status flow between operator interface and automation logic

## Common Patterns When Extending

1. **New equipment controllers**: Follow `FB_CameraControl` pattern with internal step management
2. **Custom permissives**: Extend `ST_PermIntlk_cfg` descriptions for domain-specific interlocks  
3. **Specialized steps**: Inherit from `FB_SequenceStep` pattern for equipment-specific behaviors
4. **HMI integration**: Use `SM_HMI` structure family for consistent operator interfaces

## Library Dependencies
- **Tc2_Standard**: Core TwinCAT function blocks (TON timers, R_TRIG edge detection)
- **TwinCAT 3.1.4024.13+**: Required for XML structure and data type support
