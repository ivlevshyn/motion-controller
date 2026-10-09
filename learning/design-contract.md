# Controller design contract

These decisions coordinate all modules. Amend deliberately when measurements justify a change; record the reason in progress.md. They are engineering choices for this small trainer, not claims about every motion controller.

## Units and position

Internally, position is signed `int32_t` in **pulses** (called steps in APIs). A full step equals one pulse only in full-step mode. Use `int64_t` intermediates for target differences and conversions. Speeds are pulses/s and acceleration is pulses/s². Reject invalid/out-of-range input before conversion. Initially cap speed at 200 pulses/s; raise only after measurements, with 1000 pulses/s a provisional course ceiling, not a guaranteed motor capability.

Position means commanded/emitted pulse estimate. `position_valid` starts false, becomes true after successful homing, and becomes false after reset, drive disable that allows manual movement, suspected stall or abrupt fault abort. Track `homed`, `enabled`, `moving` and fault reason explicitly. Do not report the requested target as achieved before the pulse backend finishes.

Use mechanical steps/mm only after M07. Example: 200 full steps/rev and an 8 mm **lead** gives 25 pulses/mm; at 1/8 microstepping, 200 pulses/mm. Lead and repeatability must be measured on the actual mechanism.

## States and commands

Boot: disabled, unhomed, no queued work. Normal states: DISABLED, IDLE, HOMING, MOVING, FAULT. Homing has bounded substates. Its final slow latch is followed by a fixed, bounded backoff to a confirmed released switch; define zero at that released location so ordinary IDLE is not left sitting on an asserted limit. Ordinary absolute moves require a valid home and travel envelope. Early bench tests use a bounded relative test move with the mechanism detached; disable that mode for the finished mechanism.

A software STOP requests a controlled stop once acceleration exists. Limit hits, command-watchdog expiry and internal faults request ABORT: stop STEP generation, disable the driver, clear queued work, invalidate position and latch the cause. ABORT is intentionally abrupt and can lose position. Releasing a switch or clearing a fault must not resume motion. CLEAR is rejected while the cause persists. Recovery from a tripped limit is a separately bounded away-from-limit operation, not unrestricted jogging.

Initial line protocol: ASCII, newline terminated, maximum **80 bytes including newline**, excluding any storage-only NUL terminator. Allocate at least 81 bytes if storing that full frame plus NUL. No heap. Accept CRLF by stripping a single CR immediately before LF. On overflow discard until newline and report one error; never execute a truncated prefix.

| Command | Introduced | Meaning |
|---|---|---|
| `STATUS` | M06 | State, position validity, fault and current limits |
| `ENABLE` / `DISABLE` | M06 | Deliberately energize an otherwise healthy idle controller / abort work and disable; enabling never establishes homing |
| `MOVE_REL <signed_steps> <rate>` | M06 | Bounded bench move; later requires homing/soft limits |
| `STOP` | M06 | Immediate bounded stop initially; controlled stop after M08 |
| `HOME` | M07 | Bounded homing sequence |
| `MOVE_ABS <steps> <rate>` | M07 | Absolute target within calibrated travel |
| `RECOVER <signed_steps>` | M07 | Deliberate tiny low-rate move away from the single active limit, within a measured bound; never restores position validity |
| `CLEAR` | M09 | Clear a resolved fault; leaves position invalid |
| `PING` | M09 | Refresh host lease while remote motion is active |
| `SAVE` / `LOAD` | M12 | Idle-only settings operations; never restore live position |

Distinguish `OK accepted id=…` from `DONE id=…`. A duplicate sequence ID must not cause unintended replay; either reject duplicates or implement a documented idempotency cache. Use four pending move slots plus one active move record. STOP/ABORT clears all pending slots and emits cancellation results where communication remains available. Reserve an out-of-band stop/abort flag so a full motion queue cannot prevent stopping. Invalid requests do not modify motion state.

## Pulse backend and timing

TIM1_CH1/PA8 produces STEP. Do not generate precision steps from an RTOS tick or a busy-wait loop in the final design. The learning implementation uses bounded blocks of **1–256 pulses** based on TIM1 edge-aligned upcounting, one-pulse mode and its repetition counter. `RCR = pulse_count - 1`. PWM mode 2 can keep the output inactive at CNT=0 and active in the latter part of a period; the update at the final overflow stops the counter at an inactive level. Verify startup/idle behavior in the actual configuration, including MOE, CC1E and output-idle controls.

Before a block: disable the counter/output; configure direction while STEP is inactive; load prescaler, ARR, CCR and repetition count; generate the required update to latch preloads; clear the resulting flags; establish inactive output; then start. Do not count the software update as a completed block. Verify N=1 and N=256 electrically before motor use. Zero-length moves emit zero edges. Direction setup/hold and STEP high/low times must meet the A4988 datasheet; the initial low rates provide generous margin. Use explicit minimum timing constants rather than implicit luck.

A completion ISR reports a finished block. A later block may begin after a software scheduling gap. This design bounds unintended extra pulses even if the task is delayed, but **does not guarantee gap-free streaming**. Choose small blocks, measure gaps and their effect on acceleration, and report the limitation. Lower speed or improve scheduling if it is unacceptable; do not claim uninterrupted precision from an unmeasured callback chain. A seamless DMA/timer-chain backend is an optional extension.

TIM2 external-clock mode counts the STEP signal via the PA8→PA0 loopback. It is useful for edge-count verification, but shares the MCU clock environment and is not independent calibration equipment. A motor motion test and a timer-count test establish different facts.

## Persistent settings

On the 512 KiB STM32F401RE only, reserve Flash sectors 6 and 7 for two redundant records. Sector 6 starts at `0x08040000`, sector 7 at `0x08060000`; each is 128 KiB. Limit application Flash to the first 256 KiB in the linker script **before** writing settings. Check the link map and generated configuration after regeneration. Do not apply these addresses to another MCU.

Records have explicit encoding, magic, format version, length, monotonic sequence, settings payload, CRC and a final commit marker. Store limits/calibration, never an instruction to resume motion or a trusted current position. Validate every decoded value. Write the inactive slot and verify it before programming its commit marker; keep the prior valid record until the new one is committed. Erase only an inactive slot. Saving is permitted only when motion is stopped, drive disabled and host notified. Flash erase/program can stall instruction fetch on this device; ISR-driven motion cannot be assumed to survive it.

## Enable and limit-recovery admission

ENABLE is allowed only when idle/disabled without an unresolved fault; it never sets position valid. HOME requires an explicitly enabled controller and accepts the already-active HOME case only through its bounded away/backoff substate. At boot an active HOME switch permits that deliberate bounded homing entry or recovery, not ordinary moves; both switches active is a wiring/mechanics fault. A new unexpected assertion outside the expected homing/recovery state aborts.

RECOVER is the one narrow movement exception for an active-limit fault: the requested direction must lead away from that single limit, speed is fixed low, distance and time are capped from the actual mechanism, the opposite limit and STOP remain effective, and queued/ordinary motion stays forbidden. On completion disable and retain invalid position; require resolved cause, CLEAR and a new HOME before ordinary moves. Failure to release within the bound keeps the fault latched. Do not use RECOVER to bypass a software/internal fault.
