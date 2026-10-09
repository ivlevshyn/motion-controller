# Development environment — Apple Silicon, macOS 27

## Baseline and compatibility

Host supplied by learner: MacBook Pro M5 Max, macOS 27. Target: NUCLEO-F401RE / STM32F401RET6. Firmware: C17 application, STM32CubeF4 HAL and CMSIS, initially bare metal. The target executable is Arm Cortex-M4F code; it does not run as a macOS executable.

**macOS 27 remains an unverified host for this course.** ST's published VS Code requirements checked on 2026-10-04 list macOS 15 and 26, including aarch64. An ST moderator reports that CubeIDE's supported host list does not yet include 27. We have not installed tools on your Mac or exercised a board. A native Apple Silicon download is not proof of full macOS 27 compatibility. M01 explicitly establishes a working baseline.

## Primary route

Use the current official Apple Silicon STM32CubeIDE package (2.2.0 was the published candidate during research), the separate STM32CubeMX configuration application, and STM32CubeProgrammer. Install one coherent release set from ST, following its current macOS installation instructions. STM32CubeIDE 2.x workflows may open CubeMX separately; do not follow old tutorials assuming its pin configurator is built into the IDE.

1. Record `sw_vers`, `uname -m`, and existing tools before installing. Expect `arm64` for a native terminal.
2. Obtain the applications from the official links in sources.md. Use the Apple Silicon variant when offered. Install through normal macOS package/application workflows. Record exact versions, including CubeF4 firmware package and bundled GCC.
3. Connect the Nucleo through a **data-capable USB-A to mini-B cable**, with a USB-C adapter if needed. Keep its onboard ST-LINK jumpers in their delivered state. Use no motor supply yet.
4. In CubeProgrammer, choose ST-LINK, SWD, conservative probe speed, connect, and read target identification. Do not mass-erase or change option bytes as a discovery step. Record MCU and probe firmware versions.
5. In standalone CubeMX, create a Board Selector project for NUCLEO-F401RE. Inspect rather than blindly accept peripheral defaults. Keep Serial Wire debug. Start from HSI 16 MHz; do not depend on an external crystal or ST-LINK MCO routing. Select the matching STM32CubeIDE project generator. Generate into the actual repository path chosen in M01-L02.
6. Build in CubeIDE. Confirm the debugger can reach `main`, set a breakpoint in application code, continue and reset. Record the exact reproducible build route before moving on.
7. Serial output is added in M02-L03 via USART2 and the onboard ST-LINK virtual COM port. List available serial devices using `ls /dev/cu.*`. Do not assume a fixed device suffix or hard-code it into committed configuration.

If the installed tool's screens differ, the teacher must consult that version's documentation and explain the equivalent settings. A teaching document cannot guarantee future menu labels.

## Troubleshooting by layer

| Observation | Investigation | What not to infer |
|---|---|---|
| Board absent from macOS USB inventory | Data cable, adapter, connector, power LED, direct connection | That the C code is broken |
| USB present but programmer cannot connect | ST-LINK firmware, selected interface, probe speed, target identity, jumpers | That a new debugger must be purchased |
| Programmer connects but IDE debug fails | Actual server executable path, architecture, permissions, debug configuration, other tools holding probe | That flashing and debugging are the same operation |
| `Bad CPU type in executable` | Inspect that exact executable with `file`; prefer correct native package; some components may need Rosetta | That every bundled component is native |
| Program runs but serial is empty | USART2 mapping, baud, virtual COM device, one terminal owner, newline policy | That application USB firmware is required |

Do not delete broad system directories, disable Gatekeeper globally, install untrusted drivers or downgrade macOS automatically. Capture the specific error and resolve the failing layer.

## Fallback route if primary debug remains blocked

Use VS Code, an Arm GNU embedded compiler, CMake/Ninja, and OpenOCD or probe-rs for C ELF flashing/debugging, with CubeMX-generated CMake where supported. Both flashing and an actual breakpoint must be demonstrated. Choose **one** fallback configuration and record it; do not teach several toolchains concurrently. Probe-rs also supports C, and offers native Apple Silicon releases, but it is not prevalidated here on macOS 27. This route preserves the same board and application rather than requiring Windows hardware.

Host tools may be installed via their official installers or a package manager the learner already uses. If Xcode command line tools are missing, `xcode-select --install` obtains the host compiler used by pure-C tests. Host `clang` and target `arm-none-eabi-gcc` serve different roles.

## Version record to fill in M01

| Item | Recorded value |
|---|---|
| macOS build and terminal architecture | Pending local setup |
| IDE / extension and architecture | Pending |
| CubeMX | Pending |
| CubeProgrammer / ST-LINK firmware | Pending |
| STM32CubeF4 package | Pending |
| Arm GCC / GDB | Pending |
| Build command or documented IDE build | Pending |
| Host clang | Pending |
| Git commit establishing known-good environment | Pending review |

Pin the functioning release set for the course. Upgrade deliberately in a separate commit, with rebuild, flash, breakpoint and serial regression checks. Do not require the newest version for every lesson.

## Clock and build milestones

M01–M04 use HSI 16 MHz with generated defaults explicitly recorded. M05 introduces a nominal 84 MHz SYSCLK from HSI through PLL: M=16, N=336, P=4; AHB /1, APB1 /2 and APB2 /1. Verify voltage scaling, Flash latency and all device limits in CubeMX and RM0368. With this tree, TIM1 and TIM2 kernel clocks are nominally 84 MHz; do not equate APB1's 42 MHz bus with its timer clock. Recalculate UART/timer configuration after switching.

Introduce application warning flags from c-practices.md gradually. Generated code changes must be reviewable. Verify regeneration does not erase application work. Do not commit build folders, binary output, serial-device names or private machine paths.
