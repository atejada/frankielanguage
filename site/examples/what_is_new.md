# What's New — Per Version

Each version of Frankie ships a runnable `whats_new_vX.fk` example file that demonstrates every new feature introduced in that release. Every install (Homebrew, git clone, or unpacked zip) bundles the same set — grab one with `frankiec examples whats_new_v122` or run it straight from the source tree:

```bash
frankiec run examples/whats_new_v122.fk
```

## v1.22 — `whats_new_v122.fk` — "More FORTRAN" 🧮

**Theme:** One last feature pass before the freeze locks back in — leaning harder into Frankie's FORTRAN heritage. Pure Python stdlib, zero external dependencies, as always.

| Feature | Summary |
|---|---|
| Matrix math | `+`/`-`/`/`/`%`/`**` now recurse through nested vectors, so matrices do real element-wise math instead of silently concatenating rows; `matrix_new`, `matrix_identity`, `matrix_shape`, `matrix_transpose`, `matrix_scale`, `matrix_multiply` |
| Complex numbers | `complex_new(real, imag)` exposes Python's native complex type — `+`/`-`/`*`/`/`/`**` all work for free; `complex_real`/`complex_imag`/`complex_conj`/`complex_abs`; prints as `3+4i` |
| Math intrinsics | `sin`/`cos`/`tan`/`asin`/`acos`/`atan`/`atan2`, `exp`/`log`/`log10`/`log2`, `factorial`/`gcd`/`lcm`, and constants `PI`/`E` |
| Labeled loops | `loop`/`while`/`until`/`for` take an optional `:name` label; `break :name`/`next :name` target that exact loop from anywhere nested inside it |
| `:symbol` is its own token | Previously indistinguishable from a plain string at the parser level — needed for `break :label` to be unambiguous; every existing use of `:name` as a value is unaffected |
| 🧊 Language freeze moves to v1.22.0 | This was the explicitly-promised "last big addition" — only bug fixes and doc corrections from here |

---

## v1.21 — `whats_new_v121.fk` — "The Book Edition" 🧊

**Theme:** Real bitmap sprites and a proper synth API for `frankiecanvas`, a binary-safe file primitive underneath both, and a language freeze while the language gets a book written about it. Pure Python stdlib, zero external dependencies, as always.

| Feature | Summary |
|---|---|
| `load_image(path)` + `draw_image(g, x, y, image, w:, h:)` | Real PNG/JPEG/GIF sprites in `frankiecanvas`, client-cached after the first frame; `frankiegame` gets matching calls with a `▒` placeholder for cross-engine parity |
| `synth_play(g, freq:, wave:, dur:, gain:)` | Real WebAudio oscillator tones (sine/square/sawtooth/triangle) in `frankiecanvas`; `game_beep` is now a thin wrapper over it |
| `file_read_base64(path)` | Binary-safe file read + Base64 in one call — what `load_image` uses internally |
| `frankiec fmt` data-loss fix | Single-statement blocks with an assignment or postfix-if/unless body no longer silently collapse to `nil` |
| 🧊 Language freeze | Grammar, stdlib, and stitch APIs locked while "The Book of Frankie" is written against this exact version — patch releases only until the freeze lifts |

---

## v1.20 — `whats_new_v120.fk`

**Theme:** "It's Alive… and Playing" — two game engines, one shared API. Pure Python stdlib, zero external dependencies, as always.

| Feature | Summary |
|---|---|
| `frankiegame` | Terminal game engine — raw keys via stdlib `termios`, double-buffered ANSI rendering, headless-safe for CI |
| `frankiecanvas` | Browser game engine — same API, real pixels via HTML5 canvas streamed over WebSockets, multiplayer for free |
| Shared engine API | `game_new`, `on_key`, `on_tick`, `draw`, `draw_sprite`, `collide?`, `run_game`, `render_frame`, `max_ticks:` for CI |
| `on_player_key` (canvas-only) | Per-player input — every connected browser tab is a player |
| `frankiec new --game` | Scaffold a playable starter with `frankiegame` pre-installed |
| Showcase games | 🐍 `snake` (terminal), 🧟 `zombie_invaders` (browser), 🏓 `pong` (browser, multiplayer) |

**Patch — v1.20.1:** arrow keys fixed in terminal games (`term_key()` now reads raw bytes); `frankiec examples [name]` to browse/copy bundled examples; official Homebrew formula (`brew install atejada/frankie/frankie`).

---

## v1.19 — `whats_new_v119.fk`

**Theme:** "Under the microscope" — types, a real debugger, coverage. Pure Python stdlib, zero external dependencies, as always.

| Feature | Summary |
|---|---|
| `def area(r: Float) -> Float` | Gradual type annotations — optional, checked statically by `check`/CI/LSP |
| Stepping debugger | `(fkdb)` gains `s`/`n`/`stack`; `frankiec run --debug` breaks at line 1 |
| `frankiec test --coverage` | Line coverage report + `.frankie_coverage.json`, read by the LSP |
| TLS + UDP | `tcp_connect(tls: true)`, `ws_connect("wss://...")`, `udp_listen`/`udp_send` |
| `frankiec docs --html` | Doc-comments rendered as styled single-page HTML |
| Stitch installs from any URL | `frankiec stitch install https://.../foo.fk`, pinned in `stitch.lock` |
| Showcase projects | 🧟 `zombie_chat`, ⚡ `word_reanimator`, 💰 `frankie_ledger` |

---

## v1.18 — `whats_new_v118.fk`

**Theme:** "From projects to products" — edit (LSP), ship (bundle), connect (WebSockets), debug (breakpoint). Pure Python stdlib underneath, zero external dependencies, as always.

| Feature | Summary |
|---|---|
| `frankiec lsp` | Language Server over stdio — live diagnostics, completion, hover docs in VS Code, Neovim, Helix, Zed |
| `frankiec bundle` | Compile a program + all `require`/`import`/`stitch` targets into ONE self-contained `.py` — runs anywhere with Python 3.8+ |
| WebSockets | `app.websocket("/ws/:room") do \|ws\| ... end` server + `ws_connect(url)` client — hand-rolled RFC 6455 |
| `breakpoint` | Pause into a scoped debug REPL — `vars`, `where`, any Frankie expression; skipped when stdin isn't a tty |
| `enum Status(pending, active, done)` | Named symbolic values — `Status.pending`, `.values`, `.include?`, `case/when` matching |
| `benchmark "label" do ... end` | Time a block — prints `⏱ label: 12.3ms`, returns elapsed ms |
| Set operations | `.union` / `.intersect` / `.difference` — order-preserving, deduplicating |
| Numeric literals | Underscores `1_250_000` + scientific notation `6.022e23` |
| Project-wide tooling | `frankiec check .` and `frankiec fmt --write .` recurse into directories |
| Stitch lockfile | `stitch.lock` with sha256 pins — `frankiec stitch verify` / `update` |

---

## v1.17 — `whats_new_v117.fk`

**Theme:** "Programs that grow" — namespacing, real static checking, typed errors, first-class ranges, easy concurrency, TCP sockets, and a stitch installer. Everything still pure Python stdlib, zero dependencies.

| Feature | Summary |
|---|---|
| `import "path" as name` | Load a `.fk` file into its own namespace — `geo.circle_area(5)`, `geo.PI`; modules cached; `require` unchanged |
| `error TypeName` | User-defined error types — `raise ValidationError, "msg"`, caught with `rescue ValidationError => e` |
| `rescue Type => e` | New Ruby-style rescue binding (old `rescue Type e` form still works) |
| First-class ranges | `r = 1..10` prints as `1..10`; `.to_vec`, `.include?`, `.step(n)`, `.sum`, `.first`, `.last` |
| Range `case/when` | `when 90..100` tests membership — works with range values in variables too |
| Record dot access | `p1.x` instead of `p1["x"]` — hash-style access still works |
| `parallel_map` | `parallel_map(urls, workers: 8) do \|u\| ... end` — thread-pool map, results in input order |
| TCP sockets | `tcp_connect` / `tcp_listen` / `tcp_serve` — `send_line`, `recv_line`, `peer`, threaded server loop |
| `frankiec check` | Real static analysis — undefined names, wrong arg counts, unused locals; `--strict` for CI |
| Cross-file tracebacks | Runtime errors point at the right file *and* line, in requires, imports and stitches |
| `test` groups | `test "name", tags: ["slow"] do ... end` — filter with `--filter` / `--tag` |
| `stub` / `unstub` | Swap any global function during tests — `stub("shell", ->(cmd) { fake })` |
| Stitch installer | `frankiec stitch install frankiecolor [--global]`, `frankiec stitch list` |
| REPL upgrades | Bare expressions echo `=> value`, `_` holds last result, `help <function>` shows docs |

---

## v1.16.2 — `whats_new_v116.fk`

**Theme:** System Integration & Scripting Power — shell commands, environment loading, infinite loops, nil-coalescing assign, richer hash and string tools, and three new stitches for email, CLI parsing, and caching.

| Feature | Summary |
|---|---|
| `shell(cmd)` | Run an OS command — returns `{stdout, stderr, exit_code, ok}` |
| `dotenv(path)` | Load a `.env` file into the process environment — returns hash of loaded vars |
| `loop do...end` | Infinite loop, exits only via `break` — cleaner than `while true` |
| `\|\|=` | Nil-coalescing assign — `config \|\|= load_defaults()` |
| `Hash#transform_values` | New hash with same keys, block-transformed values |
| `Hash#transform_keys` | New hash with block-transformed keys, same values |
| `Hash#deep_merge` | Recursive nested hash merge — unlike shallow `.merge` / `\|` |
| `String#scan` | Extract all regex matches as a vector |
| `frankiemail` stitch | Send email via SMTP — plain text, HTML, CC/BCC — zero deps |
| `frankiecli` stitch | CLI argument parsing — flags, options, subcommands — zero deps |
| `frankiecache` stitch | In-memory TTL cache — set/get/delete/clear/size — zero deps |

---

## v1.15 — `whats_new_v115.fk`

**Theme:** Language Polish & Developer Ergonomics — cleaner syntax, more powerful stdlib, and two new web stitches.

| Feature | Summary |
|---|---|
| `Ternary operator` | `x > 0 ? "pos" : "neg"` — right-associative, valid in any expression position |
| `Keyword default params` | `def connect(host, port: 5432, ssl: true)` — `:` syntax now accepted alongside `=` |
| `Splat in multi-assign` | `a, *rest = [1,2,3]` and `first, *mid, last = arr` — capture remaining elements |
| `const` keyword | Explicit constant declaration — reassignment warns and preserves original value |
| `fn.(args)` everywhere | Lambda call syntax now works in all contexts, not just web middleware |
| `json_encode` | Preferred name for `json_dump`; accepts `pretty: true` |
| `hmac_sign` / `hmac_verify` | Promoted from internal `_fk_*` to public stdlib — signed tokens and webhook verification |
| `base64_encode` / `base64_decode` | Standard Base64 encode/decode — useful for HTTP Basic Auth headers |
| `String#format` | Named placeholder substitution: `"Hello, {name}!".format({name: "Alice"})` |
| `.chars` / `.bytes` | Unicode-aware character vector and byte values from a string |
| Path helpers | `path_join`, `path_dirname`, `path_basename`, `path_extname`, `path_stem`, `path_absolute` |
| Date arithmetic | Date objects support `+` / `-` (days) and comparison operators |
| `zip_with` block | `a.zip_with(b) do \|x, y\| x + y end` — element-wise combination |
| `assert_not_nil` / `assert_in` | New test assertions for nil checks and membership |
| `req.query_int/float/bool` | Typed query parameter helpers with defaults |
| `frankieauth` stitch | HTTP Basic Auth + Bearer token auth — zero dependencies |
| `frankieratelimit` stitch | Per-IP sliding-window rate limiting — zero dependencies |
| `frankiec check` | Parse without executing — useful in CI |
| `frankiec new` | Scaffold a complete project layout |
| `frankiec watch` | Official file watcher — re-run on save |

---

## v1.14 — `whats_new_v114.fk`

**Theme:** Frankie for real web apps — concurrency, templates, signed cookies, middleware, static files, and two language upgrades that make working with data cleaner everywhere.

| Feature | Summary |
|---|---|
| `spawn { }` | Run a block in a background thread — response goes out immediately, work continues behind the scenes |
| `timeout(n) { }` | Kill a block that exceeds `n` seconds — raises `TimeoutError`, wrap in begin/rescue for graceful fallback |
| `Async routes` | `app.get_async` / `app.post_async` etc. — non-blocking handlers, use `await` inside for slow I/O |
| `Middleware stack` | `app.use do |req, next_fn|` — chain auth, logging, and rate-limiting; call `next_fn.(req)` to continue or return early to short-circuit |
| `Static file serving` | `app.static("./public")` — serve a directory at `/`; `app.static("./assets", "/static")` for a URL prefix |
| `frankietemplate` | Mustache-compatible template stitch — `{{ var }}, {{{ raw }}}, {{# section }}, {{^ inverted }}, {{> partial }}, {{! comment }}` |
| `frankiecookie` | HMAC-signed cookies via Python's `hmac` stdlib — `set_signed_cookie` / `get_signed_cookie` returns `nil` if tampered |
| `Hash destructuring` | `{name, age} = user` — pull symbol keys directly into variables; missing keys become `nil` |
| `Shape pattern matching` | `case user when {role: "admin"}` — `when` now accepts hash literals, matches on subset of keysfrankiec new scaffold |
| `Global stitch install` | `install.py` now correctly copies all stitches to `~/.frankie/stitches/` and lists each file installed |
| `page_links` docs | Full web pagination example added — `paginate` + `page_slice` + `page_links` + `frankietemplate` working together | 

## v1.13.1 — `whats_new_v1131.fk`

**Theme:** Gap closers — no new syntax, everything real programs needed and didn't have.

| Feature | Summary |
|---|---|
| `frankiestring` v2 | Rewritten stitch: `pad_left`, `pad_right`, `truncate`, `slugify`, `word_wrap`, `indent_lines` — replaces clunky `lfill`/`rfill` API |
| `Vector .sum do \|x\|` | Block form now works — projected sum without a two-step `.map` + `sum` |
| `Vector .flat_map do \|x\|` | Multi-line block bodies now parse correctly |
| `assert_approx_eq(a, b, delta, msg)` | Float comparison in tests with configurable delta (default `0.001`) |
| `run_tests()` | Now a public stdlib function — callable from any `.fk` file, not just `frankiec test` |
| `session(req, resp)` | Cookie-backed session hash — read, mutate, `.save()`. Zero server state, single JSON cookie `_fk_session` |
| `frankiec fmt` blank lines | Intentional blank lines between statement groups inside function bodies now preserved |
| `frankiec fmt` multi-line threshold | Hashes/vectors whose inline form exceeds 60 chars auto-expand to one element per line |
| `frankiec fmt` idempotency | Running `fmt --write` twice now produces identical output — safe for pre-commit hooks and CI |
| Heredoc in `do...end` blocks | Fixed lexer bug — heredocs now work anywhere a string expression is valid |
| `Hash.each do \|k, v\|` | Two-parameter block iteration confirmed and end-to-end tested |
| Symbol key round-trip in `fmt` | `{host: "x"}` no longer becomes `{"host": "x"}` after formatting |

---

## v1.13 — `whats_new_v113.fk`

**Theme:** Stitches — a zero-dependency, zero-registry package system.

| Feature | Summary |
|---|---|
| `stitch "name"` keyword | Load a package by name — resolves from `./stitches/` then `~/.frankie/stitches/` |
| `frankieforms` | Form field validation — `required`, `min_length`, `max_length`, `email`, `numeric`, `alpha`, `matches_pattern` |
| `frankietable` | ASCII table rendering from a vector of hashes — `table(rows)` / `table(rows, cols)` |
| `frankiecolor` | ANSI color helpers — `red`, `green`, `yellow`, `bold`, `success`, `error`, `warn`, `info`, `colorize`, `strip_color` |
| `frankiepager` | Pagination math — `paginate(opts)`, `page_slice(items, page, per_page)`, `page_links(pager, url_template)` |
| `frankieconfig` | Layered config loading: defaults → JSON file → env vars → overrides, with type coercion |
| `frankiestring` | String utilities stitch (v1, superseded by v2 in v1.13.1) |
| `?` in function names | `def even?(n)` now works — `?` compiled transparently to `_q` in generated Python |

---

---

## v1.12 — `whats_new_v112.fk`

**Theme:** Filling stdlib gaps, fixing error handling, tooling polish.

| Feature | Summary |
|---|---|
| `String .gsub` with block | Transform each regex match via a block — `"hello".gsub("[aeiou]") do \|m\| m.upcase end` |
| `Hash .map_hash` | Transform a hash into a new hash — block returns `[new_key, new_value]` |
| `round(x, n)` | Round to n decimal places — `round(3.14159, 2)` → `3.14` |
| `Vector .product(other)` | Cartesian product — `[1,2].product([3,4])` → `[[1,3],[1,4],[2,3],[2,4]]` |
| `String .chars` | Promoted to first-class documented method — chains into iterators |
| `each_with_object` + Hash | Hash accumulator now explicitly documented and tested |
| `assert_match` / `assert_nil` | Two new test runner assertions |
| `rescue FileNotFoundError` | `file_read` and `file_lines` now raise the correct type |
| `frankiec watch` | Re-run on save — `frankiec watch main.fk` |
| `frankiec repl --no-banner` | Headless REPL for scripts and pipes |
| Friendlier runtime errors | Type errors now say "Integer and String", not Python internals |

---

## v1.11 — `whats_new_v111.fk` *(in docs/13_v111_features.md)*

**Theme:** Ergonomics — removing paper cuts, making the common case obvious.

| Feature | Summary |
|---|---|
| Implicit return | Last expression in a function is returned automatically |
| Inline `if` expression | `grade = if score >= 90 then "A" else "B" end` |
| `String .replace(old, new)` | The method new users always reach for first |
| `String .format(hash)` | `"Hello, {name}!".format({name: "Alice"})` |
| `.zip_with do \|a, b\|` | Pair-wise transform two vectors in one pass |
| Multiple return values | `lo, hi = min_max(data)` — destructuring from functions |
| `frankiec check` boxed errors | Parse errors now use the same box as runtime errors |
| REPL multi-line history | `↑` recalls a full `def...end` block |
| `frankiec fmt` heredoc support | Heredoc bodies preserved verbatim during formatting |
| `String .delete(chars)` | `"hello".delete("l")` → `"heo"` — documented |

---

## v1.10 — `whats_new_v110.fk`

**Theme:** Expressiveness — more ways to write less.

| Feature | Summary |
|---|---|
| `String * n` / `Vector * n` | `"ha" * 3` → `"hahaha"` · `[0] * 5` → `[0,0,0,0,0]` |
| Heredoc `<<~TEXT` | Multi-line strings with auto indent-stripping and interpolation |
| `times()` standalone | `times(5) do \|i\|` alongside `5.times do` |
| `flatten(depth)` | Flatten exactly n levels — `flatten(1)` vs deep `flatten` |
| `map_with_index` | `v.map_with_index do \|x, i\|` |
| `pp` — pretty print | `pp({host: "localhost", port: 3000})` |
| `encode` / `decode` | `"hi".encode` → `[104, 105]` · `[104, 105].decode` → `"hi"` |
| Exit codes | `exit(0)` / `exit(1)` — propagated to shell |
| `--help` flag | `frankiec run --help` etc. |

---

## v1.9 — `whats_new_v19.fk`

**Theme:** Safety and tooling.

| Feature | Summary |
|---|---|
| Hash `.dig` | `config.dig("db", "host")` — safe nested access, returns nil |
| `zip()` standalone | `zip([1,2,3], ["a","b","c"])` → `[[1,"a"],[2,"b"],[3,"c"]]` |
| Record types | `record Point(x, y)` — named data objects built on Hash |
| `frankiec fmt` | AST-based auto-formatter with `--write` and `--check` |
| `frankiec docs` | Extract `##` doc-comments to Markdown |
| REPL readline + history | Arrow keys, Ctrl+R, tab completion, persistent `~/.frankie_history` |
| `.env` auto-loading | `.env` in the project root is loaded automatically at startup |

---

## v1.8 — `whats_new_v18.fk`

**Theme:** Functional programming and collection power.

| Feature | Summary |
|---|---|
| Lambdas | `->(x) { x * 2 }` — storable, passable first-class functions |
| Hash merge `\|` | `h1 \| h2` — right wins on conflict, chains naturally |
| `group_by` | Bucket elements into a hash of arrays by block result |
| `each_slice(n)` | Iterate in non-overlapping chunks |
| `each_cons(n)` | Iterate with a sliding window |

---

## v1.7 — `whats_new_v17.fk`

**Theme:** Safety and real-world utility.

| Feature | Summary |
|---|---|
| Nil safety `&.` | `name&.upcase` returns nil instead of crashing |
| `template(str, hash)` | `{{key}}` placeholder substitution |
| File system ops | `file_mkdir`, `dir_exists`, `dir_list`, `file_copy`, `file_rename` |
| `assert_raises_typed` | Test that a specific error type is raised |

---

## v1.6 — `whats_new_v16.fk`

**Theme:** Control flow completeness.

| Feature | Summary |
|---|---|
| Compound assignment | `+=`, `-=`, `*=`, `/=`, `//=`, `**=`, `%=` |
| Typed rescue | `rescue ZeroDivisionError e` — catch specific error types |
| `.find` / `.detect` | First element matching a block condition |
| `frankiec test` | Built-in test runner |

---

## v1.5 — `whats_new_v15.fk`

**Theme:** Loop control, randomness, and collection utilities.

| Feature | Summary |
|---|---|
| `next` | Skip to next iteration — like `continue` |
| `break` | Exit a loop early |
| `break value` | Exit with a result — `result = loop.break n * 10` |
| Constants | `UPPER_CASE` — warn on reassignment |
| `rand_int`, `rand_float` | Random number functions |
| `shuffle`, `sample` | Random collection operations |
| `sleep(n)` | Pause execution |
| `sort_by` | Sort by computed key |
| `min_by`, `max_by`, `sum_by` | Aggregate by key |
| `unzip` | Split vector of pairs into two vectors |

---

## v1.4 — `whats_new_v14.fk`

**Theme:** Built-in web server.

| Feature | Summary |
|---|---|
| Web server | `web_app()` — Sinatra-style routing |
| Route definitions | `.get`, `.post`, `.put`, `.delete`, `.patch` |
| Path parameters | `/users/:id` → `req.params["id"]` |
| Query parameters | `req.query["page"]` |
| JSON body | `req.json` — auto-parsed |
| Before/after filters | `app.before do`, `app.after do` |
| Response helpers | `html_response`, `json_response`, `redirect`, `halt` |

---

## v1.3 — `whats_new_v13.fk`

**Theme:** External data and project structure.

| Feature | Summary |
|---|---|
| JSON | `json_read`, `json_write`, `json_parse`, `json_dump` |
| CSV | `csv_read`, `csv_write`, `csv_parse`, `csv_dump` |
| DateTime | `now()`, `today()`, `date_parse()`, `date_from()` |
| HTTP client | `http_get`, `http_post`, `http_put`, `http_delete` |
| `frankiec new` | Project scaffolding |

---

## v1.2 *(no whats_new file)*

**Theme:** Database and multi-file programs.

| Feature | Summary |
|---|---|
| SQLite built-in | `db_open`, `db.query`, `db.insert`, `db.exec`, transactions |
| `require` | Load other `.fk` files — each loaded at most once |

---

## v1.1 — `whats_new_v11.fk`

**Theme:** Collection power.

| Feature | Summary |
|---|---|
| `select` / `reject` | Filter a vector by a condition |
| `reduce` / `inject` | Fold a vector into a single value |
| `any?` / `all?` / `none?` | Boolean tests across a collection |
| `each_with_object` | Iterate with a shared accumulator |
| `take`, `drop`, `chunk`, `tally`, `compact` | Collection utilities |
| `flat_map`, `zip` | Flattening and pairing |
| `case / when` | Pattern matching on values |
| Destructuring assignment | `a, b, c = [1, 2, 3]` |
| New string methods | `.chomp`, `.chop`, `.center`, `.ljust`, `.rjust`, `.squeeze`, `.tr` |
| `count` with block | Count elements matching a condition |

---

## v1.0 *(initial release)*

The foundation: variables, strings with `#{}` interpolation, integers, floats, booleans, nil, vectors, hashes, `if/elsif/else/unless`, `while/until/for`, functions with default params, `begin/rescue/ensure`, `raise`, regex (`matches`, `match_all`, `sub`, `gsub`, `=~`), file I/O (`file_read`, `file_write`), R-style statistics (`mean`, `stdev`, `median`, `sum`), vectorised arithmetic, the pipe operator `|>`, `seq`, `linspace`, the REPL, and `frankiec run/build/check`.
