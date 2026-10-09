# STM32 motion controller: learn by building

Version 1.0 • prepared 4 October 2026 • teaching language: English

Build one small, horizontal, single-axis positioning stage, beginning with an STM32 board and later a motor. Learn C, electronics, STM32 peripherals, real-time design, testing and FreeRTOS through the same product. This is a detailed course and teaching specification, not prebuilt controller firmware.

## Start here

1. Read [the shopping list](shopping-list.md). Order **Cart A only** for the cheapest start, or A+B to avoid another order before the inputs lessons. The motor and mechanism can wait.
2. Read [the course map](course-map.md) and [environment guide](environment.md).
3. Put this entire `learning/` directory at the root of your Git repository. Leave the rest of the repository structure open. The first lessons introduce actual project files.
4. Start with **M01-L01**, using the teaching prompt below.
5. Complete each whole lesson, run its end checks, answer its questions, then commit and push. Give the mentor the lesson ID and full commit SHA for review.
6. Record the review using [progress.md](progress.md) and the [evidence template](evidence/README.md). Continue after required fixes are resolved.

You know Java and Swift; no previous C, STM32 or electronics experience is assumed. Nominal lessons take 60–120 minutes; setup, mechanics and concurrency can take 2–3 hours plus troubleshooting. There is no speed requirement.

## Teaching prompt for a fresh conversation

```text
You are my embedded-development teacher.
Repository: <GitHub URL>
Branch / starting commit: <branch and full SHA>
Lesson: M01-L01

First read learning/mentor.md, learning/progress.md,
learning/course-map.md, learning/environment.md, learning/hardware.md,
and the complete module containing this lesson. Inspect the relevant existing
project code. Confirm what you could actually access; request an archive or
specific missing files if necessary, rather than inventing their contents.

Teach the entire requested lesson in one message where practical. Explain
both theory and code for a Java/Swift programmer new to C and electronics.
Give all implementation steps together. Put formal checks and questions at
the end. Do not insert mandatory interactive checkpoints. Use explanations,
worked examples and scaffolding; let me implement the feature.
```

## Review prompt

```text
Review my completed lesson <Mxx-Lyy>.
Repository: <GitHub URL>
Lesson starting SHA: <full SHA>
Submission SHA: <full SHA>
Evidence: learning/evidence/<Mxx-Lyy>.md and linked attachments

Read learning/mentor.md and the complete lesson. Inspect the full relevant
implementation at the submission SHA, not only the diff. Assess every
acceptance criterion, the evidence and my answers. Distinguish code inspection,
commands actually executed, and my hardware observations. Report required
fixes separately from optional improvements. Give a whole-lesson verdict.
```

Repository visibility is an access constraint: a link does not guarantee a chat can read GitHub, especially a private repository. An uploaded ZIP of the same commit is an equivalent input. If read access fails, the teacher can still explain general theory, but cannot claim a code-specific review.

## Navigation

- [Mentor contract](mentor.md): binding teaching and review behavior.
- [Course map](course-map.md): all 36 lessons and hardware gates.
- [Shopping list](shopping-list.md): Polish shop links, quantities and staged costs.
- [Hardware](hardware.md): electrical design, pin map and mechanics contract.
- [Environment](environment.md): Apple Silicon workflow and recorded versions.
- [Design contract](design-contract.md): commands, units, states and timing policy.
- [C practices](c-practices.md): conventions introduced progressively.
- [Glossary](glossary.md): terms in plain language.
- [Sources](sources.md): primary technical references and verification limits.
- [Progress](progress.md): authoritative course state.

## What completion means

Your controller homes a lightweight horizontal axis, accepts bounded position commands, executes measured motion profiles, stops and reports faults, saves settings, and passes a repeatability test. Software position is an **estimate from emitted pulses**, not an encoder measurement. The core course does not claim industrial functional-safety certification or camera-grade precision.

Advanced firmware depth comes from timing, ownership, concurrency, fault handling and evidence—not from buying more boards. Optional encoder, multi-axis and Rust extensions are listed in the course map; they are not required purchases.
