# Module 07 — Mechanics and homing

**Starting point:** M06 complete; Cart D, a matched horizontal mechanism and two NC limit switches. Firmware-only previews are allowed, but physical criteria remain pending without the mechanism.

Read the [mentor contract](../mentor.md) before teaching. Each lesson below is one complete teaching unit. Deliver its explanation and implementation together; formal checks, questions and review happen at the end. Use [the evidence template](../evidence/README.md) for the submission. Every lesson also requires a clean applicable build, no unexplained new warnings, and no regressions in previously completed behavior.

## M07-L01 — Assemble the axis and calibrate distance

**Prerequisite:** M06-L03. **Product change:** Motor pulses acquire a measured relationship to carriage travel.

### Explanation and worked example

A rail guides motion; a belt or screw transmits it. A screw's lead is distance per revolution, which differs from thread pitch on a multi-start screw. With 200 full steps/revolution and 8 mm lead, nominal scale is 25 pulses/mm. At one-eighth microstepping it becomes 200 pulses/mm, but smaller commanded increments do not automatically produce the same increase in positioning accuracy.

Backlash is lost motion when direction reverses. Compliance is deflection under load. Missed steps are motion the controller requested but the motor did not achieve. A software pulse count cannot distinguish these physical errors. Calibration establishes scale over a measured distance; repeatability testing later evaluates consistency.

Normally closed switches read low in the ordinary region and high when actuated or a wire breaks. This detects some open-circuit faults, not every possible wiring fault. A short to ground can hide an actuated switch. It is not a certified safety system.

### Build it, in order

1. Select a matched low-cost horizontal mechanism using hardware.md: NEMA17 mounting, 5 mm shaft interface, 100–300 mm useful travel and suitable end-stop mounting. Confirm the complete bill of materials before ordering; keep this decision in learning/.
2. With power removed, assemble and align the empty stage, secure its base and check smooth travel by hand. Avoid couplers imposing side load on the motor. Document mechanical dimensions and fasteners.
3. Mount both NC switches before hard stops, allowing generous low-speed stopping margin. Wire HOME to PC0 and FAR to PC1 with external pull-ups. Measure continuity and observe raw levels throughout travel with motor disabled.
4. Initially place the carriage near the middle. Allow only short, low-rate bench moves with both limit/abort inputs active. Verify logical positive direction moves away from HOME.
5. Measure a modest commanded travel using a ruler/caliper, repeat in both directions and calculate nominal versus observed pulses/mm. Keep raw measurements; do not “calibrate away” inconsistent missed steps or backlash.

### Troubleshooting

Binding in one region is a mechanical problem before it is a tuning problem. If measured distance depends strongly on direction, investigate backlash and alignment before changing scale. A permanently active switch can mean an open NC circuit or incorrect COM/NC identification.

### End-of-lesson checks and acceptance

- The mounted axis is horizontal, secured and free-moving with two functioning limit switches before hard stops.
- Positive direction, microstep setting, nominal pulses/mm and measured travel are recorded.
- Switch activation/open-wire conditions request abort; short moves stop before hard stops. Software position is still unhomed after reset.

### End-of-lesson questions

1. How do screw lead and thread pitch differ?
2. Why does microstepping not guarantee equivalent physical accuracy?
3. Which wiring failure can an NC switch detect, and which can it hide?

### Submit for whole-lesson review

Include mechanism specifications, photos, switch measurements and the travel calibration table. If using only a shaft pointer, mark this lesson's physical criteria pending.

Commit the implementation and `learning/evidence/M07-L01.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M07-L02 — Implement bounded homing

**Prerequisite:** M07-L01. **Product change:** The axis establishes a repeatable reference without unbounded travel.

### Explanation and worked example

Homing is a state machine, not “move until a switch changes.” Use SEEK, BACKOFF, SLOW_APPROACH, FINAL_RELEASE and SET_ZERO, with explicit entry conditions, maximum distance and maximum time for every moving substate. If HOME is already active, begin a bounded move away from it; never assume the switch will eventually release.

An expected HOME assertion during SEEK is an event in homing, whereas FAR or an unexpected limit during ordinary motion is a fault. Keep this distinction explicit so a generic fault handler neither blocks all homing nor ignores real limits. Stop on raw assertion promptly; use stable release/confirmation for bounce handling.

The second, slower approach reduces variability caused by speed and stopping distance. After the confirmed slow latch, perform a documented fixed bounded backoff and require HOME to be stably released. Establish zero at that released location. FINAL_RELEASE has its own distance/time bound; ordinary IDLE must not be left on an asserted switch. The usable soft-limit interval must reflect this zero convention. Timer pulses estimate position; a successful home establishes a reference, not proof that later motion cannot stall.

### Build it, in order

1. Specify homing direction, seek/approach rates, maximum travel/time, backoff distance and zero offset from the actual axis. Start conservatively and ensure all travel bounds fit the mechanism.
2. Implement a pure homing state machine driven by switch state, elapsed time and pulse-backend results. The controller remains its sole owner.
3. Handle HOME active at entry, failure to release, failure to find HOME, unexpected FAR, STOP, reset and driver disable. Every failure clears work, disables and invalidates position.
4. Integrate HOME command admission: require deliberate enable, allow the documented already-active HOME backoff case, and reject both switches active or concurrent ordinary motion; accepted and final result carry an ID. Only successful SET_ZERO makes position valid.
5. Test all branches with simulated inputs before a low-speed physical run, then home from three different safe starting positions. Do not force a carriage into a hard stop to test timeout.

### Troubleshooting

Repeated oscillation at the switch often means missing substate memory or insufficient backoff. A home routine that never exits needs independent distance/time bounds, not a longer timeout. An active switch ignored outside homing is a fault-policy bug.

### End-of-lesson checks and acceptance

- Each moving substate has tested distance and time limits, including already-active and stuck-switch cases.
- STOP/FAR/failure never produces a valid position or automatic retry.
- Physical homing succeeds from three safe starts and defines zero consistently; simulated cases are labelled separately.

### End-of-lesson questions

1. Why is a timeout alone weaker than combined time and travel bounds?
2. Why can HOME assertion be expected in one state and a fault in another?
3. What fact becomes valid at successful homing, and what uncertainties remain?

### Submit for whole-lesson review

Submit the substate transition table, simulation tests, parameter rationale and actual homing observations.

Commit the implementation and `learning/evidence/M07-L02.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M07-L03 — Enforce soft limits and absolute moves

**Prerequisite:** M07-L02. **Product change:** The homed axis accepts only reachable bounded targets.

### Explanation and worked example

Hard switches detect reaching a physical boundary. Soft limits reject a request before movement, using a valid reference and configured travel range. Neither compensates for a lost physical position. After reset, abrupt abort or manual movement while disabled, ordinary absolute moves must be rejected until homing restores validity.

Calculate target in a wider type before checking range. For relative motion, `candidate = (int64_t)position + displacement`; compare to the allowed envelope, then narrow only after validation. A queue later needs the planned endpoint rather than current instantaneous position for sequential relative requests.

Define the usable interval with margin inside the switches. With zero at the final released-home location, choose the usable interval inside both physical switch boundaries; minimum usable position can be zero or a documented positive margin. Avoid silently clamping an out-of-range command: the host should know it requested something impossible.

### Build it, in order

1. Record measured usable min/max positions in pulses, with explanation of physical margins and zero convention. Add checked mm-to-pulse conversion for the host display while keeping the wire protocol in documented pulse units.
2. Implement MOVE_ABS and update MOVE_REL admission to require valid homing and in-range target on the assembled axis. Remove or explicitly lock out unrestricted bench mode in the normal stage build.
3. Validate target/rate as a transaction before changing backend direction or state. Reject out-of-envelope requests with a useful error.
4. Treat any unexpected HOME/FAR activation as abort. Implement the separate RECOVER command from design-contract.md: only away from the single active limit, fixed low speed, measured small distance/time cap, opposite limit and STOP enforced, no ordinary queued work. Finish disabled with position invalid; require CLEAR and HOME afterward. Never let CLEAR alone move the axis.
5. Test just-inside, exact-boundary and just-outside requests in simulation, then exercise safe interior positions physically. Verify a reset invalidates position and rejects MOVE_ABS.

### Troubleshooting

Soft limits that fail only after a relative move often use stale current position or unchecked addition. If the axis can move after disabling/re-enabling without homing, position validity is being inferred from an old numeric counter.

### End-of-lesson checks and acceptance

- Both command forms reject out-of-range targets without emitting pulses; arithmetic extremes are covered.
- Normal motion requires valid homing, and reset/disable/abort removes that validity.
- Physical interior moves finish at plausible positions, and a deliberately activated limit causes a latched abort without automatic recovery motion.

### End-of-lesson questions

1. Why is a numeric position insufficient without a validity flag?
2. Why is silent clamping a poor default command policy?
3. Why should recovery from a tripped limit be a separate operation?

### Submit for whole-lesson review

Include the travel envelope, conversion rules, boundary tests and reset/limit serial evidence. Preserve the mechanical measurement uncertainty.

Commit the implementation and `learning/evidence/M07-L03.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.
