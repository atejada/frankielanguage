# Integer Methods

## Conversion

### `.to_f`

Convert to Float. 

```ruby
puts 42.to_f 

→ 42.0
```

### .to_s

Convert to String. 

```ruby
puts 42.to_s 

→ "42"
```

### `.to_i`

Return self — no-op, but useful in generic code where the type isn't known.

```ruby
puts 42.to_i

→ 42
```

---

## Iteration

### `.times do |i|`

Repeat a block exactly n times. i counts from 0 to n-1. 

```ruby
3.times do |i|
  puts i 
end

→ 0, 1, 2
```

---

## `Standalone functions`

| What do you want | How you do it Frankie |
|---|---|
| `Absolute value` | abs(n) |
| `Is it zero?` | n == 0 |
| `Is it even?` | n % 2 == 0 |
| `Is it odd?` | n % 2 != 0 |
| `Is it positive?` | n > 0 |
| `Clamp to range` | clamp(n, lo, hi) |
| `Square root` | sqrt(n) |
| `Random integer` | rand_int(a, b) |
| `Type check` | is_integer(n) |
