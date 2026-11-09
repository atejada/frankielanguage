# Showcase

The showcase is a single runnable file that demonstrates every major Frankie feature from v1.0 through v1.22. 

It is the fastest way to see the whole language in one place.

```bash
frankiec run examples/showcase.fk
# or, without a local checkout:
frankiec examples showcase && frankiec run showcase.fk
```

Want the playable versions instead? Grab the standalone [showcase projects](index.md#showcase-projects-v119-v120) (Snake, Pong, Zombie Chat, and more) with `frankiec examples`.

---

## What It Covers

The showcase is divided into 83 numbered sections, one per feature area. Each section is self-contained and prints its own output.

| # | Section | Introduced |
|---|---|---|
| 1 | Error Handling — `begin / rescue / ensure` | v1.0 |
| 2 | Nil Safety — `&.` operator | v1.7 |
| 3 | String Templates — `template()` | v1.7 |
| 4 | File System — `file_mkdir`, `dir_list`, `file_copy` | v1.7 |
| 5 | Regex — `matches`, `match_all`, `sub`, `gsub`, `=~` | v1.0 |
| 6 | File I/O — `file_read`, `file_write`, `file_lines`, `file_append` | v1.0 |
| 7 | Collections & Iterators — `select`, `reject`, `map`, `reduce`, `each_with_object` | v1.1 |
| 8 | Compound Assignment — `+=`, `-=`, `*=`, `//=`, `**=` | v1.6 |
| 9 | R-style Statistics — `mean`, `stdev`, `median`, pipe `\|>` | v1.0 |
| 10 | JSON & CSV | v1.3 |
| 11 | SQLite Database | v1.2 |
| 12 | DateTime | v1.3 |
| 13 | Randomness — `rand_int`, `shuffle`, `sample` | v1.5 |
| 14 | Constants & Loop Control — `next`, `break`, `break value` | v1.5 |
| 15 | Mini Log Analyser — a real-world composition | v1.0 |
| 16 | Lambdas — `->(x) { }`, `.call`, higher-order functions | v1.8 |
| 17 | Hash Merge Operator `\|` | v1.8 |
| 18 | `group_by` | v1.8 |
| 19 | `each_slice` and `each_cons` | v1.8 |
| 20 | Record Types — `record Point(x, y)` | v1.9 |
| 21 | Hash `.dig` — safe deep access | v1.9 |
| 22 | `zip()` — pair two vectors | v1.9 |
| 23 | `frankiec fmt` and `frankiec docs` | v1.9 |
| 24 | String & Vector `*` repetition | v1.10 |
| 25 | Heredoc `<<~TEXT` | v1.10 |
| 26 | `times()` standalone + `flatten(depth)` + `map_with_index` + `pp` | v1.10 |
| 27 | `encode` / `decode` | v1.10 |
| 28 | Implicit Return | v1.11 |
| 29 | String `.replace()` | v1.11 |
| 30 | String `.format(hash)` | v1.11 |
| 31 | `.zip_with` — pair-wise transform | v1.11 |
| 32 | Multiple Return Values | v1.11 |
| 33 | String `.gsub` with Block | v1.12 |
| 34 | Hash `.map_hash` | v1.12 |
| 35 | `round(x, n)` | v1.12 |
| 36 | Vector `.product(other)` | v1.12 |
| 37 | String `.chars` | v1.12 |
| 38 | `rescue FileNotFoundError` from file I/O | v1.12 |
| 39 | `stitch` — package system | v1.13 |
| 40 | `?` in user-defined function names | v1.13 |
| 41 | Hash Destructuring — `{name, age} = user` | v1.14 |
| 42 | Shape Pattern Matching — `when {role: "admin"}` | v1.14 |
| 43 | `spawn` + `timeout` | v1.14 |
| 44 | Ternary Operator — `x > 0 ? "pos" : "neg"` | v1.15 |
| 45 | Keyword Default Params — `port: 5432` | v1.15 |
| 46 | Splat Multi-Assign — `a, *rest = vec` | v1.15 |
| 47 | `const` keyword | v1.15 |
| 48 | `json_encode` + `base64` / `hmac` public | v1.15 |
| 49 | `String#format` + path helpers + date arithmetic | v1.15 |
| 50 | `shell()` + `dotenv()` | v1.16 |
| 51 | `loop do...end` + `\|\|=` | v1.16 |
| 52 | `transform_values` + `transform_keys` | v1.16 |
| 53 | `String#scan` + `Hash#deep_merge` | v1.16 |
| 54 | `frankiecli` stitch | v1.16 |
| 55 | `frankiecache` stitch | v1.16 |
| 56 | First-class Ranges — `.to_vec`, `.include?`, `.step` | v1.17 |
| 57 | User-defined Error Types — `error` / `rescue Type => e` | v1.17 |
| 58 | Namespaced Imports — `import "path" as name` | v1.17 |
| 59 | Record Dot Access — `p1.x` | v1.17 |
| 60 | `parallel_map` — thread-pool map | v1.17 |
| 61 | TCP Sockets — `tcp_connect` / `tcp_serve` | v1.17 |
| 62 | Test Stubs — `stub` / `unstub` | v1.17 |
| 63 | Numeric Underscores + Scientific Notation — `1_000_000`, `2.5e-3` | v1.18 |
| 64 | Set Operations — `.union` / `.intersect` / `.difference` | v1.18 |
| 65 | `enum Status(pending, active, done)` | v1.18 |
| 66 | `benchmark` — friendly timing | v1.18 |
| 67 | WebSockets — `app.websocket` / `ws_connect` | v1.18 |
| 68 | Gradual Type Annotations — `def f(x: Float) -> Float` | v1.19 |
| 69 | UDP Sockets — `udp_listen`, `send_to`, `recv` | v1.19 |
| 70 | TLS Clients — verified `wss://` / `tcp_connect(tls: true)` | v1.19 |
| 71 | v1.19 Tooling — `--debug`, `--coverage`, `docs --html`, stitch URL installs | v1.19 |
| 72 | `frankiegame` — terminal game engine, `run_game` / `max_ticks` | v1.20 |
| 73 | Engine Helpers — `collide?`, `GAME_COLORS` | v1.20 |
| 74 | `frankiecanvas` — same API, real HTML5-canvas pixels | v1.20 |
| 75 | Image Sprites — `load_image` / `draw_image` | v1.21 |
| 76 | WebAudio Synth — `synth_play`, `game_beep` | v1.21 |
| 77 | `file_read_base64` — binary-safe reads | v1.21 |
| 78 | 🧊 Language Freeze (historical) — The Book of Frankie | v1.21 |
| 79 | Matrix Math — `matrix_identity`, `matrix_transpose`, `matrix_scale`, `matrix_multiply` | v1.22 |
| 80 | Complex Numbers — `complex_new`, `complex_abs`, `complex_conj` | v1.22 |
| 81 | Math Intrinsics — `sin`, `cos`, `atan2`, `exp`, `log`, `factorial`, `gcd`, `lcm`, `PI`, `E` | v1.22 |
| 82 | Labeled Loops — `loop :name`, `break :name`, `next :name` | v1.22 |
| 83 | 🧊 Language Freeze — The Book of Frankie | v1.22 |

---

## A Taste

Here is a small excerpt from sections 7, 9, and 16 to show the feel:

```ruby
# ── 7. Collections & Iterators ──
data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

evens = data.select do |x| x % 2 == 0 end
puts evens   # [2, 4, 6, 8, 10]

squared = data.map do |x| x * x end
puts squared   # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

total = data.reduce(0) do |acc, x| acc + x end
puts total   # 55

# ── 9. R-style Statistics ──
nums = [23, 45, 12, 67, 34, 89, 56]
puts mean(nums)      # 46.57...
puts stdev(nums)     # 24.99...
puts median(nums)    # 45
nums |> sum |> puts  # 326

# ── 16. Lambdas ──
double = ->(x) { x * 2 }
add    = ->(a, b) { a + b }
puts double.call(5)    # 10
puts add.call(3, 4)    # 7

def apply(fn, val)
  fn.call(val)
end
puts apply(double, 9)  # 18
```

---

## Running Specific Sections

The showcase runs from top to bottom as a single program. To run just one section, copy the relevant lines into a new file or into the REPL:

```bash
frankiec repl
```

Then paste any section and press Enter.
