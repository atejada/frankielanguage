# Frankiecolor

Terminal Colors

## `Color functions`

`black`, `red`, `green`, `yellow`, `blue`, `magenta`, `cyan`, `white`

wrap a string in that ANSI color.

```ruby
stitch "frankiecolor"

puts red("Error: something failed")
```

## `Style functions`

`bold`, `dim`, `italic`, `underline`, `inverse` — apply text styles.

```ruby
stitch "frankiecolor"

puts underline("Click here")
```

## Utilities

### `colorize(str, color_name)`

generic color by name string.

```ruby
stitch "frankiecolor"

puts colorize("hello", "magenta")
```

### `strip_color(str)`

remove all ANSI escape codes (for writing colored output to a log file).

```
stitch "frankiecolor"

msg = warn("Deprecated API")
puts msg
puts strip_color(msg)
```
