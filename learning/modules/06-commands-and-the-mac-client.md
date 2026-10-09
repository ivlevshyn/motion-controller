# Module 06 — Commands and the Mac client

**Starting point:** M05 complete. Bench motor only; no automatic carriage movement. Reuse the onboard serial link.

Read the [mentor contract](../mentor.md) before teaching. Each lesson below is one complete teaching unit. Deliver its explanation and implementation together; formal checks, questions and review happen at the end. Use [the evidence template](../evidence/README.md) for the submission. Every lesson also requires a clean applicable build, no unexplained new warnings, and no regressions in previously completed behavior.

## M06-L01 — Frame serial input without losing boundaries

**Prerequisite:** M05-L03. **Product change:** The controller accepts bounded complete lines and rejects overflow safely.

### Explanation and worked example

UART delivers a byte stream, not messages. One read can contain half a command or several commands. A line framer remembers bytes across reads until LF arrives. The maximum frame is 80 bytes including LF, with separate storage for NUL if a C string is needed.

On overflow, enter a discard-until-LF state. Executing a truncated prefix can turn malformed input into a valid but unintended move. Report one error for the discarded line, reset at its boundary and accept the next complete line. Normalize a single CR immediately before LF; do not silently strip arbitrary embedded control characters.

Start with interrupt reception into a fixed single-producer/single-consumer byte ring. The ISR owns the write index, foreground owns the read index; publish a byte before the new write index. Follow a documented ordering/critical-section scheme on this target. Ring overflow is distinct from an overlong command and should force resynchronization, not silently drop arbitrary bytes.

### Build it, in order

1. Implement and host-test the pure byte-to-line framer. Define behavior for LF, CRLF, empty lines, NUL and other disallowed control bytes.
2. Add a bounded RX ring with explicit full/empty detection. Explain why reserving one slot is one way to distinguish full from empty.
3. Configure receive interrupts using the actual installed HAL API; copy each received byte quickly and rearm reception as required. Handle UART error status without printing inside the ISR.
4. Drain the ring in foreground and pass complete lines to a placeholder dispatcher that initially supports STATUS only.
5. Keep TX bounded and foreground-only. Test fragmented and back-to-back input while a bench move is running.

### Troubleshooting

Receiving exactly one byte then stopping usually means the HAL receive operation was not rearmed. Garbled command boundaries suggest silent ring overflow or treating every read as a whole line. Distinguish UART hardware overrun from your ring's full condition.

### End-of-lesson checks and acceptance

- Tests cover all split points of a sample line, multiple lines per chunk, LF/CRLF, empty input, exact limit and limit+1.
- An overlong or overflow-corrupted line never executes a prefix; the next valid line is recoverable.
- RX interrupt work is bounded and motion remains responsive during command traffic.

### End-of-lesson questions

1. Why is a UART read boundary not a command boundary?
2. Why must overflow discard through the delimiter?
3. What is the difference between a UART overrun and a software ring overflow?

### Submit for whole-lesson review

Include framer/ring tests, actual serial captures and a precise capacity convention. Record the ISR/foreground ownership rule.

Commit the implementation and `learning/evidence/M06-L01.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M06-L02 — Parse and dispatch commands transactionally

**Prerequisite:** M06-L01. **Product change:** Validated commands control finite bench motion and report accepted versus completed.

### Explanation and worked example

Parsing has three separate jobs: recognize grammar, convert values and validate semantics. `atoi` cannot reliably distinguish invalid input from zero and gives inadequate range-error reporting. Use a checked conversion such as `strtol`/`strtoll` with `errno`, end-pointer and explicit target-range validation, or a carefully tested bounded decimal parser.

A request is a candidate until all checks pass. Do not update direction, queue or target while still parsing later arguments. A successful parse produces a command value; the controller then accepts or rejects it based on current state.

`OK accepted id=12` means the request entered execution/queue policy. `DONE id=12` means execution finished. A host timeout does not reveal which happened. Introduce an explicit sequence-ID syntax now, for example `12 MOVE_REL 200 50`, while keeping STOP available through a bounded priority path. Document exact grammar and duplicate policy.

### Build it, in order

1. Specify the grammar for STATUS, ENABLE, DISABLE, MOVE_REL and STOP, including token counts, whitespace, signed displacement and positive rate. Add IDs consistently to motion commands and responses.
2. Implement parsing as a pure function with no GPIO or controller mutation. Reject overflow, trailing garbage, unsupported commands, extra tokens and non-ASCII input.
3. Implement ENABLE/DISABLE as deliberate commands: healthy disabled→idle on enable, no automatic homing, and abort/queue clearing/position invalidation on disable. Dispatch a validated move only when the state and bench limits allow it. Initially reject another move while busy; queueing comes later.
4. Send accepted/completed/error responses with IDs. Implement a small last-ID policy that rejects duplicate accepted commands within the current boot session; communicate reset/session limitations to the client.
5. Make STOP bypass ordinary move admission. A host parser alone cannot replace the physical STOP input.

### Troubleshooting

A value like `12abc` being accepted means the conversion end-pointer was ignored. Duplicate motion after a timeout means acceptance and retry semantics are underspecified. Never solve uncertainty by blindly resending a relative move.

### End-of-lesson checks and acceptance

- Parser tests cover numeric extrema, extra/missing fields, negative rate, zero displacement and forbidden state combinations.
- Invalid requests leave state/output unchanged; duplicates do not cause replay in the documented session policy.
- Actual serial evidence shows accepted then DONE, rejection while busy and STOP during a move.

### End-of-lesson questions

1. Why is a parsed integer not yet a valid motion request?
2. Why can retrying a relative move be dangerous after a timeout?
3. What facts distinguish accepted, started and completed?

### Submit for whole-lesson review

Submit the protocol specification, table-driven parser tests and representative serial conversations with IDs.

Commit the implementation and `learning/evidence/M06-L02.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.

## M06-L03 — Build a small Swift host client

**Prerequisite:** M06-L02. **Product change:** A familiar-language Mac tool sends commands and records reproducible sessions.

### Explanation and worked example

Keep the embedded firmware in C; use Swift for the host because you already know it. A serial descriptor is still a byte stream. Foundation FileHandle and Darwin/POSIX serial configuration expose operating-system resources whose lifetime must be managed explicitly.

The host must configure the selected port for 115200, eight data bits, no parity, one stop bit and raw input without unintended line translation or hardware flow control. Read the APIs available in the installed Swift SDK; do not invent a portable serial package API. A small wrapper around `open`, `termios`, `poll`/nonblocking I/O and `close` is adequate. Explain timeout units and partial writes/reads.

Serial reading must continue while a command awaits DONE, and STOP must remain sendable. Give each operation a deadline and record the controller's boot/session identity. A reconnect is not evidence that a previous move completed; obtain STATUS and require deliberate recovery.

### Build it, in order

1. Choose the host-client directory now and record it in progress.md. Build a small Swift command-line program with explicit `--port` selection; never select an unrelated `/dev/cu.*` device automatically.
2. Encapsulate port open/configure/close and line buffering. Handle partial writes, EOF/disconnect and timeouts. Retain the terminal as a troubleshooting fallback, closed while the client runs.
3. Add status, enable, disable, bounded bench move and stop operations. Allocate IDs, wait for accepted/DONE separately, and do not automatically retry a timed-out move.
4. Save timestamped raw request/response logs plus a concise human summary. Keep host wall-clock timestamps separate from MCU timing measurements.
5. Run the same short test sequence several times, unplug/reconnect only the USB connection during an idle test, and demonstrate a clear disconnected/unknown-result state.

### Troubleshooting

Hanging reads usually indicate missing deadlines, canonical terminal mode or a read loop expecting the entire message at once. Lost STOP responsiveness suggests one blocking operation owns both UI and receive processing. Fix ownership rather than adding a second process on the port.

### End-of-lesson checks and acceptance

- The client builds on the actual Mac and executes STATUS and one finite move with logged acceptance/completion.
- Fragmented responses, timeout and idle disconnect produce clear bounded behavior without blind retries.
- Port settings and resource closure are explicit. The physical STOP remains usable independently of the client.

### End-of-lesson questions

1. Why must the host buffer partial lines just like the MCU?
2. What should the user be told when connection loss leaves completion uncertain?
3. Why is a Mac timestamp not a measurement of the timer's pulse width?

### Submit for whole-lesson review

Provide Swift build/run instructions, a sanitized session log and disconnect/timeout evidence. Explain the remaining uncertainty after reconnect.

Commit the implementation and `learning/evidence/M06-L03.md`, update the submission entry in [progress](../progress.md), and push. Supply the full SHA. The reviewer must assess every criterion above and the relevant surrounding code; evidence that cannot be inspected remains unverified.
