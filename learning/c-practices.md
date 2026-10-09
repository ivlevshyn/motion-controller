# C practices used by this course

These rules are introduced when relevant. They are not prerequisites to memorize.

## Language and builds

Use C17 for application C where the selected GCC version supports it. Keep vendor-generated code and library conventions intact. Enable `-Wall -Wextra` first; progressively add `-Wpedantic -Wconversion -Wshadow -Wundef -Wstrict-prototypes -Wformat=2` for application code. Do not suppress warnings globally to make a vendor package quiet. Add stronger warning policy only after understanding findings. Commit the `.ioc`, application source, linker script and reproducible dependency/version record. Ignore generated binaries and personal IDE caches.

## Values and memory

Use `<stdint.h>` types for hardware quantities, `<stdbool.h>` for flags, and `size_t` for capacities/indexes. Include units in names: `steps_per_s`, `position_steps`, `timeout_ms`. Check ranges before narrowing casts. Unsigned wrap is defined; signed overflow is not. Avoid computing `abs(INT32_MIN)`; widen first. For time elapsed use `(uint32_t)(now - then)` with intervals much less than one wrap, not `now >= then + interval`.

Begin without dynamic allocation. A bounded queue still needs an explicit full policy. A static buffer still needs bounds checks. Never return a pointer to a local array. Every interface using a buffer states its length, owner and valid lifetime. `const` describes allowed modification through that interface; it is not synchronization.

A pointer is an address plus a type in the compiler's model. It is not a Java reference or a Swift ARC-managed reference. Passing a pointer neither copies nor extends an object's lifetime. Teach `.`, `->`, `&` and `*` through actual controller data.

## Interfaces and modules

Separate pure calculations/state transitions from pin/timer/serial operations as they become substantial. Prefer small C structs, enums and explicit function arguments. Avoid a global 'everything' object. Functions report invalid arguments and busy/fault conditions. Headers contain declarations and interface contracts; implementation details and internal helpers use file-local `static`.

Use descriptive names over clever macros. Parenthesize necessary function-like macros, avoid repeated side effects, and prefer inline functions when types matter. Do not pack structs merely to save a few bytes. Never transmit or store raw structs as a permanent binary format: padding, endian order and versioning matter.

## Errors and real-time behavior

No silent HAL return-code discard in application logic. Distinguish rejectable commands from latched faults. The controller boots disabled and position unknown. A reset must never resume queued motion. Log faults outside timing-critical interrupt paths. No `printf`, blocking serial transmission, heap allocation, sleep or Flash erase in a pulse/limit ISR.

`volatile` is appropriate for peripheral registers and some externally changed observations; it does not make increment, multi-field snapshots or queues atomic. Use short critical sections, verified atomics or RTOS primitives appropriate to context. Save and restore the previous interrupt mask; do not blindly enable interrupts that were already disabled by a caller.

One owner modifies motion state. ISRs communicate completion/fault facts through narrowly defined mechanisms. Document which shared values are read and written in each context. FreeRTOS `FromISR` functions have priority constraints; a very urgent IRQ cannot call them just because its name ends in `FromISR`.

## Tests and evidence

Test pure logic on the Mac with the host compiler; use hardware integration tests for actual peripherals. Host sanitizers can reveal many C errors but do not model MCU registers or all timing. Keep failure cases: zero moves, boundaries, overflow, queue full, malformed input, switch stuck, communication loss and corrupted settings.

Define pass criteria before measuring. A command acknowledgement is not completed motion. A pulse count is not measured carriage position. Distinguish nominal timer resolution, measured timing, physical repeatability and absolute accuracy.
