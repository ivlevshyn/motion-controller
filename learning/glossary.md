# Working glossary

Read terms when they first appear; memorization is not a prerequisite.

| Term | Meaning in this project |
|---|---|
| MCU / STM32 | The microcontroller that executes firmware and controls peripherals. |
| Nucleo | The development board containing the target MCU, programmer/debugger and connectors. |
| Firmware | The program built for the MCU, normally stored in its Flash. |
| Toolchain | Compiler, linker, debugger and related build utilities. |
| Cross compiler | A compiler running on the Mac that produces code for another target. |
| ELF | A linked executable file with code/data and, in debug builds, symbols. |
| Linker script / map | Placement rules for memory; the map reports what was actually placed where. |
| Flash / RAM | Nonvolatile program/settings storage; volatile working memory. |
| Stack / heap | Automatic call/local storage; dynamically allocated storage. Neither makes C pointers lifetime-safe. |
| Translation unit | A source file after its included headers have been processed. |
| Pointer / lifetime | An address with a type; the period during which the addressed object validly exists. |
| Undefined behavior | An operation for which C imposes no required result; signed overflow is one example. |
| HAL / LL | ST's hardware abstraction and lower-level peripheral interfaces. |
| Register | A hardware control/status location, often accessed through memory-mapped addresses. |
| GPIO / alternate function | A general digital input/output pin; routing that pin to a peripheral such as a timer. |
| Pull-up / pull-down | A resistor establishing a default high/low level when nothing actively drives a signal. |
| Active-low | A signal asserted by a low voltage, such as driver ENABLE or pressed STOP. |
| Ground | The circuit's common voltage reference and return network; current-path layout still matters. |
| NO / NC / COM | Normally open, normally closed and common switch contacts, in the unactuated state. |
| Debounce | Filtering mechanical contact transitions into stable events. |
| SWD / ST-LINK | Target debug interface; the onboard tool that bridges the Mac to it. |
| UART / virtual COM | Byte-oriented serial peripheral; USB-accessible host serial interface from ST-LINK. |
| ISR / NVIC / EXTI | Interrupt handler; interrupt controller; external GPIO interrupt routing. |
| Critical section | A bounded region protected from specified concurrent access, often briefly masking interrupts. |
| Race condition | Correctness depending on an unintended execution/interruption order. |
| Volatile | A C access qualifier useful for hardware/observed state; not general atomicity or synchronization. |
| Timer / PWM | Hardware counter; waveform derived from counter comparisons. |
| PSC / ARR / CCR | Prescaler, auto-reload period limit and compare threshold registers. |
| OPM / RCR | One-pulse mode and repetition counter used for a bounded TIM1 pulse block. |
| DMA | Hardware moving data between a peripheral and memory with less CPU copying. |
| Ring buffer | Fixed storage reused circularly, with explicit producer/consumer positions and overflow policy. |
| Stepper / driver | Motor moving through magnetic step positions; power electronics controlling its coils. |
| STEP / DIR | Pulse input requesting one step/microstep; direction-select input. |
| VREF / sense resistor | Driver reference voltage and resistor used to determine regulated phase current. |
| Microstep | A subdivided commanded coil-current position; not a guarantee of equivalent physical accuracy. |
| Homing | Bounded motion locating a physical reference before trusting absolute coordinates. |
| Soft limit | Software target constraint based on valid homing and measured usable travel. |
| Open loop | No continuous physical position feedback correcting or detecting all motion errors. |
| Backlash / compliance | Lost motion on reversal; elastic deflection under load. |
| Lead / pitch | Screw travel per revolution; spacing of adjacent threads. Multi-start screws distinguish them. |
| Accuracy / repeatability | Closeness to the intended position; consistency when repeating a procedure. |
| STOP / ABORT | Controlled stop after M08; immediate fault shutdown with position invalidation. |
| Lease / watchdog | Time-limited host authority; independent hardware reset mechanism for loss of software progress. |
| RTOS / task | Real-time operating system; a scheduled execution context with its own stack. |
| Queue / notification | Bounded message transfer; lightweight event/wakeup mechanism with defined delivery semantics. |
| CRC / commit marker | Accidental-corruption check; final indicator that a settings transaction finished. |
