# Architecture Notes

These notes describe the supplied handwritten design drawings. They distinguish visible design intent from facts that require the final implementation. The diagrams have not been checked against HDL or a synthesis report.

## Evidence Available

| Artifact | What it shows |
| --- | --- |
| [High-level state machine](../images/architecture/hlsm.png) | Four states: Off `00`, Calibration `01`, Tracking `10`, Acknowledge `11`; control inputs labeled `c`, `t`, `a`, and `f`. |
| [Calibration FSM](../images/architecture/calibration-fsm.png) | A six-state sequence: idle/reset, load `r0`, load `r1`, load `r2`, load `r3`, and load the average. |
| [Engine FSM](../images/architecture/engine-fsm.png) | Rise and Fall states controlled by `p_lt`; `mag_en=1` is written on the Rise-to-Fall transition. |
| [Tracking FSM](../images/architecture/tracking-fsm.png) | Inactive, tracking, and alert states; the alert state's output mode is held until `ack_active`. |
| [Datapath PDF](../hardware/datapath.pdf) | Sample-ROM access, arithmetic and storage, calibration, threshold comparison, FSM interfaces, seven-segment and LED logic. |

## Sample and Magnitude Processing

The datapath shows a ROM access module whose data output is labeled 5-bit. A bitwise inversion, addition of one, and a multiplexer controlled by the sample's high bit appear before `cur_reg`. This is consistent with two's-complement magnitude conversion, but the interpretation and handling of the most-negative input require confirmation from the RTL.

Registers labeled `cur_reg`, `prev_reg`, and `prev2_reg` retain samples. A 5-bit comparator drives `p_lt`; a magnitude register is enabled by `mag_en`. The engine FSM has two states named Rise and Fall. Its transition from Rise to Fall asserts `mag_en`. The drawings suggest a comparison-based magnitude-capture mechanism; they do not by themselves establish the exact sample alignment or final peak-detection behavior.

The address-generation area contains two four-way multiplexers controlled by `sw_1` and `sw_0`, an incrementer, a comparator, and an adder. The multiplexers are labeled with waveform maxima and starting addresses. A red annotation says that bit width depends on the waveform files. TODO: supply the ROM image, waveform lengths, sample format, and address bounds.

## Calibration Baseline

The calibration FSM sequentially asserts load signals for four registers and then the average register. The datapath connects four 5-bit calibration values to a two-level adder tree, a right shift by two, and `cal_avg`.

At the level of the drawing, the intended operation is:

```text
cal_avg = (cal_reg0 + cal_reg1 + cal_reg2 + cal_reg3) >> 2
```

The first pairwise sums are labeled 6-bit and the combined sum 7-bit. The final adder is nevertheless labeled “5-bit adder,” so the actual intermediate widths need checking. TODO: confirm rounding/truncation, sample spacing, reset behavior, and whether calibration records raw magnitudes or captured peaks.

## Threshold and Alert

The magnitude register and calibration average feed a subtractor; its output is compared with a constant labeled `5`. The comparator output is labeled `gt5` and enters the alert FSM. This supports describing a threshold comparison relative to a calibration baseline. It does not establish a physical seismic magnitude scale or calibrated acceleration unit.

The tracking FSM moves from an inactive state to a monitoring state when `trk_active` is asserted. A true `gt5` condition moves it into an alert state with `trkout_mode=1`; it remains there until `ack_active` is asserted. TODO: verify how leaving Tracking mode affects this subcontroller and how arithmetic underflow or negative differences are represented.

## Displays

The seven-segment section contains four ROM access paths, per-character address arithmetic, a shared display multiplexer, and a 5 x 5-bit register file. A handwritten note identifies the register file as storage for the first-character address of each message. Window logic is annotated with `(I + offset) % N`, using character offsets 0 through 3 and a message length `N`.

The LED section contains a slide-bar module and a two-to-four decoder connected to LEDs labeled `LD15` through `LD0`. TODO: confirm the exact meaning of each display output and how the state and alert indications are encoded.

## Implementation Questions

| Question | Why it needs confirmation |
| --- | --- |
| Is the implemented datapath 16-bit or 5-bit? | The brief says 16-bit; sample and arithmetic blocks in the drawing are mainly labeled 5-bit. Sixteen LED outputs do not prove a 16-bit arithmetic datapath. |
| How do Pause and Acknowledge relate? | The brief names Pause; the HLSM instead names Acknowledge. Their equivalence is not established. |
| Which HDL language and FPGA tools were used? | Verilog appears as a possible technology in the brief, but no source or build project is supplied. |
| What is the clock and sample rate? | The drawing includes a clock-divider annotation involving 50,000,000, but that is insufficient to establish the implemented input clock, divider behavior, or sample rate. |
| Are the green `async_tick` connections clock inputs or enables? | The drawing labels them, but the final RTL and timing constraints are needed to describe clocking safely and accurately. |
| What is the threshold's unit and exact arithmetic? | A comparison to 5 is visible; signedness, scaling, overflow, and underflow behavior are unspecified. |
| What are the stored signals? | ROM access is shown, but provenance, sample count, sampling interval, and physical units are absent. |
| What was personally implemented and verified? | Individual ownership, team responsibilities, and test records were not supplied. |
