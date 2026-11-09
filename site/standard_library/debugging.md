# Debugging & Benchmarking

Introduced in **v1.18**, with a full stepping debugger added in **v1.19**. A debugger you can drop anywhere, and friendly block timing — both built into the language, zero dependencies.

---

## `breakpoint` — Stepping Debugger

Drop `breakpoint` anywhere. When execution reaches it in a terminal, the program pauses into a scoped REPL:

```ruby
def price(qty)
  total = qty * 4.5
  breakpoint
  total
end

price(10)
```

```
🧟 breakpoint — pricing.fk:3
   ──▶ 3 │   breakpoint
   (c)ontinue · s(tep) · n(ext) · stack · vars · where · exit · or type any Frankie expression

(fkdb) vars
  qty = 10
  total = 45.0
(fkdb) total * 2
  => 90.0
(fkdb) c
```

### Commands

| Command | Effect |
|---|---|
| `c` | Continue execution |
| `s` | Step — execute one line, stepping into calls *(v1.19)* |
| `n` | Next — one line, stepping over calls *(v1.19)* |
| `stack` | Show the Frankie call stack, innermost first *(v1.19)* |
| `vars` | List local variables in the paused scope (clean — no stdlib noise) |
| `where` | Show the current file and line |
| `exit` | Abort the program |
| *anything else* | Evaluated as a Frankie expression against the paused scope |

Stepping is line-accurate everywhere thanks to the compiler's line maps — including inside `require`d files.

### Debug from the very first line (v1.19)

```bash
frankiec run --debug app.fk
```

Breaks before the first statement runs, so you can step through a program you've never read — no need to add a `breakpoint` line yourself.

### Conditional breakpoints

Postfix conditions work:

```ruby
breakpoint if qty > 100
```

### CI-safe by design

When stdin isn't a terminal (CI, pipes, `frankiec test` in a pipeline), breakpoints print a notice and are skipped — your test runs never hang.

`breakpoint` is a contextual keyword — a variable named `breakpoint` keeps working.

---

## `benchmark` — Friendly Timing

Times a block, prints the elapsed time, and returns the elapsed milliseconds:

```ruby
ms = benchmark "crunch" do
  heavy_work()
end
# ⏱  crunch: 132.4ms

benchmark do        # label optional
  quick_thing()
end
```

Because it returns the elapsed ms, you can assert on it in tests or collect timings in a vector:

```ruby
timings = [100, 1000, 10000].map do |n|
  benchmark "n=#{n}" do
    (1..n).to_vec.map do |x| x * x end
  end
end
puts timings
```

`benchmark` is a contextual keyword — a variable named `benchmark` keeps working.

---

## Quick Reference

| Syntax | Description |
|---|---|
| `breakpoint` | Pause into a scoped debug REPL (skipped when stdin isn't a tty) |
| `breakpoint if cond` | Conditional breakpoint |
| `frankiec run --debug app.fk` | Break at the first line automatically (v1.19) |
| `(fkdb) s` / `n` / `stack` | Step in / step over / show call stack (v1.19) |
| `benchmark "label" do ... end` | Time a block — prints `⏱ label: 12.3ms`, returns elapsed ms |
| `benchmark do ... end` | Same, without a label |
