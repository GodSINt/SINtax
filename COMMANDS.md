# SINtax — Command Reference

Every statement the compiler knows, with the words it answers to.

This list was taken from the compiler's own keyword tables (`sintax.c`), not from
prose, and the forms shown were compiled to check they produce code. Where the
compiler has a trap, it is called out.

**Keep this file in step with the compiler.** An ability the language gains is
appended here in the same change that adds it, including anything found already
working and never written down. When the language is reshaped, the entries it
reshapes are corrected here — an entry that has quietly stopped being true is
worse than a missing one. Anything not yet compiled and run says so.

---

## How a line is read

| piece | meaning |
|---|---|
| `(name)` | you are **naming** something new |
| `{name}` | you are **referring** to something that already exists |
| `"text"` | the payload, or an attribute list |
| `key: value \| key: value` | attributes, separated by pipes |
| `#` | the rest of the line is a comment |

- **Word order is free.** `create a variable named (x) holding "1"`,
  `create (x) holding "1" as a variable` and `create variable "1" (x)` are the
  same statement.
- **Filler is ignored.** "please make me a variable called (x) that holds "1"
  and thanks" compiles. Only the pieces above are read.
- **Order of attributes is free**, and unknown attributes are kept, not dropped.
- **Blocks end with a word.** `if` → `end if`, `repeat` → `end repeat` (or
  `end loop`), `for each` → `end for`, `while` → `end while`, `action` →
  `end action`, `class` → `end class`, `box` → `end box`.
- **Every command answers to several words.** The synonym list is exact.

### Values

**Every value is quoted.** A quoted string that is a number becomes a number.

Words like *slot*, *field*, *called*, *holding*, *containing*, *storing* are
prose, not keywords — the compiler only needs `create` and the `(name)`. So
`make a slot named (x) holding "1"` works, but `declare a field (x)` does not:
`declare` is not one of its words.

```
create (n) holding "7"          # n is a number, 7
create (g) holding "hello"      # g is text
create (price) holding "99.9"   # works: the quotes make it a number
create (n) holding 7            # NO -- bare numbers do not compile
create (price) holding 99.9     # NO
```

**Every value interpolates.** A name in braces is what that name holds, whoever
is holding the value — `print`, `set`, `write`, an attribute, or a creation:

```
create (g) holding "hi"
create (copy) holding "{g}"     # copy holds hi
print "g is {g}"                # prints: g is hi
```

**Braces you want kept are escaped with apostrophes.** `'{g}'` is the word
`{g}`, not the value of `g`:

```
create (template) holding "'{g}'"    # template holds {g}
set {template} "'{g}'"               # still {g}
```

A brace that names nothing is left exactly as typed, which is why JSON, CSS
and HTML pass through a value untouched: `body { color: red }` keeps its braces
and `{"a": 1}` keeps its own.

### True and false

`true` / `yes` mean true; `false` / `no` mean false. In element attributes
`off` and `0` are also false, and an empty value is true.

### Traps the compiler will not warn you about

| trap | what happens |
|---|---|
| `create (n) holding 7` | does not compile — quote it: `holding "7"` |
| `create (copy) holding "{g}"` | stores the literal text `{g}`; use `set {copy} "{g}"` |
| `declare a field (x)` | the line is skipped (`declare` is not a word) |
| recording twice to one `output` | the second take deletes the first file |
| `if` / `repeat` / `box` without `end …` | the rest of the file is swallowed by the block |
| `create action (a)` with no `end action` | refused since 2026-11-01 — it used to swallow the rest of the file silently |

These four words work as values, in comparisons, and on their own:

```
create (flag) holding "yes"
if {flag} = true
while {flag}
set {feed} "record: yes"
```

---

## Contents

1. [Variables and values](#1-variables-and-values)
2. [Printing](#2-printing)
3. [If, else, end](#3-if-else-end)
4. [Loops](#4-loops)
5. [Actions, calls](#5-actions-calls)
6. [Classes and instances](#6-classes-and-instances)
7. [Windows and elements](#7-windows-and-elements)
8. [Element attributes](#8-element-attributes)
9. [Camera, video and recording](#9-camera-video-and-recording)
10. [Objects, lists, input](#10-objects-lists-input)
11. [Files](#11-files)
12. [Files as memory: map, seek](#12-files-as-memory-map-seek)
13. [Clock and time](#13-clock-and-time)
14. [Numbers, maths, secrets](#14-numbers-maths-secrets)
15. [Database](#15-database)
16. [Network](#16-network)
17. [Tensors](#17-tensors)
18. [Learning](#18-learning)
19. [Models and text](#19-models-and-text)
20. [Shell and system](#20-shell-and-system)
21. [Synonym index](#21-synonym-index)

---

## 1. Variables and values

```
create a variable named (greeting) holding "Hello"
create (status) holding "Active" as a variable
create item (mode) "Autonomous"
create variable "Armored" (integrity)
make a slot named (uptime) holding "99.9"
build a var called (rate) holding "0.1"
```

**Words:** `create` · `make` · `build` · `init` — with `variable` · `item` ·
`element` · `var` (and any prose you like around them)

Changing a value:

```
set {greeting} "Goodbye"
set {n} "5"
```

`set` also writes an element's attribute, either way round:

```
set {header.bgcolor} "#2411d4"
set "bgcolor: #2411d4" into {header}
```

**Words:** `set` · `assign` · `update` · `put` · `store`

---

## 2. Printing

```
print "hello"
print {greeting}
print "g is {greeting}"
```

A reference prints as its value; text in quotes is interpolated first.

**Words:** `print` · `display` · `show` · `output` · `echo` · `log`

> `show` is also the word that opens a window (`show window {w}`); the window
> form wins when a window is named.

---

## 3. If, else, end

```
if {n} = "2"
    print "two"
else
    print "not two"
end if
```

Comparisons: `=`, `is`, `is not`, `isn't`, `not =`, `>`, `<`, `>=`, `<=`,
`has` (text search). Truth words compare by meaning, so `{flag} = true` asks the
same question whether the flag holds `yes` or `true`.

```
if {name} has "bo"
if {n} > 4
if {flag}
```

**Words:** `if` · `when` for the test; `else` · `otherwise` for the branch.

Closing with `end` alone also works, but `end if` is the one to use inside
another `if`.

---

## 4. Loops

**Counted:**

```
repeat 200 times
    print "again"
end repeat

repeat {windows} times as (i)
    print "row {i}"
end repeat
```

**Words:** `loop` · `cycle` · `repeat` — with the word `times`.

**Over a list:**

```
for each (c) in {colors}
    print "c={c}"
end for
```

**Words:** `for` · `iterate` · `each`

**While:**

```
while {n} < 5
    add {n} and 1 into (n)
end while
```

**Words:** `while` · `during` · `until`

**Leaving early:**

```
break        # break · exit · stop
continue     # continue · skip · next
```

**A name made inside a loop is still there afterwards.** The braces of a loop do
not decide how long a name lives; the program does. `create variable (k)`,
`create matrix (m)`, `create list (l)` and anything an `into (name)` introduces
are declared at the top of the function that wrote them, and the line inside the
loop only fills them in:

```
repeat 200 times as (i)
    create matrix (batch) {width} by {width}
    fill {batch} with random normal 0.08 7
end repeat
print {batch}[0, 0]      # the matrix the last pass made
```

The value itself is made again on every pass, so what the name holds afterwards
is whatever the last pass left there. A name a loop only sometimes reaches keeps
the value from the pass that did reach it. The same holds inside an `if`: a name
made in a branch is still the program's name after `end if`.

---

## 5. Actions, calls

```
create action (roll)
    set {feed.record} "yes"
end action

do {roll}
```

**Words:** `action` · `function` · `routine` · `task` · `procedure` ·
`operation` · `method`

**Words for calling:** `do` · `run` · `call` · `execute` · `perform`

An action is the thing a button's `action:` attribute names, and it can also be
called directly with `do {name}`.

---

## 6. Classes and instances

```
create class (Hero) "name: bob"
end class

spawn {Hero} into (h)
set {h.name} "bob"
```

**Words:** `class` · `blueprint` · `template` · `schema`

**Words for making one:** `spawn` · `instantiate` · `new`
`spawn {Class} into (name)` names the instance.

```
inspect {h}                     # inspect · examine · check · debug · probe
```

---

## 7. Windows and elements

```
create window (main) "text: Title | size: 1000x800 | grid: 15x20 | bgcolor: #30303f"
create box (header) "bgcolor: #2411d4 | pad: 6 | rounded: 6"
create label (title) "text: Hello | color: #ffffff | fontsize: 32"
create button (go) "text: GO | bgcolor: #0ceb29 | action: do {roll}"
create textbox (name_box) "text: | width: 20"
create image (logo) "image: icon.png"
create link (reg) "text: Register Here | action: do {go_reg}"
create link (join) "text: discord.gg/H5k9UJ25Mn | url: https://discord.gg/H5k9UJ25Mn"
create dropdown (menu) "options: a; b; c"
```

| element | words it answers to |
|---|---|
| window | `window` · `stage` · `canvas` |
| box | `box` · `panel` · `deck` · `container` |
| label | `label` · `text_label` |
| text box | `textbox` · `field` · `input_box` |
| button | `button` · `btn` · `clickable` |
| image | `image` · `img` · `picture` |
| link | `link` · `anchor` |
| dropdown | `dropdown` · `select_menu` |
| camera view | `camview` · `camera_view` |
| video | `video` · `movie` |
| audio | `audio` · `sound` |

**Placing an element:**

```
nest {header} "parent: {main} | span: 1,1; 4,20"
nest {title} "parent: {header} | span: 2,1; 2,8"
```

`span` is `start_row, start_col; row_count, col_count` on the grid of whatever
holds the element.

A box given a `grid:` of its own is a page inside the page, and its children
count from 1 again:

```
create box (talk_box) "grid: 10x10 | bgcolor: #332f44"
nest {prompt} "parent: {talk_box} | span: 10,1; 10,8"   # its bottom row
```

A box given no grid is a window onto the grid it sits in, so `span: 8,1; 9,6`
inside it means the same rows and columns it means on the window — the panel is
where the page says it is, not a page of its own.

**Words:** `nest` · `mount` · `reparent` · `house`

**Anything that opens something:**

```
create link (join) "text: discord.gg/H5k9UJ25Mn | url: https://discord.gg/H5k9UJ25Mn"
create button (dj) "text: Join the SIN Nation | url: https://discord.gg/H5k9UJ25Mn"
create image (banner) "image: invite.png | url: https://discord.gg/H5k9UJ25Mn"
create label (line) "text: or read the invitation | url: https://example.com"
```

`url:` makes an element pressable without an action — the press is the address.
It works on a `link`, a `button`, an `image` and a `label`, so a button can open a
page instead of running a statement. A Discord invitation is handed to the
Discord app first (`discord://-/invite/<code>`, built from `discord.gg/<code>`,
`discord.com/invite/<code>` or `discordapp.com/invite/<code>`) and opens a
browser tab only when the app cannot take it. Every other address opens in the
browser. An `action:` of its own beats `url:`, so an element may open a page or
run a statement.

> `connect http "…"` asks the address for content and brings it back into the
> program. It does not open anything on the desktop. To open an app or a page,
> the words are `url:` on something pressable.

A **label that names a link** — `create label (or_lbl) "text: or {join}"` — is
pressable too, and its press goes where the link's goes. The link then is not
drawn a second time, so the words appear once.

**Showing and closing:**

```
show window {main}
destroy window {main}           # destroy · remove · kill · delete
```

```
destroy {box}                   # take one element off the page
set {box.hidden} "yes"          # or hide it, and "no" to bring it back
```

> `destroy` works on a window and on any element. A destroyed element is gone
from the page for good — a redraw does not bring it back, and its camera or
recording is closed on the way out. `set … hidden` is the reversible one.

Elements may also be nested with closing blocks (`create box … end box`), and
anything inside a box travels with it when the box is nested elsewhere.

---

## 8. Element attributes

Attributes are `key: value`, separated by `|`. These are the ones the compiler
recognises for elements:

`text` · `color` · `bgcolor` · `backgroundimage` · `bgimage` · `image` ·
`picture` · `backdrop` · `font` · `fontsize` · `grid` · `span` · `size` · `pad` ·
`rounded` · `border` · `shadow` · `align` · `wrap` · `opacity` · `visible` ·
`parent` · `action` · `url` · `source` · `record` · `output` · `file` · `audio` ·
`autoplay` · `loop` · `controls` · `volume` · `title` · `width` · `height`

`hidden` is the one that actually hides an element (`set {box.hidden} "yes"`);
`visible` is read on windows.

**Long text wraps.** Text in a `textbox`, an `input` and a `label` wraps inside
the room its span gives it, so a paragraph read from a file fills the box instead
of running off the edge. `wrap: no` keeps it on one line. An attribute is carried
in 8191 characters, so a text longer than that is cut — a page of prose fits, a
book does not.

```
read file "invite.txt" into (invitation)
create box (invite) "text: An Open Invitation | grid: 10x10"
create textbox (invite_text) "text: {invitation} | editable: no"
nest {invite_text} "parent: {invite} | span: 2,1; 9,10"
```

Reading one back:

```
print {header.bgcolor}
print {title.text}
print {feed.parent}
```

Setting one:

```
set {header.bgcolor} "#000000"
set {cam_feed.record} "yes"
```

---

## 9. Camera, video and recording

```
create camview (feed) "source: local | record: no | output: /tmp/take.mov | audio: yes"
```

- `record: yes` at creation records as soon as the window appears.
- `record: no` waits. Turn it on and off with `set`:

```
set {feed.record} "yes"
set {feed.record} "no"
set {feed.output} "/tmp/second.mov"     # retarget the next take
```

An `output` path may name a variable: `output: {clip}` or `output {clip}`.

A separate recorder (its own capture, not the one on screen):

```
create recorder (clip_one) "source: camera | audio: yes | output: /tmp/take.mov"
start recording {clip_one}
end recording {clip_one}
```

`source: camera` is a camera; `source: screen` records the screen.

**Words:** `recorder` · `record` · `recording` · `capture` with `start` ·
`begin` · `commence`, and `stop` · `halt`, or `end`.

> A take **replaces** the file at that path. Give each clip its own `output`
> when you want to keep more than the last one.
> Closing the window finishes an open recording before the window goes.

Video and audio elements:

```
create video (film) "source: clip.mov | autoplay: yes | loop: no | controls: yes | volume: 60"
```

---

## 10. Objects, lists, input

**Object** — named fields, dotted access, nesting:

```
create object (sinner) "username: unknown | title: guest | accepted: false"
set {sinner.username.title.accepted} "bob | admin | true"
print {sinner.title}            # admin
```

An object placed in a field is held, not copied:

```
create object (Boog) "species: Monkey"
create object (Goob) "colour: Grey"
set {Goob.friend} "{Boog}"
set {Boog.species} "Ape"
print {Goob.friend.species}     # Ape
```

**List** — pipe-separated at creation:

```
create list (colors) "red | green | blue"
for each (c) in {colors}
    print {c}
end for
```

**Words:** `list` · `array` · `collection`

**Input** — a place for a typed line:

```
create input (you) "Type something: "
listen (you)
print "you said {you}"
```

**Words:** `input` · `prompt` · `ask` for the name; `listen` · `hear` to fill it.

A field holds text like anything else, so `slice` reads what is typed in it
and `length {you} into (n)` is how many characters are in it right now.

**Slice** — a piece of text cut into a list, one item per piece:

```
create variable (doc) "The cat sat on the mat. A dog ran fast."
slice sentences {doc} (sents)
for each (s) in {sents}
    print {s}
end for
```

What is read comes from a named value in braces. Where it goes is `(name)`
and the list is made for you, or `{name}` and it must already be a list:

```
create list (words_list) ""
slice words {doc} into {words_list}
```

Calling a name replaces what it held — a slice does not add to an old list.

What is read is any name the language can resolve: a word, a field of an
object, a file mapped into memory, or an element's live text. An input gives
what is in it at that moment — inside an action as much as outside one:

```
create window (main) "text: NTM | size: 900x600 | grid: 16x20"
create input (prompt) "text: Speak Here"
nest {prompt} "parent: {main} | span: 10,1; 10,8"

create function (chop)
    slice words {prompt} into (words)      # what is typed, right now
    length {words} into (n)
    print {n}
end function
```

A window is not text: `slice words {main} (w)` says so instead of building
a program that fails at its first line. Slice `{main.title}` when the title
is the text you want.

| what you write | each item is |
|---|---|
| `slice {doc} (x)` or `slice lines {doc} (x)` | one line |
| `slice sentences {doc} (x)` | one sentence, ending included |
| `slice words {doc} (x)` | one word — and punctuation on its own, `...` kept whole |
| `slice characters {doc} (x)` | one character; `\r` and `\n` are dropped |
| `slice pos {doc} (x)` or `slice tags {doc} (x)` | `word\|tag`, e.g. `the\|det`, `cat\|n`, `mat\|?` |
| `slice svo {doc} (x)` or `slice triples {doc} (x)` | `(subject, verb, object)`, e.g. `(cat, sat, on the mat)` |
| `slice type {doc} (x)` or `slice mood {doc} (x)` | `declarative`, `interrogative`, `exclamatory`, `imperative`, `fragment` |

Measured on `"The cat sat on the mat. A dog ran fast."`: 1 line, 2
sentences, 12 items as words (10 words and 2 full stops), 39 characters,
10 tags, 2 triples, 2 types.

> The mode is chosen by the word between `slice` and `{`. Any word it does
> not recognise means **lines**, so `slice {doc} (x)` and `slice sense {doc}
> (x)` both give lines. Tags and triples assume English word order and lean
> on `lexicon.txt`; a word it has never seen comes back tagged `?`.

**Words:** `slice` · `split` · `chunk`

---

## 11. Files

```
write "out.txt" "hello"
append "out.txt" "more"
read "out.txt" into (text)
print {text}
```

**Words:** `write` · `save` · `store` · `dump` — `append` · `attach` ·
`push_file` — `read` · `load` · `fetch_file`

---

## 12. Files as memory: map, seek

```
map "big.txt" into (page)               # read-only
map "log.txt" into (log) | append       # writable from the end
map "draft.txt" into (buf) | writable   # writable from the start
length {page} into (size)
segment {page} from 0 for 40 into (head)
seek from {page} "<h1> | </h1>" into (headings)
seek all from {page} "<h1> | </h1>" into (all)
```

**Words:** `map` · `mmap` · `mount` · `bind` — `length` · `size` —
`segment` · `excerpt` — `seek`.

A `seek` delimiter is `opener | closer`, so the words between them come back.

---

## 13. Clock and time

```
create clock (now) "mode: normal | zone: local"
tick {now}

create timestamp (today) "mode: date | zone: local"
print {today.year}
print {today.month}
print {today.date}
print {today.day}
print {today.weekday_name}
print {today.days}
```

`set {today.offset} "1"` moves a month or a day on.

Pausing:

```
freeze 2                        # freeze · sleep · pause · wait · delay
```

---

## 14. Numbers, maths, secrets

```
add {a} and {b} into (sum)      # add · subtract · multiply · divide
quotient {a} and {b} into (q)   # quotient · intdiv
remainder {a} and {b} into (r)  # remainder · mod · modulo
scale {a} by 3 into (bigger)
```

`into (name)` names a new value; `into {name}` replaces an existing one.

**Secrets:**

```
create random (salt) "type: bytes | length: 16"
create masterkey (mk) "{password} | {salt}"
create handshake (hs) "{salt} | {mk}"
create filepath (weights) "model.gguf"
```

**Words:** `random` · `nonce` · `salt` — `masterkey` · `master_key` · `key` —
`handshake` · `digest` · `signature`.

---

## 15. Database

```
connect database "app.db"

create column (c1) "name: id | type: integer | args: pk; ai"
create column (c2) "name: username | type: varchar | args: uq; nn"
create table (sinners) "{c1} | {c2}"

insert into table {sinners} "username: bob | title: guest"
search table {sinners} "username: bob" into (found)
print {found.username}
print {found.count}
update table {sinners} "title: admin"
execute database "app.db" "VACUUM;"

close database
```

**Words:** `connect` · `open_db` — `close_db` · `disconnect` —
`table` · `schema` — `insert` — `search` · `query` · `select` · `find` —
`update` · `modify` · `patch` — `execute` · `raw_sql` · `sql` —
`column` · `field` — `close` · `finish` · `done` · `end`.

> `execute database "file.db" "SQL;"` takes the statement as the **last** quoted
> string; a file name comes first.

---

## 16. Network

```
fetch "https://example.com" into (answer)
transmit "https://example.com/notes" "weight: 42" into (thanks)
stream "https://example.com/frame.png" into "frame.png"
```

**Words:** `fetch` · `get` — `transmit` · `post` · `send` — `stream` ·
`download`

**A server:**

```
create server (api) "port: 8080"
repeat 100000 times
    accept {api} into (call)
    if {call.path} = "/hello"
        reply {call} with "hello"
    else
        set {call.status} "404"
        reply {call} with "no such path"
    end if
end repeat
```

**Shapes:**

```
parse {call.body} as json into (order)     # text -> object
render {order} as json into (answer)       # object -> text
convert {markup} as windows into (source)  # a page -> element statements
```

**Words:** `parse` — `render` — `convert` · `adopt` — `accept` · `receive` —
`reply` · `respond` — `server` · `service` · `endpoint`

> `parse … into (name)` names its own variable, and `render` / `convert` write
into one. Naming it first also works — an existing variable is pointed at the
text afterwards, rather than written through.

**Wire settings:**

```
use socket      # a plain socket instead of the usual transport
use secure      # TLS / curl transport
```

**Words:** `use` · `mode` · `set_mode`

---

## 17. Tensors

```
create matrix (a) 2 by 3
create vector (v) 3
create tensor (t) 2 by 2

fill {a} with 1
put 5 into {a} at 1, 2
print {a}[1, 2]
take {a}[0, 0] into (cell)
```

**Words:** `tensor` · `matrix` · `vector` · `grid` · `ndarray` — `fill` ·
`fill_with` — `take` · `peek`

**Arithmetic and shapes:**

```
add {a} and {b} into (c)            # subtract, multiply, divide
scale {a} by 3 into (e)
matmul {m} and {n} into (p)
dot {v} and {w} into (n)
transpose {a} into (at)
halve {a} at 2 into (l) and (r)     # a column: a number or a {name}
join {l} and {r} into (back)
clip {a} between -30 and 30 into (c)
norm {a} into (mag)
apply exp to {a} into (e)           # exp, log, sqrt, abs, tanh, relu, sin, cos…
```

**Reductions:**

```
sum {a} into (total)                # one number for everything
sum {a} across into (columns)       # one number per column
mean {a} down into (rows)           # one number per row
max {a} across into (peaks)         # also min, mean, argmax
```

**Words:** `sum` · `mean` · `average` · `minimum` · `maximum` · `min` · `max` ·
`argmax` — `matmul` · `mat_mul` · `matrix_multiply` — `dot` · `inner` —
`transpose` · `transposed` — `halve` · `bisect` — `join` · `concatenate` ·
`concat` — `clip` · `clamp` — `norm` · `magnitude` — `scale` — `sample` ·
`gamble`

**Where a product runs.** A `matmul` above a few million multiply-adds is sent
to the graphics card when the machine has one. Apple's GPUs have no double
precision, so the product travels as float and comes back as double: the answer
keeps about seven digits instead of sixteen, which a gradient does not notice
and a run trained through it reaches the same place. Below that size the round
trip through the card costs more than the arithmetic, so small products stay on
the processor -- a 128 x 128 x 128 product is four million and stays put.

```
SINTAX_GPU=off    ./myprogram   # every product on the processor
SINTAX_GPU=force  ./myprogram   # every product on the card, whatever its size
SINTAX_GPU=on     ./myprogram   # the same as saying nothing: the sizes decide
```

Any value that is not `off`/`no`/`false`/`0`/`cpu` or `force`/`always` means the
sizes decide.

The first time a product actually goes over, one line on stderr names the card
and the size where it starts. A program whose products are all small prints
nothing and never wakes the card. Nothing about this is written in the source:
`matmul` is the same word either way.

Measured here on the transformer shipped in `examples/`, at dim 256 and
context 128, one hundred training windows: **17.7 s on the processor, 3.0 s on
the card** -- 177 ms a window against 30 ms. Both learn, and the loss descends
the same way through both; the difference is that a pass over a 2.8 MB corpus
is about three hours instead of about twenty.

**Text and bytes:**

```
bytes {text} into (ids)             # text -> numbers
text {ids} into (back)              # numbers -> text
before {ids} into (all_but_last)
after {ids} into (all_but_first)
onehot {ids} and 256 into (x)
flatten {x} into (row)
sample {scores} down with heat 0.8 into (pick)
```

`bytes` takes a quoted string, a text variable, or a mapped file, and a quoted
string reads its braces like every other value does:

```
bytes "user: {prompt}" into (ids)          # the question, not the word
bytes "'{prompt}'" into (ids)              # the braces, kept
```

Pieces can still be built separately and joined; `join` puts two streams side by
side, and for a stream of byte numbers that is concatenation:

```
bytes "user: " into (lead)
bytes {prompt} into (question)
join {lead} and {question} into (seed)
```

Interpolation is the same everywhere: `print`, `set`, `write`, `bytes`,
attributes, `snprintf`, and a value being created. A name in braces becomes its
value; `'{name}'` keeps the braces; and a brace that names nothing stays exactly
as typed.

**Two things that bite, learned the hard way:**

`halve` splits at whatever the column holds, written or named. `halve {q} at
16` and `halve {q} at {half}` are the same instruction, so a width can be a
dial: change `(half)` once and every split in the file follows. What it cannot
take is arithmetic in the column — `at {dim} / 2` reports *halve needs a
column* and skips the line — so keep a `(half)` beside your `(dim)` and set
them together. A line that fails this way makes nothing, and the next line that
uses its halves fails too.

A tensor first made **inside** a `repeat` does not exist in the next `repeat`.
Two loops that run the same forward pass therefore cannot both say `into
(odds)` — the second one is a name the C compiler has never seen. Declare the
working tensors once, above both loops, and both can use them:

```
create matrix (odds) 1 by 1         # a place to put it, shaped by the first op

repeat 10 times as (i)
    matmul {x} and {w} into (odds)
end repeat

repeat 300 times as (i)
    matmul {x} and {w} into (odds)  # same name, and now it exists
end repeat
```

`examples/transformer.stx` is a whole program built on this: two heads of
causal self-attention, a feed-forward layer, attention written out of `matmul`,
`exp`, `sum` and `divide`, and a mask built with nested `repeat` and `put`.

---

## 18. Learning

```
create matrix (w) 2 by 1
fill {w} with random normal 0.5
learn {w}

repeat 200 times
    matmul {x} and {w} into (y)
    loss {y} against {wanted} into (l)
    train {l}
    step {w} by 0.05
end repeat

gradient {w} into (g)
save {w} to "brain.txt"
load {w} from "brain.txt"
```

`loss … as cross entropy` selects cross entropy instead of squared error.

**The path is a value too.** Both take a quoted path, and braces inside it are
read the way braces are read everywhere else, so a program can keep its
checkpoint wherever it likes:

```
create variable (brain) "examples/transformer.brain"
save {w} to "{brain}"
load {w} from "{brain}"
```

**One file, many tensors.** `save` reads what is already in the file, keeps
every tensor it does not recognise, and writes the one you named back — so a
model made of nine tensors is one checkpoint and nine lines, not nine files:

```
save {tok_emb} to "brain.txt"
save {pos_emb} to "brain.txt"
save {w_out} to "brain.txt"
```

`load` finds a tensor **by the name in the braces**, so those names are the keys
in the file: `load {pos_emb} from "brain.txt"` looks for `pos_emb` and nothing
else. Rename the variable and the tensor it used to be is invisible. A name with
no match is reported — *holds no tensor named pos_emb* — and the tensor stays
empty rather than stopping the program, so the mistake surfaces as a shape error
on the next `matmul`.

`examples/transformer.stx` writes such a checkpoint after training, and
`examples/transformer_talk.stx` loads it and answers: about thirty seconds to
train, three tenths of a second to use.

**Words:** `learn` · `teach` — `loss` · `error` · `misfit` — `train` ·
`backward` · `backprop` — `gradient` · `gradients` · `slope` — `descend` —
`step` · `tick` — `save` · `checkpoint` — `load` · `restore` · `reawaken`

---

## 19. Models and text

```
create model (Anima) "temperature: .7 | stops: end; user | template: user: {prompt}"

process {Anima} "hello there" into (reply)
translate {weights} "9707 847" into (words)   # ids -> words
inspect {Anima}

train model (cat) "the_cat_in_the_hat.txt"
```

**Words for ids:** `tokenize` · `tokenise` — `detokenize` · `detokenise` ·
`untokenize`

```
tokenize {weights} "Hello my friend." into (ids)
detokenize {weights} {ids} into (back)
```

> `translate` decodes ids when given a tokenizer path. Given a `model` it
answers the text the same way `process` does.

**Words:** `model` · `mind` — `process` · `infer` — `translate` · `decode` —
`tokenize` · `tokenise` — `detokenize` · `detokenise` · `untokenize` —
`train` · `backward`

---

## 20. Shell and system

```
pop program "open -a Calculator"
pop program "examples/anima_awake"
pop program {script_name}
```

**Words:** `pop` · `launch`

> `pop` runs the thing and does not wait. A `.stx` target is compiled first.

A name with no `/` in it means the file **beside the program** — `pop program
"calendar"` looks for `./calendar`, not for something on `PATH`. So a program
that is popped sits in the same folder as the one popping it. The child keeps
the folder it was started from, which is why two programs that both
`connect database "sss.db"` are reading and writing one file.

A link or a button pops another program through an ordinary action:

```
create link (cal) "text: Calendar | action: do {open_cal}"

create action (open_cal)
pop program "calendar"
end action
```

---

## 21. Synonym index

Every table in the compiler, in source order. Any word in a row may be used
wherever another word in the same row can.

| purpose | words |
|---|---|
| create | create, make, build, init |
| set | set, assign, update, put, store |
| nest | nest, mount, reparent, house |
| variable | variable, item, element, var |
| print | print, display, show, output, echo, log |
| action / function | function, action, routine, task, procedure, operation, method |
| end | end, close, finish, done |
| call | do, run, call, execute, perform |
| class | class, blueprint, template, schema |
| instance | spawn, instantiate, new |
| inspect | inspect, examine, check, debug, probe |
| if | if, when |
| else | else, otherwise |
| loop | loop, cycle, repeat |
| list | list, array, collection |
| input | input, prompt, ask |
| for each | for, iterate, each |
| while | while, during, until |
| break | break, exit, stop |
| continue | continue, skip, next |
| write file | write, save, store, dump |
| append file | append, attach, push_file |
| read file | read, load, fetch_file |
| listen | listen, hear |
| pop program | pop, launch |
| server | server, service, endpoint |
| parse | parse |
| render | render |
| convert | convert, adopt |
| accept | accept, receive |
| reply | reply, respond |
| slice | slice, split, chunk |
| tokenize | tokenize, tokenise |
| detokenize | detokenize, detokenise, untokenize |
| process prompt | process, infer |
| translate | translate, decode |
| learn | learn, teach |
| loss | loss, error, misfit |
| train | train, backward, backprop |
| gradient | gradient, gradients, slope |
| descend | descend |
| picture | image, bgimage, backgroundimage, picture, backdrop |
| take a cell | take, peek |
| length | length, size |
| seek | seek |
| onehot | onehot, one_hot, spread |
| bytes | bytes, encode |
| text from numbers | text |
| segment | segment, excerpt |
| flatten | flatten, unroll, straighten |
| save weights | save, checkpoint, store |
| load weights | load, restore, reawaken |
| transpose | transpose, transposed |
| apply | apply |
| clip | clip, clamp |
| halve | halve, bisect |
| join | join, concatenate, concat |
| norm | norm, magnitude |
| timestamp | timestamp, time, date |
| column | column, field |
| random | random, nonce, salt |
| masterkey | masterkey, master_key, key |
| handshake | handshake, digest, signature |
| freeze | freeze, sleep, pause, wait, delay |
| clock | clock, timer, ticker |
| tick | tick, step |
| connect db | connect, open_db |
| close db | close_db, disconnect |
| table | table, schema |
| insert | insert |
| update row | update, modify, patch |
| search | search, query, select, find |
| raw sql | execute, raw_sql, sql |
| wire mode | use, mode, set_mode |
| fetch | fetch, get |
| transmit | transmit, post, send |
| stream | stream, download |
| window | window, stage, canvas |
| box | box, panel, deck, container |
| label | label, text_label |
| text box | textbox, field, input_box |
| button | button, btn, clickable |
| image element | image, img, picture |
| link | link, anchor |
| dropdown | dropdown, select_menu |
| camera view | camview, camera_view |
| recorder | recorder, record, recording, capture |
| start recording | start, begin, commence |
| stop recording | stop, halt |
| video | video, movie |
| audio | audio, sound |
| destroy | destroy, remove, kill, delete |
| map file | map, mmap, mount, bind |
| tensor | tensor, matrix, vector, grid, ndarray |
| fill | fill, fill_with |
| scale | scale |
| matmul | matmul, mat_mul, matrix_multiply |
| dot | dot, inner |
| reduce | sum, mean, average, minimum, maximum, min, max, argmax |
| elementwise | add, subtract, multiply, divide |
| model | model, mind |
| remainder | remainder, mod, modulo |
| quotient | quotient, intdiv |
| sample | sample, gamble |
