# Automatic Differential Pressure Control System (PFEIFFER Compartment)

Design, PLC programming, and functional validation of an automated differential pressure control system for an enclosed PFEIFFER compartment using a **Siemens LOGO! 8.3** PLC and a **Nidec Commander S100** variable frequency drive (VFD).

## Goal

Maintain a controlled differential pressure inside the PFEIFFER compartment (separated from the adjacent workspace by a sealed wall and an airlock) to ensure required operating conditions and prevent uncontrolled airflow. The system continuously measures differential pressure and automatically adjusts exhaust fan speed in discrete frequency stages, with built-in start/stop control and fault alarm signaling.

## System Architecture & Hardware

- **PLC:** Siemens LOGO! 8.3 (programmed in LOGO! Soft Comfort using FBD)
- **Differential Pressure Transmitter:** HOTCOLD DPC250-D (`0–10 V` analog output mapped linearly to `0–100 Pa`: `0 V = 0 Pa`, `5 V = 50 Pa`, `10 V = 100 Pa`)
- **Variable Frequency Drive (VFD):** Nidec Commander S100 with three preset speed stages:
  - Stage 1: `30 Hz` (minimum operating speed)
  - Stage 2: `~38 Hz` (intermediate operating speed)
  - Stage 3: `50 Hz` (maximum operating speed)
- **Motor & Fan:** Lenze `m550-H71/L4` three-phase motor (`0.55 kW` at `50 Hz`, `1430 rpm`, wired in `230 V` delta configuration)

## PLC I/O Configuration

| Type | Address | Description |
| --- | --- | --- |
| Analog Input | `AI1` | Differential pressure signal (`0–10 V`) from DPC250-D |
| Digital Input | `I5` | Enable + Run push-button |
| Digital Output | `Q1` | VFD preset command (`~38 Hz`) |
| Digital Output | `Q2` | VFD preset command (`50 Hz`) |
| Digital Output | `Q3` | VFD Enable |
| Digital Output | `Q4` | VFD Run |
| Digital Output | `Q5` | Alarm output |

## Control Strategy & PLC Logic

The control program was built in **Siemens LOGO! Soft Comfort** using **Function Block Diagram (FBD)** with analog scaling, threshold comparators, logic gates, RS latches, and timers:

1. **Analog signal processing:** The `0–10 V` signal on `AI1` is scaled and conditioned (starting with analog block `B001`, configured with `Gain = 0.12`, `Offset = 20`, `Point = 0`) and routed through comparison blocks (`B009`, `B002`, `B011`, `B003`) to evaluate the pressure error ($e = \Delta P_{SP} - \Delta P_{measured}$).
2. **Discrete frequency stage selection:** Based on the processed pressure thresholds, the PLC activates the corresponding digital outputs (`Q1`, `Q2`) to select the `30 Hz`, `~38 Hz`, or `50 Hz` VFD preset.
3. **Start/Stop sequence:** Motor operation is conditioned by the `Enable + Run` logic (`I5`, `Q3`, `Q4`), ensuring controlled startup and immediate shutdown when disabled.
4. **Alarm monitoring:** Output `Q5` is triggered under abnormal conditions, including underpressure, overpressure, and VFD faults.

## Validation & Functional Test Plan

| ID | Test | Acceptance Criterion |
| --- | --- | --- |
| `T1` | DPC250-D calibration | Reads `0 Pa` at equalized pressures |
| `T2` | VFD parameterization check | Drive parameters configured properly |
| `T3` | Frequency stage selection | Switches between `30 / 38 / 50 Hz` per pressure condition |
| `T4` | Closed-loop test | Differential pressure converges toward setpoint |
| `T5` | Minimum frequency check | Motor does not operate below `30 Hz` |
| `T6` | VFD fault simulation | Alarm output (`Q5`) activates |
| `T7` | Underpressure alarm test | Alarm activates after the configured delay |
| `T8` | Overpressure alarm test | Frequency drops to `30 Hz` and alarm activates |

## Tools & Technologies

Siemens LOGO! 8.3 PLC, Siemens LOGO! Soft Comfort (FBD), Nidec Commander S100 VFD, HOTCOLD DPC250-D Transmitter, 0–10 V Analog Signal Scaling, Three-Phase Motor Control.

## Repository Structure

```text
differential-pressure-control/
├── README.md
└── docs/            # Full project documentation (PDF)
