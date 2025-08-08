# TwinCAT PLC Programming Standards

These standards keep code consistent across libraries and projects. They match the existing State Machine library and extend it for device/actuator FBs.

## Naming and casing

- Use lower camelCase for variables and FB internals: stepTimerSeconds, inPermAdv.
- Prefixes by role:
  - in* for inputs from field/other logic: inPermAdv, inFbkAdvanced.
  - out* for outputs to field/other logic: outValveAdvance.
  - pcmd* for program commands (automatic): pcmdAdvance.
  - ocmd* for operator commands (manual/HMI): ocmdRetract.
  - cmd* for generic commands (e.g., cmdReset).
  - sts* for status flags: stsFaulted, stsMoving.
  - cfg* for configuration: cfgAdvTimeoutS, cfgHoldAtAdvanced.
- Structures/Enums use PascalCase type names: ST_TwoPosActuator_Cfg, ST_TwoPosActuator_Sts, E_TwoPosActuator_State, E_TwoPosActuator_Fault.
- Public FBs use FB_* prefix: FB_TwoPosActuator.

## File and folder layout

- Keep shared DUTs in State Machine/DUTs/<Category>/.
- Keep FBs in State Machine/POUs/ (or a subfolder per device family if desired).
- One DUT per .TcDUT file for clarity.

## Permissions (Permissives)

- Use FB_Permissives for all permission/interlock evaluations.
- Each permission set uses ST_PermIntlk_HMI for HMI binding:
  - Read .cfg from HMI struct to get bypass and descriptions.
  - Write computed .sts.OK and .sts.PermIntlk back to HMI.
- Inputs carrying live permissive bits are INT and use two’s complement -1 to mean “all OK”.

## Commands and modes

- FBs should accept both pcmd* and ocmd* and OR them internally.
- Conflicting commands (e.g., advance + retract) must fault with a clear code and de-energize outputs.
- Provide cmdReset to clear a latched fault when safe.

## State machines and timers

- Prefer small explicit enums for device states (e.g., Undefined, Retracted, Advancing, Advanced, Retracting, Fault).
- Require feedback transitions to declare success; otherwise time out to Fault.
- Use TON timers for motion timeouts. Configure in seconds via cfg*TimeoutS.
- On success, support hold outputs via cfgHold* options.

## Status and faults

- Group public statuses in a struct (ST_*_Sts): state enum, booleans (stsAdvanced, stsRetracted, stsMoving, stsFaulted), faultCode, and lastFaultCode.
- Fault codes use an enum (E_*_Fault) with specific reasons: None, AdvanceTimeout, RetractTimeout, BothFeedbackOn, UndefinedState, PermissivesNotOK, ConflictingCommands, etc.

## IO mapping

- Prefer a single IN_OUT IO struct for mapping field IO to the FB: inputs (feedbacks) and outputs (valve commands) together.
- FB logic only drives the out* fields; it never writes in* fields.

## HMI integration

- Expose permissive HMI structs (ST_PermIntlk_HMI) per motion direction when relevant, so naming/desc and bypass can be set from HMI.
- Keep FB outputs stable and HMI-friendly (e.g., provide one-shots like osAdvanced/osRetracted if needed by screens).

## Error handling and safety

- De-energize both outputs on fault and on undefined state.
- Never energize opposing valves simultaneously.
- Enter Undefined state if both feedbacks are TRUE or both are FALSE at rest; fault accordingly.

## Example prefixes at a glance

- Inputs: inFbkAdvanced, inPermAdv
- Outputs: outValveAdvance, outValveRetract
- Program commands: pcmdAdvance, pcmdRetract
- Operator commands: ocmdAdvance, ocmdRetract
- Config: cfgAdvTimeoutS, cfgHoldAtAdvanced
- Status: stsMoving, stsFaulted, faultCode
