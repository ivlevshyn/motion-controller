# Module 11 — FreeRTOS with clear ownership

**Starting point:** M10 complete and a tagged/identified working bare-metal baseline. Use the same project, hardware and protocol; RTOS adoption is a measured engineering experiment.

Read the [mentor contract](../mentor.md) before teaching. Each lesson below is one complete teaching unit. Deliver its explanation and implementation together; formal checks, questions and review happen at the end. Use [the evidence template](../evidence/README.md) for the submission. Every lesson also requires a clean applicable build, no unexplained new warnings, and no regressions in previously completed behavior.

## M11-L01 — Introduce tasks without moving pulse timing into the scheduler

**Prerequisite:** M10-L03. **Product change:** Foreground services become tasks while TIM1 retains electrical timing.

### Explanation and worked example

An RTOS schedules threads called tasks. Each task has its own stack; switching tasks changes CPU execution context, not the motor timer's waveform. A millisecond tick is suitable for service wakeups, not for generating precise STEP edges.

Keep one motion/controller owner. A communication task parses input and sends command values; a motion task owns state, queue and backend requests; a lower-priority diagnostics task reports snapshots. More tasks are not automatically more robust. Avoid a task for every GPIO.

Use static allocation where supported and size stacks from measurements plus margin. In FreeRTOS APIs, stack depth is commonly in StackType_t units rather than bytes; CMSIS wrappers may use different units. Verify the actual API. If SysTick becomes the RTOS tick, move the HAL timebase to reserved TIM5 using the appropriate generated configuration; do not allocate TIM1 or TIM2 again.

### Build it, in order

1. Preserve the bare-metal commit and write an ownership/task table before adding the kernel. Select a compatible STM32CubeF4/FreeRTOS version and record its exact API layer; avoid casually mixing CMSIS and native APIs.
2. Configure static task/queue storage and explicit priorities. Keep pulse output in TIM1 hardware and its bounded completion ISR.
3. Move serial service/parsing and motion state progression into their assigned tasks. Replace busy loops with blocking event waits or periodic delay-until semantics as appropriate.
4. Resolve HAL/RTOS timebase ownership with TIM5 and verify existing timeout behavior. Do not use HAL_Delay as a synchronization primitive between tasks.
5. Enable stack-overflow and assertion hooks, measure stack high-water marks under a basic workload, and keep hooks minimal with outputs disabled on fatal conditions.

### Troubleshooting

A task that never runs may be starved by a higher-priority loop that never blocks. Timing changed by a factor of two can indicate competing tick configuration. A stack allocation interpreted in the wrong units can either waste RAM or corrupt adjacent memory.

### End-of-lesson checks and acceptance

- The board boots disabled and existing basic commands work under the recorded RTOS configuration.
- Timer/timebase ownership is unique, and STEP generation does not depend on task tick timing.
- Task ownership, priorities, stack units/allocations and measured initial high-water marks are documented.

### End-of-lesson questions

1. Why should the RTOS tick not generate motor STEP pulses?
2. Why is one owner of controller state useful across multiple tasks?
3. Why must stack-size units be checked for the actual API layer?

### Submit for whole-lesson review

Submit the task table, kernel configuration, basic regression log and stack measurements. Reference the preserved bare-metal baseline SHA.

Commit the implementation and `learning/evidence/M11-L01.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M11-L02 — Transfer events safely across tasks and interrupts

**Prerequisite:** M11-L01. **Product change:** Queues and notifications replace unsynchronized shared state.

### Explanation and worked example

A queue copies a value when configured for that item size; passing a pointer copies only the pointer. Buffer lifetime still matters. A task notification is lightweight but may coalesce events depending on mode. Use it for a wakeup/fact only when lost multiplicity is acceptable or separately counted.

Only the kernel's `FromISR` APIs are allowed from eligible interrupts. On Cortex-M, lower numeric NVIC priority values represent higher urgency. Interrupts more urgent than the configured syscall threshold must not call FreeRTOS APIs. Check implemented priority bits and the distinction between shifted register values and library-format priorities in the selected port.

A mutex protects shared access and can provide priority inheritance; it does not make a long operation acceptable in the motion path. Prefer immutable snapshots and ownership transfer. STOP/ABORT still needs a bounded priority path that cannot be blocked by a full ordinary command queue.

### Build it, in order

1. Define value types for validated commands and controller snapshots. Transfer them through bounded static queues; retain one task as the controller owner.
2. Replace the bare-metal completion handoff with a documented notification/event mechanism. Handle completion-after-abort and operation-generation identity exactly as before.
3. Audit every ISR that calls a kernel API against NVIC priority and the FreeRTOS syscall threshold. Enable configASSERT where supported and explain the actual numeric settings.
4. Implement STOP/fault delivery with priority and bounded latency even when normal queues are full. Keep the immediate output-abort mechanism compatible with ISR/task ownership.
5. Test queue-full, notification-before-wait, stale completion and competing serial/diagnostic activity. Do not log from an ISR or hold a mutex while waiting for motion completion.

### Troubleshooting

An assertion only when an interrupt fires often indicates priority/API misuse. Intermittent corrupted commands often mean queued pointers outlive the parser buffer. Lost completion events may come from treating a coalescing notification as a counted queue.

### End-of-lesson checks and acceptance

- Every shared object has an owner or explicit synchronization rule, including DMA buffers and diagnostic snapshots.
- ISR priorities and FromISR usage satisfy the selected FreeRTOS port's rules.
- Full queues and stale notifications cannot prevent STOP or restart aborted work; race-oriented sequences pass.

### End-of-lesson questions

1. What is copied when a queue item is a pointer?
2. Why can a higher-urgency interrupt be forbidden from calling kernel APIs?
3. When is event coalescing safe, and when would it lose required information?

### Submit for whole-lesson review

Provide the shared-object/ISR priority audit, queue/notification tests and stress traces for stop and cancellation.

Commit the implementation and `learning/evidence/M11-L02.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M11-L03 — Compare RTOS and bare-metal behavior under load

**Prerequisite:** M11-L02. **Product change:** RTOS complexity is justified by measurements rather than appearance.

### Explanation and worked example

Concurrency adds scheduling choices, stack usage and failure modes. The meaningful comparison is whether the same product requirements still hold under the same workload. A task architecture can improve organization without improving electrical pulse timing, because TIM1 already supplies that timing.

Measure worst observed stop latency, block restart gap, serial loss, stack margin, CPU service time and total RAM/Flash. A watermark reports the unused stack pattern observed so far, not a proof of maximum future stack need. Include rare error paths and deep formatting calls in stress tests.

Watchdog feeding must now depend on progress of all required tasks. One healthy supervisor task cannot assume the others are healthy merely because it runs. Use per-service progress counters/deadlines and define which stalls should prevent feeding.

### Build it, in order

1. Reproduce M10's workload on both identified builds with the same clock, rates, logging policy and hardware. Store the comparison table.
2. Stress continuous serial traffic, four pending moves, repeated STATUS, STOP and a bounded diagnostics backlog. Verify edge totals and physical low-speed behavior.
3. Add task health reporting to the watchdog policy. Inject one stalled noncritical task and one stalled critical service with motor power disconnected; explain the intended difference.
4. Measure stack high-water marks and memory map after stress. Add justified margin without claiming an untested maximum.
5. Decide whether to keep the RTOS for the release. Either decision is acceptable if requirements and learning evidence are met; record the tradeoffs and maintain one primary configuration.

### Troubleshooting

Better average latency with a worse maximum can still violate the motion margin. A watchdog that resets during harmless logging backlog may classify optional diagnostics as critical; one that never resets after motion-task failure is too permissive.

### End-of-lesson checks and acceptance

- A reproducible before/after table covers timing, resources and functional results under matched workloads.
- Critical-task stalls cause disabled/reset recovery through the watchdog policy; optional work does not falsely establish system health.
- The chosen release architecture meets the tested operating envelope, with remaining block-gap limits disclosed.

### End-of-lesson questions

1. What does an RTOS add when hardware already generates pulses?
2. Why is a measured stack watermark not a mathematical maximum?
3. What evidence would justify returning to bare metal for this product?

### Submit for whole-lesson review

Submit comparative measurements, watchdog fault-injection results and a concise keep/revert decision with the final architecture SHA.

Commit the implementation and `learning/evidence/M11-L03.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.
