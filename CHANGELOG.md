# Changelog

What each build of the compiler changed. Newest first.

The compiler ships as a binary, so this file is the only way to see what a
particular `bin/sintax` knows. If a line here is missing from the binary you
have, that build predates it.

---

## 4 November 2026 — current

### The compiler

- **`bytes` writes as well as reads.** `bytes {ids} to "tone.wav"` takes a
  tensor of numbers and writes one byte each, so a program can put binary back
  on disk. There was a door in and no door out: `write` carries text, and a C
  string stops at the first zero byte — which is most of a WAV header and all of
  a stretch of silence. A file read with `map` and turned into numbers by
  `bytes` now comes back out byte for byte.
- A number outside 0..255 is rounded and held at the edge rather than wrapping,
  the same way `text` holds it. The path reads braces, so takes can number
  themselves: `bytes {ids} to "take_{n}.wav"`.

### The reference

- `COMMANDS.md` documents the outlet under *Text and bytes*, with the arithmetic
  for taking a 16-bit sample apart and putting it back.

---

## 3 November 2026

### The compiler

- **A name made inside a loop now exists after the loop.** `repeat`, `while`,
  `for each` and `if` bodies are C blocks, and a name declared inside one died
  with the closing brace. Declarations are now lifted to the top of the action
  that wrote them, while the value is still made on the line, on the pass, you
  wrote it — so a loop that reads a count still reads it every pass. What the
  name holds afterwards is whatever the last pass left there.
- **`nest` keeps the whole parent name.** A parent was stored into
  `sizeof(char *)` bytes, so any name longer than seven characters was cut
  (`talk_box` became `talk_bo`) and the element was silently never drawn.
  Reparenting into a longer name could also overrun the old buffer.
- **A box with no `grid:` of its own inherits the grid it sits in.** It used to
  become a 1×1 grid, which put every child at a `span` far outside the box —
  nothing inside it appeared.
- **`slice` reads an element's live text, inside an action or out.**
  `slice words {prompt}` compiled to a C name that never existed, so a field
  could not be sliced anywhere; a variable worked, which made it look like an
  action-only problem. A window is refused with a message instead of compiling
  into a crash.
- **`length` counts characters** for text, a field, or a list. It assumed every
  name was a tensor and emitted a tensor count that did not compile.
- **No fixed ceilings.** Source lines and string literals grow as needed; the
  old limits that cut long lines and long strings are gone.
- **Tensor math runs on the GPU** through Metal, with Accelerate on the CPU
  side.

### The example

- `sin.stx` no longer writes the masterkey onto the `sinner` object. The salt
  and the handshake stay in the table; the object carries identity only.
- Stray characters after `destroy window {landing}` in `login_good_ok` removed.

### The reference

- `COMMANDS.md` documents the loop rule, the grid inheritance, and what `slice`
  and `length` accept as a source.

---

## 1 November 2026 — first release

- The compiler, and the SIN Nation example: landing, registration, login, the
  invitation in `invite.txt`, and the button that opens Discord.
- `COMMANDS.md`, the SIN Nation License, and a `.gitignore` written as a
  manifest — nothing reaches the repo unless it is named there.
