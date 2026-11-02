# SINtax

A language that listens for keywords and compiles to a native program.

SINtax reads a statement by its shape, not its word order. You name new things in `( )`,
reference existing things in `{ }`, and write as much English around them as you like —
none of it matters. The compiler walks each line for the pieces it already knows, emits
ordinary C, and hands that to your system compiler.

```sin
create a variable named (greeting) holding "Hello, Sovereign World"
create window (main) "text: SIN Nation | size: 800x600 | grid: 15x20 | bgcolor: #30303f"
create label (hello) "text: {greeting} | fontsize: 18"
nest {hello} "parent: {main} | span: 3,4; 5,16"
show window {main}
```

That compiles to a real window. There is no runtime to install and no framework to learn
first — one binary reads your file and produces a program.

## What's in here

```
bin/sintax      the compiler (macOS, Apple Silicon)
sin/sin.stx     the SIN Nation example program — registration, login, the invitation
sin/            every image and text file sin.stx expects, beside it
COMMANDS.md     the command reference: every statement and every synonym it answers to
LICENSE.MD      the SIN Nation License (v1.1)
```

The compiler is shipped as a binary. The source is not in this repo — see the license.

## Running it

macOS on Apple Silicon today. Apple's Gatekeeper will block a downloaded binary until you
clear the quarantine flag, so run this once:

```sh
chmod +x bin/sintax
xattr -d com.apple.quarantine bin/sintax
```

Then compile the example and run it:

```sh
cd sin
../bin/sintax sin.stx
./sin
```

The compiler writes the program **next to your `.stx` file and names it after that file** —
`sin.stx` becomes `./sin`. There is no `-o`; that's by design, and it means a program always
lands beside the assets it needs.

First launch opens the landing window. Click the door, register a name and password, then
log in. The invitation is on the other side.

Nothing else is required: the database (`sss.db`) is created on first run, and deleted if
you want a clean slate. Starting fresh means deleting that file, nothing more.

If you would rather read the language before running anything, open `COMMANDS.md`. It is
the whole surface of the language in one file.

## Five things that make the language click

1. **`(name)` introduces something. `{name}` uses it.** `create window (main)` makes `main`;
   `show window {main}` calls it. Calling a name that was never named is an error, not a guess.
2. **Attributes are written `key: value`, separated by `|`.** `"text: OK | bgcolor: #e9cf0f | fontsize: 18"`.
3. **Order is free.** `create a variable named (x) holding "1"` and `create variable "1" (x)`
   are the same statement. So are most permutations you'd naturally type.
4. **The words around a statement are ignored.** Write comments inline, write whole sentences
   around your code — the compiler only looks for the pieces it knows. `#` silences the rest
   of a line.
5. **Elements carry their children.** A `box` with a header nested inside it moves as one piece,
   so a header you build once can be dropped into another window with a single `nest`.

## Words and synonyms

Every keyword answers to a family of synonyms, so a statement reads the way you'd say it.
`create`, `make`, `declare`, `spawn`, `forge` all create. `true` and `false`, `yes` and `no`
are interchangeable. `COMMANDS.md` lists the alternatives under each statement — that table
is the language, and it's the first place to look when a line seems to do nothing.

## Reading the example

`sin/sin.stx` is a working program, not a fragment, and it's meant to be read in order:

- **The database** — columns, a table, and a `connect`/`close` around each block.
- **The gate** — a landing window with a door that leads to login, and a link between login
  and registration.
- **The registry** — a password never stored as typed: a random salt and a handshake are
  what land in the table.
- **The invitation** — read from `invite.txt` into a variable and shown in a box, with a
  button whose `url:` opens Discord.

Every asset it names sits beside it. Change one, recompile, and the program changes.

## License

**The SIN Nation License (v1.1)** — see [LICENSE.MD](LICENSE.MD).

Personal, educational and non-profit use is free: you may study it, build on it, and pass it
on. Commercial use is not granted by this license — including selling it, wrapping it in a
paid product or service, or using it to train a proprietary model. Commercial use is arranged
by written agreement with the author.

## Questions, or something that isn't possible yet

The invitation inside the program leads to the SIN Nation Discord — that's where the author
is, anytime, for questions about what you're building or just conversation.

~GodSINt~
