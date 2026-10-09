# Module 12 — Persistence and project release

**Starting point:** M11 complete with a chosen primary architecture. Back up a working firmware image/configuration before the Flash lesson.

Read the [mentor contract](../mentor.md) before teaching. Each lesson below is one complete teaching unit. Deliver its explanation and implementation together; formal checks, questions and review happen at the end. Use [the evidence template](../evidence/README.md) for the submission. Every lesson also requires a clean applicable build, no unexplained new warnings, and no regressions in previously completed behavior.

## M12-L01 — Store settings without risking the application image

**Prerequisite:** M11-L03. **Product change:** Calibration and limits survive reset while motion state remains untrusted.

### Explanation and worked example

Flash erases in sectors and programs under alignment/voltage constraints. It cannot be treated like ordinary RAM. On this exact 512 KiB F401RE, sectors 6 and 7 are 128 KiB each starting at 0x08040000 and 0x08060000. Reserve them by limiting the application linker region to the first 256 KiB before any settings write; otherwise an erase could destroy firmware.

A raw struct dump is a fragile format because padding, endianness and future layout changes matter. Encode fields explicitly with magic, format version, payload length, sequence number and CRC. CRC detects accidental corruption; it is not authentication.

Use two slots transactionally: retain the valid old record, erase the inactive slot, write/verify the candidate, then program a final commit marker. On boot choose the newest fully valid supported record; if none exists use conservative defaults and report it. Flash operations can stall execution on this device, so SAVE requires motion stopped, driver disabled and host informed. Never persist a trusted position or an automatic-resume request.

### Build it, in order

1. Update the linker script and check the map to prove no application section occupies reserved sectors. Add a build-time/map check if practical. Refuse this layout on any different MCU/Flash size.
2. Define an explicit versioned byte encoding for calibration, limits and tested speed/acceleration settings. Implement pure encode/decode/validate/CRC functions with corrupt/truncated/unsupported-version tests.
3. Implement two-slot selection and commit logic with sequence-wrap policy. Follow actual Flash programming-width/alignment and status requirements from RM0368/HAL; keep the previous valid record until commit succeeds.
4. Add idle/disabled-only SAVE and LOAD commands. Account for the IWDG timeout during Flash erase/program using the device’s worst-case operation time; choose a suitable watchdog budget and explicit maintenance state before starting. Do not assume an ISR can keep feeding it during a Flash stall. LOAD validates settings before replacing live configuration and leaves position invalid. Prevent repeated automatic saves that would consume erase endurance.
5. Model interruption after each write stage in host tests. On hardware with motor power disconnected, verify normal persistence and a deliberately incomplete inactive record. Do not erase random addresses to simulate failure.

### Troubleshooting

Firmware disappearing after SAVE means reservation/addressing failed; stop writes and restore the known image. A CRC-valid record can still contain unreasonable settings, so semantic validation remains required. A commit marker written before the payload defeats the transaction.

### End-of-lesson checks and acceptance

- Link-map evidence proves sector isolation on the exact F401RE, with application size within the reduced region.
- Corruption/interrupted-write tests preserve an old valid record or conservative defaults; unsupported versions are handled explicitly.
- Actual reset restores validated settings but remains disabled/unhomed. SAVE during motion or while enabled is rejected.

### End-of-lesson questions

1. Why must the linker reservation precede any Flash erase?
2. Why is a CRC insufficient to decide that settings are safe to use?
3. Why must position and automatic resume be excluded from persisted state?

### Submit for whole-lesson review

Provide linker/map evidence, format specification, interruption tests and real SAVE/reset/LOAD results. Label simulated power-loss cases accurately.

Commit the implementation and `learning/evidence/M12-L01.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M12-L02 — Measure repeatability and declare an operating envelope

**Prerequisite:** M12-L01. **Product change:** The finished axis has evidence for what it can actually do.

### Explanation and worked example

Accuracy is closeness to the intended physical coordinate; repeatability is how consistently the same procedure returns to a point. Resolution is the smallest commanded or measured increment. These are different. A ruler with 1 mm divisions cannot establish 0.01 mm repeatability regardless of the number of decimal places in a spreadsheet.

Test approach direction, home repeatability, unloaded travel and conservative payload separately. Backlash can make same-direction repeatability much better than bidirectional accuracy. Temperature, belt tension and current affect results, so record the setup rather than presenting a universal motor specification.

A measured operating envelope includes travel range, rate, acceleration, load, duty cycle, switch margin and measurement resolution. It should also disclose the software block-gap limit. No encoder means lost steps can remain undetected until a physical check or rehome.

### Build it, in order

1. Choose a reachable reference mark and a measurement method available to you. State resolution/uncertainty before collecting data; borrow a dial indicator only if you need a finer claim.
2. Run at least ten home-and-return cycles using the same approach direction, then ten bidirectional approaches. Record all raw positions, not only the best outcomes.
3. Calculate min, max, spread and mean offset where meaningful. Compare results with electrical pulse counts to distinguish what each measurement establishes.
4. Repeat a conservative longer sequence at the intended rate/acceleration with an empty carriage first. Check for drift, heat symptoms, cable interference and missed-step suspicion without intentionally stalling the mechanism.
5. Freeze conservative release settings and document what would require recalibration: microstep change, mechanics change, motor/driver replacement or increased load. Rehome after any uncertainty.

### Troubleshooting

A perfect zero spread at coarse ruler resolution means “below the resolution of this method,” not perfect repeatability. A growing offset suggests lost motion/steps, not random measurement noise. Fix mechanical/electrical causes before adjusting scale.

### End-of-lesson checks and acceptance

- Raw data covers the specified repeated approaches and states instrument resolution and setup.
- Conclusions distinguish accuracy, repeatability, resolution and pulse estimate without unsupported precision.
- A conservative operating envelope and known limitations are recorded, and any drift/stall suspicion is resolved or limits are reduced.

### End-of-lesson questions

1. Can an axis be repeatable but inaccurate? Give an example from this project.
2. Why do same-direction and bidirectional tests reveal different behavior?
3. What can this open-loop controller fail to notice during an apparently successful move?

### Submit for whole-lesson review

Submit raw measurements, calculations/plots, setup photos and the final tested envelope. Physical measurements are required for project completion.

Commit the implementation and `learning/evidence/M12-L02.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M12-L03 — Prepare a reproducible portfolio release

**Prerequisite:** M12-L02. **Product change:** Another developer can build, understand and assess the completed controller.

### Explanation and worked example

A useful embedded portfolio explains decisions and evidence, not just a video of motion. Show the chain from commands to validated targets, planning, hardware pulse generation and physical observation. State what remains approximate or unmeasured.

Reproducibility requires source, configuration, tool versions, build commands, pin map and a clean setup path. A binary without its matching configuration is weak evidence. Record the release commit and identify the exact build used for demonstrations. Do not put credentials, machine-specific paths or unlicensed redistributed tool packages into the repository.

Instructional material stays under learning/. Application code, tests, host client and build files now have the layout established by actual needs. A minimal root README may link to learning/README.md; place the detailed build/operator instructions within learning/ to honor the agreed structure.

### Build it, in order

1. Build from a fresh checkout or clean copy at the intended release SHA using the recorded environment. Run host regression tests and inspect warnings/map/Flash reservation.
2. Execute a concise hardware acceptance script: disabled boot, home, interior absolute move, short queued sequence, controlled STOP, limit/lease fault and recovery, settings save/reset. Use low-load conservative settings.
3. Write learning/release-notes.md with architecture, wiring revision, protocol, tool versions, measured envelope, known limitations and unresolved items. Include a diagram only where it clarifies ownership or event flow.
4. Record a short demo with matching serial log and commit identifier. Explain one C memory issue, one timer issue and one fault-recovery issue you solved.
5. Request the final whole-project review. Tag a version only after required fixes are resolved; publishing/pushing remains your explicit action. Choose an optional next extension only after recording completion status.

### Troubleshooting

A clean checkout failure often reveals an untracked generated input or an absolute local path. A demo made from a different binary cannot validate the tagged source. An impressive feature list should not obscure pending physical evidence.

### End-of-lesson checks and acceptance

- A clean source copy builds and passes the applicable regression suite with recorded commands.
- The final hardware script passes with traceable evidence; missing physical claims remain Verification pending.
- Release notes disclose limitations, match the tested SHA/configuration and support a new chat or developer continuing without prior conversation memory.

### End-of-lesson questions

1. Which pieces of evidence best demonstrate engineering judgment in this project?
2. Which operating claims remain conditional on the measured setup?
3. What would you redesign first for continuous high-speed motion or a heavier machine?

### Submit for whole-lesson review

Submit the release candidate SHA, clean-build/test results, hardware acceptance log and release notes. The reviewer assesses the complete relevant project and all unresolved course gates before marking completion.

Commit the implementation and `learning/evidence/M12-L03.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.
