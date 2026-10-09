# Module 08 — Acceleration and queued motion

**Starting point:** M07 complete; empty low-load stage. Keep conservative tested rates. Read the bounded-block timing limitation in design-contract.md.

Read the [mentor contract](../mentor.md) before teaching. Each lesson below is one complete teaching unit. Deliver its explanation and implementation together; formal checks, questions and review happen at the end. Use [the evidence template](../evidence/README.md) for the submission. Every lesson also requires a clean applicable build, no unexplained new warnings, and no regressions in previously completed behavior.

## M08-L01 — Plan acceleration and controlled stopping

**Prerequisite:** M07-L03. **Product change:** Moves ramp their pulse rate instead of abruptly demanding full speed.

### Explanation and worked example

A stepper has limited torque and can miss steps when started too fast. A trapezoidal profile accelerates, cruises and decelerates. In consistent pulse units, stopping distance from speed v with deceleration a is approximately `v²/(2a)`. For 400 pulses/s and 800 pulses/s², that is 100 pulses before allowance for discrete scheduling and latency.

This equation is a planning approximation, not a guarantee of physical stopping under every load. Integer arithmetic needs wide intermediates for v² and explicit rounding; stopping distance should be rounded conservatively upward. Reject zero acceleration.

The course backend emits constant-rate blocks, so its profile is a staircase approximation. Smaller blocks improve rate-update granularity but increase scheduling overhead and interblock gaps. Bound speed changes and measure gaps; do not claim a smooth continuous ramp from a list of desired rates alone. Controlled STOP decelerates within a planned bound. A limit/fault ABORT remains immediate disable with position invalidation.

### Build it, in order

1. Implement a pure planner producing the next block count/rate from remaining distance, present rate and acceleration limits. Start with small blocks and a conservative nonzero starting rate appropriate to the tested motor.
2. Explain each rate update using elapsed block duration or a distance-based v² relation. Include the scheduling-gap approximation in the design notes and choose a conservative policy for delayed service.
3. Begin deceleration early enough to fit remaining distance plus rounding/latency margin. Keep emitted pulse count exactly equal to the requested displacement.
4. Add controlled STOP: cancel future queued requests, calculate a bounded stopping trajectory and preserve position only if execution completes normally with no suspected stall. If a safe stop cannot fit the envelope, reject motion earlier or abort and invalidate; do not cross a limit to satisfy the profile.
5. Test planner outputs on the host before a low-speed physical ramp. Log block rate/count/timing and compare requested versus observed movement.

### Troubleshooting

Stalling at start suggests excessive initial rate/acceleration or mechanical load. Oscillating rate near the endpoint usually indicates inconsistent rounding or phase selection. Gaps large enough to dominate block duration require lower rate or scheduling changes, not prettier plots.

### End-of-lesson checks and acceptance

- Host traces have bounded rates/acceleration changes, exact total pulse count and conservative deceleration decisions.
- Controlled STOP clears future work and stops within the documented software bound; ABORT remains distinct.
- A physical ramp completes at conservative settings, and block gaps/measurement limits are reported rather than hidden.

### End-of-lesson questions

1. Why does doubling speed quadruple ideal stopping distance?
2. What tradeoff changes when pulse blocks become smaller?
3. When can a controlled stop preserve position validity, unlike an abrupt abort?

### Submit for whole-lesson review

Submit planner tests, a table or plot of block rates, the stopping-margin calculation and actual low-speed observations.

Commit the implementation and `learning/evidence/M08-L01.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M08-L02 — Handle short moves and reversals

**Prerequisite:** M08-L01. **Product change:** The planner works at the boundaries where simple profiles often fail.

### Explanation and worked example

A short move may have no cruise segment. A triangular profile accelerates to a lower peak and then decelerates. For a symmetric ideal rest-to-rest move, `v_peak = sqrt(a × distance)` before accounting for nonzero start rate and discrete pulses. Use that equation to reason about shape, not as an unchecked floating-point shortcut in every interrupt.

Zero distance is a valid no-motion completion. One pulse still needs a legal electrical waveform. Odd distances may assign one extra pulse to one phase; the total must remain exact. Reversal must finish deceleration, hold STEP inactive, change DIR with setup/hold margin and then accelerate in the opposite direction.

For the core course, stop fully between separately queued moves. This makes direction transitions understandable and keeps the limitations honest. Continuous look-ahead blending is an optional future project, not an implied property of this queue.

### Build it, in order

1. Extend the planner's phase logic for zero, one, two and short triangular moves. Define how integer rounding allocates pulses between phases.
2. Test long moves where cruise exists and the exact boundary where it disappears. Include maximum allowed displacement and both signs without taking an overflowing absolute value.
3. Implement explicit end-of-move inactive/direction-hold handling before a reverse start. Keep DIR unchanged during a block.
4. Run a repeated small forward/back sequence away from limits at low rates. Compare pulse estimates with an external carriage mark; distinguish backlash from a software count error.
5. Document the core policy of full stop between queued moves and the measured speed/acceleration envelope used so far.

### Troubleshooting

A short move that never completes often waits for a cruise phase that does not exist. Direction glitches often come from configuring the next command before the prior block has finished. Physical reversal error can exist even with perfect electrical pulse totals.

### End-of-lesson checks and acceptance

- Tests cover 0,1,2,odd/even short moves, phase-boundary distances, negative displacement and large valid values.
- All profiles emit exact pulse totals without zero-length hardware blocks, illegal rate or division by zero.
- DIR changes only after the prior pulse sequence ends and the required hold time; physical reversal observations state backlash uncertainty.

### End-of-lesson questions

1. Why does a short move lack a cruise phase?
2. How can an odd pulse count complicate symmetric planning?
3. Why can a perfect forward/back pulse total still leave the carriage displaced?

### Submit for whole-lesson review

Provide boundary-case planner traces and direction timing reasoning. Include a physical reversal measurement with its resolution.

Commit the implementation and `learning/evidence/M08-L02.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M08-L03 — Queue commands with clear cancellation

**Prerequisite:** M08-L02. **Product change:** The controller accepts a small sequence without losing STOP priority.

### Explanation and worked example

A fixed-capacity queue makes resource limits explicit. With four entries, the fifth request must be rejected or back-pressured; silently replacing an earlier move violates the protocol. Copy command values into queue storage rather than retaining pointers into the parser's reused buffer.

Relative queued moves are relative to the planned endpoint after preceding accepted moves, not to the current carriage position when the command arrives. Validate that whole chain transactionally. Resetting the queue also resets its planned endpoint consistently.

STOP and ABORT must not need an empty move slot. A separate bounded flag/event path lets them take priority. Cancelling an accepted command requires an explicit response, such as `CANCELLED id=…`, so the host does not wait forever for DONE. Only executed completion is DONE.

### Build it, in order

1. Implement a fixed four-entry command queue with pure enqueue/dequeue/clear operations and no heap. Choose whether capacity includes the active move and document it; prefer four pending entries plus one active record.
2. Validate each new target against the current planned endpoint and soft limits before enqueueing. Preserve queue/state unchanged on rejection.
3. Start the next command only after the previous finishes and direction/timing conditions permit. Use the full-stop policy from L02.
4. Define STOP, ABORT, timeout and reset cancellation semantics. Give every accepted queued ID a terminal DONE, CANCELLED or FAULT result when communication remains available.
5. Update the Swift client to submit and display a short sequence without treating acceptance as completion. Exercise a full queue and STOP through its priority path.

### Troubleshooting

Wrong relative endpoints usually mean mixing actual and planned position. Corrupted queued commands suggest storing pointers to a temporary parser object. A queue that prevents STOP is an architecture error, not a capacity-tuning problem.

### End-of-lesson checks and acceptance

- FIFO order, wraparound, full rejection and clear behavior pass host tests.
- Relative target chains are correct and rejected requests do not alter the planned endpoint.
- STOP/ABORT with a full queue prevents subsequent starts and reports cancellations; reset never resumes stored work.

### End-of-lesson questions

1. Why must a queue own its command data?
2. Which position should validate the second queued relative command?
3. Why should STOP not share ordinary queue admission limits?

### Submit for whole-lesson review

Submit queue tests, an accepted/executed/cancelled ID trace and a physical short-sequence demonstration.

Commit the implementation and `learning/evidence/M08-L03.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.
