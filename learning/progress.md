# Progress and handoff record

Course version: 1.0, 4 October 2026. **No lessons have been completed or hardware-tested by creation of this package.** This file records the learner's actual work and reviews.

## Current context

| Field | Value |
|---|---|
| Current lesson | M01-L01 |
| Last completed lesson | None |
| Host | MacBook Pro M5 Max, macOS 27; exact build to record in M01 |
| Prior languages | Java, Swift |
| Repository URL / branch | Record when created |
| Last reviewed implementation SHA | None |
| Firmware source/build paths | Choose during M01-L02 |
| Host tests path / command | Choose during M02-L01 |
| Swift client path / command | Choose during M06-L03 |
| Board / revision | Planned NUCLEO-F401RE; record actual marking on arrival |
| Wiring revision | Planned H1 from hardware.md; record actual changes |
| Driver / sense resistor / VREF | Not purchased or measured yet |
| Mechanics / microstep setting | Not selected; initial motor tests full-step |
| Tool versions | Fill environment.md after actual installation |
| Physical verification status | None yet |
| Unresolved issues | macOS 27 flash/debug compatibility must be demonstrated in M01 |

## Lesson ledger

Status vocabulary: **Not started**, **In progress**, **Submitted**, **Needs revision**, **Verification pending**, **Complete**. Only an actual review can establish Complete. A row may link to multiple review attempts in its evidence file.

| Lesson | Status | Submitted implementation SHA | Review / evidence |
|---|---|---|---|
| M01-L01 | Not started | — | — |
| M01-L02 | Not started | — | — |
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

1. Before the lesson, record its starting full SHA in the evidence file.
2. Complete the lesson and its end checks/questions. Commit evidence and implementation together, then push.
3. Supply the resulting SHA in the chat and ledger on a subsequent commit if needed. A commit cannot contain its own final hash.
4. After review, record verdict, reviewed implementation SHA, required fixes and the next lesson. This review-record commit is separate from the reviewed implementation commit.
5. If fixes are requested, preserve the earlier review and submit a new SHA. Do not mark Complete while required evidence is absent.

## Decision history

| Date / lesson | Decision | Reason | Affected files / verification |
|---|---|---|---|
| Course setup | C on NUCLEO-F401RE; low-load single-axis motion controller | Budget path, onboard debugger, learn C and STM32 from first principles | hardware.md, environment.md, design-contract.md |
| Course setup | All instructions under learning/; source layout emerges during lessons | Learner's requested repository structure | mentor.md |

Append hardware substitutions, pin changes, actual toolchain selection, timer changes, operating limits and keep/revert decisions here. Update canonical documents as well; do not rely on a chat-only decision.
