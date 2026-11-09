# Changelog

## v1.22.3 (2026) — Patch

### `frankiec fmt` no longer eats your comments

- **`frankiec fmt --write` stripped every comment from a file.** The formatter is AST-based and the lexer discarded comments, so they were lost on rewrite. Comments are now recorded on the side and re-attached to the statement they belong to (full-line comments above it, trailing comments after it). Formatting is idempotent.
- **`frankiec fmt` could drop parentheses and change the meaning of an expression** (e.g. `(a + b) * c`, or range bounds like `(n - 1)..m`). It now uses a precedence table and keeps the parentheses that matter.
- **Stitch doc comments restored** in `frankiegame` and `frankiecanvas` (`##` comments consumed by `frankiec docs`).

### New: the Frankie Playground 🌐

Try Frankie in the browser with nothing to install. See [The Playground](../getting_started/playground.md).

No new syntax, no stdlib/stitch signature changes — the language freeze for The Book of Frankie stays in effect.

---

## v1.22.2 (2026) — Patch

### Bug fixes found while writing The Book of Frankie

- **`app.static("./dir")` (1-arg, root-mounted form) was shadowing every dynamic route with a 404.** When a request didn't match a static file, the dispatcher returned 404 immediately instead of falling through to check regular routes. Fixed: the 2-arg, explicit-prefix form (`app.static("/static", "./dir")`) still 404s outright since it owns that URL space exclusively, but the root-mounted 1-arg form now falls through to routes when no static file matches.
- **Assigning to a keyword-shaped identifier crashed at runtime.** `import = "value"` (and compound assignment to a keyword-shaped name, e.g. `class += 1`) generated invalid Python, since `gen_assign`/`gen_compound_assign` weren't running the identifier through the same `_py_safe()` mangling used everywhere else in codegen. Fixed — both now mangle consistently.
- **Homebrew tap was tagged `v1.22` instead of `v1.22.0`**, inconsistent with every other release tag and the `FRANKIE_VERSION` constant. Retagged, and the formula/README now point at the correct tarball.

No new syntax, no stdlib/stitch signature changes — the language freeze for The Book of Frankie stays in effect.

---

## v1.22.0 (2026) — "More FORTRAN" 🧮

### One last feature pass before the freeze locks back in

The freeze briefly lifted from v1.21.0 for a single, deliberate, final round leaning harder into Frankie's FORTRAN heritage — real matrix math, complex numbers, math intrinsics, and labeled loops. This is the last planned feature release before the book ships; only bug fixes and doc corrections from here.

---

### New Stdlib

**Matrix math**

- Arithmetic (`+`, `-`, `/`, `%`, `**`) on vectors now recurses through nested vectors at every depth, so a matrix (a vector of vectors) does genuine element-wise math instead of silently concatenating rows. `[[1,2],[3,4]] + [[5,6],[7,8]]` → `[[6,8],[10,12]]`. Plain (non-nested) vector arithmetic is unchanged.
- `matrix_new(rows, cols, fill = 0)`, `matrix_identity(n)`, `matrix_shape(m)`, `matrix_transpose(m)`, `matrix_scale(m, n)` (element-wise scalar multiply — `*` stays as vector-repeat, unchanged), and `matrix_multiply(m1, m2)` (real row·column multiplication, raises on shape mismatch).

**Complex numbers**

- `complex_new(real, imag = 0)` builds a native Python `complex` — `+`, `-`, `*`, `/`, `**` all work for free with zero changes to arithmetic codegen. `complex_real`, `complex_imag`, `complex_conj`, `complex_abs` round out the accessors.
- Printed in conventional math notation (`3+4i`), not Python's electrical-engineering `3+4j`.

**Math intrinsics**

- `sin`, `cos`, `tan`, `asin`, `acos`, `atan`, `atan2`, `exp`, `log` (natural or any base), `log10`, `log2`, `factorial`, `gcd`, `lcm`, and the constants `PI`/`E`.

**Labeled loops**

- `loop`, `while`, `until`, and `for` can carry a `:name` label right after the keyword. `break :name` / `next :name` target that exact loop from anywhere nested inside it — including through other unlabeled loops in between — solving the classic "break out of two loops at once" problem without extra flag variables.
- `:name` reuses Frankie's existing symbol-literal syntax, but as of this release it's tokenized separately from plain strings (previously `:name` and `"name"` produced an identical token). This is what makes `break :label` unambiguous from `break "a string value"` — see Fixes below.

---

### Fixes

- **`:symbol` and `"string"` were the same token.** `:name` has always been valid Frankie syntax (used as hash values, `enum`-style constants, etc.) but the lexer collapsed it straight into a plain string token, indistinguishable from `"name"`. This surfaced while building labeled loops — `break :label` and `break "some string"` were literally unparseable-apart. Fixed by giving `:name` its own `SYMBOL` token that still parses to the exact same value everywhere it was already used, so this is invisible unless you're doing something new like labeled loops.
- **`frankie_fmt.py` silently dropped loop labels.** Before this was caught, running `frankiec fmt --write` on a file using labeled loops stripped every label from `loop`/`while`/`until`/`for`/`break`/`next`, which can silently turn a correctly-terminating labeled break into an infinite loop. Fixed — the formatter now round-trips labels losslessly (verified idempotent: format → run → identical output).
- **`vector.md` and `math_statistics.md` had an inaccurate example.** Docs claimed `v * 2` on a vector scales every element (`[2,4,6,8,10]`); it actually repeats the vector Python-list-style (`[1,2,3,4,5,1,2,3,4,5]`). This was already true before v1.22 — just never caught until writing the new Matrix docs alongside it. Corrected, and `matrix_scale`/manual addition are now documented as the way to get true scalar scaling.

---

### 🧊 Language Freeze

The freeze moved from v1.21.0 to v1.22.0 for this one release, per Blag's explicit decision — "the last big addition" before the book ships. v1.22.0 is now the frozen version: grammar, stdlib signatures, and stitch APIs are locked again, patch releases (bug fixes, doc corrections) continue as normal.

---

## v1.21.0 (2026) — "The Book Edition" 🧊

### Image sprites, WebAudio synth, and a language freeze

---

### New Stdlib

**Image sprites in `frankiecanvas`**

- `load_image(path)` reads a PNG/JPEG/GIF file and returns `{src: "data:image/...;base64,..."}`. Call it once before `run_game`, not per frame — it hits disk.
- `draw_image(g, x, y, image, w: 1, h: 1)` places the image at cell `(x, y)`, sized `w × h` cells. Every connected browser decodes and caches the image the first time it's drawn; later frames just resend coordinates.
- `frankiegame` (terminal) gained matching `load_image`/`draw_image` calls for API parity — since a tty can't show real pixels, `draw_image` there renders a `▒` placeholder block instead. The same function calls compile against either engine.

**WebAudio synth API**

- `synth_play(g, freq: 440, wave: "sine", dur: 0.08, gain: 0.06)` plays a real oscillator tone in every connected browser. `wave` is `"sine"`, `"square"`, `"sawtooth"`, or `"triangle"`.
- `game_beep(g)` is now a thin wrapper over `synth_play` (520Hz square, 0.07s) — fully backward compatible.
- `frankiegame` (terminal) gained `synth_play` too, mapped to the terminal bell — parameters are accepted but ignored, so cross-engine code still compiles.

**`file_read_base64(path)`**

- Binary-safe file read + Base64 encode in one call. `file_read` is text-mode and corrupts binary data (images, audio, zip files); `load_image` uses `file_read_base64` internally. Generally useful for embedding any binary file as text.

---

### Fixes

- **Data-loss bug in `frankiec fmt`** — a single-statement block whose body was an assignment or a postfix `if`/`unless` (e.g. `game["shots"].each do |s| hit = true if collide?(...) end`) silently formatted to `do |s| nil end`, discarding the logic. `_is_stmt_only()` didn't recognize `Assign`, `PostfixIf`, and several other statement node types. Fixed by falling back to the safe multi-line block form for any statement kind that can't be inlined. If you ran `frankiec fmt --write` on code with this pattern before v1.21, it's worth a diff review.

---

### 🧊 Language Freeze

v1.21 is the last planned feature release for a while — Frankie is language-frozen while "The Book of Frankie" is written against this exact version. Grammar, stdlib signatures, and stitch APIs are locked; patch releases (bug fixes, doc corrections) continue as normal, but no new syntax or behavior changes until the freeze lifts.

---

### Compatibility notes

Fully backward compatible. `game_beep` behaves identically to v1.20. No existing function signatures changed — `synth_play` and `draw_image` are additive on both stitches.

---

## v1.20.1 (2026)

### Patch: Arrow keys, `frankiec examples`, Homebrew

---

- **Arrow keys work in terminal games.** `term_key()` was reading from Python's buffered stdin, which swallowed the tail of arrow-key escape sequences — every arrow decoded as `esc`. It now reads raw bytes from the file descriptor. Snake finally obeys.
- **`frankiec examples [name]`** — list the bundled examples and showcase games, or copy one into the current directory ready to run (`frankiec examples snake && cd snake && frankiec run main.fk`). Works for brew, git-clone, and install.py installs alike.
- **Homebrew support** — official formula in `packaging/homebrew/`: `brew tap atejada/frankie && brew trust atejada/frankie && brew install frankie`. Standard stitches now also resolve from the installation directory, so brew installs work out of the box.

---

## v1.20.0 (2026)

### Theme: "It's Alive… and Playing" — two game engines, one shared API

---

### New Language Features

**The shared engine API**

- `game_new(width:, height:, fps:)`, `on_key`, `on_any_key`, `on_tick`, `run_game`, `stop_game`, `clear`, `draw`, `draw_sprite`, `text`, `collide?` (AABB), `render_frame`, `game_beep`
- Write a game once against this API — the `stitch` line picks the renderer
- Also: `max_ticks:` to bound the loop for CI-safe, headless test runs

**`frankiegame` — terminal renderer**

- Raw keyboard mode (no Enter needed) via stdlib `termios`; arrows map to `"up"/"down"/"left"/"right"`; `q` quits by default
- Double-buffered ANSI rendering — one write per frame, flicker-free
- Headless-safe: without a tty, input and rendering no-op, so test suites can play the game via `max_ticks` + `render_frame`
- New stdlib TUI primitives: `term_raw_on/off`, `term_key`, `term_size`, `term_hide_cursor`/`term_show_cursor`, `term_clear`, `term_render`, `clock_ms`, `beep`

**`frankiecanvas` — browser renderer**

- Same API, real pixels: an HTML5 canvas page served from Frankie's own web server, draw ops streamed over WebSockets (~20–30 fps)
- Key and mouse events stream back with the same key names as the terminal; mouse clicks arrive as `"click:x,y"`
- `rect(g, x, y, w, h, color)`, `on_player_key` (every browser tab is a player), `players_count(g)` — multiplayer costs zero extra code, built on the v1.18 WebSocket architecture

**`frankiec new --game`**

- Scaffolds a playable starter (a zombie you steer with the arrows) with `frankiegame` pre-installed and lockfile-pinned

**Showcase games** *(`examples/projects/`)*

- 🐍 `snake/` (terminal) — the whole engine API in ~80 lines
- 🧟 `zombie_invaders/` (browser) — rects, sprites, waves, beeps
- 🏓 `pong/` (browser) — multiplayer via `on_player_key`, two tabs, one ball

---

### Fixes

- Blocks whose single statement is an assignment (`do |g| g["x"] = 1 end`) or a postfix conditional now compile correctly (previously a codegen error)

---

## v1.19.0 (2026)

### Theme: "Under the microscope" — types, a real debugger, coverage

---

### New Language Features

**Gradual type annotations**

- `def area(r: Float) -> Float` — optional everywhere; un-annotated code is untouched, and annotations cost nothing at runtime
- Checked statically by `frankiec check`, in CI, and live in the editor via the LSP
- Type names: `Int`/`Integer`, `Float`/`Number`, `String`/`Str`, `Bool`/`Boolean`, `Vector`, `Hash`, `Lambda`, `Range`, `Nil`, `Any`
- Return annotations are checked against every `return` and the implicit final expression; simple local inference follows literals, arithmetic, and calls to annotated functions
- Coexists with keyword defaults: `def connect(host: String, port: 5432)` — a bare reserved type name is an annotation, anything else is still a keyword default

---

### Tooling

**Full stepping debugger**

- `breakpoint` grew up: `(fkdb)` now understands `s` (step into calls), `n` (next, step over), `stack` (call stack, innermost first), `vars` (clean locals, no stdlib noise), any expression, and `c` (continue)
- `frankiec run --debug app.fk` breaks at the very first line — step through a program you've never read
- Line-accurate everywhere via the compiler's line maps, including inside `require`d files

**Test coverage — `frankiec test --coverage`**

- Percentages and missing-line ranges mapped back to `.fk` source
- Writes `.frankie_coverage.json` — the LSP reads it and shows uncovered lines as hints in the editor

**`frankiec docs --html`**

- Renders `##` doc-comments (with `@param`/`@return`/`@example`) into a styled single-page HTML matching the website's theme: `frankiec docs --html mystitch.fk --output mystitch.html`

**Stitch installs from any URL**

- `frankiec stitch install https://example.com/stitches/foo.fk` — pinned in `stitch.lock` with the URL recorded as its source; `verify` and `update` work exactly as with registry stitches

---

### New Stdlib

**TLS clients + UDP sockets**

- `ws_connect("wss://...")` — TLS WebSockets
- `tcp_connect(host, port, tls: true)` — TLS TCP, certificate-verified via stdlib `ssl`
- `udp_listen(port)` → socket with `.recv()` (returns `{data:, host:, port:}`) and `.send_to(host, port, msg)`
- `udp_send(host, port, msg)` — fire-and-forget
- Server-side TLS stays on the wish list

---

### Showcase Projects

Three complete programs in `examples/projects/`, each a single `.fk` file, each shippable as one `.py` via `frankiec bundle`:

- 🧟 `zombie_chat/` — multi-room WebSocket chat, server + web UI in one file
- ⚡ `word_reanimator/` — multiplayer browser hangman, shared state over WebSockets
- 💰 `frankie_ledger/` — terminal expense tracker: SQLite, stitches + `stitch.lock`, R-style stats

---

### Fixes

- The formatter now round-trips `return x if cond` and similar postfix statements correctly (previously could collapse to `nil`)

---

## v1.18.0 (2026)

### Theme: "From projects to products" — edit, ship, connect, debug

---

### Tooling

**Language Server — `frankiec lsp`**

- LSP over stdio, pure Python stdlib — live diagnostics (via the v1.17 analyzer + lex/parse errors), completion (stdlib, user symbols, keywords) and hover docs (`##` doc-comments included)
- VS Code extension upgraded from grammar-only to a full client; Neovim, Helix and Zed setups documented

**`frankiec bundle <file.fk> [-o out.py]`**

- Compiles a program plus every statically referenced `require`/`import`/`stitch` into ONE self-contained `.py` with the stdlib inlined
- Runs anywhere with `python3 out.py` — no Frankie installation needed
- Dynamic paths (computed at runtime) can't be bundled and produce a warning

**Stitch lockfile**

- `frankiec stitch install` writes `stitch.lock` (sha256, source, size, date)
- `frankiec stitch verify` — ✓ pinned / ⚠ modified / ✗ missing, exit 1 on problems (CI-ready)
- `frankiec stitch update [name]` re-fetches + re-pins

**Project-wide `check` and `fmt`**

- Both commands accept directories and recurse into `.fk` files: `frankiec check .`, `frankiec fmt --write .`

---

### New Language Features

**WebSockets — `app.websocket` + `ws_connect`**

- `app.websocket("/ws/:room") do |ws| ... end` on the built-in server — hand-rolled RFC 6455 (handshake, framing, ping/pong, close), one thread per connection, auto-close on handler return
- `ws_connect("ws://host:port/path")` client; `ws.send` / `ws.recv` / `ws.close` / `ws.params` / `ws.path` / `ws.peer`

**`breakpoint` — debugger-lite**

- Pauses into a scoped REPL: `vars`, `where`, `exit`, `c`, or any Frankie expression evaluated against the paused scope
- Skipped with a notice when stdin isn't a terminal — CI never hangs
- Postfix conditions work: `breakpoint if qty > 100`

**`enum Status(pending, active, done)`**

- Members are their own names as strings: `Status.pending` → `"pending"`
- `Status.values`, `Status.include?(x)`, iteration and `case/when` matching
- Contextual keyword — `enum` stays valid as a variable name

**`benchmark ["label"] do ... end`**

- Times a block, prints `⏱ label: 12.3ms`, returns the elapsed ms
- Works as an expression: `ms = benchmark "x" do ... end`

**Numeric literals**

- Underscore separators: `1_000_000`, `3.141_592`
- Scientific notation: `1e6`, `2.5e-3`, `1E+9`

**Set operations on vectors**

- `.union(v)` / `.intersect(v)` / `.difference(v)` — order-preserving, deduplicating

---

### Fixes

- Formatter: parenthesizes indexed expressions correctly — `(a | b)["x"]` no longer loses its parens on reformat
- Formatter emits floats the lexer can always re-read (`1e-06` now lexes thanks to scientific-notation support)
- `loop`/`spawn`/`record`/`import` etc. are now valid as the last statement of a web route handler (previously: codegen crash)

---

## v1.17.0 (2026)

### Theme: "Programs that grow" — namespacing, real checking, typed errors

---

### New Language Features

**Namespaced imports — `import "lib/math" as math`**

- Loads a `.fk` file into its own namespace: `math.circle_area(5)`, `math.PI`
- Alias optional — `import "lib/math"` defines `math`
- Modules are cached; `require` is unchanged and still merges into scope

**User-defined error types — `error TypeName`**

- `error TimeoutError` declares a type; `raise TimeoutError, "msg"` raises it
- New Ruby-style rescue binding: `rescue TimeoutError => e` (old `rescue Type e` still works)
- Typed raises auto-declare their type; generic `rescue` still catches everything
- `assert_raises_typed` understands user-defined types
- User types take precedence over Python builtins of the same name

**First-class ranges**

- Ranges print in Frankie syntax: `1..10` (not `range(1, 11)`)
- `.to_vec` / `.to_a`, `.include?`, `.step(n)`, `.sum`, `.first`, `.last`
- `case/when` with a range value now tests membership: `when 90..100`

**Record dot access — `p.x`**

- `record Point(x, y)` instances now support `p1.x` (previously `p1["x"]` only)
- Same dispatch powers module constants and zero-arg methods uniformly

**`test` blocks — `test "name", tags: ["slow"] do ... end`**

- Named, filterable test groups for the built-in harness (contextual keyword)

---

### New Stdlib

**`parallel_map(vec, workers: 4) do |x| ... end`**

- Thread-pool map via `concurrent.futures` — results in input order, first worker exception re-raised. Built for I/O-bound work.

**TCP sockets — `tcp_connect` / `tcp_listen` / `tcp_serve`**

- `tcp_connect(host, port, timeout: 5)` → socket with `send`, `send_line`, `recv(n)`, `recv_line`, `peer`, `close`
- `tcp_listen(port)` → server with `accept` / `close`
- `tcp_serve(port) do |client| ... end` — threaded accept loop, auto-close

**`stub(name, fn)` / `unstub(name)`**

- Swap any global function (`shell`, `http_get`, ...) during tests; restore with `unstub(name)` or `unstub()`

---

### Tooling

**`frankiec check` — real static analysis**

- Undefined variables/functions and wrong argument counts → errors (exit 1)
- Unused local variables → warnings; `--strict` fails on warnings (CI mode)
- Resolves `require`/`stitch`/`import` targets; checks `#{interpolation}` too

**Accurate cross-file tracebacks**

- CodeGen now emits a precise py→fk line map for every compiled file
- Runtime errors in `require`d files, imports and stitches point at the right file and the right line (previously: wrong line, main file only)

**`frankiec test --filter <name> --tag <tag>`**

- Run a subset of `test` groups; skipped groups reported in the summary

**`frankiec stitch install <name> [--global]` / `frankiec stitch list`**

- Installs stitches from the Frankie GitHub registry into `./stitches/` or `~/.frankie/stitches/` — stdlib HTTP only, zero dependencies

**REPL upgrades**

- Bare expressions echo their value: `fk> 2 + 3` → `=> 5`
- `_` holds the last result
- `help <function>` prints the signature + docs of any function

---

### Fixes

- `raise` as the last statement of a function, lambda or block no longer crashes codegen ("Unknown expression node: RaiseStmt")
- Generated f-strings avoid nested quotes — compiled output now runs on Python 3.8–3.11, not just 3.12+
- `Integer#chr`, `String#hex`, `String#oct` wired into codegen
- `TimeoutError` added to the rescue type map and builtin error registry
- Generic zero-arg method calls dispatch through a uniform mechanism, with a clear error message for missing fields/methods
- Function-call blocks now respect the block's exact parameter count, and keyword arguments order correctly around blocks: `parallel_map(urls, workers: 4) do |u| ... end`

---

## v1.16.2 (2026)

### Patch: Multi-Line Call & Definition Support

---

**Multi-line function calls now parse correctly**
Function calls with the closing `)` on its own line — or with arguments spread across multiple lines — previously caused a parse error (`Unexpected token in expression: NEWLINE`). The parser now skips newlines after every opening `(`, after every `,` between arguments, and before the closing `)` in all call sites (plain calls, method calls, safe-navigation calls).

```ruby
# All of these now work:
result = send_mail(
  to:      "alice@example.com",
  subject: "Hello",
  body:    "Hi!"
)

config = deep_configure(
  host,
  port,
  opts
)
```

**Multi-line function definitions also supported**
The same fix applies to `def` parameter lists — parameters can now span multiple lines:

```ruby
def create_user(
  name,
  email,
  role: "viewer"
)
  # ...
end
```

---

## v1.16.1 (2026)

### Patch: Bug Fixes & Compatibility

---

**Regex patterns must be strings, not `/literals/`**
Frankie does not have a regex literal syntax. All patterns passed to `scan`, `gsub`, `match_all`, `matches`, etc. must be plain strings with double-escaped backslashes: `"\\d+"` not `/\d+/`. The lexer rejects `/` in expression position when followed by a pattern character — fixed all stdlib docs and examples to use the string form.

**`||=` applies to simple variables only**
The nil-coalescing assignment operator (`||=`) works on plain variable names only. Subscript expressions such as `hash["key"] ||= value` are not supported and will cause a parse error. Use an explicit nil check instead:
```ruby
if cache["user:1"] == nil
  cache["user:1"] = {name: "Alice"}
end
```

**Tuple/parallel assignment not supported**
Multi-assignment syntax (`a, b = b, a + b`) causes a parse error. Use a temporary variable:
```ruby
tmp = a + b
a   = b
b   = tmp
```

**`hmac_verify(token, secret)` takes exactly 2 arguments**
The subject is embedded in the token by `hmac_sign` — you do not pass it separately to `hmac_verify`. Passing 3 arguments raises a runtime error.

**`.inspect` is not a Frankie method**
Ruby's `.inspect` does not exist in Frankie. Use string interpolation (`"#{value}"`) or `puts value` directly.

**Stitch names use the `frankie` prefix throughout**
All built-in stitches are named with the full `frankie` prefix: `frankieforms`, `frankietable`, `frankiecolor`, `frankiepager`, `frankieconfig`, `frankiestring`. The abbreviated `franki` prefix is incorrect.

---

### Theme: System Integration & Scripting Power

---

### New Language Features

**`loop do...end` — Infinite Loops**

A clean infinite loop construct that exits only via `break`. Clearer intent than `while true`.

```ruby
i = 0
loop do
  i += 1
  break if i >= 10
end
puts i   # 10
```

**`||=` — Nil-Coalescing Assignment**

Assign a value only if the variable is currently `nil` or `false`. Ruby-compatible semantics.

```ruby
config ||= load_defaults()
cache  ||= {}

# Common pattern — memoized computation
result ||= expensive_query()
```

---

### New Stdlib Functions

**`shell(cmd)`** — run any OS command and get structured output: `{stdout, stderr, exit_code, ok}`.

**`dotenv(path = ".env")`** — parse a `.env` file and load its variables into the process environment. Returns a hash of the loaded keys.

**`Hash#transform_values do |v| ... end`** — new hash with same keys, transformed values.

**`Hash#transform_keys do |k| ... end`** — new hash with transformed keys, same values.

**`Hash#deep_merge(other)`** — recursively merge nested hashes (unlike `.merge` / `|` which are shallow).

**`String#scan(pattern)`** — extract all regex matches from a string as a vector.

---

### New Stitches

**`frankiemail`** — send email via SMTP with a simple keyword-argument API. Supports plain text, HTML, CC/BCC. Uses Python's `smtplib` — zero extra dependencies.

**`frankiecli`** — structured CLI argument parsing built on `argv()`. Returns flags, named options, positional args, and a first-argument subcommand. Pure Frankie, zero dependencies.

**`frankiecache`** — in-memory key/value cache with optional TTL. Ideal for caching database results, config, or rate-limit counters within a running script or server. Pure Frankie, zero dependencies.

---

## v1.15.0 (2026)

### Theme: Language Polish & Developer Ergonomics

---

### New Language Features

**Ternary Operator**

Right-associative ternary expressions, valid in any expression position — assignments, string interpolation, function arguments, and vector literals.

```ruby
label = score >= 90 ? "A" : score >= 70 ? "B" : "C"
msg   = n == 1 ? "one item" : "#{n} items"
```

**Keyword-Style Default Parameters**

Both `=` and `:` syntax are now accepted in `def` signatures. Mixed signatures work correctly.

```ruby
def connect(host, port: 5432, ssl: true, timeout: 30)
  puts "#{host}:#{port} ssl=#{ssl}"
end

connect("localhost")                        # localhost:5432 ssl=true
connect("prod", port: 3306, ssl: false)     # prod:3306 ssl=false
```

**Splat in Multi-Assign — `*rest`**

Capture remaining elements with a splat variable in destructuring assignments.

```ruby
a, b, *rest       = [1, 2, 3, 4, 5]    # rest = [3, 4, 5]
first, *mid, last = [1, 2, 3, 4, 5]    # mid  = [2, 3, 4]
head, *tail       = [10, 20]            # tail = []
```

**`const` Keyword**

Explicit constant declaration. Reassignment prints a runtime warning and preserves the original value. The existing ALL_CAPS auto-detection continues to work — `const` is additive.

```ruby
const PI          = 3.14159
const MAX_RETRIES = 3
const BASE_URL    = env("BASE_URL", "http://localhost:3000")
```

**Lambda Call Syntax — `fn.(args)` Everywhere**

`fn.(args)` previously only worked inside web route middleware. Now valid in all contexts.

```ruby
double = ->(x) { x * 2 }
puts double.(5)    # 10

transforms = [->(x) { x * 2 }, ->(x) { x + 10 }]
result = 5
transforms.each do |f|
  result = f.(result)
end
puts result    # 20
```

---

### Stdlib

**`json_encode`** — preferred alias for `json_dump`. Accepts `pretty: true` for formatted output.

**`hmac_sign(subject, secret)` / `hmac_verify(token, secret)`** — previously internal (`_fk_hmac_*`), now public. Useful for signed tokens, webhooks, and tamper-proof values.

**`base64_encode(s)` / `base64_decode(s)`** — standard Base64 encode/decode, useful for HTTP Basic Auth headers.

**`String#format`** — named placeholder substitution: `"Hello, {name}!".format({name: "Alice"})`. Raises a descriptive error for missing keys.

**`.chars` / `.bytes`** — `.chars` returns a vector of individual characters (Unicode-aware); `.bytes` returns byte values.

**Path Helpers** — `path_join`, `path_dirname`, `path_basename`, `path_extname`, `path_stem`, `path_absolute`.

**Date Arithmetic** — date objects now support `+` / `-` with integers (days) and comparison operators (`<`, `==`, etc.).

**`Vector#zip_with(other) do |x, y|`** — element-wise combination with a block. Without a block, returns pairs.

**`assert_not_nil(val, msg)` / `assert_in(val, collection, msg)`** — new test assertions.

**`FrankieRequest` Query Helpers** — `req.query_int(key, default)`, `req.query_float(key, default)`, `req.query_bool(key, default)` for typed query parameter access.

---

### New Stitches

**`frankieauth`** — HTTP Basic Auth + Bearer token authentication. Zero dependencies — uses the built-in `hmac_sign` / `hmac_verify` and `base64_decode`. Functions: `basic_auth_ok?`, `auth_token_create`, `auth_token_verify`, `bearer_required`.

**`frankieratelimit`** — In-memory per-IP sliding-window rate limiting middleware. Zero dependencies. `rate_limit_check(req, next_fn, max: 60, window: 60)` and `rate_limit_reset(ip, window: 60)`.

---

### Tooling

**`frankiec check`** — parse without executing; exits 0 on success, 1 on error. Useful in CI (`frankiec check src/*.fk`).

**`frankiec new <name>`** — scaffold a complete project layout: `main.fk`, `test.fk`, `lib/`, `stitches/`, `views/`, `public/`, `data/`, `.env.example`, `.gitignore`, `README.md`.

**`frankiec watch` (now official)** — re-run a file on save. `frankiec watch test.fk --test` re-runs tests on every save.

---

## v1.14.0 (2026)

### Theme: Frankie for Real Web Apps

---

### New Language Features

**`spawn { }` — Background Blocks**
- Run any block in a background thread — returns immediately, response goes out while work continues
- Works in web routes and standalone scripts
- Spawned blocks receive a copy of variables at spawn time; mutations do not affect outer scope
- Backed by Python's `threading.Thread` — zero dependencies

**`timeout(n) { }` — Time-Bounded Execution**
- Kill any block that exceeds `n` seconds — raises `TimeoutError`
- Works inside `begin/rescue TimeoutError` for graceful fallback
- Essential for external HTTP calls, slow database queries, and any operation that can hang
- Backed by Python's `threading` with a sentinel thread — zero dependencies

**Hash Destructuring — `{name, age} = user`**
- Pull hash keys directly into local variables in one assignment
- Missing keys evaluate to `nil` — no error
- Works anywhere an assignment is valid: top level, inside functions, inside route handlers, inside loops
- Bareword (symbol) keys only — matches the existing hash literal convention

**Shape Pattern Matching — `case user when {role: "admin"}`**
- `when` clauses now accept hash literals — matches any hash containing at least those key/value pairs
- Extra keys in the subject hash are ignored (subset match)
- Works with records — `record Point(x, y)` is a hash, so `when {x: 0}` is valid
- Mix shape and value `when` clauses in the same `case`

---

### New Web Features

**Async Routes — `app.get_async` / `app.post_async` etc.**
- Non-blocking route handlers — slow I/O in one handler does not hold up others
- All five HTTP methods have async variants: `get_async`, `post_async`, `put_async`, `delete_async`, `patch_async`
- Use `await` inside async blocks for non-blocking calls

**Middleware Stack — `app.use do |req, next_fn|`**
- Chain middleware that wraps every request — auth, logging, rate-limiting, CORS
- Each layer calls `next_fn.(req)` to pass control forward, or returns a response to short-circuit
- Runs in registration order

**Static File Serving — `app.static(dir)` / `app.static(dir, prefix)`**
- Serve a directory of static files with one line
- Serves HTML, CSS, JS, images, fonts, JSON automatically
- Optional URL prefix: `app.static("./assets", "/static")`
- Directory listing disabled by default

---

### New Stitches

**`frankietemplate`**
- Mustache-compatible template engine — zero dependencies, pure `.fk`
- `{{ variable }}` — HTML-escaped interpolation
- `{{{ variable }}}` — raw / unescaped output
- `{{# section }} ... {{/ section}}` — truthy blocks and vector iteration
- `{{^ inverted }} ... {{/ inverted}}` — falsy / empty blocks
- `{{> partial_name }}` — include from `./views/partials/<n>.html`
- `{{! comment }}` — stripped from output
- `render(template, data)` — render a string
- `render_file(path, data)` — load and render a file
- `partial(name)` — load a partial by name

**`frankiecookie`**
- HMAC-SHA256 signed cookies via Python's `hmac` + `hashlib` stdlib — zero dependencies
- `set_signed_cookie(resp, name, value, secret, opts)` — write a tamper-proof cookie
- `get_signed_cookie(req, name, secret)` — read and verify — returns `nil` if missing or tampered
- `delete_cookie(resp, name)` — expire a cookie immediately
- `cookie_set?(req, name)` — check if a cookie is present
- Supports all cookie options: `path`, `max_age`, `same_site`, `http_only`, `secure`

---

### Tooling

**`frankiec new` — Scaffold Updated for Stitches**
- Generated project now includes a `stitches/` folder with a `README.md` explaining the stitch convention
- Generated project includes a `views/partials/` folder for template projects
- Generated `README.md` documents `stitch "name"` usage
- Version string in `main.fk` banner updated to v1.14

**Global Stitch Install — `install.py`**
- `install.py` now correctly copies all bundled stitches to `~/.frankie/stitches/` at install time
- Installation output lists each stitch file copied
- `frankiecookie` and `frankietemplate` included in the bundled set
- `python3 install.py --uninstall` removes `~/.frankie/stitches/` and `~/.frankie/` if empty

## v1.13.1 (2026)

### Bug Fixes & Gap Closers

**Parser / Compiler**

- Heredoc inside `do...end` blocks — Fixed lexer bug that discarded tokens after `<<~DELIM` on the same line. Heredocs now work anywhere a string expression is valid, including inside route and iterator blocks. The only documented workaround in the language is removed.
- Vector `.sum do |x| ... end` — Block form was swallowed by the internal method map before block detection. Now correctly routes to `_fk_sum_by` for projected sums.
- Vector `.flat_map do |x| ... end` — Multi-line block bodies now parse correctly.
- `Hash.each do |k, v|` — Two-parameter block iteration confirmed and end-to-end tested. No workaround needed.

**Standard Library**

- `assert_approx_eq(actual, expected, delta, msg)` — Float comparison assertion with configurable delta (default `0.001`). Replaces `assert_true(abs(a-b) < delta, ...)` boilerplate.
- `run_tests()` — Now a public stdlib function callable from any `.fk` file, not just `frankiec test`.
- `session(req, resp)` — Cookie-backed session helper. Read, mutate, and write back with `.save()`. Single JSON cookie (`_fk_session`), zero server-side state.
- String `.ljust(n)` / `.rjust(n)` / `.center(n)` — Promoted to documented stdlib status with examples.
- String `.start_with?(s)` / `.end_with?(s)` — Documented as first-class predicates.
- Hash `.keys` / `.values` / `.has_key?(k)` — All three consistently documented; `.values` and `.has_key?` reference gaps closed.

**Stitches**

- `frankiestring` v2 — Rewritten with a clean API. Old `lfill/rfill` replaced by `pad_left(str, n, char), pad_right(str, n, char), truncate(str, n, suffix), slugify(str), word_wrap(str, width), indent_lines(str, n)`.

**Tooling**

- `frankiec fmt` — symbol key round-trip — Symbol keys (`host: "x"`) are preserved after formatting; `{host: "val"}` no longer becomes `{"host": "val"}`.
- `frankiec fmt` — blank line preservation — Intentional blank lines between statement groups inside function bodies are now preserved.
- `frankiec fmt` — multi-line threshold — Hashes and vectors whose inline form exceeds 60 characters auto-expand to one element per line.
- `frankiec fmt` — idempotency — Running `fmt --write` twice now produces identical output. Safe for pre-commit hooks and CI.

**REPL**

- Multi-line REPL input — Fixed `_is_incomplete` edge cases: standalone `do |x|` lines now correctly hold the `...` prompt open; comment lines are skipped during depth counting.

## v1.13.0 (2026)

### New Features

**Language — `stitch "name"` keyword**
- New keyword for loading third-party Frankie packages by name
- Resolution order: `./stitches/<n>.fk` (project-local) → `~/.frankie/stitches/<n>.fk` (user-global)
- Friendly error when not found: tells you exactly where to put the file
- Each stitch is loaded at most once — safe to call multiple times
- Uses the same underlying `require` machinery — stitch files are plain `.fk` files
- Establishes a clear convention: `lib/` = your code, `stitches/` = third-party packages

**Language — `?` in user-defined function names**
- `def even?(n)` and `def palindrome?(s)` now work correctly
- `?` is compiled to `_q` in generated Python — transparent to the programmer
- Applies to function definitions, calls, and assignments

**Stitch — `frankieforms`**
- Form field validation returning a Hash of `{field: error_message}` pairs
- Rules: `required, min_length, max_length, email, min_value, max_value, numeric, alpha, matches_pattern`
- `validate(form, rules)` → error hash · `valid?(form, rules)` → boolean

**Stitch — `frankietable`**
- ASCII table rendering from a vector of hashes
- `table(rows)` — all columns · `table(rows, cols)` — specific columns in given order
- Column widths auto-sized to content

**Stitch — `frankiecolor`**
- ANSI color and style helpers for terminal output
- Color functions: `red, green, yellow, blue, cyan, magenta, white, black`
- Style functions: `bold, dim, italic, underline, inverse`
- Semantic helpers: `success, error, warn, info`
- `colorize(str, color)` — generic · `strip_color(str)` — remove ANSI codes

**Stitch — `frankiepager`**
- Pagination math for web apps and CLI tools
- `paginate(opts)` → full pager hash with `page, total_pages, from, to, has_prev, has_next, prev_page, next_page`
- `page_slice(items, page, per_page)` — slice a vector to the current page
- `page_links(pager, url_template)` — navigation link vector

**Stitch — `frankieconfig`**
- Layered configuration loading: defaults → JSON file → environment variables → overrides
- Type coercion from env var strings to match default types (Integer, Float, Boolean)
- `load_config(opts) · config_get(config, key, fallback)`

## v1.12.0 (2026)

### New Features

**Standard Library — String `.gsub` with Block**
- `"hello".gsub("[aeiou]") do |m| m.upcase end` → `"hEllO"`
- Block form transforms each match; the block receives the matched substring and returns the replacement
- The fixed-string form `gsub(pattern, replacement)` continues to work unchanged
- Uses `re.sub` with a callable internally — no new dependencies

**Standard Library — Hash `.map_hash do |k, v|`**
- `{a: 1, b: 2}.map_hash do |k, v| [k, v * 2] end` → `{a: 2, b: 4}`
- Transforms a hash into a new hash in one idiomatic step
- Block must return a two-element vector `[new_key, new_value]`; any other return raises a clear runtime error
- Fills the gap between `.map` (returns vector of pairs) and a true hash transform

**Standard Library — `round(x, n)`**
- `round(3.14159, 2)` → `3.14`
- Rounds to n decimal places; n defaults to 0
- Available as a top-level function alongside `floor` and `ceil`
- Wired to Python's built-in `round()` — no surprises on banker's rounding edge cases

**Standard Library — `Vector .product(other)`**
- `[1,2].product([3,4])` → `[[1,3],[1,4],[2,3],[2,4]]`
- Cartesian product — every combination of elements from two vectors
- Pure nested loop, zero dependencies
- Natural companion to `.zip` and `.zip_with` — completes the combinatorics trio

**Standard Library — `String .chars` (promoted)**
- `.chars` was already in the method map but undocumented
- Now a first-class documented method alongside `.bytes` and `.lines`
- Chains naturally into iterators: `"hello".chars.select do |c| c != "l" end`

**Standard Library — `Vector .each_with_object` with Hash Accumulator (documented)**
- Hash accumulators already worked implicitly; now explicitly documented and tested
- `[1,2,3].each_with_object({}) do |x, h| h[x] = x * x end` → `{1: 1, 2: 4, 3: 9}`

**Tooling — `assert_match` and `assert_nil`**
- `assert_match(value, pattern, msg)` — checks a regex match; pattern can be a regex or string
- `assert_nil(value, msg)` — checks for nil; cleaner than `assert_eq(x, nil)`
- Both available in `frankiec test` with the same output style as existing assertions

**Tooling — `frankiec watch <file.fk>`**
- Polls file mtime and re-runs automatically on every save
- `frankiec watch test.fk --test` runs in test mode
- Zero dependencies — uses `os.stat` and `time.sleep`
- Ctrl-C to stop; gracefully ignores `exit()` calls in watched files

**Tooling — `frankiec repl --no-banner`**
- Skips the ASCII art and version header
- Makes the REPL usable piped into other tools or embedded in scripts

### Bug Fixes & Improvements

**Runtime — `FileNotFoundError` from File I/O**
- `file_read` and `file_lines` previously raised `RuntimeError`, silently defeating `rescue FileNotFoundError`
- Now raise a genuine `FileNotFoundError` with a clean message — no `[Frankie]` prefix noise
- `file_copy` and `file_rename` also raise `FileNotFoundError` when the source is missing
- `file_delete` and `file_exists` unchanged — returning `false` for missing files is correct for those
- `rescue FileNotFoundError e` now works as expected for all file I/O

**Runtime — Friendlier Error Messages**
- `TypeError` now reads as `"Type mismatch — can't use '+' with Integer and String"` instead of the raw Python message
- `IndexError` now reads as `"Index out of bounds — vector index does not exist"` instead of `"list index out of range"`
- `FileNotFoundError` strips the raw Python `[Errno 2] No such file or directory:` prefix
- The friendly dict in `frankiec.py` is now backed by three focused helper functions for easier future extension

---

## v1.11.0 (2026)

### New Features

**Language — Implicit Return**
- The last expression in a function body is now automatically returned — `return` is optional
- Early `return` statements are still fully supported for mid-function exits
- Applies to all function bodies including nested functions
- Only expressions trigger implicit return; assignments, loops, and `puts` at the end still return `nil`
- Zero breaking change risk: all existing programs using explicit `return` continue to work identically

**Language — Inline `if` Expression**
- `x = if cond then a else b end` — `if` is now usable as an expression, not just a statement
- `then` keyword is optional; a newline after the condition also works
- `elsif` clauses are supported: `if a then x elsif b then y else z end`
- Missing `else` clause evaluates to `nil`
- New `THEN` token type added to the lexer; new `IfExpr` AST node added
- Avoids introducing a `?:` ternary operator that clashes with Frankie's readable style

**Standard Library — String `.replace(old, new)`**
- `"hello world".replace("world", "Frankie")` → `"hello Frankie"`
- Replaces the first occurrence — an alias for `sub()`
- The method name new users always reach for before remembering `sub`/`gsub`
- `sub()`, `gsub()` continue to work as before

**Standard Library — String `.format(hash)`**
- `"Hello, {name}!".format({name: "Alice"})` — named `{key}` placeholder replacement
- Method form of the existing `template()` function
- Uses `{key}` syntax (vs `template()`'s `{{key}}` syntax)
- Runtime dispatches on argument type: dict → string format; non-dict → datetime format

**Standard Library — `.zip_with do |a, b|`**
- `[1,2,3].zip_with([10,20,30]) do |a, b| a + b end` → `[11, 22, 33]`
- Pair-wise transform two vectors in a single pass
- Completes the R-style vector pipeline alongside `.zip`, `.map`, `.select`
- Stops at the shorter vector, matching `.zip` behaviour

**Tooling — `frankiec check` Boxed Error Output**
- Parse and lex errors from `frankiec check` now use the same boxed format as runtime errors
- Includes file path, line number, and source context with `──▶` pointer
- Essential for editor integration — output is now machine-parseable and visually consistent
- Exit codes unchanged: 0 = clean, 1 = error

**Tooling — REPL Multi-line History Recall**
- `↑` now recalls a complete `def...end` block as a single history entry
- Previously, each line of a multi-line block was stored separately
- Implementation: per-line readline entries are removed and replaced with a single joined entry
- History file (`~/.frankie_history`) updated on block submission, not just on exit
- Gracefully degrades on readline bindings that don't support `remove_history_item`

**Tooling — `frankiec fmt` Heredoc Support**
- Heredoc string bodies are now preserved verbatim during formatting
- Multiline string literals are re-emitted as `<<~HEREDOC` blocks
- Fixes a v1.10 regression where heredoc content could be mangled by the formatter

### Documentation
- New `docs/13_v111_features.md` with full feature reference and examples
- Multiple return values via destructuring documented as an official pattern
- String `.delete(chars)` promoted from hidden stdlib to documented method

---

## v1.10.0 (2026)

### New Features

**Language — String & Vector `*` Repetition**
- `"ha" * 3` → `"hahaha"` — string repetition
- `[0] * 5` → `[0, 0, 0, 0, 0]` — vector fill
- `[1, 2] * 3` → `[1, 2, 1, 2, 1, 2]` — pattern repeat
- Integer on either side works: `3 * "hi"` → `"hihihi"`
- Implemented in `_fk_arith` — zero new syntax

**Language — Heredoc `<<~TEXT`**
- `<<~DELIM ... DELIM` multiline string with automatic indent-stripping
- `<<DELIM` variant (no strip) also supported
- Full `#{}` interpolation inside heredoc bodies
- Pure lexer change — no new tokens or AST nodes
- The codegen `gen_string` rewritten to use `repr()` + concatenation for multiline interpolated strings, eliminating triple-quote/backslash edge cases entirely

**Language — Named Rescue Without Variable**
- `rescue TypeError` is now valid without a binding variable
- `rescue TypeError e` still works when the message is needed
- Parser fix: variable binding is now truly optional after a typed rescue

**Standard Library — `times(n) do |i|` standalone**
- `times(n) do |i| ... end` functional form added alongside `n.times do`
- `times(n)` with no block returns `[0..n-1]` as a list
- `FuncCall` AST node gains an optional `block` field; parser attaches trailing `do...end` blocks to function calls; codegen emits a `for` loop for `times`

**Standard Library — `flatten(depth)`**
- `.flatten` with no argument now does **full deep** flatten (breaking change from v1.9's one-level-only behaviour)
- `.flatten(n)` flattens exactly `n` levels; `.flatten(0)` is a no-op
- Backed by new `_fk_flatten_deep(iterable, depth)` in stdlib

**Standard Library — `map_with_index`**
- `.map_with_index do |x, i| ... end` — index available in map block
- Single-expression blocks compile to a list comprehension; multi-statement blocks use a helper function

**Standard Library — `pp(value)` pretty-print**
- Indented multiline output for hashes, vectors, and records
- Records printed as `RecordName(\n  field: value,\n  ...)`
- Flat vectors printed on one line; nested structures indented recursively

**Standard Library — `encode` / `decode`**
- `"hello".encode` → `[104, 101, 108, 108, 111]` (UTF-8 bytes as vector)
- `"hello".encode("ascii")` — explicit encoding
- `[104, 105].decode` → `"hi"`
- `[104, 105].decode("utf-8")` — explicit encoding

**Runtime — Exit Code Propagation**
- `exit(42)` in Frankie code now propagates the exact code to the shell
- `frankiec run` catches `SystemExit` and calls `sys.exit(e.code)` instead of re-raising

**CLI — `--help` Flag**
- `frankiec --help` prints the full usage docstring
- `frankiec <cmd> --help` prints a short description for that specific command
- All commands covered: `run`, `repl`, `test`, `fmt`, `docs`, `build`, `check`, `new`, `version`

### Bug Fixes
- `gen_string` multiline interpolation: single-line f-strings were emitted with literal embedded newlines (invalid Python syntax). Fixed by using `repr()` + string concatenation for all multiline interpolated strings.
- `flatten` semantics changed to full-deep by default; use `.flatten(1)` for the old one-level behaviour.

---

## v1.9.0 (2026)


### New Features

**Language — Record Types (`record`)**
- `record Point(x, y)` defines a lightweight named data object
- Constructor function generated automatically: `p = Point(3, 4)`
- Prints as `Point(x: 3, y: 4)` — clean, readable output
- Records are hashes under the hood — all hash methods, iterators, `dig`, `|`, and `.merge` work on them
- New lexer token `RECORD`, new AST node `RecordDef`, new `gen_record_def` in codegen
- `_fk_to_str` updated to detect `__type__` and display record notation

```ruby
record Employee(name, dept, salary)
emp = Employee("Alice", "Engineering", 95000)
puts emp               # Employee(name: Alice, dept: Engineering, salary: 95000)
puts emp["dept"]       # Engineering
by_dept = employees.group_by do |e| e["dept"] end
```

**Standard Library — `hash.dig(key, ...)`**
- Safe nested access: returns `nil` at the first missing key, never crashes
- Works on hashes (string/symbol keys) and vectors (integer indices)
- Chains correctly with `&.` for nil-safe navigation

```ruby
config = {db: {host: "localhost", pool: {max: 10}}}
puts config.dig("db", "pool", "max")   # 10
puts config.dig("db", "missing")       # nil  (no crash)
```

**Standard Library — Standalone `zip(*vecs)`**
- `zip(a, b)` function form alongside the existing `.zip` method
- Accepts two or more vectors; stops at the shortest
- Consistent with Frankie's R-inspired functional style

```ruby
zip(["Alice", "Bob"], [95, 87])   # [["Alice", 95], ["Bob", 87]]
```

**Tooling — `frankiec fmt` (Auto-Formatter)**
- New command: `frankiec fmt <file.fk>` — print canonically formatted source
- `--write` flag: reformat in-place
- `--check` flag: exit 1 if not already formatted (CI-friendly)
- Implemented in `frankie_fmt.py` — walks the AST, zero new dependencies
- Canonical style: 2-space indent, single-expr blocks inlined, blank line after top-level `def`

**Tooling — `frankiec docs` (Documentation Generator)**
- New command: `frankiec docs <file.fk>` — extract `##` doc-comments to Markdown
- `--output <file.md>` flag: write to file instead of stdout
- Supports `@param`, `@return`, `@example` tags
- Works on directories: `frankiec docs lib/` generates docs for every `.fk` file
- Implemented in `frankie_docs.py` — pure Python, zero new dependencies

**REPL — readline, Tab Completion, History Persistence**
- Arrow key navigation (`↑`/`↓`) and Ctrl+R reverse-search via Python's built-in `readline`
- Tab completion for Frankie keywords, stdlib functions, and common method names
- History saved to `~/.frankie_history` on exit; restored at next startup (max 1000 entries)
- `.env` auto-loaded from the current working directory at REPL startup

**Runtime — `.env` Auto-Loader**
- `frankiec run` and `frankiec repl` automatically load `.env` from the current directory
- Keys already set in the shell environment take precedence
- Values accessible via the existing `env(key, default)` stdlib function
- No crash if `.env` is absent

### Bug Fixes
- `record` added as a reserved keyword — programs that used `record` as a variable name must rename it (the existing `.fk` examples in the repo have been updated)
- `scaffold.py` version string updated from `v1.3` to `v1.9`

---

## v1.8.0 (2026)


### New Features

**Language — Lambda / Anonymous Functions (`->`)**
- Store functions as first-class values: `double = ->(x) { x * 2 }`
- Call with `.call(args)`: `double.call(5)` → `10`
- Single-expression bodies use brace syntax: `->(x) { x * 2 }`
- Multi-statement bodies use `do...end`: `->(x) do ... end`
- Default parameters are supported: `->(x, y = 1) { x + y }`
- Lambdas can be stored in variables, vectors, and hashes
- Lambdas can be passed to functions as arguments (higher-order functions)
- Lambdas can be returned from functions
- New token: `ARROW` (`->`)
- New AST node: `LambdaLiteral`

```ruby
double = ->(x) { x * 2 }
add    = ->(a, b) { a + b }
puts double.call(7)      # 14
puts add.call(3, 4)      # 7

def apply(fn, val)
  return fn.call(val)
end
puts apply(double, 9)    # 18
```

**Language — Hash Merge Operator `|`**
- `h1 | h2` merges two hashes; right-hand keys win on conflict
- Returns a new hash — neither operand is modified
- Chains naturally: `a | b | c`
- Complements the existing `.merge(other)` method

```ruby
defaults = {color: "blue", size: "medium"}
overrides = {color: "red"}
puts defaults | overrides   # {color: red, size: medium}
```

**Standard Library — `group_by`**
- `vector.group_by do |x| key end` buckets elements by block return value
- Returns a hash whose values are arrays of matching elements, in original order
- Pairs naturally with `.tally`, `.sort_by`, `.each`, and the new `|` operator

```ruby
words = ["ant", "ape", "bear", "bee", "cat"]
puts words.group_by do |w| w[0] end
# {"a" => ["ant", "ape"], "b" => ["bear", "bee"], "c" => ["cat"]}
```

**Standard Library — `each_slice` and `each_cons`**
- `vector.each_slice(n)` — iterate non-overlapping chunks of size `n`
- `vector.each_cons(n)` — iterate all consecutive windows of size `n` (sliding window)
- Both accept an optional `do |var| ... end` block; without a block, return a vector of slices/windows
- Mirrors Ruby's API; natural fit for data-processing, batch operations, and rolling statistics

```ruby
[1,2,3,4,5,6].each_slice(2) do |s|
  puts s     # [1,2]  [3,4]  [5,6]
end

[10, 13, 11, 15].each_cons(2) do |w|
  puts w[1] - w[0]    # 3  -2  4
end
```

### Bug Fixes
- **`_block_to_lambda`** — blocks whose body ends with a control-flow expression
  (`if`, `case`, `unless`) now correctly capture the result value instead of
  raising a `CodeGenError`. Affects `select`, `reject`, `sort_by`, `min_by`,
  `max_by`, `sum_by`, `find`, `flat_map`, and the new `group_by`.

---

## v1.7.1 (2025)

### Bug Fixes
- **`N.times do`** — `def _gen_times` method definition was accidentally dropped from
  `compiler/codegen.py` during v1.7 development, causing an `AttributeError` at runtime.
  The method body was present but the `def` header was missing.
- **`test_v17.fk`** — example test file used `def()` anonymous function syntax inside
  `assert_raises_typed` which does not exist in Frankie yet. Rewritten to use
  `begin/rescue` blocks instead.

---

## v1.7.0 (2025)

### New Features

**Language — Nil Safety Operator `&.`**
- `x&.method` — call `.method` on `x`, returning `nil` if `x` is nil (no crash)
- `x&.method(args)` — nil-safe method call with arguments
- `x&.property` — nil-safe property / zero-arg attribute access
- Chains naturally: `a&.b&.c` — short-circuits at the first nil
- Works with any value: strings, hashes, vectors, custom objects
- No language keywords added — `&.` is a single new operator token

```ruby
user = {name: "Alice"}
missing = nil

puts user["name"]&.upcase   # ALICE
puts missing&.upcase        # nil  (no crash)
puts missing&.upcase&.reverse  # nil  (chain short-circuits)
```

**Standard Library — `template(str, hash)`**
- Replace `{{key}}` placeholders in a string with values from a hash
- Clean alternative to `sprintf` / `#{}` when keys are dynamic or templates are stored externally
- Raises `KeyError` if a placeholder key is missing from the hash

```ruby
msg = template("Hello, {{name}}! Age: {{age}}.", {name: "Alice", age: 30})
puts msg   # Hello, Alice! Age: 30.
```

**Standard Library — File System Operations**
- `file_rename(src, dst)` — rename or move a file
- `file_copy(src, dst)` — copy a file (preserving metadata); returns `dst`
- `file_mkdir(path)` — create a directory; creates intermediate dirs by default (like `mkdir -p`)
- `file_mkdir(path, false)` — create a single directory only (no parents)
- `dir_exists(path)` — return `true` if path is an existing directory
- `dir_list(path)` — return a sorted vector of filenames in a directory (default: `"."`)
- All use Python's built-in `os` / `shutil` — zero external dependencies

```ruby
file_mkdir("/tmp/myapp/data")
puts dir_exists("/tmp/myapp/data")     # true

file_write("/tmp/myapp/data/a.txt", "hello")
file_copy("/tmp/myapp/data/a.txt", "/tmp/myapp/data/b.txt")
file_rename("/tmp/myapp/data/b.txt", "/tmp/myapp/data/c.txt")

puts dir_list("/tmp/myapp/data")       # [a.txt, c.txt]
```

**Standard Library — `assert_raises_typed(fn, type, msg)`**
- Extends the test runner to assert that a specific error *type* was raised
- `type` can be a string (`"ZeroDivisionError"`) or a Python exception class
- Fails with a clear message if no error is raised, or if the wrong type is raised
- Supported type names mirror typed `rescue`: `RuntimeError`, `TypeError`, `ValueError`,
  `ZeroDivisionError`, `IndexError`, `KeyError`, `IOError`, `FileNotFoundError`,
  `OverflowError`, `NameError`, `AttributeError`, `Exception` / `Error`

```ruby
assert_raises_typed(def()
  x = 1 // 0
end, "ZeroDivisionError", "division by zero raises correctly")

assert_raises_typed(def()
  file_read("/no/such/file.txt")
end, "RuntimeError", "missing file raises RuntimeError")
```

### Bug Fixes
- `&.` operator correctly short-circuits chains — once a nil is encountered, remaining
  method calls in the chain are skipped without raising errors

---

## v1.6.0 (2025)

### New Features

**Language — Compound Assignment Operators**
- `+=` — add and assign: `x += 5`
- `-=` — subtract and assign: `x -= 3`
- `*=` — multiply and assign: `x *= 2`
- `/=` — divide and assign (float): `x /= 4`
- `//=` — integer-divide and assign (Fortran): `x //= 3`
- `**=` — exponentiate and assign (Fortran): `x **= 2`
- `%=` — modulo and assign: `x %= 7`
- All operators also work on vector elements: `v[i] += 1`

**Language — Typed Rescue Clauses**
- `rescue TypeError e` — catch only `TypeError` errors
- `rescue ZeroDivisionError e` — catch only division by zero
- Multiple `rescue` clauses on one `begin...end` block, checked in order
- Full list of supported types: `RuntimeError`, `TypeError`, `ValueError`,
  `ZeroDivisionError`, `IndexError`, `KeyError`, `IOError`,
  `FileNotFoundError`, `OverflowError`, `NameError`, `AttributeError`,
  `StopIteration`, `Exception` / `Error` (catch-all aliases)
- Untyped `rescue e` remains valid and catches everything

**Standard Library — `.find` / `.detect`**
- `.find do |x| ... end` — return the first element for which the block is true, or `nil`
- `.detect do |x| ... end` — alias for `.find`
- Works on any vector, including vectors of hashes
- Chains naturally with `.select`, `.map`, `.sort_by`, etc.

**Tooling — `frankiec test`**
- `frankiec test` — run `test.fk` in the current directory
- `frankiec test <file.fk>` — run a named test file
- Built-in assertions (no imports needed):
  - `assert_true(cond, msg)` — pass if condition is truthy
  - `assert_eq(actual, expected, msg)` — pass if values are equal
  - `assert_neq(actual, expected, msg)` — pass if values differ
  - `assert_raises(fn, msg)` — pass if calling `fn` raises any error
- Live `✓` / `✗` output per assertion
- Summary line with pass count, fail count, and elapsed time
- Exits with code 1 if any assertion fails (CI-friendly)

### Bug Fixes
- `_fk_test_suite` singleton now uses a fresh isolated copy per `frankiec test` run,
  preventing state leakage between multiple test invocations in the same process

---

## v1.5.0 (2025)

### New Features

**Language — Loop Control**
- `next` — skip to the next iteration (like `continue` in other languages); supports postfix `next if cond`
- `break` — exit a loop early; supports postfix `break if cond`
- `break value` — exit a loop and store a result in `_fk_break_val`; supports postfix form

**Language — Constants**
- `UPPER_CASE = value` — UPPER_SNAKE_CASE identifiers are treated as constants
- Reassignment prints a warning and preserves the original value
- Works with any type: integers, floats, strings, vectors, hashes

**Standard Library — Randomness**
- `random()` — random Float in [0.0, 1.0)
- `rand(n)` — random Integer in [0, n)
- `rand_int(a, b)` — random Integer in [a, b] (both inclusive)
- `rand_float(a, b)` — random Float in [a, b)
- `shuffle(vec)` — return a shuffled copy of a vector
- `sample(vec, n)` — return n randomly chosen elements (no repeats)
- `rand_seed(n)` — seed the RNG for reproducible results

**Standard Library — Sorting**
- `.sort_by do |x| key end` — sort a vector by any computed key
- `.min_by do |x| key end` — element with the smallest key
- `.max_by do |x| key end` — element with the largest key
- `.sum_by do |x| val end` — sum the values the block returns

**Standard Library — Other**
- `sleep(n)` — pause execution for n seconds (float supported)
- `unzip(vec)` — inverse of zip: vector of pairs → vector of columns
- `format(fmt, ...)` — alias for `sprintf`

### Bug Fixes
- Block parameters named `p` (e.g. `do |p|`) now parse correctly — `p` was always tokenised as the debug-print keyword, preventing it from being used as a loop variable
- `p[...]` and `p.method` now correctly treated as variable access rather than a debug-print call
- `break if cond` and `next if cond` (postfix forms) parse without errors

---

## v1.4.0 (2025)

### New Features

**Web Server** (built-in `http.server` — zero deps)
- `web_app()` — create a new Frankie web application
- `app.get(path) do |req| end` — register a GET route
- `app.post(path) do |req| end` — register a POST route
- `app.put(path) do |req| end` — register a PUT route
- `app.delete(path) do |req| end` — register a DELETE route
- `app.patch(path) do |req| end` — register a PATCH route
- `app.before do |req| end` — before-filter (runs before every matched route)
- `app.after do |req, res| end` — after-filter (runs after every matched route)
- `app.not_found do |req| end` — custom 404 handler
- `app.run(port)` / `app.run(port, host)` — start the server (blocking, multi-threaded)
- Path parameters with `:name` segments: `"/users/:id"` → `req.params["id"]`
- Query string access: `req.query["page"]`
- JSON body parsing: `req.json` — returns parsed hash/vector or nil
- Form body parsing: `req.form` — returns decoded hash
- `response(body, status, headers)` — plain-text response
- `html_response(body, status)` — HTML response
- `json_response(data, status)` — JSON response (auto-serializes hashes and vectors)
- `redirect(location, status)` — redirect response (default 302)
- `halt(status, body)` — error response shortcut
- Returning a plain string from a handler auto-wraps as `200 text/plain`
- Returning a hash or vector auto-wraps as `200 application/json`
- Full request object: `.method`, `.path`, `.params`, `.query`, `.headers`, `.body`, `.json`, `.form`
- See `docs/09_web.md` and `examples/webapp.fk` for full reference and demo

### Bug Fixes
- `raise expr if cond` (postfix `if` on `raise`) now parsed correctly — was leaving the `if` clause unconsumed, causing `rescue` to be seen as an unexpected token in `begin/rescue` blocks
- `data |> sum |> puts` — `puts` and `print` now accepted as bare pipe targets (previously raised an unexpected token error)

---

## v1.3.0 (2025)

### New Features

**Language**
- Default parameter values: `def greet(name, msg="Hello", punct="!")`
- Keyword-named parameters (e.g. `times`, `each`) now usable as variable/param names
- Triple-quoted multi-line strings: `"""..."""` and `'''...'''` with interpolation

**JSON** (built-in `json` module — zero deps)
- `json_parse(str)` — parse JSON string → Frankie value
- `json_dump(obj, pretty)` — serialize to JSON string
- `json_read(path)` — read and parse JSON file
- `json_write(path, obj, pretty)` — serialize and write JSON file

**CSV** (built-in `csv` module — zero deps)
- `csv_parse(text, headers)` — parse CSV text → vector of hashes
- `csv_dump(data, headers)` — serialize vector of hashes → CSV string
- `csv_read(path, headers)` — read and parse CSV file
- `csv_write(path, data, headers)` — write CSV file

**DateTime** (built-in `datetime` module — zero deps)
- `now()` — current date and time
- `today()` — today's date at midnight
- `date_from(year, month, day, hour, minute, second)` — construct a date
- `date_parse(str, fmt)` — parse a date string (default fmt: `%Y-%m-%d`)
- `.year`, `.month`, `.day`, `.hour`, `.minute`, `.second` — accessors
- `.format(fmt)` — format with `strftime` directives
- `.add_days(n)`, `.add_hours(n)`, `.add_minutes(n)` — arithmetic
- `.diff_days(other)`, `.diff_seconds(other)` — differences
- `.weekday()`, `.weekday_name()` — day of week
- `.is_before(other)`, `.is_after(other)` — comparison
- `.timestamp()` — Unix timestamp

**HTTP** (built-in `urllib` — zero deps)
- `http_get(url, headers)` — GET request
- `http_post(url, data, headers)` — POST request (auto JSON-encodes dicts)
- `http_put(url, data, headers)` — PUT request
- `http_delete(url, headers)` — DELETE request
- Response: `.status`, `.body`, `.headers`, `.json()`, `.ok()`
- `url_encode(hash)` — encode params as query string
- `url_decode(str)` — decode query string → hash

**Tooling**
- `frankiec new <project>` — scaffold a new project with `main.fk`, `test.fk`, `lib/`, `data/`, `README.md`, `.gitignore`
- Better error messages — compile and runtime errors now show a boxed display with source context and line pointer (──▶)
- Syntax highlighting for **VS Code** (`.tmLanguage.json` + `package.json` + `language-configuration.json`)
- Syntax highlighting for **Vim/Neovim** (`frankie.vim`)
- Syntax highlighting for **Sublime Text / TextMate** (`frankie.tmLanguage.json`)
- All editor files in `editors/`

### Bug Fixes
- Fixed `print` output disappearing when running from a different working directory
- `times`, `each`, `map` keywords can now be used as parameter and variable names
- Template placeholders in `frankiec new` use `.replace()` to avoid `str.format()` key conflicts

---

## v1.2.0 (2025)

### New Features

**Database Access (SQLite)**
- `db_open(path)` — open or create a SQLite database; `":memory:"` for in-memory
- `db.exec(sql, params)` — run DDL/DML with `?` placeholders; returns row count
- `db.query(sql, params)` — SELECT → vector of hashes keyed by column name
- `db.query_one(sql, params)` — SELECT → first row as hash or nil
- `db.insert(table, hash)` — insert a hash of column→value; returns new row id
- `db.insert_many(table, rows)` — bulk insert a vector of hashes
- `db.find_all(table)` — all rows as vector of hashes
- `db.find(table, where)` — filtered rows (where is a hash, conditions ANDed)
- `db.find_one(table, where)` — first matching row or nil
- `db.update(table, data, where)` — update matching rows; returns count
- `db.delete(table, where)` — delete matching rows; returns count
- `db.count(table)` / `db.count(table, where)` — row counts
- `db.last_id` — rowid of last INSERT
- `db.tables` — list of table names in the database
- `db.columns(table)` — column info as vector of hashes
- `db.transaction do...end` — atomic block; rolls back on any error
- `db.begin` / `db.commit` / `db.rollback` — explicit transaction control
- `db.close` — close the connection
- Zero external dependencies — uses Python's built-in `sqlite3`

**Multi-line Strings**
- Triple-quoted strings `"""..."""` and `'''...'''` spanning multiple lines
- String interpolation `#{}` works inside triple-double-quoted strings
- Perfect for embedding multi-line SQL, templates, or long text

### Bug Fixes
- `isinstance(obj, FrankieDB)` cross-namespace failure — fixed with duck typing
  (`hasattr` checks instead) so DB objects work correctly inside `exec()` globals
- `db.delete(table, where)` was intercepted by string/hash delete handler — now
  correctly dispatches based on argument count (2 args = DB call)
- `.count("sub")` on a DB object was routing to `_fk_str_count` — fixed via
  `_fk_count_dispatch` with duck typing
- Transaction `BEGIN`/`COMMIT`/`ROLLBACK` now uses explicit `isolation_level=None`
  with an `_in_tx` flag for correct per-operation autocommit and block rollback

---

## v1.1.0 (2025)

### New Features

**Iterators & Collections**
- `.select do |x|` — filter elements where block is true
- `.reject do |x|` — filter elements where block is false
- `.reduce(init) do |acc, x|` — fold to a single value (also `.inject`)
- `.each_with_object(init) do |x, obj|` — iterate with shared accumulator
- `.any? do |x|` — true if any element matches
- `.all? do |x|` — true if all elements match
- `.none? do |x|` — true if no elements match
- `.count do |x|` — count matching elements (or `.count("sub")` for strings)
- `.flat_map do |x|` — map then flatten one level
- `.take(n)` — first n elements
- `.drop(n)` — all but first n elements
- `.tally` — count occurrences → Hash
- `.compact` — remove nil values
- `.chunk(n)` — split into sub-vectors of size n
- `.zip(other)` — zip two vectors together

**Control Flow**
- `case/when/else/end` — pattern matching on values or conditions
- Bare `case` (no subject) — uses truthy when-clauses

**Destructuring Assignment**
- `a, b, c = [1, 2, 3]` — unpack vector into named variables
- Pads with `nil` if vector is shorter than target count

**String Methods (new)**
- `.chars` — vector of individual characters
- `.bytes` — vector of byte values
- `.lines` — vector of lines
- `.chomp` — remove trailing newline
- `.chop` — remove last character
- `.count("sub")` — count substring occurrences
- `.center(w, pad)` — center in field
- `.ljust(w, pad)` — left-justify in field
- `.rjust(w, pad)` — right-justify in field
- `.squeeze` — collapse consecutive duplicates
- `.tr(from, to)` — translate characters
- `.each_char do |c|` — iterate over characters
- `.each_line do |l|` — iterate over lines
- `.lstrip` / `.rstrip` — directional whitespace trim

**REPL (Interactive Mode)**
- `frankiec repl` — starts the interactive REPL
- `frankiec` with no arguments also launches the REPL
- Multi-line block detection — automatically waits for `end`
- `vars` — show all user-defined variables and functions
- `clear` — reset the session
- `load <file.fk>` — load a file into the current session
- `help` — show available commands
- Persistent state across expressions in a session

### Bug Fixes
- `do...while` body was accidentally consuming the `while` keyword
- Postfix `if`/`unless` now works after `puts` (not just expressions)
- `matches()` and all regex functions had flipped argument order — fixed to `(string, pattern)`
- `s[-5..-1]` negative range ends now parse as `(-5)..(-1)` correctly
- `[x, x * 2]` vector literal with multiplication was mis-parsed as destructuring — fixed with backtracking
- `gen_pipe` method lost its `def` header during code insertion — restored
- `count` method now correctly dispatches: `.count("sub")` for strings, `.count do` for filtering, `.count` for length

### Compiler Version
- Version header in generated files updated to v1.1

---

## v1.0.0 (2025)

### Initial Release

**Core language**
- 7 data types: Integer, Float, String, Boolean, Nil, Vector, Hash
- Full arithmetic: `+`, `-`, `*`, `/`, `//`, `%`, `**`
- String interpolation with `#{}`
- Ranges: `1..10` (inclusive), `1...10` (exclusive)
- Conditionals: `if/elsif/else/end`, `unless/end`
- Postfix `if`/`unless`
- Loops: `while`, `until`, `do...while`, `for...in`
- Iterators: `.times`, `.each`, `.each_with_index`, `.map`
- Functions with `def...end` and explicit `return`
- Named arguments: `func(x, sep: "-")`
- Pipe operator `|>`
- Destructuring (v1.1)

**Collections**
- Vectors with R-style vectorized arithmetic
- Hashes with symbol and integer keys, nil-safe access
- Full method suites for both types

**Standard Library**
- Math: `sqrt`, `abs`, `floor`, `ceil`, `min`, `max`
- Statistics: `sum`, `mean`, `median`, `stdev`, `variance`
- Sequences: `seq`, `linspace`, `rep`, `clamp`
- String formatting: `sprintf`, `paste`
- Regex: `matches`, `match`, `match_all`, `sub`, `gsub`, `=~`, `regex()`
- File I/O: `file_read`, `file_write`, `file_append`, `file_lines`, `file_exists`, `file_delete`
- Type conversion: `to_int`, `to_float`, `to_str`
- Type checking: `is_integer`, `is_float`, `is_string`, `is_vector`, `is_nil`, `is_bool`
- System: `exit`, `argv`, `env`

**Error handling**
- `begin/rescue/ensure/end`
- `raise`

**Multi-file**
- `require "filename"` — load another `.fk` file once

**Tooling**
- `frankiec run` — run a program
- `frankiec build` — compile to Python source
- `frankiec check` — syntax check
- `frankiec version`
- `python3 install.py` — install `frankiec` to `frankie/bin/`
