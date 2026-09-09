# FPGA Earthquake Detection System

A Digilent Basys 3 project that combines stored sample data, calibration, finite state machines, and a digital datapath to indicate a seismic event through LEDs and a seven-segment display.

**Focus:** FPGA logic · Datapath and control design · FSMs · ROM data · Hardware interfaces

[![Basys 3 board with OFF shown on its seven-segment display](images/fpga-demo.jpg)](media/fpga-demo.mp4)

*Frame at 00:05 from the [FPGA demonstration](media/fpga-demo.mp4), showing the Basys 3 board with “OFF” on its seven-segment display.*

## Overview

This project implements an earthquake detection demonstrator on a Basys 3 FPGA board. It uses stored sample data, a calibration baseline, and detection logic, with push-button controls and visual status outputs. The project brings together arithmetic, sequential control, ROM access, and user-interface logic in one hardware system.

The available materials include design drawings, a board demonstration, and Lab 4b simulation, timing, and utilization evidence. HDL, constraints, ROM initialization files, testbenches, and logs are still needed to reproduce the implementation.

## Modes of Operation

| Mode / controller state | Behavior reported in Lab 4b |
| --- | --- |
| Off (`00`) | Reset state; `LD3:0 = 0001` and display shows `OFF`. |
| Calibration (`01`) | Captures four samples, latches the baseline, and displays `DONE`. |
| Tracking / Measuring (`10`) | Scrolls `READING`; the LED bar reflects `reading - baseline`. A difference above 5 latches an alert. |
| Acknowledge (`11`) | Clears the alert latch/value and scrolls `ACKNOWLEDGED`; the next mode advance returns to Off. |

These behaviors come from the lab's scenario table; see the [simulation evidence](docs/verification.md#simulation-evidence) for the reported results and unresolved error counters.

## Datapath

The earlier Lab 4a design drawing shows:

- A ROM sample source, waveform-selection multiplexers, and address-generation logic.
- Current and previous sample registers, a comparator, a magnitude register, and a small engine FSM that generates `mag_en`.
- Four calibration registers whose values are added and shifted right by two before being stored as a baseline average.
- Subtraction of the calibration baseline from a stored magnitude, followed by a comparator with an output labeled `gt5`.
- ROM-based display data, character-window logic, a display multiplexer, an LED bar module, and a state decoder.

![Datapath drawing with sample processing, calibration, detection, and display logic](images/architecture/datapath.png)

The Lab 4b retrospective documents later changes: 16-bit seismic samples, smaller control FSMs, and a single looping wave file that replaced the planned multiple wave-pattern files and separate iteration engine. The drawing preserves the earlier design; [architecture notes](docs/architecture.md) explain the changes and remaining implementation questions.

## Control FSMs

These drawings record the design history. Lab 4b confirms that smaller calibration and alert FSMs were sequenced by the main controller; final HDL is needed to compare their exact implementation with the sketches.

| Diagram | Responsibility |
| --- | --- |
| [High-level state machine](images/architecture/hlsm.png) | Coordinates the operating modes. |
| [Calibration FSM](images/architecture/calibration-fsm.png) | Sequences the four calibration-register loads and the average-register load. |
| [Engine FSM](images/architecture/engine-fsm.png) | Uses comparator input `p_lt` to generate a magnitude-register enable on a state transition. |
| [Tracking FSM](images/architecture/tracking-fsm.png) | Enters an alert state when `gt5` is asserted and holds that state until acknowledgement. |

## Inputs / Outputs

| Interface | Purpose |
| --- | --- |
| Push buttons | User control of the operating modes. Exact mapping is TODO. |
| Switches | The datapath drawing uses `sw_1` and `sw_0` to select stored-waveform parameters. Confirm the implemented mapping. |
| Stored sample data | Supplies the input sequence used by the detector. Sample provenance, units, and ROM contents are TODO. |
| LEDs | Lab 4b identifies `LD3:0` as mode indicators and `LD15:6` as the reading-minus-baseline bar; the bar flashes during an alert. |
| Seven-segment display | Lab 4b reports `OFF`, `DONE`, `READING`, `ALERT`, and `ACKNOWLEDGED` messages. Update rate remains to be documented. |

## Verification, Debugging & Results

The Lab 4b deliverable includes controller and integration waveforms, five integration scenarios marked **Pass**, and Vivado timing and utilization summaries. The scenarios cover reset, four-sample calibration, tracking, a persistent threshold alert, and acknowledgement/return to Off.

The waveform screenshots also show nonzero error counters: **4** for the controller and **3** for integration at the displayed times. Those counters must be reconciled with the scenario table before claiming a clean regression. See the [verification results and evidence](docs/verification.md) for the figures and limitations.

- **Implementation refinement:** the team replaced difficult low-level control logic with smaller alert and calibration FSMs under a main controller. The seismic unit was adapted to 16-bit samples, and multiple wave-pattern ROM files were consolidated into one looping wave file (lab deliverable, p. 10).
- **Reported timing:** the implementation summary shows **2.642 ns setup slack**, **0.125 ns hold slack**, and zero failing timing endpoints. The accompanying utilization report is labeled Fully Placed; final routed timing and clock constraints remain to be added (p. 6).
- **Placed resources:** **314 LUTs (1.51%)**, **229 registers (0.55%)**, **3.5 BRAM tiles (7%)**, and **0 DSPs** for `eew_top` on `xc7a35tcpg236-1`, using Vivado 2025.2.1 (pp. 6-8).

The documented changes explain how the design evolved; the source does not identify the cause of the waveform error counters or provide a confirmed fix. The repository still needs the HDL, testbenches, logs, and memory files for reproducible verification.

## Media

- [Simulation waveforms and timing summaries](docs/verification.md)
- [High-level state machine](images/architecture/hlsm.png)
- [Calibration controller](images/architecture/calibration-fsm.png)
- [Magnitude engine controller](images/architecture/engine-fsm.png)
- [Tracking / alert controller](images/architecture/tracking-fsm.png)
- [Full datapath PDF](hardware/datapath.pdf)
- [FPGA demonstration video](media/fpga-demo.mp4)
- [Board preview at 00:05](images/fpga-demo.jpg)

## Repository Structure

```text
README.md
docs/
  architecture.md          # Diagram interpretation and open implementation questions
  verification.md          # Lab results, debugging changes, and follow-up checks
hardware/
  datapath.pdf             # Original design drawing
images/
  architecture/            # FSM images and readable datapath preview
  verification/            # Controller, integration, and timing evidence
  fpga-demo.jpg            # Frame at 00:05 from the board demonstration
media/
  fpga-demo.mp4            # Board demonstration video
```

Add `src/`, `sim/`, and `constraints/` when the actual HDL, testbenches, and board constraints are available. Preserve their original organization if the source project already has one.

## Reproducing the Project

Reproduction is pending the source files. The hardware platform is Digilent Basys 3. Lab 4b identifies Vivado 2025.2.1, top-level design `eew_top`, and device `xc7a35tcpg236-1`. TODO: add the source and ROM file list, board constraints, testbenches, simulation commands, and programming steps.

## Next Steps

- Publish the HDL and stored sample data with provenance and any required attribution.
- Confirm final arithmetic widths, signedness, and hardware input mapping from HDL.
- Add a concise demonstration walkthrough with inputs and expected outputs.
- Reconcile the waveform error counters with the scenario table and record a reproducible clean rerun.
