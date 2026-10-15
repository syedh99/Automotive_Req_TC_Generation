# AEB Pilot Test Cases

**Version:** 0.1 | **Date:** 2026-10-15  
**Source:** [AEB_Spec.md](AEB_Spec.md), CFTS-AEB-001 v0.1  
**Status:** AI-assisted draft; specification consistency checked; human review and execution pending.

## Use and timing conventions

These 15 cases are pilot reference examples for a fictional controller. They are not execution results or a production validation suite. Each case has requirement metadata followed by exactly three test sections. Preconditions are repeated so cases can later be used independently.

- Apply calibrations and bypass settings through the assumed INCA-style interface; set raw inputs through the simulated input source. Never write effective measurements or outputs directly.
- Run the controller every 10 ms and log applied settings, effective inputs, and outputs synchronously after each execution. Record applied-change timestamps; INCA click times are not timing references.
- A 20 ms deadline is measured from the applied change. For activation/release tests, verify the effective input change within 20 ms too; do not restart the output deadline when the effective input changes.
- Unless otherwise stated, hold each stimulus for 100 ms. This is a chosen test observation window, not a feature debounce or response requirement. For a transition, verify the expected value by 20 ms and at every subsequent logged sample through 100 ms. For “remain” checks, verify every sample throughout the 100 ms window.
- Logical simulation comparisons use the stated values exactly. Implementation-specific floating-point tolerances are not specified and must be documented before execution against a concrete implementation.
- Keep all inputs/calibrations unchanged except those explicitly modified. Begin each independent case or parameterized run with its own preconditions. After recording a run, set `AEB_Enable_C = 0`, then set all four bypass enables to `0`; initialize the next run from its listed setup.
- Case 001 needs an initialization observation hook before normal cyclic execution. This is a test-harness capability to implement, not an invented ECU signal. Ordinary post-cycle logging alone cannot establish the initial values at that instant.

## Coverage index

| Case | Focus | Primary requirement |
| --- | --- | --- |
| AEB-TC-001 | Initialization | AEB-REQ-001 |
| AEB-TC-002 | Bypass selection and ignored raw source | AEB-REQ-002 |
| AEB-TC-003 | Raw selection and ignored bypass source | AEB-REQ-002 |
| AEB-TC-004 | TTC activation boundary | AEB-REQ-003 |
| AEB-TC-005 | Minimum eligible speed | AEB-REQ-003 |
| AEB-TC-006 | Maximum eligible speed | AEB-REQ-003 |
| AEB-TC-007 | Disabled-feature inhibition | AEB-REQ-004 |
| AEB-TC-008 | Missing-target inhibition | AEB-REQ-004 |
| AEB-TC-009 | Invalid-TTC inhibition | AEB-REQ-004 |
| AEB-TC-010 | Outside-speed-range inhibition | AEB-REQ-004 |
| AEB-TC-011 | Active-state hysteresis | AEB-REQ-005 |
| AEB-TC-012 | TTC release boundary | AEB-REQ-006 |
| AEB-TC-013 | Target-loss release | AEB-REQ-006 |
| AEB-TC-014 | Feature-disable release | AEB-REQ-006 |
| AEB-TC-015 | Deceleration request and output synchronization | AEB-REQ-007 |

## AEB-TC-001 — Initial output values

**Primary requirement:** AEB-REQ-001  
**Supporting context:** Specification section 2; initialization observation hook required  
**Specification version:** CFTS-AEB-001 v0.1  
**Review / execution:** Pending / Not executed

### Preconditions

1. Set the test harness to capture outputs immediately after the controller initialization routine and before the first normal 10 ms execution.
2. Set simulated raw inputs to `VehSpd_Raw = 50.0 km/h`, `TargetDetected_Raw = 1`, `TTC_Raw = 3.0 s`, and `TTCValid_Raw = 1`. Set the initial calibration image to the defaults in specification section 5 and all four bypass enables to `0`.

### Test Procedure/Test Steps

1. Set the controller to its initialized state by invoking the simulated initialization routine once; capture `AEB_Active` and `AEB_DecelReq` before normal cyclic execution.
2. Verify the captured initial outputs before allowing the first normal cycle.

### Acceptance Criteria

1. Verify at the initialization observation point that `AEB_Active = 0` and `AEB_DecelReq = 0.0 m/s²`.
2. Verify the observation precedes the first normal evaluation. If that observation is unavailable, record this case as blocked, not passed.

## AEB-TC-002 — Bypassed input selection

**Primary requirement:** AEB-REQ-002  
**Supporting context:** Specification sections 2–6 and 9  
**Specification version:** CFTS-AEB-001 v0.1  
**Review / execution:** Pending / Not executed

### Preconditions

1. Set the simulated controller execution period to 10 ms and synchronized post-execution logging to 10 ms; log all applied changes and relevant measurements.
2. Set `AEB_Enable_C = 0`; set `AEB_MinSpd_C = 10.0 km/h`, `AEB_MaxSpd_C = 80.0 km/h`, `AEB_TTCActivate_C = 1.5 s`, `AEB_TTCRelease_C = 2.0 s`, and `AEB_DecelCmd_C = -3.0 m/s²`.
3. Set `VehSpd_Raw = 50.0 km/h`, `TargetDetected_Raw = 1`, `TTC_Raw = 3.0 s`, and `TTCValid_Raw = 1` through the simulated input source; set all four bypass enables to `0`.
4. Set `VehSpd_BypVal = 50.0 km/h`, `TargetDetected_BypVal = 1`, `TTC_BypVal = 3.0 s`, and `TTCValid_BypVal = 1`; then set `VehSpd_BypEn = 1`, `TargetDetected_BypEn = 1`, `TTC_BypEn = 1`, and `TTCValid_BypEn = 1`.
5. Verify within 20 ms that `VehSpd_Eff = 50.0 km/h`, `TargetDetected_Eff = 1`, `TTC_Eff = 3.0 s`, and `TTCValid_Eff = 1`; verify `AEB_Active = 0` after the disabled setup has been applied for 20 ms.
6. Set `AEB_Enable_C = 0` and hold for 100 ms; verify `AEB_Active = 0` throughout this window because TTC is 3.0 s.
7. Set up each row below as an independent run. The prefix denotes the exact `_Raw`, `_BypEn`, `_BypVal`, and `_Eff` names in the specification. Use the unit listed in the row.

| Input prefix | Unit | Raw A | Bypass B | Raw C | Bypass D |
| --- | --- | --- | --- | --- | --- |
| `VehSpd` | km/h | 40.0 | 50.0 | 60.0 | 70.0 |
| `TargetDetected` | Boolean | 0 | 1 | 0 | 0 |
| `TTC` | s | 3.0 | 1.4 | 2.5 | 1.2 |
| `TTCValid` | Boolean | 0 | 1 | 0 | 0 |

For Boolean rows, Raw C equals Raw A; the ignored-source check below explicitly toggles raw 0 → 1 → 0 instead.


### Test Procedure/Test Steps

1. Set the selected input’s bypass enable to `0`; set its raw input to Raw A and its bypass value to Bypass B. Verify the effective input equals Raw A within 20 ms.
2. Set the selected bypass enable to `1`; verify the effective input becomes Bypass B within 20 ms and remains at Bypass B for the rest of the 100 ms window.
3. Set the raw input to Raw C for numeric rows. For Boolean rows, set raw to `1` for 100 ms, then `0` for 100 ms. Verify the effective input remains Bypass B throughout every window.
4. Set the selected bypass value to Bypass D; verify the effective input becomes Bypass D within 20 ms and remains there through 100 ms. Repeat from preconditions for each input row.

### Acceptance Criteria

1. Verify for all four rows that enabling the bypass selects Bypass B within 20 ms.
2. Verify raw-source changes never alter the selected effective value while bypass remains enabled.
3. Verify changing the selected bypass value produces Bypass D within 20 ms; no output-state assertion is required for this selector-focused case.

## AEB-TC-003 — Raw input selection

**Primary requirement:** AEB-REQ-002  
**Supporting context:** Specification sections 2–6 and 9  
**Specification version:** CFTS-AEB-001 v0.1  
**Review / execution:** Pending / Not executed

### Preconditions

1. Set the simulated controller execution period to 10 ms and synchronized post-execution logging to 10 ms; log all applied changes and relevant measurements.
2. Set `AEB_Enable_C = 0`; set `AEB_MinSpd_C = 10.0 km/h`, `AEB_MaxSpd_C = 80.0 km/h`, `AEB_TTCActivate_C = 1.5 s`, `AEB_TTCRelease_C = 2.0 s`, and `AEB_DecelCmd_C = -3.0 m/s²`.
3. Set `VehSpd_Raw = 50.0 km/h`, `TargetDetected_Raw = 1`, `TTC_Raw = 3.0 s`, and `TTCValid_Raw = 1` through the simulated input source; set all four bypass enables to `0`.
4. Set `VehSpd_BypVal = 50.0 km/h`, `TargetDetected_BypVal = 1`, `TTC_BypVal = 3.0 s`, and `TTCValid_BypVal = 1`; then set `VehSpd_BypEn = 1`, `TargetDetected_BypEn = 1`, `TTC_BypEn = 1`, and `TTCValid_BypEn = 1`.
5. Verify within 20 ms that `VehSpd_Eff = 50.0 km/h`, `TargetDetected_Eff = 1`, `TTC_Eff = 3.0 s`, and `TTCValid_Eff = 1`; verify `AEB_Active = 0` after the disabled setup has been applied for 20 ms.
6. Set `AEB_Enable_C = 0` and hold for 100 ms; verify `AEB_Active = 0` throughout this window because TTC is 3.0 s.
7. Set up each row below as an independent run. The prefix denotes the exact `_Raw`, `_BypEn`, `_BypVal`, and `_Eff` names in the specification. Use the unit listed in the row.

| Input prefix | Unit | Raw A | Bypass B | Raw C | Bypass D |
| --- | --- | --- | --- | --- | --- |
| `VehSpd` | km/h | 40.0 | 50.0 | 60.0 | 70.0 |
| `TargetDetected` | Boolean | 0 | 1 | 0 | 0 |
| `TTC` | s | 3.0 | 1.4 | 2.5 | 1.2 |
| `TTCValid` | Boolean | 0 | 1 | 0 | 0 |

For Boolean rows, Raw C equals Raw A; the ignored-source check below explicitly toggles raw 0 → 1 → 0 instead.


### Test Procedure/Test Steps

1. Set the selected raw input to Raw A and bypass value to Bypass B; set its bypass enable to `1`. Verify its effective input equals Bypass B within 20 ms.
2. Set the selected bypass enable to `0`; verify the effective input becomes Raw A within 20 ms and remains there through 100 ms.
3. Set the unselected bypass value to Bypass D and hold for 100 ms; verify the effective input remains Raw A throughout.
4. Set raw input to Raw C for numeric rows. For Boolean rows, set raw to `1` for 100 ms, then `0` for 100 ms. Verify each raw change appears at the effective input within 20 ms and persists for the rest of its window. Repeat from preconditions for each input row.

### Acceptance Criteria

1. Verify for all four rows that disabling the bypass selects Raw A within 20 ms.
2. Verify changes to the unselected bypass value do not alter the effective input.
3. Verify selected raw-source changes appear at the effective input within 20 ms; no output-state assertion is required for this selector-focused case.

## AEB-TC-004 — Activation at the TTC threshold

**Primary requirement:** AEB-REQ-003  
**Supporting context:** AEB-REQ-004 for the above-threshold check; sections 2–6  
**Specification version:** CFTS-AEB-001 v0.1  
**Review / execution:** Pending / Not executed

### Preconditions

1. Set the simulated controller execution period to 10 ms and synchronized post-execution logging to 10 ms; log all applied changes and relevant measurements.
2. Set `AEB_Enable_C = 0`; set `AEB_MinSpd_C = 10.0 km/h`, `AEB_MaxSpd_C = 80.0 km/h`, `AEB_TTCActivate_C = 1.5 s`, `AEB_TTCRelease_C = 2.0 s`, and `AEB_DecelCmd_C = -3.0 m/s²`.
3. Set `VehSpd_Raw = 50.0 km/h`, `TargetDetected_Raw = 1`, `TTC_Raw = 3.0 s`, and `TTCValid_Raw = 1` through the simulated input source; set all four bypass enables to `0`.
4. Set `VehSpd_BypVal = 50.0 km/h`, `TargetDetected_BypVal = 1`, `TTC_BypVal = 3.0 s`, and `TTCValid_BypVal = 1`; then set `VehSpd_BypEn = 1`, `TargetDetected_BypEn = 1`, `TTC_BypEn = 1`, and `TTCValid_BypEn = 1`.
5. Verify within 20 ms that `VehSpd_Eff = 50.0 km/h`, `TargetDetected_Eff = 1`, `TTC_Eff = 3.0 s`, and `TTCValid_Eff = 1`; verify `AEB_Active = 0` after the disabled setup has been applied for 20 ms.
6. Set `AEB_Enable_C = 1` and hold for 100 ms; verify `AEB_Active = 0` throughout this window because TTC is 3.0 s.

### Test Procedure/Test Steps

1. Set `TTC_BypVal = 1.6 s`; verify `TTC_Eff = 1.6 s` within 20 ms and verify `AEB_Active = 0` throughout 100 ms.
2. Set `TTC_BypVal = 1.5 s`; verify `TTC_Eff = 1.5 s` and `AEB_Active = 1` within 20 ms of this applied change; hold for 100 ms.

### Acceptance Criteria

1. Verify AEB remains inactive at 1.6 s when initially inactive.
2. Verify AEB activates at exactly 1.5 s within 20 ms and remains active through the observation window.

## AEB-TC-005 — Activation at the minimum eligible speed

**Primary requirement:** AEB-REQ-003  
**Supporting context:** Specification sections 2–6 and 9  
**Specification version:** CFTS-AEB-001 v0.1  
**Review / execution:** Pending / Not executed

### Preconditions

1. Set the simulated controller execution period to 10 ms and synchronized post-execution logging to 10 ms; log all applied changes and relevant measurements.
2. Set `AEB_Enable_C = 0`; set `AEB_MinSpd_C = 10.0 km/h`, `AEB_MaxSpd_C = 80.0 km/h`, `AEB_TTCActivate_C = 1.5 s`, `AEB_TTCRelease_C = 2.0 s`, and `AEB_DecelCmd_C = -3.0 m/s²`.
3. Set `VehSpd_Raw = 50.0 km/h`, `TargetDetected_Raw = 1`, `TTC_Raw = 3.0 s`, and `TTCValid_Raw = 1` through the simulated input source; set all four bypass enables to `0`.
4. Set `VehSpd_BypVal = 10.0 km/h`, `TargetDetected_BypVal = 1`, `TTC_BypVal = 3.0 s`, and `TTCValid_BypVal = 1`; then set `VehSpd_BypEn = 1`, `TargetDetected_BypEn = 1`, `TTC_BypEn = 1`, and `TTCValid_BypEn = 1`.
5. Verify within 20 ms that `VehSpd_Eff = 10.0 km/h`, `TargetDetected_Eff = 1`, `TTC_Eff = 3.0 s`, and `TTCValid_Eff = 1`; verify `AEB_Active = 0` after the disabled setup has been applied for 20 ms.
6. Set `AEB_Enable_C = 1` and hold for 100 ms; verify `AEB_Active = 0` throughout this window because TTC is 3.0 s.

### Test Procedure/Test Steps

1. Set `TTC_BypVal = 1.4 s`; verify `TTC_Eff = 1.4 s` within 20 ms; hold for 100 ms.
2. Verify `VehSpd_Eff = 10.0 km/h` throughout and verify `AEB_Active = 1` within 20 ms of the TTC change.

### Acceptance Criteria

1. Verify the inclusive minimum speed of 10.0 km/h permits activation when the other activation conditions are satisfied.
2. Verify activation occurs within 20 ms and persists through 100 ms.

## AEB-TC-006 — Activation at the maximum eligible speed

**Primary requirement:** AEB-REQ-003  
**Supporting context:** Specification sections 2–6 and 9  
**Specification version:** CFTS-AEB-001 v0.1  
**Review / execution:** Pending / Not executed

### Preconditions

1. Set the simulated controller execution period to 10 ms and synchronized post-execution logging to 10 ms; log all applied changes and relevant measurements.
2. Set `AEB_Enable_C = 0`; set `AEB_MinSpd_C = 10.0 km/h`, `AEB_MaxSpd_C = 80.0 km/h`, `AEB_TTCActivate_C = 1.5 s`, `AEB_TTCRelease_C = 2.0 s`, and `AEB_DecelCmd_C = -3.0 m/s²`.
3. Set `VehSpd_Raw = 50.0 km/h`, `TargetDetected_Raw = 1`, `TTC_Raw = 3.0 s`, and `TTCValid_Raw = 1` through the simulated input source; set all four bypass enables to `0`.
4. Set `VehSpd_BypVal = 80.0 km/h`, `TargetDetected_BypVal = 1`, `TTC_BypVal = 3.0 s`, and `TTCValid_BypVal = 1`; then set `VehSpd_BypEn = 1`, `TargetDetected_BypEn = 1`, `TTC_BypEn = 1`, and `TTCValid_BypEn = 1`.
5. Verify within 20 ms that `VehSpd_Eff = 80.0 km/h`, `TargetDetected_Eff = 1`, `TTC_Eff = 3.0 s`, and `TTCValid_Eff = 1`; verify `AEB_Active = 0` after the disabled setup has been applied for 20 ms.
6. Set `AEB_Enable_C = 1` and hold for 100 ms; verify `AEB_Active = 0` throughout this window because TTC is 3.0 s.

### Test Procedure/Test Steps

1. Set `TTC_BypVal = 1.4 s`; verify `TTC_Eff = 1.4 s` within 20 ms; hold for 100 ms.
2. Verify `VehSpd_Eff = 80.0 km/h` throughout and verify `AEB_Active = 1` within 20 ms of the TTC change.

### Acceptance Criteria

1. Verify the inclusive maximum speed of 80.0 km/h permits activation when the other activation conditions are satisfied.
2. Verify activation occurs within 20 ms and persists through 100 ms.

## AEB-TC-007 — Feature-disabled inhibition

**Primary requirement:** AEB-REQ-004  
**Supporting context:** Specification sections 2–6 and 9  
**Specification version:** CFTS-AEB-001 v0.1  
**Review / execution:** Pending / Not executed

### Preconditions

1. Set the simulated controller execution period to 10 ms and synchronized post-execution logging to 10 ms; log all applied changes and relevant measurements.
2. Set `AEB_Enable_C = 0`; set `AEB_MinSpd_C = 10.0 km/h`, `AEB_MaxSpd_C = 80.0 km/h`, `AEB_TTCActivate_C = 1.5 s`, `AEB_TTCRelease_C = 2.0 s`, and `AEB_DecelCmd_C = -3.0 m/s²`.
3. Set `VehSpd_Raw = 50.0 km/h`, `TargetDetected_Raw = 1`, `TTC_Raw = 3.0 s`, and `TTCValid_Raw = 1` through the simulated input source; set all four bypass enables to `0`.
4. Set `VehSpd_BypVal = 50.0 km/h`, `TargetDetected_BypVal = 1`, `TTC_BypVal = 3.0 s`, and `TTCValid_BypVal = 1`; then set `VehSpd_BypEn = 1`, `TargetDetected_BypEn = 1`, `TTC_BypEn = 1`, and `TTCValid_BypEn = 1`.
5. Verify within 20 ms that `VehSpd_Eff = 50.0 km/h`, `TargetDetected_Eff = 1`, `TTC_Eff = 3.0 s`, and `TTCValid_Eff = 1`; verify `AEB_Active = 0` after the disabled setup has been applied for 20 ms.
6. Set `AEB_Enable_C = 0` and hold for 100 ms; verify `AEB_Active = 0` throughout this window because TTC is 3.0 s.

### Test Procedure/Test Steps

1. Set `TTC_BypVal = 1.4 s`; verify `TTC_Eff = 1.4 s` within 20 ms; hold for 100 ms.
2. Verify `AEB_Enable_C = 0` persists and verify `AEB_Active = 0` at every sample throughout the window.

### Acceptance Criteria

1. Verify AEB remains inactive for the full 100 ms with `AEB_Enable_C = 0`, despite TTC being below the activation threshold.

## AEB-TC-008 — Missing-target inhibition

**Primary requirement:** AEB-REQ-004  
**Supporting context:** Specification sections 2–6 and 9  
**Specification version:** CFTS-AEB-001 v0.1  
**Review / execution:** Pending / Not executed

### Preconditions

1. Set the simulated controller execution period to 10 ms and synchronized post-execution logging to 10 ms; log all applied changes and relevant measurements.
2. Set `AEB_Enable_C = 0`; set `AEB_MinSpd_C = 10.0 km/h`, `AEB_MaxSpd_C = 80.0 km/h`, `AEB_TTCActivate_C = 1.5 s`, `AEB_TTCRelease_C = 2.0 s`, and `AEB_DecelCmd_C = -3.0 m/s²`.
3. Set `VehSpd_Raw = 50.0 km/h`, `TargetDetected_Raw = 1`, `TTC_Raw = 3.0 s`, and `TTCValid_Raw = 1` through the simulated input source; set all four bypass enables to `0`.
4. Set `VehSpd_BypVal = 50.0 km/h`, `TargetDetected_BypVal = 0`, `TTC_BypVal = 3.0 s`, and `TTCValid_BypVal = 1`; then set `VehSpd_BypEn = 1`, `TargetDetected_BypEn = 1`, `TTC_BypEn = 1`, and `TTCValid_BypEn = 1`.
5. Verify within 20 ms that `VehSpd_Eff = 50.0 km/h`, `TargetDetected_Eff = 0`, `TTC_Eff = 3.0 s`, and `TTCValid_Eff = 1`; verify `AEB_Active = 0` after the disabled setup has been applied for 20 ms.
6. Set `AEB_Enable_C = 1` and hold for 100 ms; verify `AEB_Active = 0` throughout this window because TTC is 3.0 s.

### Test Procedure/Test Steps

1. Set `TTC_BypVal = 1.4 s`; verify `TTC_Eff = 1.4 s` within 20 ms; hold for 100 ms.
2. Verify `TargetDetected_Eff = 0` persists and verify `AEB_Active = 0` at every sample throughout the window.

### Acceptance Criteria

1. Verify AEB remains inactive for the full 100 ms with `TargetDetected_Eff = 0`, despite TTC being below the activation threshold.

## AEB-TC-009 — Invalid-TTC inhibition

**Primary requirement:** AEB-REQ-004  
**Supporting context:** Specification sections 2–6 and 9  
**Specification version:** CFTS-AEB-001 v0.1  
**Review / execution:** Pending / Not executed

### Preconditions

1. Set the simulated controller execution period to 10 ms and synchronized post-execution logging to 10 ms; log all applied changes and relevant measurements.
2. Set `AEB_Enable_C = 0`; set `AEB_MinSpd_C = 10.0 km/h`, `AEB_MaxSpd_C = 80.0 km/h`, `AEB_TTCActivate_C = 1.5 s`, `AEB_TTCRelease_C = 2.0 s`, and `AEB_DecelCmd_C = -3.0 m/s²`.
3. Set `VehSpd_Raw = 50.0 km/h`, `TargetDetected_Raw = 1`, `TTC_Raw = 3.0 s`, and `TTCValid_Raw = 1` through the simulated input source; set all four bypass enables to `0`.
4. Set `VehSpd_BypVal = 50.0 km/h`, `TargetDetected_BypVal = 1`, `TTC_BypVal = 3.0 s`, and `TTCValid_BypVal = 0`; then set `VehSpd_BypEn = 1`, `TargetDetected_BypEn = 1`, `TTC_BypEn = 1`, and `TTCValid_BypEn = 1`.
5. Verify within 20 ms that `VehSpd_Eff = 50.0 km/h`, `TargetDetected_Eff = 1`, `TTC_Eff = 3.0 s`, and `TTCValid_Eff = 0`; verify `AEB_Active = 0` after the disabled setup has been applied for 20 ms.
6. Set `AEB_Enable_C = 1` and hold for 100 ms; verify `AEB_Active = 0` throughout this window because TTC is 3.0 s.

### Test Procedure/Test Steps

1. Set `TTC_BypVal = 1.4 s`; verify `TTC_Eff = 1.4 s` within 20 ms; hold for 100 ms.
2. Verify `TTCValid_Eff = 0` persists and verify `AEB_Active = 0` at every sample throughout the window.

### Acceptance Criteria

1. Verify AEB remains inactive for the full 100 ms with `TTCValid_Eff = 0`, despite TTC being below the activation threshold.

## AEB-TC-010 — Speed-range inhibition

**Primary requirement:** AEB-REQ-004  
**Supporting context:** Specification sections 2–6 and 9  
**Specification version:** CFTS-AEB-001 v0.1  
**Review / execution:** Pending / Not executed

### Preconditions

1. Set the simulated controller execution period to 10 ms and synchronized post-execution logging to 10 ms; log all applied changes and relevant measurements.
2. Set `AEB_Enable_C = 0`; set `AEB_MinSpd_C = 10.0 km/h`, `AEB_MaxSpd_C = 80.0 km/h`, `AEB_TTCActivate_C = 1.5 s`, `AEB_TTCRelease_C = 2.0 s`, and `AEB_DecelCmd_C = -3.0 m/s²`.
3. Set `VehSpd_Raw = 50.0 km/h`, `TargetDetected_Raw = 1`, `TTC_Raw = 3.0 s`, and `TTCValid_Raw = 1` through the simulated input source; set all four bypass enables to `0`.
4. Set `VehSpd_BypVal = 50.0 km/h`, `TargetDetected_BypVal = 1`, `TTC_BypVal = 3.0 s`, and `TTCValid_BypVal = 1`; then set `VehSpd_BypEn = 1`, `TargetDetected_BypEn = 1`, `TTC_BypEn = 1`, and `TTCValid_BypEn = 1`.
5. Verify within 20 ms that `VehSpd_Eff = 50.0 km/h`, `TargetDetected_Eff = 1`, `TTC_Eff = 3.0 s`, and `TTCValid_Eff = 1`; verify `AEB_Active = 0` after the disabled setup has been applied for 20 ms.
6. Set `AEB_Enable_C = 1` and hold for 100 ms; verify `AEB_Active = 0` throughout this window because TTC is 3.0 s.
7. Set up two independent runs with speed values 9.9 km/h and 80.1 km/h; repeat the full preconditions before each run.

### Test Procedure/Test Steps

1. Set `VehSpd_BypVal` to the run’s speed; verify `VehSpd_Eff` equals that value within 20 ms while TTC remains 3.0 s.
2. Set `TTC_BypVal = 1.4 s`; verify `TTC_Eff = 1.4 s` within 20 ms; hold for 100 ms and verify `AEB_Active = 0` throughout.

### Acceptance Criteria

1. Verify AEB remains inactive throughout each run: at 9.9 km/h and at 80.1 km/h, despite all other activation conditions being satisfied.

## AEB-TC-011 — Active-state retention in the hysteresis band

**Primary requirement:** AEB-REQ-005  
**Supporting context:** Specification sections 2–6 and 9  
**Specification version:** CFTS-AEB-001 v0.1  
**Review / execution:** Pending / Not executed

### Preconditions

1. Set the simulated controller execution period to 10 ms and synchronized post-execution logging to 10 ms; log all applied changes and relevant measurements.
2. Set `AEB_Enable_C = 0`; set `AEB_MinSpd_C = 10.0 km/h`, `AEB_MaxSpd_C = 80.0 km/h`, `AEB_TTCActivate_C = 1.5 s`, `AEB_TTCRelease_C = 2.0 s`, and `AEB_DecelCmd_C = -3.0 m/s²`.
3. Set `VehSpd_Raw = 50.0 km/h`, `TargetDetected_Raw = 1`, `TTC_Raw = 3.0 s`, and `TTCValid_Raw = 1` through the simulated input source; set all four bypass enables to `0`.
4. Set `VehSpd_BypVal = 50.0 km/h`, `TargetDetected_BypVal = 1`, `TTC_BypVal = 3.0 s`, and `TTCValid_BypVal = 1`; then set `VehSpd_BypEn = 1`, `TargetDetected_BypEn = 1`, `TTC_BypEn = 1`, and `TTCValid_BypEn = 1`.
5. Verify within 20 ms that `VehSpd_Eff = 50.0 km/h`, `TargetDetected_Eff = 1`, `TTC_Eff = 3.0 s`, and `TTCValid_Eff = 1`; verify `AEB_Active = 0` after the disabled setup has been applied for 20 ms.
6. Set `AEB_Enable_C = 1` and hold for 100 ms; verify `AEB_Active = 0` throughout this window because TTC is 3.0 s.
7. Set `TTC_BypVal = 1.4 s`; verify `TTC_Eff = 1.4 s` and `AEB_Active = 1` within 20 ms of the applied change; hold for 100 ms and verify the active state persists.

### Test Procedure/Test Steps

1. Set `TTC_BypVal = 1.6 s`; verify `TTC_Eff = 1.6 s` within 20 ms and verify `AEB_Active = 1` throughout 100 ms.
2. Set `TTC_BypVal = 1.9 s`; verify `TTC_Eff = 1.9 s` within 20 ms and verify `AEB_Active = 1` throughout another 100 ms.

### Acceptance Criteria

1. Verify an already active controller remains active at 1.6 s and 1.9 s while eligible; exceeding the activation threshold alone shall not release AEB.

## AEB-TC-012 — Release at the TTC threshold

**Primary requirement:** AEB-REQ-006  
**Supporting context:** AEB-REQ-005 for retention at 1.9 s; sections 2–6  
**Specification version:** CFTS-AEB-001 v0.1  
**Review / execution:** Pending / Not executed

### Preconditions

1. Set the simulated controller execution period to 10 ms and synchronized post-execution logging to 10 ms; log all applied changes and relevant measurements.
2. Set `AEB_Enable_C = 0`; set `AEB_MinSpd_C = 10.0 km/h`, `AEB_MaxSpd_C = 80.0 km/h`, `AEB_TTCActivate_C = 1.5 s`, `AEB_TTCRelease_C = 2.0 s`, and `AEB_DecelCmd_C = -3.0 m/s²`.
3. Set `VehSpd_Raw = 50.0 km/h`, `TargetDetected_Raw = 1`, `TTC_Raw = 3.0 s`, and `TTCValid_Raw = 1` through the simulated input source; set all four bypass enables to `0`.
4. Set `VehSpd_BypVal = 50.0 km/h`, `TargetDetected_BypVal = 1`, `TTC_BypVal = 3.0 s`, and `TTCValid_BypVal = 1`; then set `VehSpd_BypEn = 1`, `TargetDetected_BypEn = 1`, `TTC_BypEn = 1`, and `TTCValid_BypEn = 1`.
5. Verify within 20 ms that `VehSpd_Eff = 50.0 km/h`, `TargetDetected_Eff = 1`, `TTC_Eff = 3.0 s`, and `TTCValid_Eff = 1`; verify `AEB_Active = 0` after the disabled setup has been applied for 20 ms.
6. Set `AEB_Enable_C = 1` and hold for 100 ms; verify `AEB_Active = 0` throughout this window because TTC is 3.0 s.
7. Set `TTC_BypVal = 1.4 s`; verify `TTC_Eff = 1.4 s` and `AEB_Active = 1` within 20 ms of the applied change; hold for 100 ms and verify the active state persists.

### Test Procedure/Test Steps

1. Set `TTC_BypVal = 1.9 s`; verify `TTC_Eff = 1.9 s` within 20 ms and verify `AEB_Active = 1` throughout 100 ms.
2. Set `TTC_BypVal = 2.0 s`; verify `TTC_Eff = 2.0 s` and `AEB_Active = 0` within 20 ms of the applied change; hold for 100 ms.

### Acceptance Criteria

1. Verify AEB remains active below release at 1.9 s.
2. Verify AEB deactivates at exactly 2.0 s within 20 ms and remains inactive through 100 ms.

## AEB-TC-013 — Release on target loss

**Primary requirement:** AEB-REQ-006  
**Supporting context:** Specification sections 2–6 and 9  
**Specification version:** CFTS-AEB-001 v0.1  
**Review / execution:** Pending / Not executed

### Preconditions

1. Set the simulated controller execution period to 10 ms and synchronized post-execution logging to 10 ms; log all applied changes and relevant measurements.
2. Set `AEB_Enable_C = 0`; set `AEB_MinSpd_C = 10.0 km/h`, `AEB_MaxSpd_C = 80.0 km/h`, `AEB_TTCActivate_C = 1.5 s`, `AEB_TTCRelease_C = 2.0 s`, and `AEB_DecelCmd_C = -3.0 m/s²`.
3. Set `VehSpd_Raw = 50.0 km/h`, `TargetDetected_Raw = 1`, `TTC_Raw = 3.0 s`, and `TTCValid_Raw = 1` through the simulated input source; set all four bypass enables to `0`.
4. Set `VehSpd_BypVal = 50.0 km/h`, `TargetDetected_BypVal = 1`, `TTC_BypVal = 3.0 s`, and `TTCValid_BypVal = 1`; then set `VehSpd_BypEn = 1`, `TargetDetected_BypEn = 1`, `TTC_BypEn = 1`, and `TTCValid_BypEn = 1`.
5. Verify within 20 ms that `VehSpd_Eff = 50.0 km/h`, `TargetDetected_Eff = 1`, `TTC_Eff = 3.0 s`, and `TTCValid_Eff = 1`; verify `AEB_Active = 0` after the disabled setup has been applied for 20 ms.
6. Set `AEB_Enable_C = 1` and hold for 100 ms; verify `AEB_Active = 0` throughout this window because TTC is 3.0 s.
7. Set `TTC_BypVal = 1.4 s`; verify `TTC_Eff = 1.4 s` and `AEB_Active = 1` within 20 ms of the applied change; hold for 100 ms and verify the active state persists.

### Test Procedure/Test Steps

1. Set `TargetDetected_BypVal = 0`; hold for 100 ms with TTC still 1.4 s.
2. Verify `TargetDetected_Eff = 0` and `AEB_Active = 0` within 20 ms of the applied change.

### Acceptance Criteria

1. Verify AEB deactivates within 20 ms and remains inactive through 100 ms despite TTC remaining below the activation threshold.

## AEB-TC-014 — Release on feature disable

**Primary requirement:** AEB-REQ-006  
**Supporting context:** Specification sections 2–6 and 9  
**Specification version:** CFTS-AEB-001 v0.1  
**Review / execution:** Pending / Not executed

### Preconditions

1. Set the simulated controller execution period to 10 ms and synchronized post-execution logging to 10 ms; log all applied changes and relevant measurements.
2. Set `AEB_Enable_C = 0`; set `AEB_MinSpd_C = 10.0 km/h`, `AEB_MaxSpd_C = 80.0 km/h`, `AEB_TTCActivate_C = 1.5 s`, `AEB_TTCRelease_C = 2.0 s`, and `AEB_DecelCmd_C = -3.0 m/s²`.
3. Set `VehSpd_Raw = 50.0 km/h`, `TargetDetected_Raw = 1`, `TTC_Raw = 3.0 s`, and `TTCValid_Raw = 1` through the simulated input source; set all four bypass enables to `0`.
4. Set `VehSpd_BypVal = 50.0 km/h`, `TargetDetected_BypVal = 1`, `TTC_BypVal = 3.0 s`, and `TTCValid_BypVal = 1`; then set `VehSpd_BypEn = 1`, `TargetDetected_BypEn = 1`, `TTC_BypEn = 1`, and `TTCValid_BypEn = 1`.
5. Verify within 20 ms that `VehSpd_Eff = 50.0 km/h`, `TargetDetected_Eff = 1`, `TTC_Eff = 3.0 s`, and `TTCValid_Eff = 1`; verify `AEB_Active = 0` after the disabled setup has been applied for 20 ms.
6. Set `AEB_Enable_C = 1` and hold for 100 ms; verify `AEB_Active = 0` throughout this window because TTC is 3.0 s.
7. Set `TTC_BypVal = 1.4 s`; verify `TTC_Eff = 1.4 s` and `AEB_Active = 1` within 20 ms of the applied change; hold for 100 ms and verify the active state persists.

### Test Procedure/Test Steps

1. Set `AEB_Enable_C = 0`; hold for 100 ms with TTC still 1.4 s.
2. Verify `AEB_Active = 0` within 20 ms of the applied enable-calibration change.

### Acceptance Criteria

1. Verify AEB deactivates within 20 ms and remains inactive through 100 ms despite TTC remaining below the activation threshold.

## AEB-TC-015 — Deceleration request follows state and calibration

**Primary requirement:** AEB-REQ-007  
**Supporting context:** AEB-REQ-003/006 for transitions; section 2 timing and section 5 calibration. -2.0 m/s² is a deliberate test value, not a changed default.  
**Specification version:** CFTS-AEB-001 v0.1  
**Review / execution:** Pending / Not executed

### Preconditions

1. Set the simulated controller execution period to 10 ms and synchronized post-execution logging to 10 ms; log all applied changes and relevant measurements.
2. Set `AEB_Enable_C = 0`; set `AEB_MinSpd_C = 10.0 km/h`, `AEB_MaxSpd_C = 80.0 km/h`, `AEB_TTCActivate_C = 1.5 s`, `AEB_TTCRelease_C = 2.0 s`, and `AEB_DecelCmd_C = -3.0 m/s²`.
3. Set `VehSpd_Raw = 50.0 km/h`, `TargetDetected_Raw = 1`, `TTC_Raw = 3.0 s`, and `TTCValid_Raw = 1` through the simulated input source; set all four bypass enables to `0`.
4. Set `VehSpd_BypVal = 50.0 km/h`, `TargetDetected_BypVal = 1`, `TTC_BypVal = 3.0 s`, and `TTCValid_BypVal = 1`; then set `VehSpd_BypEn = 1`, `TargetDetected_BypEn = 1`, `TTC_BypEn = 1`, and `TTCValid_BypEn = 1`.
5. Verify within 20 ms that `VehSpd_Eff = 50.0 km/h`, `TargetDetected_Eff = 1`, `TTC_Eff = 3.0 s`, and `TTCValid_Eff = 1`; verify `AEB_Active = 0` after the disabled setup has been applied for 20 ms.
6. Set `AEB_Enable_C = 1` and hold for 100 ms; verify `AEB_Active = 0` throughout this window because TTC is 3.0 s.

### Test Procedure/Test Steps

1. Verify `AEB_Active = 0` and `AEB_DecelReq = 0.0 m/s²` at every logged sample for 100 ms.
2. Set `TTC_BypVal = 1.4 s`; verify activation within 20 ms and verify the same post-execution sample first reporting `AEB_Active = 1` reports `AEB_DecelReq = -3.0 m/s²`; hold for 100 ms.
3. Set `AEB_DecelCmd_C = -2.0 m/s²`; verify `AEB_DecelReq = -2.0 m/s²` on the first controller execution that reads the applied calibration (within 20 ms). Verify `AEB_Active = 1` throughout 100 ms.
4. Set `TTC_BypVal = 2.0 s`; verify release within 20 ms and verify the same post-execution sample first reporting `AEB_Active = 0` reports `AEB_DecelReq = 0.0 m/s²`; hold for 100 ms.

### Acceptance Criteria

1. Verify inactive state always corresponds to 0.0 m/s² in the logged windows.
2. Verify active state corresponds to the currently applied deceleration calibration: -3.0 m/s² initially and -2.0 m/s² after the calibration update is read.
3. Verify no logged controller cycle has mismatched active status and deceleration request. This validates cycle-level synchronization only, not sub-cycle write ordering.

## Review notes and remaining coverage

- These are 15 case records, with parameterized runs inside cases 002, 003, and 010. They are not 15 independent requirements or a complete training corpus.
- Coverage is intentionally partial. Additional cases could cover release on invalid TTC or speed departure, simultaneous eligibility loss and low TTC, selector behavior while active, calibration threshold changes, and reactivation after release.
- Case 001 is blocked for execution until initialization capture is implemented. No actual test harness or controller has been executed for this document.
- The per-case model input must include the linked requirement text and relevant specification context. Include these timing conventions and expand parameterized rows if exporting a single-row example. Do not train on case text while omitting definitions it relies on.
- Related cases and their paraphrases/numeric variants must remain in the same dataset split. This pilot is intended for review and format development; no train/evaluation split is assigned here.
- Human reviewer: pending. Execution evidence: none. Cases were drafted with ChatGPT assistance against CFTS-AEB-001 v0.1.
