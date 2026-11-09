# Regular Expressions

## `.match(pattern)`

Return the first match object for pattern, or nil.

```ruby
s =  "The password is 123x123".match("(?<=The password is )\\S+")

→ 123x123
```

## `.match_all(pattern)`

Return a vector of all matches.


```ruby
puts  "4 languages, 1 Frankie".match_all("\\d+")

→ [4, 1]
```

## `.matches?(pattern)`

True if pattern matches anywhere. 

```ruby
puts "4 languages, 1 Frankie".matches("\\d+")

→ true
```

## `=~`

Match operator. Returns the position of the first match, or nil. 

```ruby
puts "frankie123" =~ "\\d+" 

→ 7
```

## `.sub(pattern, replacement)`

Replace the first regex match.

```ruby
puts "Frank1e".sub("\\d", "i") 

→ Frankie
```

## `.gsub(pattern, replacement)`

Replace all regex matches with a fixed string.

```ruby
puts "Frank1e 1s great!".gsub("\\d", "i") 

→ Frankie is great!
```

## `.gsub(pattern) do |m| … end` 

Replace all matches — block receives each match and returns its replacement.

```ruby
puts "Round 1 of 3 starts in 5 seconds".gsub("\\d+") do |m|
  n = m.to_i
  suffix = if n == 1 then "st" elsif n == 2 then "nd" elsif n == 3 then "rd" else "th" end
  m + suffix
end

→ Round 1st of 3rd starts in 5th seconds
```

## `sub(str, pattern, rep)`

Replace first match.

```ruby
puts sub("Frakie is good", "good", "great!")

→ Frankie is great!
```

## `gsub(str, pattern, rep)`

Replace all.

```ruby
puts gsub("Ruby is great, Ruby is slick!", "Ruby", "Frankie")

→ Frankie is great, Frankie is slick!
```


## `match(str, pattern)`

First match.

```ruby
s = match("Frankie 101 and Frankie 102", "\\d+")
puts s[0]

→ 101
```

## `match_all(str, pattern)`

All matches.

```ruby
s = match_all("Frankie 101 and Frankie 102", "\\d+")
puts s

→ [101, 102]
```

## `matches(str, pattern)`

Boolean test.

```ruby
puts matches("Frankie", "\\d+") 

→ false
```

---

## Standalone functions

### `sub(str, pattern, rep)`

Replace first match

```ruby
puts sub("Fran123ie" , "\\d+", "k")

→ Frankie
```

### `gsub(str, pattern, rep)`

Replace all

```ruby
puts gsub("Frank123ie 1is ru3nn1ing" , "\\d+", "")

→ Frankie is running
```

### `match(str, pattern)`

First match.

```ruby
matched = match("The password is 123x123", "(?<=The password is )\\S+")
puts matched[0]

→ 123x123
```

### `match_all(str, pattern)`

All matches

```ruby
matched =  "4 languages, 1 Frankie".match_all("\\d+")
puts matched

→ [4, 1]
```

### `matches(str, pattern)` 

Boolean test

```ruby
puts matches("4 languages, 1 Frankie", "\\d+")

→ true
```
