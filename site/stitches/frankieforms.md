# Frankieforms

Form Validation

## `Available rules`

| Rule | Options | Description |
|---|---|---|
| `"required"` | — | Value must be non-nil and non-empty |
| `"min_length"` | `value: n` | At least n characters |
| `"max_length"` | `value: n` | At most n characters |
| `"email"` | — | Must match `word@word.word` |
| `"min_value"` | `value: n` | Integer value ≥ n |
| `"max_value"` | `value: n` | Integer value ≤ n |
| `"numeric"` | — | Must be a number (int or float) |
| `"alpha"` | — | Letters only |
| `"matches_pattern"` | `value: pat`, `message: msg` | Must match regex pattern |

## Functions

### `validate(form, rules)`

Returns a Hash of `{field: error_message}` pairs. 

Empty hash = all valid.

```ruby
stitch "frankieforms"

form  = {name: "A", email: "alice@example", age: ""}
rules = {
  name:  [{rule: "required"}, {rule: "min_length", value: 2}],
  email: [{rule: "required"}, {rule: "email"}],
  age:   [{rule: "required"}, {rule: "min_value", value: 18}]
}

if valid?(form, rules) == false
  validate(form, rules).each do |field, msg|
    puts "#{field}: #{msg}"
  end
end

→ name: must be at least 2 characters
→ email: must be a valid email address 
→ age: is required
```

### `valid?(form, rules)`

Returns `true` if no errors.

```ruby
stitch "frankieforms"

form  = {name: "Alice", email: "alice@example", age: "25"}
rules = {
  name:  [{rule: "required"}, {rule: "min_length", value: 2}],
  email: [{rule: "required"}, {rule: "email"}],
  age:   [{rule: "required"}, {rule: "min_value", value: 18}]
}

puts valid?(form, rules)

→ true
```
