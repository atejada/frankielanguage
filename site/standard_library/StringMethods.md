# String methods

## Case and Whitespace

### `.upcase` 

Convert all characters to uppercase. 

```ruby
"hello".upcase 

→ "HELLO"
```

### `.downcase`

Convert all characters to lowercase. 

```ruby
"HELLO".downcase 

→ "hello"
```

### `.strip` 

Remove leading and trailing whitespace. 

```ruby
"  hi  ".strip 

→ "hi"
```

### `.lstrip`

Remove leading whitespace only.

### `.rstrip`

Remove trailing whitespace only.

### `.chomp`

Remove a trailing newline character. Useful after file_lines or user input.

### `.chop`

Remove the last character unconditionally. 

```ruby
"hello!".chop 

→ "hello"
```

---

## Inspection & testing

### `length`

Number of characters. 

```ruby
"Frankie".length 

→ 7
```

### `.empty?`

True if the string has zero characters.

```ruby
"".empty? 

→ true
```

### `.nil?`

Always false for a string; useful in nil-safe chains with &.


### `.include?(n)` 

True if n appears anywhere in the string.

```ruby
"Frankie".include?("n")

→ true 
```

### `.start_with?(n)` 

True if the string begins with n.

```ruby
"Frankie".start_with?("F") 
→ true 
```

### `.end_with?(s)`

True if the string ends with s.

```ruby
"Frankie".end_with?("e") 

→ true 
```

### `.count(sub)`

Count non-overlapping occurrences of sub. 

```ruby
"hello".count("l") 

→ 2
```

---

## Transformation

### `.reverse`

Reverse the string character by character. 

```ruby
"Frankie".reverse 

→ "eiknarF"
```

### `.replace(old, new)`

Replace the first occurrence of old with new. Alias for sub().

```ruby
"Franpie".replace("p","k") 

→ "Frankie"
```

### `.delete(chars)`

Remove all occurrences of any character in chars. 

```ruby
"hello".delete("l") 

→ "heo"
```

### `.squeeze`

Collapse runs of consecutive identical characters. 

```ruby
"aaabbbccc".squeeze 

→ "abc"
```

### `.tr(from, to)`

Translate characters — each character in from is replaced by the corresponding character in to. 

```ruby
"hello".tr("aeiou", "*") 

→ "h*ll*"
```

### `.center(width, pad)`

Center in a field of width, padded with pad. 

```ruby
"hi".center(10, "-") 

→ "----hi----"
```

### `.ljust(width, pad)`

Left-justify in a field of width.

```ruby
"hi".ljust(10, "-") 

→ hi--------
```

### `.rjust(width, pad)`

Right-justify in a field of width.

```ruby
"hi".rjust(10, "-") 

→ --------hi
```

### `.encode`

Convert string to a vector of byte values (UTF-8). 

```ruby
"hi".encode 

→ [104, 105]
```

---

## Splitting & joining

### `.split(sep)`

Split on a separator string, return a Vector. 

```ruby
"a,b,c".split(",") 

→ [a, b, c]
```

### `.join(sep)`

Join a vector of strings with a separator. Called on a Vector but takes a string arg. 

```ruby
["a","b"].join("-") 

→ a-b
```

### `.lines`

Split into a vector of lines, stripping newline characters

```ruby
"a,b,c".lines 

→ [a,b,c]
```

### `.chars`

Split into a vector of individual characters. Chains naturally into iterators.

```ruby
puts "abc".chars 

→ [a b c]
```

### `.bytes`

Return a vector of integer byte values.

```ruby
puts "abc".bytes

→  [97, 98, 99]
```

### `.each_char do |c|`

Iterate over each character with a block.

```ruby
"Frankie".each_char do |c| 
  puts c 
end 

→ F
  r
  a
  n 
  k
  i
  e
```

### `.each_line do |l|`

Iterate over each line with a block.

```ruby
"Frankie is great!".each_line do |c| 
  c =  c.delete("i") 
  puts c 
end

→ Franke s great!
```

---

## Slicing

### `[i]`

Character at index i. Negative indices count from the end. 

```ruby
"hello"[-1] 

→ "o"
```

---

### `[a..b]`

Inclusive slice from index a to b. 

```ruby
"hello"[1..3] 

→ "ell"
```

### `[a...b]`

Exclusive slice — up to but not including b

```
"hello"[1...3] 

→ "el"
```

### `template(str, hash)`

{{key}} placeholder replacement.

```ruby
puts template("{{name}} is awesome!", {name: "Frankie"})

→ Frankie is awesome!
```

### `sprintf(fmt, args)`

C-style formatting.

```ruby
puts sprintf("%-10s %5d %8.2f", "Frankie", 1, 3.14444)

→ Frankie        1     3.14
```

### `.format(hash)`

Named placeholder substitution. *(introduced in v1.15)*

Replaces `{key}` placeholders using values from the given hash. Raises a descriptive error if a key is missing.

```ruby
puts "Hello, {name}! You have {count} messages.".format({name: "Alice", count: 5})
→ Hello, Alice! You have 5 messages.

puts "Order {id} placed by {user}".format({id: 42, user: "Bob"})
→ Order 42 placed by Bob
```

### `paste(a, b, sep)` 

Concatenate with separator.

```ruby
puts paste("2026", "03", "14", sep: "/")

→ 2026/03/14
```

---

## Pattern Extraction *(introduced in v1.16)*

### `.scan(pattern)`

Find all non-overlapping matches of a regex pattern in the string. Returns a vector of matched strings (or capture groups if the pattern contains them).

```ruby
# Extract all words
puts "hello world frankie".scan("[a-z]+")
# ["hello", "world", "frankie"]

# Extract email addresses
text = "Contact alice@example.com or bob@test.org for help."
emails = text.scan("[\\w.]+@[\\w.]+")
puts emails
# ["alice@example.com", "bob@test.org"]

# Extract dates
log = "Errors on 2026-01-15 and 2026-02-03."
dates = log.scan("\\d{4}-\\d{2}-\\d{2}")
puts dates
# ["2026-01-15", "2026-02-03"]

# Extract numbers
prices = "Items cost $12.50, $3.99, and $45.00."
amounts = prices.scan("\\d+\\.\\d{2}")
puts amounts
# ["12.50", "3.99", "45.00"]
```

`scan` is commonly combined with iterators:

```ruby
html = "<a href='https://frankie.dev'>Frankie</a>"
urls = html.scan("href='([^']+)'")
urls.each do |url|
  puts "Found link: #{url}"
end
```
