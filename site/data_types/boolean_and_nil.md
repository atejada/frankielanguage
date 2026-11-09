# Boolean & Nil

## Boolean

A Boolean is either `true` or `false`. Booleans are produced by comparisons, logical operators, and predicate methods (`.empty?`, `.nil?`, `is_integer()`, etc.).

```ruby
flag    = true
missing = false

puts 5 > 3      # true
puts 5 < 3      # false
puts 5 == 5     # true
puts 5 != 3     # true
```

### `Logical Operators`

```ruby
puts true and false    # false — both must be true
puts true or false     # true  — at least one must be true
puts not true          # false — negation

# Short-circuit evaluation
# 'and' stops at the first false
# 'or' stops at the first true
puts false and risky()  # false — risky() never called
puts true  or  risky()  # true  — risky() never called
```

### `Truthiness`

`if`, `while`, `until`, and the inline-`if` expression follow Python-style truthiness: `false`, `nil`, `0`, `0.0`, `""`, `[]`, and `{}` are all falsy. Everything else is truthy.

```ruby
puts(if 0     then "truthy" else "falsy" end)   # falsy
puts(if ""    then "truthy" else "falsy" end)   # falsy
puts(if []    then "truthy" else "falsy" end)   # falsy
puts(if false then "truthy" else "falsy" end)   # falsy
puts(if nil   then "truthy" else "falsy" end)   # falsy
```

The one exception is `||=` (nil-coalescing assignment), which uses Ruby's model instead — only `nil` and `false` trigger the fallback, so `0`, `""`, and `[]` are kept as-is:

```ruby
a = 0
a ||= 99
puts a   # 0 — kept, not nil/false

b = nil
b ||= 99
puts b   # 99 — nil triggers the fallback
```

### `Boolean Comparison`

```ruby
puts true == true    # true
puts true == false   # false
puts true != false   # true

# Booleans are not numbers — no arithmetic
# puts true + 1   # Runtime error
```

### `Type Checking`

```ruby
puts is_bool(true)    # true
puts is_bool(false)   # true
puts is_bool(0)       # false
puts is_bool("true")  # false
```

### `Common Patterns`

```ruby
# Guard clause with boolean
def process(data)
  return nil if data == nil
  return nil if data.empty?
  data.map do |x| x * 2 end
end

# Store result of a condition
valid = age >= 18 and not banned
puts "Access: #{if valid then "granted" else "denied" end}"

# Predicate methods return booleans
puts "".empty?            # true
puts [1, 2].empty?        # false
puts "hello".include?("ell")  # true
puts is_integer(42)       # true
```

---

## Nil

`nil` represents the absence of a value. It is Frankie's equivalent of `null`, `None`, or `undefined` in other languages. There is exactly one `nil` — every `nil` is the same value.

```ruby
missing = nil
puts missing        # nil
puts missing == nil # true
```

### `When Nil Appears`

```ruby
# Unset hash keys
h = {name: "Alice"}
puts h["age"]        # nil — key doesn't exist

# Out-of-range vector access
v = [1, 2, 3]
puts v[10]           # nil

# Missing function results
def find_user(id)
  return nil if id == 0
  {id: id, name: "User#{id}"}
end
puts find_user(0)    # nil

# Short-circuit of &. chain
puts nil&.upcase     # nil
```

### `Checking for Nil`

```ruby
x = nil

# Three equivalent ways
puts x == nil        # true
puts x != nil        # false
puts x.nil?          # true — method form
puts is_nil(x)       # true — function form
```

Avoid testing `if not x` to check for nil — it also catches `false`:

```ruby
x = false

if not x             # catches both nil AND false
  puts "this runs"
end

if x == nil          # only catches nil
  puts "this does not run"
end
```

### `Nil Safety with `&.``

The safe navigation operator `&.` short-circuits on nil, returning nil instead of crashing:

```ruby
name = nil
puts name.upcase     # Runtime error
puts name&.upcase    # nil — safe

# Chain — stops at the first nil
user = nil
puts user&.name&.upcase&.length   # nil

# Useful when a value might or might not be present
config = {db: {host: "localhost"}}
puts config.dig("db", "host")&.upcase   # LOCALHOST
puts config.dig("db", "port")&.to_s     # nil
```

### `Nil in Collections`

```ruby
v = [1, nil, 2, nil, 3]
puts v.compact   # [1, 2, 3] — removes all nils

# nil propagates in arithmetic — raises an error
# puts nil + 1   # TypeError

# Nil-safe default pattern
value = get_value() || 0       # note: || is not in Frankie
# Instead use:
raw   = get_value()
value = if raw == nil then 0 else raw end
```

### `Conversion`

```ruby
puts nil.to_s    # ""   — nil becomes empty string
puts nil.to_a    # []   — nil becomes empty vector (via stdlib)
```

### `Type Checking`

```ruby
puts is_nil(nil)    # true
puts is_nil(false)  # false — false is not nil
puts is_nil(0)      # false
puts is_nil("")     # false
```
