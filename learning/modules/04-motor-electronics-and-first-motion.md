# Module 04 — Motor electronics and first motion

**Starting point:** M03 complete; Cart C and soldering/measurement capability. Read hardware.md fully. Use a secured motor with a visible shaft marker and no carriage.

Read the [mentor contract](../mentor.md) before teaching. Each lesson below is one complete teaching unit. Deliver its explanation and implementation together; formal checks, questions and review happen at the end. Use [the evidence template](../evidence/README.md) for the submission. Every lesson also requires a clean applicable build, no unexplained new warnings, and no regressions in previously completed behavior.

## M04-L01 — Assemble and current-limit the driver

**Prerequisite:** M03-L03. **Product change:** The motor circuit is electrically ready while firmware keeps it disabled.

### Explanation and worked example

A stepper has two coils. The A4988 switches coil current through H-bridges and regulates it using sense resistors; the MCU sends logical steps instead of sourcing coil current. Motor voltage rating does not mean “connect the coil to a GPIO.” The separate 12 V supply and common logic ground serve different purposes.

Current limiting depends on the actual carrier resistor value: `I_limit = VREF / (8 × R_sense)`. With R100 (0.100 Ω), a 0.25 A initial limit requires 0.200 V. With R050 it requires 0.100 V. The cheap carrier is not guaranteed to match Pololu's resistor choice. Inspect it before calculating.

The local bulk capacitor absorbs supply disturbances. Its polarity and voltage rating matter. ENABLE is active-low; its external pull-up is a hardware default that remains effective while firmware is resetting. A software assignment alone does not protect the reset interval.

### Build it, in order

1. Identify both motor coil pairs with unpowered resistance measurements; record approximate resistance and lead pairing. Read the driver pin labels and sense-resistor markings under magnification.
2. First practice a few through-hole joints on spare header/perfboard: heat pad and lead together, apply a small amount of electronics solder, let it wet both surfaces, then remove heat and hold still while it solidifies. Explain solder wetting, bridges and strain relief; use the iron stand and ventilation. Inspect with magnification and unpowered resistance checks. Assemble the driver carrier using soldered/terminal power connections, local bulk capacitor and the H1 pull resistors. Inspect solder bridges and continuity before applying either supply. Use a socket if practical; verify its row orientation.
3. Load firmware that explicitly initializes STEP low, DIR low and ENABLE high before any enabling action. Keep reset/startup behavior consistent with the external resistors.
4. With the motor disconnected and driver disabled, establish the carrier's required supply conditions and measure/adjust VREF using hardware.md. Do not guess an unidentified resistor or probe point.
5. Disconnect supplies, verify discharged VMOT, attach the motor and secure it. Reapply logic, then motor power while disabled. Measure supply polarity/voltage and confirm no spontaneous movement. No stepping is required in this lesson.

### Troubleshooting

Unexpected heating or a collapsing supply means disconnect motor power and inspect. Do not raise current as a first remedy. A motor that is stiff while disabled may have wiring or ENABLE polarity problems; a powered stepper can also have normal magnetic detent torque when disabled.

### End-of-lesson checks and acceptance

- Photos and measurements identify coil pairs, carrier orientation, sense resistors, VREF calculation/measurement and supply polarity.
- Power wiring, capacitor and reset-default resistors match H1; no motor current is routed through long signal-jumper chains.
- Firmware boot/reset keeps the driver disabled and STEP inactive; the motor remains stationary.

### End-of-lesson questions

1. Why does the MCU need common ground but not the motor's 12 V on its supply pins?
2. Why can two A4988 boards need different VREF for the same current limit?
3. Why must a coil never be unplugged while the driver is powered?

### Submit for whole-lesson review

Submit the assembly photos, measurement table and boot code. If a carrier detail is unresolved, mark power-up pending instead of inventing a value.

Commit the implementation and `learning/evidence/M04-L01.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M04-L02 — Generate the first deliberate steps

**Prerequisite:** M04-L01. **Product change:** A bounded manual test turns the shaft slowly in either direction.

### Explanation and worked example

The driver advances its internal sequence on a STEP edge; DIR selects the sequence direction. A full-step 200-step motor nominally turns 1.8° per pulse and 360° per 200 pulses. That is a commanded relationship, not a guarantee against missed steps.

At 50 pulses/s one period is 20 ms. For the first test, 10 ms high and 10 ms low gives large margin over the driver's minimum pulse widths. A delay-based loop is intentionally temporary. Direction must settle before the first STEP edge and remain stable through the required hold time after the last.

Use an explicit enable operation and a wake/setup interval. Configure DIR only while STEP is inactive and motion stopped. Shaft direction depends on coil order and viewing side; name logical positive direction and record the observation rather than assuming clockwise from a wire colour.

### Build it, in order

1. With motor power off, inspect the finite step helper before running it. Give it an explicit maximum count and reject invalid rates/counts.
2. Enable the driver only after a deliberate bench command/trigger. Allow wake/direction setup time, then issue ten full steps at 50 pulses/s using the temporary GPIO/delay method.
3. Finish STEP low. Disable only after the pulse sequence ends; record that disabling invalidates position.
4. Perform 200 pulses with a shaft marker, then 200 in the opposite direction. Keep fingers and loose leads away from the shaft.
5. Record current-limit setting, full-step selection, count/rate and observed direction. Do not debug with breakpoints while motor power is connected.

### Troubleshooting

Buzzing without rotation often means incorrect coil pairing, inadequate current or excessive load/rate. Power off before examining wiring. Back-and-forth jitter is not evidence of useful stepping. Check each hypothesis separately at low rate instead of varying every setting.

### End-of-lesson checks and acceptance

- Exactly the requested software pulse count is generated, with STEP low afterward and zero-count producing no edges in code.
- The secured motor makes a plausible one-revolution movement for 200 full-step pulses in each direction.
- Enable, direction setup, pulse-width and disable behavior are explicit. The evidence does not claim electrically measured edge accuracy yet.

### End-of-lesson questions

1. How do pulse frequency and pulse count affect different aspects of motion?
2. Why does reversing a direction bit not prove the motor moved the requested distance?
3. What can blocking delay prevent the controller from doing promptly?

### Submit for whole-lesson review

Include a short movement video or annotated observation, the bounded helper and the timing calculation. Note the limitations of visual counting.

Commit the implementation and `learning/evidence/M04-L02.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M04-L03 — Schedule a finite move cooperatively

**Prerequisite:** M04-L02. **Product change:** The motor moves while STOP, LED and diagnostics remain responsive.

### Explanation and worked example

A cooperative loop gives each service a short turn. Represent a pulse as two phases—waiting for rise and waiting for fall—instead of keeping execution inside a delay. The planner requests a move; the pulse service advances it and reports completion.

Use an explicit remaining-count field and a STEP-level field. Decrement at one documented edge, never at both. STOP must leave the output inactive and report whether the pulse estimate remains trustworthy. At this stage use the conservative abort policy: stop generation, disable, clear work and invalidate position.

A millisecond tick limits resolution and foreground latency creates jitter. At 50 pulses/s it is a useful learning backend, not the final timing design. Scheduling the next edge from “now” avoids catch-up bursts but stretches elapsed time when the loop is delayed. Recognize and record that compromise.

### Build it, in order

1. Replace the blocking step loop with `pulse_service(now_ms)`, a start request and a stop/abort operation. Use bounded input values and reject another move while busy.
2. Keep controller state ownership in the foreground. Expose completion as a fact consumed once; avoid repeatedly reporting DONE after remaining reaches zero.
3. Poll the physical STOP every loop, before ordinary diagnostics. Abort immediately on the latched request, force STEP low and disable.
4. Track emitted rises separately from requested target. Test zero, one, ten and 200 pulses with mocked GPIO callbacks on the host.
5. Run a slow physical move and press STOP. Add a bounded artificial foreground workload in a bench-only build and observe longer move duration without interpreting that as a precise jitter measurement.

### Troubleshooting

An extra final step often comes from testing the remaining count after generating the next rise. A stuck-high output means cancellation omitted the falling/idle action. Large catch-up bursts usually come from blindly replaying every missed deadline.

### End-of-lesson checks and acceptance

- Host traces show the requested number of rises, one completion and an inactive final output, including cancellation during both phases.
- STOP is serviced during movement; it disables, clears work and invalidates position without automatic restart.
- Physical motion, LED and serial remain active together. The recorded limitation identifies foreground jitter and millisecond resolution.

### End-of-lesson questions

1. Why is one completion event different from repeatedly observing an idle state?
2. What should an abort during STEP-high do electrically?
3. Why does nonblocking code alone not guarantee accurate timing?

### Submit for whole-lesson review

Provide host edge traces, a STOP demonstration and an explanation of the timing limitation that M05 will address.

Commit the implementation and `learning/evidence/M04-L03.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.
