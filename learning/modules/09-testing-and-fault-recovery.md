# Module 09 — Testing and fault recovery

**Starting point:** M08 complete. Fault injection begins in host tests or with motor power disconnected; never create a mechanical crash to test a software branch.

Read the [mentor contract](../mentor.md) before teaching. Each lesson below is one complete teaching unit. Deliver its explanation and implementation together; formal checks, questions and review happen at the end. Use [the evidence template](../evidence/README.md) for the submission. Every lesson also requires a clean applicable build, no unexplained new warnings, and no regressions in previously completed behavior.

## M09-L01 — Build a host regression suite

**Prerequisite:** M08-L03. **Product change:** Pure controller logic can be checked quickly and repeatedly without hardware.

### Explanation and worked example

A hardware abstraction is useful when it lets policy be tested without pretending to reproduce electricity. Replace time, pulse start/completion, switch reads and diagnostic output with small deterministic fakes. The fake records requested effects; it does not prove the real pin waveform or motor torque.

Test invariants in addition to examples: no move while unhomed, no output after latched abort, no invalid target admission and one terminal result per accepted command. A model sequence can generate many event combinations, but a failing case must be reproducible from a recorded seed.

Apple Clang's AddressSanitizer and UndefinedBehaviorSanitizer can find memory/undefined-behavior bugs in host builds. Host success does not prove MCU interrupt correctness, target timing or identical type/layout assumptions. Compile the same pure production C where feasible; avoid testing a separate simplified rewrite.

### Build it, in order

1. Gather existing unit tests into one documented host command or build target. Keep tests alongside actual code in a location decided now; put teaching instructions in learning/.
2. Separate HAL-dependent adapters from parser, planner, debounce, queue and controller policy only as needed for real tests. Do not refactor every file for an imagined architecture.
3. Add deterministic event-sequence tests: completion after STOP, switch during homing, queue rejection, numeric boundaries and reset from every top-level state.
4. Run host tests with strict warnings and address/undefined sanitizers. Fix findings at their cause rather than disabling checks globally.
5. Define an optional GitHub Actions job for host tests if repository access allows it. Local execution is required; a CI badge is not a substitute. Do not put credentials in files.

### Troubleshooting

Mocks that mirror implementation details can pass despite a shared design mistake. Assert external invariants and effects instead. A sanitizer failure only on the host may expose a real lifetime bug, but investigate architecture-dependent assumptions before applying target-specific workarounds.

### End-of-lesson checks and acceptance

- One command runs production-logic tests, and sanitizer-enabled execution has no unexplained findings.
- Each major invariant has a meaningful test including at least one cancellation/race-like event ordering.
- A test matrix distinguishes host logic, target build, electrical measurement and mechanical validation.

### End-of-lesson questions

1. What can a fake pulse backend prove, and what can it not prove?
2. Why should tests compile production logic rather than a rewritten model alone?
3. How do invariant tests differ from checking a few expected outputs?

### Submit for whole-lesson review

Include test command/output, coverage-by-behavior matrix and any discovered defect with its regression case. Coverage percentage is optional.

Commit the implementation and `learning/evidence/M09-L01.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M09-L02 — Handle lost hosts and invalid recovery requests

**Prerequisite:** M09-L01. **Product change:** Remote motion has an explicit communication lease and a controlled recovery path.

### Explanation and worked example

Serial connectivity is not reliable presence detection. USB can remain powered while a client crashes. Introduce a host lease: a PING refreshes a deadline while remote motion is active. A provisional 500 ms heartbeat and 2000 ms lease are reasonable starting policy values, but document and measure them. Status reads alone should not silently refresh authority unless the protocol explicitly says so.

Lease expiry requests ABORT, clears queued work and invalidates position. It does not merely stop accepting new commands while old ones continue. PING cannot revive a faulted move. A new connection must inspect state and deliberately rehome as needed.

CLEAR only removes a resolved latched fault. It does not establish position or move away from a switch. Recovery from an active boundary requires the separately constrained away-from-limit mode from M07, with the opposite boundary and STOP still enforced.

### Build it, in order

1. Add lease ownership/session semantics to the protocol and client. Start the lease when remote motion is accepted; keep it active through the queued sequence. Define when it becomes unnecessary at idle.
2. Implement wrap-safe expiry checks using injected time. Keep lease evaluation independent of ordinary logging throughput.
3. Define fault codes for host timeout, unexpected limit, homing failure and internal inconsistency. Retain the first cause plus useful diagnostic context without overwriting it with secondary noise.
4. Implement CLEAR validation and deliberate reconnect flow. Require a new HOME before absolute motion after abort. Reject stale IDs from the wrong boot/session where feasible.
5. Simulate missed heartbeats, late PING, CLEAR with persistent cause and reconnect. Physically test a stopped client only at low speed with generous free travel and the independent low-voltage disconnect reachable.

### Troubleshooting

A lease that expires during ordinary logging indicates an unbounded foreground path or overly tight policy. Extending the timeout can hide the problem; measure service latency first. A late PING that resumes motion means expiry was not latched as a real fault.

### End-of-lesson checks and acceptance

- Boundary-time tests and a real stopped-client trial show bounded abort with no queued restart.
- Late heartbeats and CLEAR do not restore position or resume cancelled motion.
- Fault reports distinguish communication loss from physical limit/homing errors, and the client displays recovery requirements.

### End-of-lesson questions

1. Why is USB power or an open serial port not proof of an active host?
2. Why should a late PING not undo a timeout fault?
3. What state must CLEAR leave unchanged or invalid?

### Submit for whole-lesson review

Provide lease timing policy, tests around expiry, observed stop latency and a reconnect transcript. Label any physical trial omitted for lack of travel.

Commit the implementation and `learning/evidence/M09-L02.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M09-L03 — Add an independent watchdog and boot diagnostics

**Prerequisite:** M09-L02. **Product change:** A stuck foreground execution path eventually resets to disabled.

### Explanation and worked example

The independent watchdog runs from a separate low-speed oscillator and resets the MCU if not refreshed. It detects some failures that a software lease cannot, because a stalled foreground loop cannot check its own deadline. Its oscillator has tolerance, so timeout values are ranges, not precise calibrated periods.

Refreshing from every interrupt can keep a broken application alive. Refresh only when the required foreground services have demonstrated progress within their deadlines. Do not equate a toggling LED with a healthy motion controller.

A watchdog reset is recovery to a known disabled state, not safe continuation. External ENABLE pull-up and timer reset behavior matter during the reset interval. Capture reset-cause flags early, report them and then clear them deliberately. A HardFault handler may capture minimal diagnostics, but must not try normal printf, allocation or motion continuation.

### Build it, in order

1. Read the F401 IWDG clock/prescaler/reload behavior and choose a conservative timeout from the oscillator range and measured service needs. Record the calculation and debug-freeze policy.
2. Add foreground health checks for input service, controller progression and communication service. Feed only after a complete healthy cycle, with no long critical section.
3. Capture boot reset causes and emit a bounded diagnostic after initialization. Ensure boot state is disabled, queue empty and position invalid regardless of cause.
4. With motor power disconnected, inject a deliberate foreground hang after a visible marker. Observe reset and watchdog cause. Remove or compile-gate the injection from the normal build.
5. Test a deliberate peripheral/service failure that prevents healthy progress. Retain the first fault and disable output while awaiting reset when execution still permits it.

### Troubleshooting

A board that resets immediately may have a timeout shorter than startup or an enabled watchdog surviving a debug session. A hang that never resets may be refreshed in an ISR or frozen by debug settings. Diagnose with motor power disconnected.

### End-of-lesson checks and acceptance

- A deliberate hang causes an observed watchdog reset within the documented range, using free-running execution.
- Reset cannot resume motion; output defaults, queue clearing and position invalidation are demonstrated.
- Feeding policy depends on meaningful service progress, and reset causes are recorded before clearing.

### End-of-lesson questions

1. Why can feeding the watchdog from an ISR conceal a broken application?
2. Why should watchdog timeout be documented as a range?
3. Why does a watchdog reset not make the last position trustworthy?

### Submit for whole-lesson review

Include timeout math, hang/reset evidence, boot diagnostic output and proof the injection is absent from the normal configuration.

Commit the implementation and `learning/evidence/M09-L03.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.
