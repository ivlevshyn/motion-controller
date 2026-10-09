# Module 02 — C for firmware

**Starting point:** M01 complete; Cart A. Teach C through controller data and diagnostics, not disconnected exercises.

Read the [mentor contract](../mentor.md) before teaching. Each lesson below is one complete teaching unit. Deliver its explanation and implementation together; formal checks, questions and review happen at the end. Use [the evidence template](../evidence/README.md) for the submission. Every lesson also requires a clean applicable build, no unexplained new warnings, and no regressions in previously completed behavior.

## M02-L01 — Represent quantities without overflow

**Prerequisite:** M01-L03. **Product change:** Controller limits use explicit units and validated numeric values.

### Explanation and worked example

C integers have a width, signedness and finite range. Java also has finite integers, but C signed overflow is undefined behavior: the compiler can optimize on the assumption that it never happens. Swift's normal checked arithmetic does not describe ordinary C behavior. Use `<stdint.h>`, `<stdbool.h>` and `<limits.h>` deliberately.

A step target can be negative, a rate cannot. Names such as `rate_steps_s` make units visible. An enum represents a small set of states; it does not automatically reject every other integer.
```c
int64_t delta = (int64_t)target_steps - (int64_t)position_steps;
```
The widening must happen before subtraction. Casting an already-overflowed result is too late. Likewise, `abs(INT32_MIN)` cannot be represented by `int32_t`. Integer division truncates, so convert with a documented rounding policy rather than accidental order of operations.

### Build it, in order

1. Introduce a small controller configuration with minimum/maximum rate, maximum bench move and a state enum. Set conservative initial limits from design-contract.md.
2. Write pure functions that validate a requested signed displacement and unsigned rate. Return a boolean or a small result enum; invalid input must not partially change configuration.
3. Calculate target differences in 64 bits and range-check before narrowing. Explain decimal, hexadecimal constants and the difference between `=` and `==`.
4. Build a tiny host C executable with assertions for these pure functions, using Apple Clang and `-std=c17 -Wall -Wextra -Wpedantic`. This introduces fast checks, not a large test framework.
5. Enable the agreed warnings for application code in the embedded build. Read warnings before changing types or adding casts; a cast can conceal a bug.

### Troubleshooting

Unexpected huge values often come from converting a negative number to unsigned or mixing signed/unsigned comparisons. Do not silence such warnings with arbitrary casts. Host `long` width need not match the MCU; rely on explicit-width fields where the representation matters.

### End-of-lesson checks and acceptance

- Minimum, maximum, zero, negative and just-outside-limit cases are checked without undefined signed overflow.
- Tests include INT32_MIN and INT32_MAX target/position combinations.
- Every numeric setting states its unit; invalid settings leave the previous state unchanged.

### End-of-lesson questions

1. Why must widening occur before subtraction?
2. When would `uint32_t` be a poor choice for position?
3. What is the difference between a compiler warning fix and a warning-suppressing cast?

### Submit for whole-lesson review

Include the configuration, pure validation code, exact host command and actual test output. Explain one overflow bug you prevented.

Commit the implementation and `learning/evidence/M02-L01.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M02-L02 — Understand pointers, storage and module boundaries

**Prerequisite:** M02-L01. **Product change:** Controller state has explicit ownership and a small C interface.

### Explanation and worked example

A C pointer is an address with a pointed-to type. It is not a Java reference with garbage collection or a Swift reference protected by ARC. It can refer to expired stack storage, the wrong type or too few elements. Passing a pointer does not pass the array length.
```c
typedef struct {
    int32_t position_steps;
    bool position_valid;
} controller_t;

void controller_init(controller_t *controller);
```
`controller->position_steps` accesses a member through a pointer. `&object` obtains an address. A local struct normally has automatic lifetime; a file-static object survives for the program's lifetime. `static` on a file-level function instead limits its linkage. Explain these distinct meanings.

An output buffer API needs both pointer and capacity. `const controller_t *` permits reading through that pointer but does not guarantee immutable storage or synchronization. Start with one foreground owner of mutable controller state; interrupts arrive later.

### Build it, in order

1. Create a controller header and implementation at paths chosen for the actual generated project. Put public types/declarations in the header, definitions in the source and private helpers at file scope with static.
2. Initialize every member explicitly through `controller_init`. Contrast zero initialization of static storage with uninitialized automatic variables.
3. Add a status-copy function whose caller supplies storage. Document whether null pointers are programmer errors or checked input; apply the policy consistently.
4. Draw a small table of important objects: owner, lifetime and who may write them. Include controller state, HAL handles and temporary formatting buffers.
5. Add include guards, compile from another source file and test repeated initialization on the host. Remove duplicated global definitions rather than patching linker errors.

### Troubleshooting

Multiple-definition errors often mean a header defines storage rather than declaring it. Returning a pointer to a local array leaves a dangling pointer even if it appears to work. Avoid heap allocation while these lifetimes can be static or caller-owned.

### End-of-lesson checks and acceptance

- Two translation units can include the header and link without duplicated storage.
- No pointer escapes the lifetime of its object; capacity is explicit for any buffer API.
- Controller initialization establishes an unhomed, disabled state with position invalid; host tests inspect all public fields.

### End-of-lesson questions

1. How does a pointer differ from a Swift reference in lifetime guarantees?
2. Why can a header declaration appear in many files but a global definition usually cannot?
3. What does `const` promise here, and what does it not promise?

### Submit for whole-lesson review

Submit the interface, implementation, ownership table and host test results. Identify any generated-code integration points.

Commit the implementation and `learning/evidence/M02-L02.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M02-L03 — Produce bounded serial diagnostics

**Prerequisite:** M02-L02. **Product change:** The controller describes its state over the onboard serial connection.

### Explanation and worked example

UART serializes bytes at an agreed rate and framing. At 115200 8N1 each byte occupies about ten bit times: about 86.8 microseconds before additional software overhead. A 100-byte message therefore takes roughly 8.7 ms on the wire. A blocking diagnostic call can already be long compared with a future motor deadline.

An array is contiguous storage indexed from zero, without automatic bounds checks. `sizeof array` measures the entire array only where it still has array type; an array parameter behaves as a pointer, so pass capacity separately. Explain indexing and a bounded loop before using them. C strings end at a NUL byte; an arbitrary received byte array is not automatically a string. For `snprintf`, the return value is the length that would have been written, excluding NUL. A negative result indicates failure; a value at least the capacity indicates truncation. Check the signed result before converting it to a size. Use `<inttypes.h>` format macros or verified matching types for fixed-width values; wrong variadic format types are not made safe by the compiler in every case.

### Build it, in order

1. Configure USART2 PA2/PA3 at 115200 8N1 with the onboard ST-LINK virtual COM connection. Identify and record the current Mac serial device.
2. Open one serial terminal with matching settings. Never let two programs compete for the same port. Emit a short boot banner and one formatted controller status line.
3. Wrap status formatting in a capacity-aware pure function and check `snprintf` results. Keep transmission separate from formatting so host tests can exercise it.
4. Use bounded blocking transmission only in the foreground at this stage. Set a timeout and handle failure. Do not log continuously every loop iteration.
5. Add a once-per-second status report alongside the nonblocking LED, then allow it to be disabled for later timing experiments.

### Troubleshooting

Garbled characters usually suggest baud/clock mismatch, wrong port or terminal settings. A disconnected terminal does not necessarily make UART transmission fail: the MCU cannot infer that the host consumed bytes. Missing NUL termination is a memory bug, not a terminal problem.

### End-of-lesson checks and acceptance

- Reset emits a readable banner and correct state on the actual Mac.
- Exact-fit, one-byte-too-small and tiny-buffer formatting tests prove bounded output and explicit truncation reporting.
- Formatting and transmission are separated, and logging is bounded in frequency and timeout.

### End-of-lesson questions

1. How long does an 80-byte line occupy a 115200 8N1 link approximately?
2. Why is successful UART transmission not proof that the host read the message?
3. What does a `snprintf` result larger than the buffer mean?

### Submit for whole-lesson review

Include an actual serial capture, buffer-boundary test results and the chosen port-opening procedure. State that this lesson's blocking UART is temporary.

Commit the implementation and `learning/evidence/M02-L03.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.
