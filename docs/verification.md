# Verification and Evidence Checklist

The following items are proposed documentation and tests. They are not completed verification results.

## Existing Evidence

- High-level state-machine diagram.
- Calibration, engine, and tracking FSM diagrams.
- Datapath PDF.
- [FPGA demonstration video](../media/fpga-demo.mp4) and a [frame at 00:05](../images/fpga-demo.jpg) showing the Basys 3 seven-segment display reading “OFF.” TODO: add a walkthrough of the remaining demonstration.

No testbenches, simulation waveforms, timing reports, utilization reports, or numerical detection results were supplied with these artifacts.

## Proposed Functional Checks

| Test | Expected behavior to define and verify |
| --- | --- |
| Reset / Off | Document initial state, register values, ROM address, LEDs, and display. |
| Mode controls | Verify every allowed transition, including repeated and simultaneous button inputs. |
| Calibration sequence | Confirm the order of the four register loads, average load, completion indication, and behavior on interruption. |
| Calibration arithmetic | Use known input samples to check average calculation, intermediate widths, and truncation. |
| Magnitude engine | Check monotonic rising, monotonic falling, flat, and turning-point sequences; verify which sample is captured and when. |
| Signed samples | Check zero, positive and negative values, and the most-negative representable sample if signed input is used. |
| Threshold boundary | Verify values just below, equal to, and above the configured threshold. |
| Baseline subtraction | Check values below the baseline and near numeric limits to expose underflow/overflow behavior. |
| Alert acknowledgement | Verify alert persistence, acknowledgement, return to monitoring, and behavior when leaving the mode. |
| ROM addressing | Check waveform selection, first/last samples, wrap behavior, and out-of-range protection. |
| Displays | Check message selection, scroll/window boundaries, digit multiplexing, state LEDs, and alert indication. |
| Physical controls | Document button synchronization and any debounce behavior implemented. |

## Results to Add When Available

For each test, include the input data or stimulus, expected output, observed output, and a waveform or captured board observation. Record tool versions and the revision tested.

- TODO: final HDL and constraints.
- TODO: ROM files and sample provenance.
- TODO: HDL language, top-level module, and build instructions.
- TODO: clock definition, generated-clock/enable strategy, and sample update rate.
- TODO: functional test results and simulation waveforms.
- TODO: synthesis utilization and implementation timing reports.
- TODO: actual detection latency and detector-performance evaluation, if measured.
- TODO: a real debugging case with root cause and confirmation of the fix.
- TODO: personal contribution and any team or course attribution.

Detection accuracy, seismic validation, and timing closure require additional test evidence.
