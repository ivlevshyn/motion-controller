# Progress and handoff record

Course version: 1.0, 4 October 2026. Last updated: **9 October 2026**. M01-L01 and M01-L02 are practically complete based on the learner's reported results and lesson discussion. The M01-L02 source and CubeMX configuration were also inspected; no build or hardware test was run by the mentor.

## Current context

| Field | Value |
|---|---|
| Current lesson | M01-L03 (next; not started) |
| Last completed lesson | M01-L02 |
| Host | MacBook Pro M5 Max, macOS 27; exact OS build not recorded |
| Prior languages | Java, Swift |
| Repository URL / branch | https://github.com/ivlevshyn/motion-controller / main |
| Last reviewed implementation SHA | No formal submission review; M01-L02 source/configuration inspected at `b1eceefef41dbba340aa3b68308cdf21e13aee08` |
| Firmware source/build paths | `firmware/`; application: `firmware/Core/Src/main.c`; CubeIDE project with Debug build output under `firmware/Debug/` |
| Host tests path / command | Choose during M02-L01 |
| Swift client path / command | Choose during M06-L03 |
| Board / revision | NUCLEO-F401RE available and detected by Mac; PCB revision not recorded |
| Wiring revision | USB-connected board only; LD2 on PA5; no external circuit assembled |
| Driver / sense resistor / VREF | Cart C not available yet |
| Mechanics / microstep setting | Not selected; initial motor tests full-step |
| Tool versions | STM32CubeIDE, CubeMX and CubeProgrammer installed. `firmware.ioc` records CubeMX 6.18.1 and STM32CubeF4 1.28.3; IDE/Programmer versions not recorded |
| Physical verification status | Learner reports USB detection and all M01-L02 build/flash, LED timing, reset and regeneration checks working |
| Unresolved issues | Build and flash work on the learner's Mac. Breakpoint, variable inspection and resume remain to be demonstrated in M01-L03 |

## Hardware currently available

The learner has **Shopping Cart A + B** from [shopping-list.md](shopping-list.md):

| Hardware | Quantity |
|---|---|
| NUCLEO-F401RE with onboard ST-LINK | 1 |
| USB-A to mini-USB-B data cable | 1 |
| USB-C to USB-A adapter | 1 |
| 830-hole breadboard | 1 |
| Jumper leads: male-male, female-female, male-female | 120-piece set |
| Tact switches | 5-piece pack |
| 10 kΩ resistors | 30-piece pack |
| UNI-T UT33A+ multimeter | 1 |

Cart C motor/driver/power assembly and Cart D limits/mechanics are not available yet. Current hardware is sufficient for Modules 1–3.

## Lesson ledger

Status vocabulary: **Not started**, **In progress**, **Submitted**, **Needs revision**, **Verification pending**, **Complete**. For this learner, Complete records practical lesson completion from reported results, discussion and available code inspection. Formal submission reviews and separate evidence files are optional; distinguish learner observations from mentor-run checks.

| Lesson | Status | Submitted implementation SHA | Review / evidence |
|---|---|---|---|
| M01-L01 | Complete | — | Learner reports board detected by Mac and requested ST applications installed |
| M01-L02 | Complete | `b1eceefef41dbba340aa3b68308cdf21e13aee08` | Learner reports all lesson checks working; source/configuration inspected. Linker placement and interrupts during blocking delays explained in discussion |
| M01-L03 | Not started | — | — |
| M02-L01 | Not started | — | — |
| M02-L02 | Not started | — | — |
| M02-L03 | Not started | — | — |
| M03-L01 | Not started | — | — |
| M03-L02 | Not started | — | — |
| M03-L03 | Not started | — | — |
| M04-L01 | Not started | — | — |
| M04-L02 | Not started | — | — |
| M04-L03 | Not started | — | — |
| M05-L01 | Not started | — | — |
| M05-L02 | Not started | — | — |
| M05-L03 | Not started | — | — |
| M06-L01 | Not started | — | — |
| M06-L02 | Not started | — | — |
| M06-L03 | Not started | — | — |
| M07-L01 | Not started | — | — |
| M07-L02 | Not started | — | — |
| M07-L03 | Not started | — | — |
| M08-L01 | Not started | — | — |
| M08-L02 | Not started | — | — |
| M08-L03 | Not started | — | — |
| M09-L01 | Not started | — | — |
| M09-L02 | Not started | — | — |
| M09-L03 | Not started | — | — |
| M10-L01 | Not started | — | — |
| M10-L02 | Not started | — | — |
| M10-L03 | Not started | — | — |
| M11-L01 | Not started | — | — |
| M11-L02 | Not started | — | — |
| M11-L03 | Not started | — | — |
| M12-L01 | Not started | — | — |
| M12-L02 | Not started | — | — |
| M12-L03 | Not started | — | — |

## Updating after a lesson

Keep this record lightweight: update the current/next lesson, a short result, actual source paths and hardware changes. Detailed inventories, screenshots, separate evidence files and formal commit-SHA reviews are optional unless the learner requests them. Capture exact tool versions, errors or measurements when needed to reproduce a build, diagnose a problem or establish a hardware limit.

## Decision history

| Date / lesson | Decision | Reason | Affected files / verification |
|---|---|---|---|
| Course setup | C on NUCLEO-F401RE; low-load single-axis motion controller | Budget path, onboard debugger, learn C and STM32 from first principles | hardware.md, environment.md, design-contract.md |
| Course setup | All instructions under learning/; source layout emerges during lessons | Learner's requested repository structure | mentor.md |
| 2026-10-09 / M01 | Firmware created in `firmware/`; HSI 16 MHz, PA5/LD2 and SysTick delay | First working build/flash and blinking LED | `firmware/firmware.ioc`, `firmware/Core/Src/main.c`; learner reports checks passed |
| 2026-10-09 / teaching preference | Lightweight progress notes; formal documentation/review optional | Learner requested more focus on building and less paperwork | This agreement supersedes the default documentation requirements for this learner |

Append hardware substitutions, pin changes, actual toolchain selection, timer changes, operating limits and keep/revert decisions here. Update canonical documents as well; do not rely on a chat-only decision.
