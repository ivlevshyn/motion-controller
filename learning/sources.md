# Sources and verification boundaries

Prepared 4 October 2026. Links are reading references, not bundled copies of third-party material. Product prices are snapshots in shopping-list.md. Recheck current release requirements when teaching setup; pin the installed versions in environment.md.

## Hardware and peripherals

| Reference | Use it for |
|---|---|
| [ST NUCLEO-F401RE product page](https://www.st.com/en/evaluation-tools/nucleo-f401re.html) | Board identity and official resources |
| [UM1724: Nucleo-64 MB1136 manual](https://www.st.com/resource/en/user_manual/um1724-stm32-nucleo64-boards-mb1136-stmicroelectronics.pdf) | Board wiring, ST-LINK, headers and solder bridges; select the F401RE tables |
| [STM32F401RE product resources](https://www.st.com/en/microcontrollers-microprocessors/stm32f401re.html) | DS10086 datasheet, alternate functions, electrical limits and errata ES0299 |
| [RM0368 reference manual, Rev 6](https://www.st.com/resource/en/reference_manual/rm0368-stm32f401xbc-and-stm32f401xde-advanced-armbased-32bit-mcus-stmicroelectronics.pdf) | RCC, GPIO, TIM1/TIM2, USART, DMA, IWDG and Flash; use section titles rather than another MCU's register examples |
| [Allegro A4988 datasheet, hosted by Pololu](https://www.pololu.com/file/0J450/A4988.pdf) | Chip electrical/timing requirements, current regulation, wake and STEP/DIR behavior |
| [Pololu A4988 carrier documentation](https://www.pololu.com/product/1182) | Carrier-level power/decoupling guidance; its exact sense resistor is not assumed for the OEM board |
| [ST STM32CubeF4 source](https://github.com/STMicroelectronics/STM32CubeF4) | Actual HAL/LL code and examples; match the installed package version |

The course's pin allocation, protocol, state policy, timer-block architecture and budgets are design choices. They must be confirmed on the actual assembled hardware. Manufacturer documentation does not certify this assembled project.

## Mac and tools

- [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html), [STM32CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html), [STM32CubeCLT](https://www.st.com/en/development-tools/stm32cubeclt.html) and [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html): official installation/release pages.
- [ST VS Code environment system requirements](https://dev.st.com/stm32cube-docs/stm32cubeide-vscode/latest/en/docs/markup/introduction/system_requirements.html): checked host architecture/OS support listing.
- [ST community macOS 27 debug issue, with ST moderator response](https://community.st.com/stm32cubeide-mcus-28/no-st-link-detected-when-trying-to-run-as-the-project-in-stm32cubeide-on-macos-27-168496): evidence of the current support gap. User workarounds are anecdotal; do not copy destructive cleanup commands.
- [probe-rs installation](https://probe.rs/docs/getting-started/installation/) and [releases](https://github.com/probe-rs/probe-rs/releases): an alternative probe stack that can work with C/C++ ELF files; installation/runtime on this exact Mac remains to be tested.
- [Swift documentation](https://www.swift.org/documentation/): host language/tooling reference. For Darwin termios/open/poll use the installed macOS SDK and `man termios`, `man open`, `man poll`; the lesson must use actual available interfaces.

**macOS 27 is not assumed vendor-supported.** M01 requires real flash, breakpoint, inspection and resume on the learner's machine. This package was not executed on that Mac or a connected STM32.

## C, testing and RTOS

- [GCC warning options](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html): understand a warning rather than silence it with a cast.
- [Clang AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html) and [UndefinedBehaviorSanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html): host test instrumentation, not automatic proof of target correctness.
- [FreeRTOS Cortex-M guidance](https://www.freertos.org/Documentation/02-Kernel/03-Supported-devices/04-Demos/ARM-Cortex/RTOS-Cortex-M3-M4): interrupt urgency, syscall thresholds and port rules.
- [FreeRTOS kernel source](https://github.com/FreeRTOS/FreeRTOS-Kernel): verify the actual selected version/configuration and API definitions.

## What was and was not validated when preparing the course

The package was checked for lesson coverage, internal links, arithmetic consistency, purchase staging and conflicting resource allocations. Public primary references and Polish product pages informed the choices. No build of the future learner-written firmware, physical wiring, motor movement or macOS 27 debug session has been performed. The lesson checks are future acceptance criteria, not reports of completed tests. Exact HAL/RTOS calls must match the installed headers when the lesson is taught.
