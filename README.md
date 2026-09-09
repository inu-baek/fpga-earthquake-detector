# FPGA Earthquake Detection System

A Digilent Basys 3 project that combines stored sample data, calibration, finite state machines, and a digital datapath to indicate a seismic event through LEDs and a seven-segment display.

**Focus:** FPGA logic · Datapath and control design · FSMs · ROM data · Hardware interfaces

## Demonstration

https://github.com/user-attachments/assets/84b5f458-4d31-4fb0-b496-760ad7489f63

*Basys 3 board demonstration with mode controls, LEDs, and a seven-segment display.*

## Overview

This project implements an earthquake detection demonstrator on a Basys 3 FPGA board. It uses stored sample data, a calibration baseline, and detection logic, with push-button controls and visual status outputs. The project brings together arithmetic, sequential control, ROM access, and user-interface logic in one hardware system.

The project is documented through design drawings, a board demonstration, and Lab 4b simulation, timing, and utilization evidence.

## Verification, Debugging & Results

The Lab 4b implementation report includes controller and integration simulations, timing summaries, and resource utilization for `eew_top` on `xc7a35tcpg236-1`, using Vivado 2025.2.1.

### Simulation Waveforms

The report marks five integration scenarios **Pass**: reset, four-sample calibration, tracking, a persistent threshold alert, and acknowledgement/return to Off.

**Controller simulation — mode transitions and active-state outputs**

![Controller simulation showing mode transitions and the recorded error counter](images/verification/controller-waveform.png)

**Integration simulation — button inputs, LED outputs, and display signals**

![Integration simulation showing LED and display outputs and the recorded error counter](images/verification/integration-waveform.png)

The screenshots show error counters of **4** for the controller and **3** for integration at the displayed times. These differ from the table's Pass entries, so the supplied evidence does not establish a clean regression run.

### Timing & Resource Results

| Reported implementation metric | Result |
| --- | ---: |
| Worst setup slack | 2.642 ns |
| Worst hold slack | 0.125 ns |
| Failing timing endpoints | 0 |
| LUTs | 314 (1.51%) |
| Registers | 229 (0.55%) |
| Block RAM tiles | 3.5 (7%) |
| DSPs | 0 |

**Implementation timing summary**

![Implementation timing summary with positive setup, hold, and pulse-width slack](images/verification/implementation-timing.png)

**Synthesis timing summary**

![Synthesis timing summary with positive setup, hold, and pulse-width slack](images/verification/synthesis-timing.png)

The accompanying utilization report is labeled **Fully Placed**; these figures do not establish post-route timing signoff.

### Debugging & Design Refinements

- **Simplified control:** replaced difficult low-level control logic with smaller alert and calibration FSMs under a main controller.
- **Reworked sample playback:** adapted the seismic unit to 16-bit samples and consolidated multiple wave-pattern ROM files into one looping wave file, removing the planned separate iteration engine.

These changes are documented in the team's implementation retrospective. See the [detailed lab evidence](docs/verification.md) for the scenario table, report references, and interpretation.

## Modes of Operation

| Mode / controller state | Behavior reported in Lab 4b |
| --- | --- |
| Off (`00`) | Reset state; `LD3:0 = 0001` and display shows `OFF`. |
| Calibration (`01`) | Captures four samples, latches the baseline, and displays `DONE`. |
| Tracking / Measuring (`10`) | Scrolls `READING`; the LED bar reflects `reading - baseline`. A difference above 5 latches an alert. |
| Acknowledge (`11`) | Clears the alert latch/value and scrolls `ACKNOWLEDGED`; the next mode advance returns to Off. |

These behaviors are recorded in the Lab 4b scenario table.

## Datapath

The earlier Lab 4a design drawing shows:

- A ROM sample source, waveform-selection multiplexers, and address-generation logic.
- Current and previous sample registers, a comparator, a magnitude register, and a small engine FSM that generates `mag_en`.
- Four calibration registers whose values are added and shifted right by two before being stored as a baseline average.
- Subtraction of the calibration baseline from a stored magnitude, followed by a comparator with an output labeled `gt5`.
- ROM-based display data, character-window logic, a display multiplexer, an LED bar module, and a state decoder.

![Datapath drawing with sample processing, calibration, detection, and display logic](images/architecture/datapath.png)

The Lab 4b retrospective documents later changes: 16-bit seismic samples, smaller control FSMs, and a single looping wave file that replaced the planned multiple wave-pattern files and separate iteration engine. The drawing preserves the earlier design; [architecture notes](docs/architecture.md) explain how the design evolved.

## Control FSMs

These drawings record the design history. Lab 4b describes smaller calibration and alert FSMs sequenced by the main controller.

| Diagram | Responsibility |
| --- | --- |
| [High-level state machine](images/architecture/hlsm.png) | Coordinates the operating modes. |
| [Calibration FSM](images/architecture/calibration-fsm.png) | Sequences the four calibration-register loads and the average-register load. |
| [Engine FSM](images/architecture/engine-fsm.png) | Uses comparator input `p_lt` to generate a magnitude-register enable on a state transition. |
| [Tracking FSM](images/architecture/tracking-fsm.png) | Enters an alert state when `gt5` is asserted and holds that state until acknowledgement. |

## Inputs / Outputs

| Interface | Purpose |
| --- | --- |
| Push buttons | User control of the operating modes. |
| Switches | The earlier datapath drawing uses `sw_1` and `sw_0` to select stored-waveform parameters. |
| Stored sample data | Supplies the input sequence used by the detector. |
| LEDs | Lab 4b identifies `LD3:0` as mode indicators and `LD15:6` as the reading-minus-baseline bar; the bar flashes during an alert. |
| Seven-segment display | Lab 4b reports `OFF`, `DONE`, `READING`, `ALERT`, and `ACKNOWLEDGED` messages. |

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
  architecture.md          # Design drawings and implementation refinements
  verification.md          # Lab results and debugging changes
hardware/
  datapath.pdf             # Original design drawing
images/
  architecture/            # FSM images and readable datapath preview
  verification/            # Controller, integration, and timing evidence
  fpga-demo.jpg            # Frame at 00:05 from the board demonstration
media/
  fpga-demo.mp4            # Board demonstration video
```
