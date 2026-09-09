# Architecture Notes

These notes describe the earlier Lab 4a handwritten design drawings. The later Lab 4b implementation deliverable records changes to that plan and supplies simulation and FPGA report evidence, summarized below and in the [verification results](verification.md). The drawings have not been checked against the final HDL.

## Changes Documented During Implementation

- The team introduced smaller calibration and alert FSMs under a main controller after the original low-level control plan became difficult to implement.
- The seismic unit was adapted to **16-bit samples**, with roughly **4,095 data points per file**. Multiple wave-pattern ROM files were consolidated into one looping wave file, removing the planned separate iteration engine. The 5-bit labels and selection logic in the drawings describe the earlier plan, not confirmed final widths or topology.
- Lab 4b uses Off, Calibration, Measuring/Tracking, and Acknowledge. It reports four-sample calibration, a persistent alert when `reading - baseline > 5`, and acknowledgement that clears the latch/value.
- The implementation reports identify Vivado 2025.2.1, top-level design `eew_top`, and device `xc7a35tcpg236-1`. The accompanying utilization report is labeled Fully Placed.

These details come from the Lab 4b deliverable, pp. 2-8 and 10. Its scenario table and waveform error counters require reconciliation; see the [simulation evidence](verification.md#simulation-evidence).

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

The LED section contains a slide-bar module and a two-to-four decoder connected to LEDs labeled `LD15` through `LD0`. Lab 4b identifies `LD3:0` as the mode indicators and `LD15:6` as the reading-minus-baseline bar, with flashing during an alert. The scenario table names `OFF`, `DONE`, `READING`, `ALERT`, and `ACKNOWLEDGED` display messages. Display timing and the final implementation still require source confirmation.

## Implementation Questions

| Question | Why it needs confirmation |
| --- | --- |
| What are the final arithmetic widths? | Lab 4b confirms 16-bit input samples, superseding the earlier 5-bit sample plan. Register and intermediate widths, signedness, and overflow handling require HDL. |
| What is the physical button mapping? | Lab 4b establishes the four mode names, but its scenario table and named testbench inputs do not establish the board-button assignments or debounce logic. |
| Which HDL language was used for the implementation? | Lab 4b names `.sv` testbenches and Vivado 2025.2.1, but the main implementation source and build project are still absent. |
| What is the clock and sample rate? | The drawing includes a clock-divider annotation involving 50,000,000, but that is insufficient to establish the implemented input clock, divider behavior, or sample rate. |
| Are the green `async_tick` connections clock inputs or enables? | The drawing labels them, but the final RTL and timing constraints are needed to describe clocking safely and accurately. |
| What is the threshold's unit and exact arithmetic? | A comparison to 5 is visible; signedness, scaling, overflow, and underflow behavior are unspecified. |
| What are the stored signals? | Lab 4b describes 16-bit samples, roughly 4,095 points per file, and one looping wave file. The actual memory contents, provenance, sampling interval, and physical units remain unavailable. |
| What was personally implemented and verified? | Lab 4b credits JP and Inu Baek and supplies team-level results; individual responsibilities and reproducible test logs are not specified. |
