# Module 05 — Timers and bounded pulse blocks

**Starting point:** M04 complete. Keep motor power off during timing/debug setup; add the PA8→PA0 signal loopback from H1.

Read the [mentor contract](../mentor.md) before teaching. Each lesson below is one complete teaching unit. Deliver its explanation and implementation together; formal checks, questions and review happen at the end. Use [the evidence template](../evidence/README.md) for the submission. Every lesson also requires a clean applicable build, no unexplained new warnings, and no regressions in previously completed behavior.

## M05-L01 — Calculate and observe hardware PWM

**Prerequisite:** M04-L03. **Product change:** TIM1 produces predictable STEP periods independently of foreground delays.

### Explanation and worked example

A timer counts clock ticks. For an upcounter, `f_counter = f_timer / (PSC + 1)` and `f_period = f_counter / (ARR + 1)`. On this STM32, APB prescalers also affect the timer clock: when an APB prescaler is greater than one, the timer typically receives twice that peripheral-bus clock. Read the actual clock tree rather than using CPU frequency by habit.

Upgrade from HSI 16 MHz to the documented nominal 84 MHz clock tree in environment.md. With a nominal 84 MHz timer clock, PSC=83 gives a 1 MHz count and ARR=9999 gives 100 pulses/s. PWM mode 2 with CCR=9990 stays inactive for 9990 counts, then active for ten counts. That late pulse will be useful when stopping at overflow. These are nominal times derived from the oscillator, not calibrated measurements.

PA8 must switch from ordinary GPIO to TIM1_CH1 AF1. TIM1 also has a main-output-enable gate; correct counter values alone do not guarantee a pin waveform.

### Build it, in order

1. Disconnect motor power. Regenerate the 84 MHz clock tree with correct bus dividers, voltage scale and Flash latency; verify SystemCoreClock and serial baud behavior. Record the configuration change.
2. Configure TIM1_CH1 on PA8, edge-aligned upcounting and PWM mode 2. Calculate PSC, ARR and CCR for 50, 100 and 200 pulses/s with a conservative 10 µs high pulse.
3. Start continuous PWM only for this isolated bench test. Explain CEN, CC1E, MOE and alternate-function routing by tracing the actual HAL setup.
4. Wire PA8 to PA0. Configure TIM2 channel 1 external-clock counting of rising edges, with a filter/polarity that admits the intended waveform. TIM2 counts physical signal edges, not calls to a software function.
5. Compare observed edge counts over a nominal timed window against the expected range. Stop PWM and establish an inactive output before reconnecting motor power in a later lesson.

### Troubleshooting

A running CNT with a flat output suggests pin AF or output gating. A count factor-of-two error suggests edge polarity or clock math. An off-by-one edge at window boundaries is not necessarily a bad timer; specify start/stop ordering and tolerance.

### End-of-lesson checks and acceptance

- Clock and timer calculations are documented for all three rates and match register/configuration values.
- TIM2 sees the real loopback edges; continuous PWM can be stopped with STEP inactive.
- Evidence distinguishes nominal clock-derived rate, electrical edge count and physical motor motion. Continuous PWM is not exposed as a normal motion command.

### End-of-lesson questions

1. Why do PSC and ARR formulas contain “+1”?
2. Why might an APB timer clock differ from its bus clock?
3. Why does loopback counting not calibrate absolute time independently?

### Submit for whole-lesson review

Include clock tree, timer registers/calculations, loopback wiring and observed counts. Keep the motor disconnected during these tests.

Commit the implementation and `learning/evidence/M05-L01.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M05-L02 — Bound pulse count in hardware

**Prerequisite:** M05-L01. **Product change:** A hardware block emits 1–256 pulses and stops without waiting for software.

### Explanation and worked example

If a foreground task or ISR stops a free-running timer after counting N callbacks, interrupt latency can permit extra pulses. Instead, use TIM1 one-pulse mode with its repetition counter. In this configuration, an update ends a block after `RCR+1` periods; the F401 repetition counter supports 1–256 pulses per block.

PWM mode 2 is inactive at CNT=0, active near the end of the period and returns inactive at overflow. With the correct polarity/idle configuration, the final update stops the counter at that inactive phase. This must be verified electrically, especially for N=1. Software update generation (`UG`) latches preloads but can also set an update flag: clear setup flags before enabling completion handling.

Treat block setup as an ordered transaction: counter/output disabled; direction stable; PSC/ARR/CCR/RCR written; preloads latched; flags cleared; inactive output established; then start. A generic HAL PWM start call may start the counter earlier than your sequence intends. Inspect the installed HAL source and choose the exact sequence deliberately.

### Build it, in order

1. Implement a small pulse-backend interface: start bounded block, query busy, abort and consume completion. Reject zero as a hardware block; handle a zero-length move above this layer without emitting an edge.
2. Configure upcounting, PWM2, one-pulse operation and RCR=N−1. Verify TIM1 output-idle/off-state controls. Keep setup changes atomic with respect to the completion interrupt.
3. In the update ISR acknowledge the actual flag and publish only a completion fact. No formatting, delay, allocation or next-move planning in the ISR.
4. Use TIM2 loopback to test N=1,2,255,256 repeatedly. Add a software test for a 257-pulse move split as 256+1; the controller coordinates blocks in L03.
5. Deliberately delay foreground consumption after starting a block, with motor disconnected. Confirm the output has already stopped at the requested edge count. Observe STEP's final level.

### Troubleshooting

Immediate false completion often comes from UG's uncleared UIF. N−1/N+1 errors can come from preload sequencing or the repetition counter's first load. A final high level requires inspecting PWM polarity/idle behavior rather than merely forcing CEN low.

### End-of-lesson checks and acceptance

- Electrical counts for 1,2,255,256 match exactly across repeated trials; the output is inactive afterward.
- Foreground delay does not add pulses; setup does not generate a spurious rising edge.
- Zero, busy-start rejection and abort are defined, and the ISR contains bounded flag handling only. Record any abort count uncertainty honestly.

### End-of-lesson questions

1. Why is hardware-bounded output stronger than stopping after N callbacks?
2. Why can a software-generated update look like a real completion?
3. What part of startup could emit an unintended first edge?

### Submit for whole-lesson review

Provide setup sequence, register values, ISR code and a requested/observed count table. A software counter alone does not satisfy the electrical-count criterion.

Commit the implementation and `learning/evidence/M05-L02.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M05-L03 — Integrate the asynchronous backend

**Prerequisite:** M05-L02. **Product change:** Finite moves use timer blocks while the controller remains responsive.

### Explanation and worked example

The backend owns timer registers; the foreground controller owns move progress. An ISR communicates a small event across that boundary. `volatile` can prevent certain compiler optimizations for observed memory, but it does not make read-modify-write sequences atomic or establish a general synchronization protocol.

For a single-core bare-metal foreground/ISR handoff, a short critical section can copy and clear an event consistently. Preserve and restore the previous interrupt-mask state; blindly enabling interrupts can break an outer critical section. Keep the section tiny and never perform serial I/O inside it.

A 600-pulse move can be split 256+256+88. The hardware bounds each block, but a software gap exists before the next starts. That means this backend does not promise continuous speed across long moves. We will measure and manage this limitation rather than silently calling the result precision streaming.

### Build it, in order

1. Replace the GPIO pulse backend with TIM1 blocks behind the same conceptual interface. The controller chooses the next count without exceeding the remaining displacement.
2. Define the completion handoff, including the race when an event arrives as the foreground reads/clears it. Explain why the chosen critical section is sufficient for these particular fields.
3. Update position from completed/emitted pulses, never from the requested endpoint at acceptance time. On abort, invalidate position even if a partial count is available.
4. Make STOP/abort shut down the timer and output, disable the driver and prevent a stale completion event from starting another block. A generation/operation token or carefully cleared state can reject stale events.
5. Test motor-disconnected edge totals for 0,1,256,257 and 600, then run low-speed shaft moves. Record interblock gaps if measurable; otherwise mark their measurement pending until M10.

### Troubleshooting

A move restarting after STOP often means an old completion event scheduled the next block. A lost completion can result from an unsafe read-then-clear sequence. Do not solve either by adding arbitrary delays.

### End-of-lesson checks and acceptance

- Edge totals are correct across block boundaries and completion is reported once per move.
- STOP at the start, middle and block boundary cannot schedule more work or preserve a false valid position.
- Shared fields have a documented synchronization rule; hardware and controller ownership are separate. Interblock gaps are explicitly acknowledged.

### End-of-lesson questions

1. What race can occur when foreground code reads and then clears an event flag?
2. Why should an old completion not be allowed to affect a new move?
3. Which guarantee does a 256-pulse hardware block provide, and which does it not?

### Submit for whole-lesson review

Submit count/STOP evidence, the ownership and handoff explanation, and a diagram or sequence table for a multi-block move.

Commit the implementation and `learning/evidence/M05-L03.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.
