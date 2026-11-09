# Testing

## Overview

Frankie has a built-in test runner — no external libraries, no configuration. 

Write assertions in any `.fk` file and run it with `frankiec test`. 

Every assertion prints a ✓ or ✗ in real time, and the runner exits with code 1 if any test fails, making it CI-friendly.

---

## Running Tests

```bash
frankiec test                  # runs test.fk in the current directory
frankiec test mytest.fk        # runs a specific file
frankiec test --filter math    # v1.17 — only test groups whose name contains "math"
frankiec test --tag slow       # v1.17 — only test groups tagged "slow"
frankiec test --coverage       # v1.19 — report line coverage after the run
```

Output looks like:

```
╔══ Frankie Test Runner ════════════════════════════════
║  test.fk
╠═══════════════════════════════════════════════════════
  ✓  addition works
  ✓  mean of [1,2,3] is 2.0
  ✗  expected 4, got 5
  ✓  FileNotFoundError raised on missing file

╠═══════════════════════════════════════════════════════
║  3/4 passed, 1 failed  (12.3ms)
╚═══════════════════════════════════════════════════════
```

---

## Assertions

All assertion functions are available without any import inside a test file.

### `assert_eq(actual, expected, msg)`

Equality

The workhorse. Passes if `actual == expected`.

```ruby
assert_eq(2 + 2,            4,     "addition")
assert_eq(mean([1,2,3]),    2.0,   "mean of 1..3")
assert_eq("hello".upcase,  "HELLO", "upcase")
assert_eq([1,2,3].length,  3,     "vector length")
```

### `assert_neq(actual, expected, msg)`

Inequality

Passes if `actual != expected`.

```ruby
assert_neq(rand_int(1, 100), 0, "rand_int never returns 0")
assert_neq("hello", "",         "non-empty string")
```

### `assert_true(value, msg)`

Truthy

Passes if `value` is truthy (anything except `false` and `nil`).

```ruby
assert_true(file_exists("/tmp"),         "tmp dir exists")
assert_true([1,2,3].include?(2),         "include? works")
assert_true("hello".start_with?("he"),   "start_with?")
assert_true(5 > 3,                       "comparison")
```

### `assert_nil(value, msg)`

Nil check

Passes if `value` is `nil`.

```ruby
assert_nil(find_user(999),      "missing user returns nil")
assert_nil([1,2,3].find do |x| x > 9 end, "find with no match")
assert_nil({a: 1}["missing"],   "missing hash key is nil")
```

### `assert_match(value, pattern, msg)`

Regex match

Passes if `pattern` matches anywhere in `value`.

```ruby
assert_match("alice@example.com", "\\w+@\\w+\\.\\w+", "valid email")
assert_match("2024-03-15",        "^\\d{4}-\\d{2}-\\d{2}$", "ISO date")
assert_match("Error: timeout",    "timeout",  "error message contains timeout")
```

### `assert_not_nil(value, msg)`

Non-nil check *(introduced in v1.15)*

Passes if `value` is anything other than `nil`.

```ruby
assert_not_nil(find_user(1),        "user exists")
assert_not_nil(result["id"],        "result has an id")
assert_not_nil("",                  "empty string is not nil")
```

### `assert_in(value, collection, msg)`

Membership check *(introduced in v1.15)*

Passes if `collection` includes `value`.

```ruby
assert_in("admin", roles,    "user has admin role")
assert_in(42,      valid_ids, "id is valid")
assert_in("fr",    ["en", "fr", "de"], "French supported")
```

### `assert_approx_eq(actual, expected, delta, msg)`

Approximate float equality *(introduced in v1.13.1)*

Passes if `|actual - expected| <= delta`. Default delta is `0.001`.

```ruby
assert_approx_eq(3.14159, 3.14, 0.01,   "pi rounded")
assert_approx_eq(mean([1,2,3]), 2.0, 0.001, "mean")
assert_approx_eq(0.1 + 0.2, 0.3, 0.0001,   "float addition")
```

### `assert_raises(fn, msg)`

Any error raised

Passes if calling `fn` raises any error.

```ruby
assert_raises(->() { x = 1 // 0 }, "division by zero raises")

assert_raises(->() { file_read("/no/such/file.txt") }, "missing file raises")
```

### `assert_raises_typed(fn, type, msg)`

Specific error type

Passes if calling `fn` raises exactly the specified error type.

```ruby
assert_raises_typed(->() { x = 1 // 0 }, "ZeroDivisionError", "division raises ZeroDivisionError")

assert_raises_typed(->() { file_read("/no/such/file.txt") }, "FileNotFoundError", "missing file raises FileNotFoundError")

assert_raises_typed(->() { "hello" + 42 }, "TypeError", "string + int raises TypeError")
```

Supported type strings: `"ZeroDivisionError"`, `"FileNotFoundError"`, `"TypeError"`, `"ValueError"`, `"IndexError"`, `"KeyError"`, `"NameError"`, `"AttributeError"`, `"TimeoutError"`, `"RuntimeError"`, `"Exception"` — plus any user-defined error type declared with `error` (v1.17).

---

## Test Groups — `test` (v1.17)

Group related assertions into named, filterable blocks with `test`:

```ruby
test "math basics" do
  assert_eq(1 + 1, 2, "addition works")
  assert_eq(abs(-5), 5, "abs negative")
end

test "slow integration", tags: ["slow", "network"] do
  assert_true(http_get("http://localhost:3000/health")["ok"])
end
```

Run a subset from the command line:

```bash
frankiec test                    # run everything
frankiec test --filter math      # groups whose name contains "math"
frankiec test --tag slow         # groups tagged "slow"
```

Skipped groups are reported in the summary. `test` is a contextual keyword — a variable named `test` keeps working.

---

## Test Coverage — `--coverage` (v1.19)

Add `--coverage` to see which lines your tests actually exercised:

```bash
frankiec test tcov.fk --coverage
```

```
╠══ Coverage ═══════════════════════════════════════════
║   80.0%  tcov.fk  missing: 6
║  ── overall: 80.0% (4/5 lines)
```

- Percentages and missing-line ranges are mapped back to `.fk` source.
- Writes `.frankie_coverage.json` in the current directory.
- The LSP reads that file — after a coverage run, uncovered lines show up as unobtrusive hints directly in your editor.

---

## Stubbing — `stub` / `unstub` (v1.17)

Swap any global function during a test — including stdlib functions like `shell` and `http_get` — to test without side effects:

```ruby
test "stubbing" do
  stub("shell", ->(cmd) { {ok: true, stdout: "fake", exit_code: 0} })
  assert_eq(shell("anything")["stdout"], "fake")
  unstub("shell")    # restore the real shell
end
```

`unstub(name)` restores one function; `unstub()` with no arguments restores everything.

---

## Structuring Tests

Use `test` groups (above), or simply place assertions anywhere in the file — they are collected and reported together. For readability, group related assertions with comments:

```ruby
# test.fk

# ── Math ──────────────────────────────────────────────────────
assert_eq(abs(-5),          5,    "abs negative")
assert_eq(abs(5),           5,    "abs positive")
assert_eq(round(3.14159,2), 3.14, "round to 2dp")
assert_eq(clamp(15, 0, 10), 10,   "clamp above max")
assert_eq(clamp(-5, 0, 10), 0,    "clamp below min")

# ── String ────────────────────────────────────────────────────
assert_eq("hello".upcase,          "HELLO",  "upcase")
assert_eq("  hi  ".strip,          "hi",     "strip")
assert_eq("a,b,c".split(",").length, 3,      "split count")
assert_match("frank123", "\\d+",             "contains digits")

# ── Vector ────────────────────────────────────────────────────
v = [3, 1, 4, 1, 5, 9, 2, 6]
assert_eq(v.length, 8,         "length")
assert_eq(min(v),   1,         "min")
assert_eq(max(v),   9,         "max")
assert_eq(v.sort[0], 1,        "sort first")
assert_eq(v.uniq.length, 7,    "uniq removes one duplicate")

# ── Error handling ────────────────────────────────────────────
assert_raises_typed(->() { 1 // 0 }, "ZeroDivisionError", "division by zero")

assert_raises_typed(->() { file_read("/no/such/file.txt") }, "FileNotFoundError", "missing file")
```

---

## Testing Your Own Functions

Require your lib files the same way `main.fk` does:

```ruby
# test.fk
require "lib/math_utils"
require "lib/string_utils"

# ── math_utils ────────────────────────────────────────────────
assert_eq(circle_area(1),    round(3.14159, 2), "unit circle")
assert_eq(circle_area(0),    0.0,               "zero radius")
assert_eq(factorial(5),      120,               "factorial 5")
assert_eq(factorial(0),      1,                 "factorial 0")

# ── string_utils ──────────────────────────────────────────────
assert_eq(slugify("Hello World"),  "hello-world",   "slugify basic")
assert_eq(slugify("  spaces  "),   "spaces",        "slugify trims")
assert_eq(truncate("hello", 3),    "hel...",        "truncate")
assert_nil(truncate("hi", 10),     "no truncate needed returns nil")
```

---

## Testing File I/O

Use a temporary path and clean up in every test:

```ruby
path = "/tmp/fk_test_#{rand_int(1000, 9999)}.txt"

# Write and read back
file_write(path, "hello frankie\n")
assert_true(file_exists(path),            "file was created")
assert_eq(file_read(path).strip, "hello frankie", "content correct")

# Append
file_append(path, "second line\n")
assert_eq(file_lines(path).length, 2,    "two lines after append")

# Delete
file_delete(path)
assert_true(not file_exists(path),       "file deleted")

# FileNotFoundError
assert_raises_typed(->() { file_read("/no/such/file.txt") }, "FileNotFoundError", "missing file raises correctly")
```

---

## Watch Mode

Run tests automatically whenever you save:

```bash
frankiec watch test.fk --test
```

Every time you save `test.fk` (or any file, since watch triggers on the target file), the test suite re-runs. 

Combine with `require` to automatically pick up changes in your lib files too.

---

## Quick Reference

| Assertion | Passes when |
|---|---|
| `assert_eq(actual, expected, msg)` | `actual == expected` |
| `assert_neq(actual, expected, msg)` | `actual != expected` |
| `assert_true(value, msg)` | `value` is truthy |
| `assert_nil(value, msg)` | `value` is nil |
| `assert_not_nil(value, msg)` | `value` is not nil |
| `assert_in(value, collection, msg)` | `collection` includes `value` |
| `assert_approx_eq(a, b, delta, msg)` | `|a - b| <= delta` (default delta `0.001`) |
| `assert_match(value, pattern, msg)` | pattern matches anywhere in value |
| `assert_raises(fn, msg)` | calling `fn` raises any error |
| `assert_raises_typed(fn, type, msg)` | calling `fn` raises that specific type |

| Helper (v1.17) | Purpose |
|---|---|
| `test "name", tags: [...] do ... end` | Named, filterable test group |
| `stub(name, fn)` | Replace a global function for the test |
| `unstub(name)` / `unstub()` | Restore one / all stubbed functions |
| `frankiec test --coverage` | Report line coverage; writes `.frankie_coverage.json` (v1.19) |
