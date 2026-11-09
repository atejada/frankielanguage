# Strings

## `What is a String?`

A string is a sequence of UTF-8 characters. Strings are used for text — output, input, file content, keys, messages, templates. In Frankie, strings are **immutable** — every method that transforms a string returns a new string rather than modifying the original.

```ruby
s = "Hello, Frankie!"
puts s.upcase    # "HELLO, FRANKIE!"
puts s           # "Hello, Frankie!" — unchanged
```

---

## `String Literals`

The standard form is a double-quoted string:

```ruby
name = "Alice"
```

Single-quoted strings are also valid but do **not** support interpolation or escape sequences:

```ruby
puts 'Hello, #{name}'   

→ Hello, #{name}  — literal, not interpolated
```

## `Triple-quoted strings` 

Span multiple lines:

```ruby
message = """
  Hello, Alice!
  Welcome to Frankie.
"""
puts message
```

`Heredocs` 

(<<~DELIM) are the cleanest way to write multi-line strings in code. 

The `~` variant strips common leading indentation automatically:

```ruby
query = <<~SQL
  SELECT name, age
  FROM employees
  WHERE age > 30
  ORDER BY name
SQL
puts query
```

Heredocs support `#{}` interpolation just like regular strings:

```ruby
table = "employees"
limit = 10

query = <<~SQL
  SELECT * FROM #{table}
  LIMIT #{limit}
SQL
```

---

## `String Interpolation`

Double-quoted strings support `#{}` to embed any expression directly:

```ruby
name  = "Alice"
score = 95

puts "Hello, #{name}!"                  → Hello, Alice!
puts "Score: #{score}"                  → Score: 95
puts "Double: #{score * 2}"             → Double: 190
puts "Upper: #{name.upcase}"            → Upper: ALICE
puts "Pi ≈ #{round(3.14159, 2)}"        → Pi ≈ 3.14
```

Any valid Frankie expression works inside `#{}`. The result is automatically converted to a string.

---

## `Escape Sequences`

Inside double-quoted strings:

| Sequence | Meaning |
|---|---|
| `\n` | Newline |
| `\t` | Tab |
| `\\` | Literal backslash |
| `\"` | Literal double quote |

```ruby
puts "line one\nline two"
puts "column\tone\ttwo"
puts "She said \"hello\""
```

---

## `String Operators`

```ruby
# Concatenation
puts "Hello, " + "Frankie!"    # Hello, Frankie!

# Repetition
puts "ha" * 3                   → hahaha
puts "-" * 40                   → ────────────────────────────────────────

# Comparison
puts "apple" == "apple"         → true
puts "apple" != "banana"        → true
puts "apple" < "banana"         → true  (lexicographic)

# Match operator — returns position of first match or nil
puts "frank123" =~ "\\d+"       → 5
puts "no digits" =~ "\\d+"      → nil
```

---

## `Accessing Characters and Slices`

Strings are 0-indexed. Negative indices count from the end.

```ruby
s = "Frankie"

puts s[0]       → F
puts s[1]       → r
puts s[-1]      → e   (last character)
puts s[-3]      → i

# Inclusive range slice
puts s[0..4]    → Frank

# Exclusive range slice (up to but not including index 5)
puts s[0...4]   → Fran

# Last N characters
puts s[-3..-1]  → kie
```

---

## `Nil Safety with Strings`

Calling a method on `nil` raises a runtime error. Use `&.` to return nil safely instead:

```ruby
name = nil

puts name.upcase    # Runtime error
puts name&.upcase   # nil — safe

# Useful when a value might or might not be present
user_input = nil
puts user_input&.strip&.downcase   # nil — whole chain short-circuits
```

---

## `Quick Reference`

| Category | Methods / Functions |
|---|---|
| Case | `.upcase` `.downcase` |
| Whitespace | `.strip` `.lstrip` `.rstrip` `.chomp` `.chop` |
| Inspection | `.length` `.empty?` `.include?` `.start_with?` `.end_with?` `.count` |
| Search | `.match` `.match_all` `.matches?` `=~` |
| Transform | `.replace` `.sub` `.gsub` `.gsub do` `.delete` `.squeeze` `.tr` `.reverse` |
| Align | `.center` `.ljust` `.rjust` |
| Split | `.split` `.lines` `.chars` `.bytes` |
| Iterate | `.each_char` `.each_line` |
| Convert | `.to_i` `.to_f` `.to_s` `.ord` `.encode` `.decode` |
| Format | `template()` `.format()` `sprintf()` `paste()` |
| Slice | `[i]` `[a..b]` `[a...b]` |
| Operators | `+` `*` `==` `!=` `<` `>` `=~` |
