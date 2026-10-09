# Module 01 — Board and debugging

**Starting point:** Cart A only. No motor circuit connected. Read environment.md; macOS 27 support is a compatibility gate, not an assumption.

Read the [mentor contract](../mentor.md) before teaching. Each lesson below is one complete teaching unit. Deliver its explanation and implementation together; formal checks, questions and review happen at the end. Use [the evidence template](../evidence/README.md) for the submission. Every lesson also requires a clean applicable build, no unexplained new warnings, and no regressions in previously completed behavior.

## M01-L01 — Identify the board and establish the toolchain

**Prerequisite:** None. **Product change:** A documented Mac-to-board development environment.

### Explanation and worked example

An MCU runs the binary you flash directly; it does not launch a Java VM or a macOS process. The compiler translates C, the linker places code and data, and the debugger controls execution through SWD. ST-LINK is a second device on the Nucleo that provides that debug connection. USB serial is a separate function of the same interface.

Distinguish host architecture (your Mac's ARM64) from target architecture (Cortex-M4). Native Apple Silicon applications can still produce Cortex-M binaries. A tool running on macOS does not prove that its bundled debug server runs there. The first module therefore proves the entire chain, including a real breakpoint.

Read the board label: NUCLEO-F401RE is the board; STM32F401RET6 is the target chip. Find LD2, reset, the user button and the ST-LINK mini-USB connector using the board manual. A USB connector fitting mechanically does not prove a cable carries data.

### Build it, in order

1. Create a Git repository with the supplied `learning/` directory. Record your Java/Swift background and host details in progress.md. Do not invent the future firmware directory structure.
2. Follow environment.md to install the current suitable ST tools, recording actual versions and download URLs. Begin with STM32CubeIDE plus standalone CubeMX if required by that release. Do not install every alternative toolchain at once.
3. Connect the board through the data cable and adapter. Inspect macOS System Information → USB and available `/dev/cu.*` devices. Record which device appears when this board is connected; do not choose a hard-coded port name from an example.
4. Locate the board user manual, MCU datasheet and reference manual through sources.md. Explain the role of each. Record board revision and jumper defaults before changing anything.
5. Establish Git ignore rules for build output and IDE user metadata as those files appear. Keep source configuration, linker scripts and the CubeMX `.ioc` file versioned.

### Troubleshooting

No USB device: try a known data cable and another adapter/port before reinstalling tools. USB visible but debug unavailable is a different failure; record the exact server error and consult environment.md. Do not erase unrelated tool installations or weaken global security settings.

### End-of-lesson checks and acceptance

- The repository contains this course and a filled host/tool inventory, without generated binary clutter.
- The actual board and USB device are identified. A serial port may be pending until the ST-LINK firmware/setup is corrected; document this explicitly.
- The installed IDE launches on this Mac. Flash/debug proof is required in the following lessons, not presumed here.

### End-of-lesson questions

1. Which software runs on the Mac and which binary runs on the MCU?
2. Why can USB serial work while SWD debugging fails?
3. Which manual would you use for a board header pin versus a timer register?

### Submit for whole-lesson review

Include board photos/markings, tool versions, USB observations, installation errors and their resolution. No passwords or personal machine identifiers are needed.

Commit the implementation and `learning/evidence/M01-L01.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M01-L02 — Generate, build and flash a minimal application

**Prerequisite:** M01-L01. **Product change:** A reproducible firmware build drives the onboard LED.

### Explanation and worked example

Generated initialization establishes clocks and peripheral state; your application adds behavior. A typical `main` initializes HAL, configures the clock, initializes GPIO and enters an infinite loop. Unlike a desktop program, returning from main is not the normal application lifecycle.

Start with the internal 16 MHz HSI clock. A simple toggle separated by 500 ms delays produces a roughly one-second on/off cycle. `HAL_Delay` blocks the foreground thread; that is acceptable for this first observation, but later would delay input handling. Explain what a function call and `while (1)` mean in C before using them.

The `.ioc` stores configuration intent; generated C and the linker script are build inputs. Regeneration can overwrite edits outside protected user sections. A clean build should be possible from committed inputs without relying on an untracked local file.

### Build it, in order

1. Create a board-specific F401RE project in a firmware location you choose now and record in progress.md. Retain Serial Wire debug and configure PA5 as the onboard LED output. Use HSI at 16 MHz; record the actual clock tree.
2. Generate code. Walk through reset/startup, main, clock setup and GPIO initialization at a conceptual level. Do not edit startup assembly yet.
3. Add LED toggling in a protected application area. Explain the HAL GPIO port and pin arguments, integer delay units and the semicolon.
4. Build. Read one compiler command and the final memory usage; distinguish a compile error from a linker error. Flash through ST-LINK, reset and observe LD2.
5. Regenerate once after a harmless configuration change, then revert that change. Confirm your application survives. Record the build and flash procedure in environment.md.

### Troubleshooting

A flashing board power LED does not prove your program ran. Check LD2/PA5, actual downloaded ELF and reset behavior. If the IDE expects integrated CubeMX that this release no longer provides, use the standalone configuration workflow described in environment.md.

### End-of-lesson checks and acceptance

- A clean build produces an ELF for STM32F401RE, and the board visibly follows the intended LD2 timing after reset.
- Regeneration preserves application changes; configuration and required generated inputs are committed.
- Build/flash commands or exact IDE actions and memory usage are recorded. No claim of precise clock calibration is made from an eyeballed LED.

### End-of-lesson questions

1. Why is an infinite loop appropriate here?
2. What does the linker decide that the compiler does not?
3. Why will a 500 ms blocking delay become a problem for a motion controller?

### Submit for whole-lesson review

Attach a short observation/video of LD2, build output summary, configuration changes and the selected application paths.

Commit the implementation and `learning/evidence/M01-L02.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M01-L03 — Use the debugger and make a status indicator

**Prerequisite:** M01-L02. **Product change:** The board reports a state without blocking the foreground loop.

### Explanation and worked example

A breakpoint suspends CPU execution so you can inspect variables and the call stack. Peripherals can continue while the CPU is halted. This is why the motor stays disconnected during debugger lessons. Step-over runs a call; step-into enters it; neither is equivalent to observing real-time behavior.

Use an unsigned millisecond tick and elapsed-time subtraction:
```c
uint32_t now = HAL_GetTick();
if ((uint32_t)(now - last_change_ms) >= interval_ms) {
    last_change_ms = now;
    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
}
```
Explain `uint32_t`, assignment, comparison, the cast and block braces. Unsigned subtraction handles a single wrap correctly for short intervals when checked regularly. It does not make arbitrary long-duration timestamp ordering safe. Updating to `now` avoids rapid catch-up flashes; later timer scheduling will make a different choice.

### Build it, in order

1. Set a breakpoint after initialization, flash and start a debug session. Inspect `now`, a local variable and the call stack; change an LED interval variable to see an observable effect.
2. Replace the delay loop with the elapsed-time pattern. Keep last-change state alive across iterations; explain why declaring and resetting it inside the loop would fail.
3. Introduce two status patterns selected by a simple application state variable. Avoid a complicated LED protocol; steady disabled/slow ready behavior is enough.
4. Demonstrate reset versus resume. Document whether reset reaches main and which state is restored.
5. Finish the Mac compatibility gate: record actual successful flash, breakpoint, variable inspection and resume using the chosen toolchain. If the debug server fails, resolve that concrete failure before motor purchases.

### Troubleshooting

Optimized variables can appear unavailable; use a debug build before assuming corruption. An LED that never toggles often has `last_change_ms` reset every loop. Tick-based behavior pauses or jumps under debugging; assess timing during free running.

### End-of-lesson checks and acceptance

- A real breakpoint, variable inspection and resume work on this Mac; a successful build alone is insufficient.
- Foreground LED behavior contains no blocking delay and resets predictably.
- A paper or small C calculation for `last=0xFFFFFFF0`, `now=0x00000010` yields 32 ms and is explained.

### End-of-lesson questions

1. Which hardware might keep running at a breakpoint?
2. Why must `last_change_ms` outlive a loop iteration?
3. What does the wraparound example establish, and what does it not establish?

### Submit for whole-lesson review

Provide the debug-session evidence, nonblocking implementation and exact working tool versions. This is the decision gate for committing to the rest of the hardware.

Commit the implementation and `learning/evidence/M01-L03.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.
