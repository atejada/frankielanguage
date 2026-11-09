# Math & Statistics

## `Overview`

Frankie has a built-in math and statistics library covering everything from basic arithmetic to descriptive statistics. 

The statistical functions are borrowed from R — `mean`, `median`, `stdev`, `variance` are first-class citizens, not an afterthought. Since v1.22, the library leans harder into its FORTRAN heritage too: real matrix math, complex numbers, and a full set of intrinsic math functions (`sin`, `log`, `factorial`, and friends).

All functions are available without any import.

---

## Arithmetic Operators

| Operator | Meaning | Example | Result |
|---|---|---|---|
| `+` | Addition | `10 + 3` | `13` |
| `-` | Subtraction | `10 - 3` | `7` |
| `*` | Multiplication | `10 * 3` | `30` |
| `/` | Float division | `10 / 3` | `3.333...` |
| `//` | Integer division | `10 // 3` | `3` |
| `%` | Modulo | `10 % 3` | `1` |
| `**` | Exponentiation | `2 ** 8` | `256` |

`/` always returns a Float. Use `//` (borrowed from Fortran) for a whole-number result.

```ruby
puts 10 / 3     # 3.3333333333333335
puts 10 // 3    # 3
puts 10 % 3     # 1
puts 2 ** 10    # 1024
puts 2 ** 0     # 1
```

### `Integer Division and Modulo with Negatives`

`//` uses floor division — it rounds toward negative infinity:

```ruby
puts 17 // 5    #  3
puts (-17) // 5 # -4  (floors toward -∞)
puts 17 % 5     #  2
puts (-17) % 5  #  3  (result has the sign of the divisor)
```

---

## Core Math Functions

### `abs(x)`

Absolute value

```ruby
puts abs(-42)    → 42
puts abs(42)     → 42
puts abs(-3.14)  → 3.14
```

### `sqrt(x)`

Square root

Always returns a Float.

```ruby
puts sqrt(144)   → 12.0
puts sqrt(2)     → 1.4142135623730951
puts sqrt(0)     → 0.0
```

### `floor(x)`

Round down

Returns the largest integer less than or equal to `x`.

```ruby
puts floor(3.7)    → 3
puts floor(3.0)    → 3
puts floor(-3.2)   → -4  (toward -∞, not toward zero)
```

### `ceil(x)`

Round up

Returns the smallest integer greater than or equal to `x`.

```ruby
puts ceil(3.2)    → 4
puts ceil(3.0)    → 3
puts ceil(-3.7)   → -3
```

### `round(x, n)`

Round to n decimal places

`n` defaults to 0. Returns a Float when `n > 0`, an integer-like Float when `n = 0`.

```ruby
puts round(3.14159, 2)   → 3.14
puts round(3.14159, 4)   → 3.1416
puts round(3.5)          → 4.0
puts round(-3.5)         → -4.0
puts round(2.71828, 3)   → 2.718
```

### `sin(x)`, `cos(x)`, `tan(x)`, `asin(x)`, `acos(x)`, `atan(x)`, `atan2(y, x)` (v1.22)

Trigonometric functions

Angles are in radians, matching FORTRAN's intrinsic `SIN`/`COS`/`TAN` family.

```ruby
puts sin(0)          → 0.0
puts cos(0)          → 1.0
puts atan2(1, 1)      → 0.7853981633974483   — π/4
```

### `exp(x)`, `log(x, base = nil)`, `log10(x)`, `log2(x)` (v1.22)

Exponential and logarithms

`log(x)` is natural log by default; pass a second argument for a specific base.

```ruby
puts exp(1)          → 2.718281828459045
puts log(E)          → 1.0
puts log(8, 2)        → 3.0
puts log10(1000)      → 3.0
puts log2(8)          → 3.0
```

### `PI` and `E` (v1.22)

Constants

```ruby
puts PI    → 3.141592653589793
puts E     → 2.718281828459045
```

### `factorial(n)`, `gcd(a, b)`, `lcm(a, b)` (v1.22)

Number theory

```ruby
puts factorial(5)    → 120
puts gcd(12, 18)      → 6
puts lcm(4, 6)         → 12
```

`factorial` raises for negative input.

### `clamp(x, lo, hi)`

Constrain to a range

Returns `lo` if `x < lo`, `hi` if `x > hi`, otherwise `x`.

```ruby
puts clamp(15, 0, 10)       → 10  — above max
puts clamp(-5, 0, 10)       → 0   — below min
puts clamp(5,  0, 10)       → 5   — within range
puts clamp(3.14, 0.0, 3.0)  → 3.0
```

### `min` and `max`

Work on two values or on a vector:

```ruby
puts min(3, 7)                → 3
puts max(3, 7)                → 7
puts min([3, 1, 4, 1, 5, 9])  → 1
puts max([3, 1, 4, 1, 5, 9])  → 9

# Also available as vector methods
puts [3, 1, 4, 1, 5, 9].min   → 1
puts [3, 1, 4, 1, 5, 9].max   → 9
```

---

## Statistics Functions

All statistics functions take a vector of numbers.

### `sum(v)`

Total

```ruby
puts sum([1, 2, 3, 4, 5])    → 15
puts [10, 20, 30].sum        → 60  — method form
```

### `mean(v)`

Arithmetic mean

```ruby
data = [23, 45, 12, 67, 34, 89, 56]
puts mean(data)    → 46.57142857142857
puts data.mean     → same — method form
```

### `median(v)`

Middle value

For even-length vectors, returns the average of the two middle values.

```ruby
puts median([1, 2, 3, 4, 5])       → 3
puts median([1, 2, 3, 4, 5, 6])    → 3.5
puts median([23, 45, 12, 67, 34])  → 34
```

### `stdev(v)`

Sample standard deviation

Measures spread. Uses `n-1` (Bessel's correction) — sample standard deviation, not population.

```ruby
data = [23, 45, 12, 67, 34, 89, 56, 11, 78, 42]
puts stdev(data)   → 26.84130316425705
```

### `variance(v)`

Sample variance

The square of the standard deviation.

```ruby
puts variance(data)   → 720.4555555555554
```

---

## Sequences and Ranges

### `seq(start, stop, step)`

Numeric sequence

Generates a vector from `start` to `stop` in steps of `step`. Borrowed from R.

```ruby
puts seq(1, 5)            → [1, 2, 3, 4, 5]
puts seq(0, 10, 2)        → [0, 2, 4, 6, 8, 10]
puts seq(5, 1, -1)        → [5, 4, 3, 2, 1]
puts seq(0.0, 1.0, 0.25)  → [0.0, 0.25, 0.5, 0.75, 1.0]
```

### `linspace(start, stop, n)`

Evenly spaced values

Generates exactly `n` values evenly distributed between `start` and `stop`, inclusive.

```ruby
puts linspace(0.0, 1.0, 5)    → [0.0, 0.25, 0.5, 0.75, 1.0]
puts linspace(0.0, 100.0, 6)  → [0.0, 20.0, 40.0, 60.0, 80.0, 100.0]
```

### `vec(range)`

Vector from a range literal

```ruby
puts vec(1..10)    → [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
puts vec(0...5)    → [0, 1, 2, 3, 4]
```

### `rep(x, n)`

Repeat a value

```ruby
puts rep(0, 5)      → [0, 0, 0, 0, 0]
puts rep(3.14, 3)   → [3.14, 3.14, 3.14]
puts rep("ha", 3)   → ["ha", "ha", "ha"]
```

---

## `Vectorised Arithmetic`

Operations on vectors apply element-wise — R does this natively, and it's exactly the whole-array math FORTRAN 90+ pioneered decades earlier (`C = A + B` adds two arrays element-wise with no explicit loop). `+`, `-`, `/`, `%`, and `**` all work this way:

```ruby
v = [1, 2, 3, 4, 5]

puts v + [10,10,10,10,10]   → [11, 12, 13, 14, 15]
puts v ** 2                 → [1, 4, 9, 16, 25]
puts v - [1, 1, 1, 1, 1]    → [0, 1, 2, 3, 4]
```

Vectors must be the same length for binary operations. Use `linspace` or `rep` to create a matching vector.

`*` is the one exception — `v * 2` repeats the vector (Python-list semantics: `[1,2,3] * 2 → [1,2,3,1,2,3]`), not element-wise scaling. For scaling, either add the vector to itself the needed number of times, or reach for `matrix_scale` if you're working with a matrix (below).

---

## `Matrix Math (v1.22)`

A matrix is just a vector of vectors, and since v1.22 arithmetic recurses through nested vectors at every depth — so `+`, `-`, `/`, `%`, and `**` on two matrices are genuinely element-wise, not a silent row-by-row concatenation:

```ruby
m1 = [[1, 2], [3, 4]]
m2 = [[5, 6], [7, 8]]

puts m1 + m2   → [[6, 8], [10, 12]]
puts m1 - m2   → [[-4, -4], [-4, -4]]
```

### `matrix_new(rows, cols, fill = 0)`

Build a filled matrix

```ruby
puts matrix_new(2, 3, 0)   → [[0, 0, 0], [0, 0, 0]]
```

### `matrix_identity(n)`

Build an n×n identity matrix

```ruby
puts matrix_identity(3)   → [[1, 0, 0], [0, 1, 0], [0, 0, 1]]
```

### `matrix_shape(m)`

Return `[rows, cols]`

```ruby
puts matrix_shape(matrix_new(2, 5, 0))   → [2, 5]
```

### `matrix_transpose(m)`

Swap rows and columns

```ruby
puts matrix_transpose([[1, 2, 3], [4, 5, 6]])   → [[1, 4], [2, 5], [3, 6]]
```

### `matrix_scale(m, n)`

Multiply every element by `n`

Since `*` stays as vector-repeat at the top level (unchanged, matching pre-v1.22 behavior), use `matrix_scale` for true element-wise scalar multiplication:

```ruby
puts matrix_scale([[1, 2], [3, 4]], 3)   → [[3, 6], [9, 12]]
```

### `matrix_multiply(m1, m2)`

Real matrix multiplication (row · column)

This is different from `*`, which stays element-wise for two matrices of the same shape — `matrix_multiply` does the actual linear-algebra dot-product multiplication, and requires `m1`'s column count to match `m2`'s row count.

```ruby
puts matrix_multiply([[1, 2], [3, 4]], [[5, 6], [7, 8]])   → [[19, 22], [43, 50]]
```

Raises if the shapes are incompatible.

---

## `Complex Numbers (v1.22)`

FORTRAN has shipped a native `COMPLEX` type since FORTRAN 66. Frankie exposes Python's built-in complex number support the same flat, no-import-needed way as everything else in this library — `+`, `-`, `*`, `/`, and `**` all work natively once you have one.

```ruby
c1 = complex_new(3, 4)
c2 = complex_new(1, 2)

puts c1              → 3+4i
puts c1 + c2          → 4+6i
puts c1 * c2          → -5+10i
```

Printed in conventional math notation (`3+4i`), not Python's electrical-engineering-flavored `3+4j`.

### `complex_new(real, imag = 0)`

Build a complex number

### `complex_real(c)` / `complex_imag(c)`

Read the real / imaginary parts

```ruby
c = complex_new(3, 4)
puts complex_real(c)   → 3.0
puts complex_imag(c)   → 4.0
```

### `complex_conj(c)`

Complex conjugate (negates the imaginary part)

```ruby
puts complex_conj(complex_new(3, 4))   → 3-4i
```

### `complex_abs(c)`

Magnitude

```ruby
puts complex_abs(complex_new(3, 4))   → 5.0   — 3-4-5 triangle
```

---

## `The Pipe Operator for Data Pipelines`

`|>` passes the left value as the argument to the right function. Combines naturally with stats:

```ruby
data = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3]

data |> sum  |> puts    → 39
data |> mean |> puts    → 3.9
data |> min  |> puts    → 1
data |> max  |> puts    → 9

# Chain transformations
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
  .select do |x| x % 2 == 0 end
  |> sum
  |> puts
  
→ 30
```

## Randomness

### `rand_int(a, b)`

Random integer

Returns a random integer between `a` and `b` inclusive.

```ruby
puts rand_int(1, 6)      → dice roll — 1 to 6
puts rand_int(0, 100)    → 0 to 100
```

### `rand_float(a, b)`

Random float

Returns a random float between `a` and `b`.

```ruby
puts rand_float(0.0, 1.0)    → e.g. 0.7493381866018054
puts rand_float(-1.0, 1.0)   → e.g. -0.234...
```

### `rand()`

Random float 0..1

```ruby
puts rand()   # e.g. 0.4217...
```

### `rand_seed(n)`

Seed for reproducibility

Fix the random seed to get the same sequence every time — useful for tests and reproducible simulations.

```ruby
rand_seed(42)
puts rand_int(1, 100)   → always the same value for seed 42
puts rand_int(1, 100)   → next value in the same sequence
```

### `shuffle(v)`

Randomly reorder a vector

```ruby
v = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
puts shuffle(v)   → e.g. [4, 9, 1, 7, 3, 10, 5, 2, 8, 6]
```

### `sample(v, n)`

Random sample without replacement

```ruby
v = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
puts sample(v, 3)   → e.g. [7, 2, 9]
puts sample(v, 1)   → e.g. [4]
```

---

## Numeric Formatting

Use `sprintf` for precise control over how numbers are displayed:

```ruby
puts sprintf("%.2f", 3.14159)                    → 3.14
puts sprintf("%.4f", 3.14159)                    → 3.1416
puts sprintf("%08.3f", 3.14)                     → 0003.140
puts sprintf("%+d", 42)                          → +42
puts sprintf("%+d", -42)                         → -42
puts sprintf("%d items at $%.2f each", 5, 1.99)  → 5 items at $1.99 each
```

Or use `round` for display without format strings:

```ruby
data = [1.23456, 7.89012, 3.45678]
puts data.map do |x| 
  round(x, 2) 
end             

→ [1.23, 7.89, 3.46]
```

---

## Quick Reference

| Function | Description | Example |
|---|---|---|
| `abs(x)` | Absolute value | `abs(-5)` → `5` |
| `sqrt(x)` | Square root | `sqrt(9)` → `3.0` |
| `floor(x)` | Round down | `floor(3.9)` → `3` |
| `ceil(x)` | Round up | `ceil(3.1)` → `4` |
| `round(x, n)` | Round to n places | `round(3.14159, 2)` → `3.14` |
| `clamp(x, lo, hi)` | Constrain to range | `clamp(15, 0, 10)` → `10` |
| `min(a, b)` / `min(v)` | Minimum | `min(3, 7)` → `3` |
| `max(a, b)` / `max(v)` | Maximum | `max(3, 7)` → `7` |
| `sum(v)` | Sum of all elements | `sum([1,2,3])` → `6` |
| `mean(v)` | Arithmetic mean | `mean([1,2,3])` → `2.0` |
| `median(v)` | Middle value | `median([1,2,3])` → `2` |
| `stdev(v)` | Sample standard deviation | `stdev([2,4,4,4,5,5,7,9])` → `2.0` |
| `variance(v)` | Sample variance | `variance([2,4,6])` → `4.0` |
| `seq(start, stop, step)` | Numeric sequence | `seq(1, 5)` → `[1,2,3,4,5]` |
| `linspace(start, stop, n)` | n evenly-spaced values | `linspace(0.0, 1.0, 3)` → `[0.0, 0.5, 1.0]` |
| `vec(range)` | Vector from range | `vec(1..5)` → `[1,2,3,4,5]` |
| `rep(x, n)` | Repeat value | `rep(0, 3)` → `[0,0,0]` |
| `rand_int(a, b)` | Random integer a..b | `rand_int(1, 6)` |
| `rand_float(a, b)` | Random float a..b | `rand_float(0.0, 1.0)` |
| `rand()` | Random float 0..1 | `rand()` |
| `rand_seed(n)` | Fix random seed | `rand_seed(42)` |
| `shuffle(v)` | Randomly reorder | `shuffle([1,2,3,4,5])` |
| `sample(v, n)` | Random sample | `sample(v, 3)` |
| `sprintf(fmt, ...)` | Formatted number string | `sprintf("%.2f", 3.14159)` |
| `sin(x)` / `cos(x)` / `tan(x)` (v1.22) | Trigonometric functions | `sin(0)` → `0.0` |
| `asin(x)` / `acos(x)` / `atan(x)` / `atan2(y, x)` (v1.22) | Inverse trig | `atan2(1, 1)` → `0.785...` |
| `exp(x)` (v1.22) | e^x | `exp(1)` → `2.718...` |
| `log(x, base = nil)` (v1.22) | Logarithm | `log(8, 2)` → `3.0` |
| `log10(x)` / `log2(x)` (v1.22) | Base-10 / base-2 log | `log10(1000)` → `3.0` |
| `PI` / `E` (v1.22) | Constants | `PI` → `3.14159...` |
| `factorial(n)` (v1.22) | n! | `factorial(5)` → `120` |
| `gcd(a, b)` / `lcm(a, b)` (v1.22) | Number theory | `gcd(12, 18)` → `6` |
| `matrix_new(rows, cols, fill)` (v1.22) | Filled matrix | `matrix_new(2, 2, 0)` → `[[0,0],[0,0]]` |
| `matrix_identity(n)` (v1.22) | Identity matrix | `matrix_identity(2)` → `[[1,0],[0,1]]` |
| `matrix_shape(m)` (v1.22) | `[rows, cols]` | `matrix_shape([[1,2,3]])` → `[1,3]` |
| `matrix_transpose(m)` (v1.22) | Swap rows/cols | `matrix_transpose([[1,2]])` → `[[1],[2]]` |
| `matrix_scale(m, n)` (v1.22) | Element-wise scalar multiply | `matrix_scale([[1,2]], 2)` → `[[2,4]]` |
| `matrix_multiply(m1, m2)` (v1.22) | Real matrix multiplication | `matrix_multiply([[1,2]], [[1],[2]])` → `[[5]]` |
| `complex_new(real, imag)` (v1.22) | Complex number | `complex_new(3, 4)` → `3+4i` |
| `complex_real(c)` / `complex_imag(c)` (v1.22) | Real / imaginary part | `complex_real(complex_new(3,4))` → `3.0` |
| `complex_conj(c)` (v1.22) | Complex conjugate | `complex_conj(complex_new(3,4))` → `3-4i` |
| `complex_abs(c)` (v1.22) | Magnitude | `complex_abs(complex_new(3,4))` → `5.0` |
