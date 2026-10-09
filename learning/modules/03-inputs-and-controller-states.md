# Module 03 — Inputs and controller states

**Starting point:** M02 complete; Cart B. USB-only low-voltage input circuit; no motor power yet.

Read the [mentor contract](../mentor.md) before teaching. Each lesson below is one complete teaching unit. Deliver its explanation and implementation together; formal checks, questions and review happen at the end. Use [the evidence template](../evidence/README.md) for the submission. Every lesson also requires a clean applicable build, no unexplained new warnings, and no regressions in previously completed behavior.

## M03-L01 — Wire and read a physical control

**Prerequisite:** M02-L03. **Product change:** A STOP button becomes a defined electrical input.

### Explanation and worked example

Voltage is measured between two points; current flows around a closed circuit. Ground is the shared reference in this circuit. A GPIO input senses voltage and should not float. A pull-up weakly connects it to 3.3 V so an open switch reads high; the pressed button connects it to ground and reads low.

A four-legged tactile button normally has two internally connected pairs. Rotating it incorrectly on a breadboard can create a permanent connection or no useful switching. Continuity mode on an unpowered circuit identifies those pairs. Breadboard power rails can be split in the middle: coloured lines are not electrical proof.

The software name should describe the meaning, such as `stop_pressed`, rather than exposing the inverted voltage level everywhere. Read a raw level at the hardware boundary, convert it once and let controller code use a boolean.

### Build it, in order

1. With USB disconnected, identify ground and PB0/A3 using hardware.md and UM1724. Map breadboard row connections and switch pairs with the meter in continuity mode.
2. Wire the STOP input to PB0 with a pull-up and normally open contact to ground. Keep all motor equipment disconnected. Verify there is no direct 3.3 V-to-ground short.
3. Configure PB0 as a GPIO input and poll it. Do not assign it EXTI0, which is reserved for HOME/PC0 later.
4. Add a raw-input adapter that returns a meaningful pressed boolean. Report changes over serial, not every polling iteration.
5. Measure input voltage released and pressed with the meter on DC voltage. Explain why current mode placed across a supply would be a short.

### Troubleshooting

An always-pressed input usually means switch orientation, incorrect breadboard row or wrong pin. A random input often lacks a pull-up or ground connection. Disconnect power before rearranging wires; measure voltage only after the continuity checks are finished.

### End-of-lesson checks and acceptance

- The physical circuit matches H1 and has defined released/pressed levels.
- Measured voltages and serial observations agree with active-low semantics.
- The controller-facing API exposes meaning, and serial reports are change-driven.

### End-of-lesson questions

1. Why is a pull-up preferable to leaving an open switch unconnected?
2. Why must continuity measurements be taken on an unpowered circuit?
3. What is the difference between a raw low level and the concept “STOP pressed”?

### Submit for whole-lesson review

Include a clear wiring photo, input-voltage measurements and a short serial trace for press/release. Label meter mode and reference points.

Commit the implementation and `learning/evidence/M03-L01.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M03-L02 — Debounce without delaying the program

**Prerequisite:** M03-L01. **Product change:** Button events are reliable while the LED and diagnostics keep running.

### Explanation and worked example

Mechanical contacts can alternate rapidly before settling. Polling faster can reveal more transitions rather than solve the problem. Debounce is a state machine that remembers a candidate level and how long it has stayed unchanged.

Track `raw`, `candidate`, `candidate_since_ms` and `stable`. When raw changes, reset the candidate timer. Adopt it only after a documented stable interval, initially 20 ms. Derive pressed/released events from changes in stable state. Use unsigned elapsed-time arithmetic from M01.

For a stop request, responsiveness matters: it is reasonable to latch a raw pressed condition immediately and debounce release/user-interface events. A short false stop costs availability; ignoring a real stop can cost a collision. This is a product decision, not a universal debounce recipe. Later HOME/FAR fault behavior will be similarly conservative.

### Build it, in order

1. Implement a pure debounce update function accepting raw level and a caller-supplied timestamp. Keep HAL_GetTick outside so tests control time.
2. Use a table of timestamped raw samples to explain one bouncing press and one bouncing release. Show why the candidate timer resets on each raw change.
3. Feed the physical input through the function in the foreground loop. Emit one event per stable transition.
4. Add a separate latched stop-request path for raw press; consuming or clearing that request must be explicit. No actual motion exists yet, so demonstrate it through state/serial output.
5. Run the LED and status reporting concurrently. Replace any debounce `HAL_Delay` with the state-based implementation.

### Troubleshooting

Repeated events while held mean the code is reporting a level as an edge. A release that never settles often has its timer overwritten every sample instead of only on candidate change. Keep test sample intervals short enough to represent the intended behavior.

### End-of-lesson checks and acceptance

- Tests cover clean press, bounce, held press, release and timestamp wrap; each yields the documented number of events.
- A raw press latches STOP promptly; debouncing does not introduce a blocking delay.
- Physical presses produce the expected events while the LED continues normally.

### End-of-lesson questions

1. Why are a level and an event different abstractions?
2. Why might assertion and release use different filtering policies?
3. How does injecting time make this code easier to test?

### Submit for whole-lesson review

Submit sample timelines with expected/actual transitions, host test output and a physical press trace. State the selected debounce interval and stop policy.

Commit the implementation and `learning/evidence/M03-L02.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M03-L03 — Define explicit state transitions

**Prerequisite:** M03-L02. **Product change:** The controller has enforceable boot, enable, stop and fault behavior.

### Explanation and worked example

A state machine makes legal transitions visible. A boolean collection can represent impossible combinations such as disabled-and-moving unless every update preserves the same rules. Use the agreed top-level states and retain separate facts, such as position validity, only when they mean something independent.

An event is an input to a transition; an action is an effect. For example, a fault event clears work, requests driver disable and invalidates position. Simply changing a displayed enum is not enough. Keep transition logic testable separately from GPIO effects, perhaps by returning a small action structure.

Reset always starts disabled and unhomed. Clearing a stop or fault is permission to accept future commands, not permission to resume an old command. This invariant will persist through UART, interrupts and FreeRTOS.

### Build it, in order

1. Write a transition table for DISABLED, IDLE, MOVING and FAULT, reserving HOMING for M07. Explain events ENABLE, DISABLE, MOVE_REQUEST, MOVE_DONE, STOP and FAULT.
2. Implement foreground-owned transition logic and a small hardware-effect adapter. Until motors exist, effects update diagnostics and a simulated busy flag.
3. Connect button STOP to the event path. Implement boot/reset and fault latching; reject a move while disabled or faulted.
4. Ensure any disable that permits physical movement invalidates position. Do not mark the simulation homed merely to simplify normal boot behavior.
5. Build table-driven tests for accepted and rejected transitions. Retain the transition table as project documentation within learning/.

### Troubleshooting

If state changes in several unrelated callbacks, locate the owner and route events to it. If the LED reports IDLE while a busy flag remains set, distinguish state from completion facts and update them together through one transition.

### End-of-lesson checks and acceptance

- Every declared event/state pair has a defined accept/reject/no-op result.
- STOP/fault clears simulated work; release never resumes it.
- Boot and disable leave position invalid, and hardware effects follow transitions rather than incidental print statements.

### End-of-lesson questions

1. Why can a state enum alone still permit inconsistent behavior?
2. What must happen besides changing the state to FAULT?
3. Why should clearing a fault not resume a queued move?

### Submit for whole-lesson review

Provide the transition table, host transition tests and a serial demonstration of enable, rejected input, STOP and reset. Clearly label simulated motion.

Commit the implementation and `learning/evidence/M03-L03.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.
