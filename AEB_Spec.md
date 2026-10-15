# Simplified AEB Feature Specification

| Field | Value |
| --- | --- |
| Specification ID | CFTS-AEB-001 |
| Version | 0.1 |
| Date | 2026-10-15 |
| Status | Initial educational specification |
| Course | CSI-4130/5130: Artificial Intelligence |

## 1. Purpose and scope

This CFTS-style specification defines a fictional Automatic Emergency Braking (AEB) controller for a generative-AI course project. It is the reference for creating requirements and reviewing generated test-case drafts.

All signal names, calibrations, thresholds, and behavior are project assumptions. This is not a production AEB specification, a claim of standards compliance, or guidance for deployment in a vehicle.

The controller requests fixed deceleration when a valid forward target has a sufficiently short time to collision (TTC). Scope includes input bypass selection, feature enable, operating-speed eligibility, target/TTC validity, activation, hysteresis, release, and deceleration request.

Out of scope: perception, sensor fusion, TTC calculation, target classification, steering avoidance, brake hydraulics, physical stopping performance, fault diagnostics/DTCs, and vehicle-program applicability. TTC is supplied as an input. Invalid numeric ranges and invalid calibration configurations are outside version 0.1.

## 2. Execution and observation conventions

| Item | Definition |
| --- | --- |
| Controller execution period | 10 ms |
| Initial state | `AEB_Active = 0`; `AEB_DecelReq = 0.0 m/s²` |
| Boolean encoding | `0` = false/disabled; `1` = true/enabled |
| Braking sign convention | Negative longitudinal acceleration |
| Processing order | Select effective inputs, evaluate state, then update outputs together |
| Priority | Release/inhibit conditions take priority over activation |
| Maximum response time | 20 ms from an applied input/calibration change to the corresponding output update |
| Test logging | Inputs and outputs logged synchronously every 10 ms |

Initialization sets the initial outputs before the first normal evaluation. Subsequent executions evaluate the conditions below. Response timing concerns controller outputs, not physical brake response. Timing is measured from when a value is applied at the controller interface, not from an INCA user-interface click. Input conditions must remain present through controller evaluation; sub-cycle pulses are not addressed.

## 3. Signal dictionary

Raw inputs originate from the simulated upstream environment. Effective measurements are the selected inputs used by the AEB logic.

| Raw input | Effective measurement | Logical type | Unit / valid range | Description |
| --- | --- | --- | --- | --- |
| `VehSpd_Raw` | `VehSpd_Eff` | Float | km/h; 0–250 | Vehicle speed |
| `TargetDetected_Raw` | `TargetDetected_Eff` | Boolean | 0 or 1 | Forward target detected |
| `TTC_Raw` | `TTC_Eff` | Float | s; 0–10 | Time to collision |
| `TTCValid_Raw` | `TTCValid_Eff` | Boolean | 0 or 1 | TTC is usable |

| Output | Access | Logical type / unit | Meaning |
| --- | --- | --- | --- |
| `AEB_Active` | Read-only measurement | Boolean | Controller active state |
| `AEB_DecelReq` | Read-only measurement | Float; m/s² | Requested longitudinal acceleration |

`TTCValid_Eff = 0` makes TTC unusable regardless of its numeric value. Types are conceptual; ECU storage types, A2L addresses, and protocol configuration are not defined here.

## 4. INCA-style bypass interface

The assumed controller contains bypass logic. INCA is the conceptual measurement/calibration interface; it does not automatically make a measurement writable. No real ECU, INCA license, or A2L file is required for the initial text-generation project.

| Effective input | Writable bypass enable | Writable bypass value |
| --- | --- | --- |
| `VehSpd_Eff` | `VehSpd_BypEn` | `VehSpd_BypVal` |
| `TargetDetected_Eff` | `TargetDetected_BypEn` | `TargetDetected_BypVal` |
| `TTC_Eff` | `TTC_BypEn` | `TTC_BypVal` |
| `TTCValid_Eff` | `TTCValid_BypEn` | `TTCValid_BypVal` |

For every input:

- `BypEn = 0`: effective input equals the raw input.
- `BypEn = 1`: effective input equals the bypass value.
- Enable flags are Boolean and default to `0`.
- Bypass values use the type, unit, and valid range of the corresponding input.
- Tests shall explicitly set a bypass value before enabling its bypass; they shall not depend on an unspecified bypass-value default.

Raw inputs are set through the simulated input source when testing the non-bypassed path. Effective inputs and outputs are monitored, not directly set. Bypass operation shall not depend on whether AEB is active.

## 5. Feature calibrations

| Calibration | Type / unit | Default | Purpose |
| --- | --- | --- | --- |
| `AEB_Enable_C` | Boolean | 1 | Master feature enable |
| `AEB_MinSpd_C` | Float; km/h | 10.0 | Minimum eligible speed, inclusive |
| `AEB_MaxSpd_C` | Float; km/h | 80.0 | Maximum eligible speed, inclusive |
| `AEB_TTCActivate_C` | Float; s | 1.5 | Activation threshold, inclusive |
| `AEB_TTCRelease_C` | Float; s | 2.0 | Release threshold, inclusive |
| `AEB_DecelCmd_C` | Float; m/s² | -3.0 | Request while active |

Valid configurations require minimum speed ≤ maximum speed and activation TTC < release TTC. Tests use these defaults unless they explicitly specify a valid change. Version 0.1 does not specify behavior for invalid configurations.

## 6. Eligibility and state logic

Eligibility requires all of the following:

1. `AEB_Enable_C = 1`.
2. `AEB_MinSpd_C ≤ VehSpd_Eff ≤ AEB_MaxSpd_C`.
3. `TargetDetected_Eff = 1`.
4. `TTCValid_Eff = 1`.

| Current state | Condition | Next state |
| --- | --- | --- |
| Either | Any eligibility condition is false | Inactive |
| Inactive | Eligible and `TTC_Eff ≤ AEB_TTCActivate_C` | Active |
| Inactive | Eligible and `TTC_Eff > AEB_TTCActivate_C` | Inactive |
| Active | Eligible and `TTC_Eff < AEB_TTCRelease_C` | Active |
| Active | Eligible and `TTC_Eff ≥ AEB_TTCRelease_C` | Inactive |

An active state is established through qualifying inputs, not by directly writing `AEB_Active`. There is no additional latching, debounce, minimum active duration, or re-arm delay in this simplified controller.

## 7. Functional requirements

| ID | Requirement |
| --- | --- |
| AEB-REQ-001 | At controller initialization, `AEB_Active` shall equal `0` and `AEB_DecelReq` shall equal `0.0 m/s²`. |
| AEB-REQ-002 | Each effective input shall follow its corresponding bypass-selection rule within 20 ms of an applied raw-input, bypass-value, or bypass-enable change. Changes to an unselected source shall not change the effective input. |
| AEB-REQ-003 | When inactive, AEB shall activate within 20 ms when all eligibility conditions are satisfied and `TTC_Eff ≤ AEB_TTCActivate_C`. |
| AEB-REQ-004 | When inactive, AEB shall remain inactive whenever any eligibility condition is false or `TTC_Eff > AEB_TTCActivate_C`. |
| AEB-REQ-005 | When active, AEB shall remain active while all eligibility conditions remain satisfied and `TTC_Eff < AEB_TTCRelease_C`. |
| AEB-REQ-006 | When active, AEB shall deactivate within 20 ms if any eligibility condition becomes false or `TTC_Eff ≥ AEB_TTCRelease_C`. |
| AEB-REQ-007 | On each controller execution, `AEB_DecelReq` shall equal `AEB_DecelCmd_C` when `AEB_Active = 1`, and `0.0 m/s²` when `AEB_Active = 0`. Both outputs shall update together. |

## 8. Boundary examples

These examples explain the requirements; they are not a completed test dataset. Default calibrations and eligibility are assumed.

| TTC | Initially inactive | Initially active |
| --- | --- | --- |
| 1.4 s | Activate | Remain active |
| 1.5 s | Activate | Remain active |
| 1.6 s | Remain inactive | Remain active |
| 1.9 s | Remain inactive | Remain active |
| 2.0 s | Remain inactive | Deactivate |
| 2.1 s | Remain inactive | Deactivate |

Vehicle speeds of exactly 10.0 and 80.0 km/h are eligible. Speeds below 10.0 or above 80.0 km/h are ineligible. Eligibility alone does not guarantee activation; the TTC condition and current state also matter.

## 9. Test-case authoring contract

Each dataset record shall retain the requirement ID and applicable specification version as metadata. Its test-case body shall contain exactly these three sections:

1. **Preconditions**
2. **Test Procedure/Test Steps**
3. **Acceptance Criteria**

Use **Set** for modified inputs, calibrations, or conditions and **Verify** for validations. State exact values and units, measurable expected outputs, and applicable timing. Establish measurement states through documented bypasses, raw-input stimulus, or controller initialization; never treat read-only measurements as writable controls.

Keep tests within the linked requirement's scope. Supply relevant signal definitions, calibrations, state rules, and bypass details with the requirement as model context. Do not invent signal names, DTCs, physical stopping behavior, or unspecified implementation details. If necessary information is missing, identify it rather than silently assuming it. Test observation durations selected for a procedure shall be distinguished from controller timing requirements.

## 10. Version history and provenance

| Version | Date | Change |
| --- | --- | --- |
| 0.1 | 2026-10-15 | Initial fictional specification based on project scoping discussions. Clarifies applied-input timing, bypass setup, and state initialization. |

Prepared with ChatGPT assistance from the project discussion. No proprietary automotive requirements or production calibration data are used. This specification will be reviewed alongside the pilot test-case dataset before model experiments.
