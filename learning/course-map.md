# Course map and dependencies

The course contains **12 modules and 36 lessons**. Complete them in order: each lesson depends on the preceding lesson, except M01-L01. Existing Java/Swift knowledge helps with general programming; the C memory model, electronics and real-time behavior are introduced explicitly.

Each module is one focused Markdown file containing three complete lesson specifications: explanation, worked reasoning/examples, ordered implementation, troubleshooting, end checks, questions and submission evidence. The mentor expands these into a lesson adapted to the current repository using mentor.md. These are teaching materials, not a completed firmware solution or a fixed source-tree scaffold.

Budget **roughly 80–130 active hours**, an estimate rather than a deadline. Early lessons often take 1–2 hours; toolchain problems, mechanics, timer setup and concurrency may take longer. Purchasing/shipping and review-fix time are additional. At two lessons a week the 36-lesson path takes about 18 weeks before extra practice/revisions. Shorter sessions can pause naturally without adding mandatory mid-lesson reviews.

## Sequence

| Lesson | Product increment / teaching topic |
|---|---|
| M01-L01 | [Identify the board and establish the toolchain](modules/01-board-and-debugging.md) |
| M01-L02 | [Generate, build and flash a minimal application](modules/01-board-and-debugging.md) |
| M01-L03 | [Use the debugger and make a status indicator](modules/01-board-and-debugging.md) |
| M02-L01 | [Represent quantities without overflow](modules/02-c-for-firmware.md) |
| M02-L02 | [Understand pointers, storage and module boundaries](modules/02-c-for-firmware.md) |
| M02-L03 | [Produce bounded serial diagnostics](modules/02-c-for-firmware.md) |
| M03-L01 | [Wire and read a physical control](modules/03-inputs-and-controller-states.md) |
| M03-L02 | [Debounce without delaying the program](modules/03-inputs-and-controller-states.md) |
| M03-L03 | [Define explicit state transitions](modules/03-inputs-and-controller-states.md) |
| M04-L01 | [Assemble and current-limit the driver](modules/04-motor-electronics-and-first-motion.md) |
| M04-L02 | [Generate the first deliberate steps](modules/04-motor-electronics-and-first-motion.md) |
| M04-L03 | [Schedule a finite move cooperatively](modules/04-motor-electronics-and-first-motion.md) |
| M05-L01 | [Calculate and observe hardware PWM](modules/05-timers-and-bounded-pulse-blocks.md) |
| M05-L02 | [Bound pulse count in hardware](modules/05-timers-and-bounded-pulse-blocks.md) |
| M05-L03 | [Integrate the asynchronous backend](modules/05-timers-and-bounded-pulse-blocks.md) |
| M06-L01 | [Frame serial input without losing boundaries](modules/06-commands-and-the-mac-client.md) |
| M06-L02 | [Parse and dispatch commands transactionally](modules/06-commands-and-the-mac-client.md) |
| M06-L03 | [Build a small Swift host client](modules/06-commands-and-the-mac-client.md) |
| M07-L01 | [Assemble the axis and calibrate distance](modules/07-mechanics-and-homing.md) |
| M07-L02 | [Implement bounded homing](modules/07-mechanics-and-homing.md) |
| M07-L03 | [Enforce soft limits and absolute moves](modules/07-mechanics-and-homing.md) |
| M08-L01 | [Plan acceleration and controlled stopping](modules/08-acceleration-and-queued-motion.md) |
| M08-L02 | [Handle short moves and reversals](modules/08-acceleration-and-queued-motion.md) |
| M08-L03 | [Queue commands with clear cancellation](modules/08-acceleration-and-queued-motion.md) |
| M09-L01 | [Build a host regression suite](modules/09-testing-and-fault-recovery.md) |
| M09-L02 | [Handle lost hosts and invalid recovery requests](modules/09-testing-and-fault-recovery.md) |
| M09-L03 | [Add an independent watchdog and boot diagnostics](modules/09-testing-and-fault-recovery.md) |
| M10-L01 | [Receive serial data with DMA](modules/10-dma-and-measured-timing.md) |
| M10-L02 | [Measure the timing budget](modules/10-dma-and-measured-timing.md) |
| M10-L03 | [Read registers to improve one measured path](modules/10-dma-and-measured-timing.md) |
| M11-L01 | [Introduce tasks without moving pulse timing into the scheduler](modules/11-freertos-with-clear-ownership.md) |
| M11-L02 | [Transfer events safely across tasks and interrupts](modules/11-freertos-with-clear-ownership.md) |
| M11-L03 | [Compare RTOS and bare-metal behavior under load](modules/11-freertos-with-clear-ownership.md) |
| M12-L01 | [Store settings without risking the application image](modules/12-persistence-and-project-release.md) |
| M12-L02 | [Measure repeatability and declare an operating envelope](modules/12-persistence-and-project-release.md) |
| M12-L03 | [Prepare a reproducible portfolio release](modules/12-persistence-and-project-release.md) |

## Module gates

| Before | Hardware / evidence needed | What can wait |
|---|---|---|
| M01 | Cart A: board and USB connection | Everything else |
| M02 | Real flash, breakpoint and resume from M01 | External electronics |
| M03 | Cart B: button, breadboard, leads, resistors and meter | Motor and mechanics |
| M04 | Cart C, inspected soldered assembly and identified current-sense resistors | Carriage and rails |
| M05 | Correct motor circuit; one signal jumper for timer loopback | External analyzer if on-target evidence is sufficient |
| M06 | Verified finite pulse blocks and STOP behavior | Mechanics |
| M07 | Matched horizontal mechanism, two NC switches and mounting | Payload, enclosure, encoder |
| M08 | Bounded physical homing and valid soft limits | Continuous look-ahead blending |
| M09 | Working conservative motion profiles and queue | Advanced UI |
| M10 | Regression tests and fault recovery | Expensive lab equipment; borrow if needed |
| M11 | Measured bare-metal baseline | Additional motors |
| M12 | Chosen architecture, tested envelope and Flash-layout review | Optional extensions |

A gate can be **Verification pending** while theory is studied. Simulated input or a shaft pointer does not satisfy carriage homing and repeatability criteria. Do not start an unsafe physical step just to keep pace with the sequence.

## Deliberate progression

- M01–M03: learn tools, C values/lifetimes, electrical inputs and explicit state while the product is a controller without a motor.
- M04–M06: make the motor move, replace blocking timing with hardware-bounded blocks, then expose a robust command interface and Swift client.
- M07–M09: add the real stage, homing, limits, acceleration and trustworthy recovery.
- M10–M12: measure bottlenecks, add DMA and an RTOS deliberately, persist settings and validate a portfolio release.

## Core limitations and optional follow-ons

The core is an open-loop, low-load training axis with measured operating limits. TIM1 bounds pulses within a block, but software starts later blocks and can introduce gaps. The release must disclose the measured effect. There is no encoder proof of actual position and no certified emergency-stop system.

After M12, choose one separate extension:

1. **Continuous pulse backend:** timer/DMA sequencing with explicit underrun handling, exact pulse termination and external waveform measurements. Re-run all count/abort tests before claiming gap-free motion.
2. **Position feedback:** add a linear/rotary encoder and compare physical measurement to pulse estimate; develop following-error detection before calling it closed-loop control.
3. **Second axis:** coordinated path planning, synchronization and resource allocation. Requires another motor/driver and suitable mechanics.
4. **Custom PCB:** transfer the proven schematic, decoupling, connectors and defaults to a board; perform layout/current-return review and bring-up.
5. **Embedded Rust comparison:** port one pure module or the controller after completing C, and compare ownership, ISR boundaries, ecosystem and generated code.

These are optional projects beyond the finished course; do not buy their hardware at the start.
