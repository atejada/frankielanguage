# Frankiestring

Additional string functions

## `pad_left(str, n, char)`

Right-align in a field of width n (space fill)

```ruby
stitch "frankiestring"

puts pad_left("Frankie", 12, "0")

→ 00000Frankie
```

## `pad_right(str, n, char)`

Left-align with custom fill character

```ruby
stitch "frankiestring"

puts pad_right("Frankie", 12, "0")

→ Frankie00000
```

## `truncate(str, n, suffix)`

Cut to n chars, append custom suffix

```ruby
stitch "frankiestring"

puts truncate("Frankie", 5, "X")

→ FranX
```

## `slugify(str)`

URL-safe lowercase slug

```ruby
stitch "frankiestring"

puts slugify("Frankie's Website")

→ frankie-s-website
```

## `word_wrap(str, width)`      

Wrap text at word boundaries

```ruby
stitch "frankiestring"

puts word_wrap("Frankie is a nice programming language", 5)

→ Frankie
  is a
  nice
  programming
  language
```

## `indent_lines(str, n)`

Prepend n spaces to every line

```ruby
stitch "frankiestring"

puts indent_lines("Frankie is a nice programming language", 5)

→     Frankie is a nice programming language 
```
