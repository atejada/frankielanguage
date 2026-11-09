# Float

A Float is a decimal number — any value with a fractional part, or a number produced by float division. Frankie uses IEEE 754 double-precision, the same standard as Python, Ruby, and R.

```ruby
x = 3.14
y = -0.5
z = 1.0      # explicitly Float, not Integer
e = 2.71828
```

## `Float Literals`

```ruby
3.14       # standard decimal
-0.5       # negative
1.0        # integer value, Float type
1.0e6      # scientific notation — 1000000.0 (v1.18)
2.5e-3     # 0.0025
3.141_592  # underscore separators (v1.18)
```

Scientific notation (`1e6`, `2.5e-3`, `1E+9`) and underscore separators (`3.141_592`) are supported since v1.18.

## `Arithmetic`

All standard operators work on Floats. Mixing Integer and Float produces a Float:

```ruby
puts 3.14 + 1.0    # 4.140000000000001 (float precision)
puts 3.14 * 2      # 6.28
puts 10.0 / 3      # 3.3333333333333335
puts 2.0 ** 8      # 256.0

# Integer / Integer always returns Float
puts 10 / 4        # 2.5
puts 10 // 4       # 2  — use // for integer result
```

## `Floating-point Precision`

Floats cannot represent all decimal values exactly. This is standard IEEE 754 behaviour, not a Frankie quirk:

```ruby
puts 0.1 + 0.2            # 0.30000000000000004
puts round(0.1 + 0.2, 2)  # 0.3  — round when displaying
```

When comparing floats, compare within a tolerance or round first:

```ruby
a = 0.1 + 0.2
b = 0.3
puts a == b                       # false — unsafe
puts abs(a - b) < 0.0001          # true  — safe comparison
puts round(a, 10) == round(b, 10) # true  — round then compare
```

## `Math Functions`

```ruby
puts abs(-3.14)         # 3.14
puts sqrt(2.0)          # 1.4142135623730951
puts floor(3.7)         # 3
puts ceil(3.2)          # 4
puts round(3.567, 2)    # 3.57
puts round(3.567)       # 4.0  (rounds to nearest, but still returns a Float)
puts clamp(1.5, 0.0, 1.0)   # 1.0
```

`sqrt` always returns a Float. `floor` and `ceil` return an Integer. `round(x)`, even with no decimal argument, always returns a Float.

## `Conversion`

```ruby
puts 3.14.to_i    # 3     — truncates toward zero
puts (-3.9).to_i  # -3    — truncates toward zero (not floor)
puts 3.14.to_s    # "3.14"
puts 3.14.to_f    # 3.14  — no-op, already a Float

puts "3.14".to_f  # 3.14
puts 42.to_f      # 42.0
```

## `Type Checking`

```ruby
puts is_float(3.14)   # true
puts is_float(3)      # false — Integer, not Float
puts is_float("3.14") # false
```
