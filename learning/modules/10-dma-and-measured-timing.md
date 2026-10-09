# Module 10 — DMA and measured timing

**Starting point:** M09 complete. Preserve a working baseline commit before peripheral changes. Motor disconnected for intrusive timing experiments.

Read the [mentor contract](../mentor.md) before teaching. Each lesson below is one complete teaching unit. Deliver its explanation and implementation together; formal checks, questions and review happen at the end. Use [the evidence template](../evidence/README.md) for the submission. Every lesson also requires a clean applicable build, no unexplained new warnings, and no regressions in previously completed behavior.

## M10-L01 — Receive serial data with DMA

**Prerequisite:** M09-L03. **Product change:** Higher serial traffic causes less per-byte CPU interrupt work.

### Explanation and worked example

DMA transfers peripheral data to memory without an interrupt for every byte. It does not parse commands or make buffer ownership disappear. Circular reception has a moving producer position; the consumer must process new spans, including a wrap split, without reading bytes still being written or processing them twice.

IDLE detection indicates a gap on the wire, not a complete application command. A newline may arrive before or after it. Half-transfer, transfer-complete and IDLE callbacks can describe overlapping progress. Consult the installed HAL receive-to-idle implementation instead of assuming callback size always means “new bytes.”

A circular DMA buffer can wrap more than once before foreground service. A modulo write index alone cannot tell that data was lost. Use half/full events with accounted progress or a rigorously bounded service interval, and treat detected overruns as a framing fault. The F401 has no data cache, but memory ordering and producer/consumer ownership still matter.

### Build it, in order

1. Measure baseline serial-service interrupt count and CPU work under a repeatable client traffic pattern. Preserve that baseline.
2. Configure a valid USART2 RX DMA stream/channel for STM32F401, using the reference manual and actual generated configuration. Check conflicts rather than copying another MCU's stream number.
3. Implement one byte-delivery adapter for DMA spans and reuse the existing tested line framer/parser unchanged. Document callback/progress semantics and wrap handling.
4. Make overrun and UART errors discard the damaged frame through a delimiter. Do not silently convert lost input into a command. Keep critical STOP/fault service independent of command throughput.
5. Stress fragmented lines, continuous traffic across many wraps, exact buffer boundaries and deliberately delayed consumer service. Keep TX simple unless a measured bottleneck requires a separate bounded TX queue.

### Troubleshooting

Duplicated characters often indicate overlapping IDLE/half/full spans being processed twice. Missing text only under load suggests a full-wrap ambiguity or a delayed consumer. Increasing the buffer may improve tolerance but does not define an overflow policy.

### End-of-lesson checks and acceptance

- Existing framing/parser tests still pass; actual traffic crosses many DMA wraps without duplication or silent truncation.
- Delayed-consumer overflow is detected or prevented by a measured enforceable bound, with recovery behavior documented.
- Interrupt work is reduced compared with the recorded baseline, without degraded stop response.

### End-of-lesson questions

1. Why is an IDLE event not a message delimiter?
2. What information is lost if only a modulo producer index is observed?
3. Which parts of the existing receive pipeline should remain unchanged?

### Submit for whole-lesson review

Provide the DMA ownership/progress design, stress logs and before/after measurements. State buffer size and maximum allowed service delay.

Commit the implementation and `learning/evidence/M10-L01.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M10-L02 — Measure the timing budget

**Prerequisite:** M10-L01. **Product change:** Timing claims have explicit measurements and limits.

### Explanation and worked example

Average CPU usage does not establish worst-case latency. The controller needs bounds for STOP sampling, completion handling, next-block start, lease checks and serial service. Interrupt masking and long high-priority work can delay all of them even when average load is low.

Use a free-running counter or Cortex-M DWT cycle counter for short execution-duration measurements after verifying availability/enabling on this target. At nominal 84 MHz, 8400 cycles corresponds to 100 µs. Unsigned subtraction handles one wrap; the 32-bit counter wraps in roughly 51 seconds at 84 MHz, so measure shorter intervals. Instrumentation itself costs time.

A GPIO trace observed by a suitable logic analyzer can reveal electrical timing independently of logs. The budget course can use on-target cycle timestamps and TIM2 edge counting, but must label the limitations: CPU timestamps do not directly prove every pin edge or absolute oscillator accuracy. If an acceptance claim needs finer external measurement, borrow a compatible analyzer rather than inventing precision.

### Build it, in order

1. Define a timing table with target, observed worst case and measurement method for each critical service. Derive a STOP-to-disable target appropriate to the tested speed and margin; start with a conservative 10 ms software service goal and verify it.
2. Instrument start/end of relevant services and block completion/restart without printing in the measured path. Store a bounded sample or maximum, then report later.
3. Run idle, continuous serial, queued motion and deliberate bounded-background-load scenarios. Measure interblock gaps and their contribution to actual average speed.
4. Calculate stopping-margin implications using measured response delay plus planned deceleration. Reduce speed/block size or remove blocking work if margins are inadequate.
5. Repeat the original electrical edge-count and physical motion checks after instrumentation. Remove unnecessary instrumentation from the normal release while preserving a diagnostic build option.

### Troubleshooting

Serial printf inside the interval changes what you measure. A constant suspiciously tiny duration can indicate an optimized-away operation or disabled cycle counter. A measurement sharing the MCU clock is useful for relative latency but not independent frequency calibration.

### End-of-lesson checks and acceptance

- The timing table includes measured worst cases under named loads, units and clock assumptions.
- Interblock gaps are quantified or explicitly pending external observation; actual average-speed error is assessed.
- STOP/limit service fits the chosen physical margin, or the operating envelope is reduced and retested. Unsupported precision claims are removed.

### End-of-lesson questions

1. Why can low average CPU use coexist with unacceptable latency?
2. How can instrumentation distort a measurement?
3. How do measured block gaps change the interpretation of requested speed?

### Submit for whole-lesson review

Submit raw timing samples or maxima, workload descriptions, calculations and the resulting tested operating envelope.

Commit the implementation and `learning/evidence/M10-L02.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M10-L03 — Read registers to improve one measured path

**Prerequisite:** M10-L02. **Product change:** One justified low-level change teaches the peripheral model without rewriting the application.

### Explanation and worked example

HAL is a useful interface, not magic. Generated code eventually writes memory-mapped registers. A volatile register declaration tells the compiler accesses have observable effects; it does not make an unsafe read-modify-write harmless.

GPIO BSRR sets/resets selected output bits without reading the output register first. Many peripheral status flags have special clearing semantics such as write-zero-to-clear; applying a generic `reg &= ~mask` can lose a concurrent event or clear unintended flags. Read the exact RM0368 register description before changing code.

Optimize a path only when measurement identifies a reason: deterministic STEP shutdown, bounded trace toggling or precise timer setup order are suitable examples. Keep the change behind the existing backend so controller tests still apply. Assembly inspection can explain instruction count, but it cannot replace target measurement.

### Build it, in order

1. Select one measured path from L02. State the concrete latency/order problem and expected improvement before editing.
2. Trace the installed HAL implementation into the relevant register operations. Identify register width, reserved bits, flag-clearing semantics and concurrency concerns.
3. Implement the narrow direct-register or LL alternative with named device-header masks and a short explanation linked to the manual section. Do not scatter numeric addresses through application logic.
4. Inspect generated assembly for the selected function and compare target timing with the same workload/build optimization as baseline.
5. Repeat related pulse-count, stop and parser/controller regressions. Keep the simpler version if the change has no material benefit or increases uncertainty.

### Troubleshooting

A “faster” result from a different optimization level is not a controlled comparison. A register write that works once but misses later interrupts often mishandles flags or preloads. Restore the known baseline while investigating.

### End-of-lesson checks and acceptance

- The change addresses a measured requirement and its register semantics are documented correctly.
- Before/after comparison uses equivalent conditions and reports actual benefit or an intentional revert.
- Relevant electrical and logic behavior remains correct, with no broad HAL rewrite.

### End-of-lesson questions

1. Why can a read-modify-write be unsafe for a peripheral flag register?
2. What does BSRR avoid compared with modifying ODR through a read-modify-write?
3. When is keeping HAL the better engineering decision?

### Submit for whole-lesson review

Submit the narrow diff, manual references, assembly observation and controlled measurement. A justified revert can complete the lesson.

Commit the implementation and `learning/evidence/M10-L03.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.
