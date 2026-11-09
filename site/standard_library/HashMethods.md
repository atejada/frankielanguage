# Hash Methods

## Access

### `h[key]`

Get the value for a key. 

Returns nil if the key doesn't exist.

```ruby
prices = {"apple": 1.20, "banana": 0.50}
puts prices["apple"]

→ 1.20
```

### `.fetch(key, default)`

Get a value with an explicit default if the key is missing.

```ruby
prices = {"apple": 1.20, "banana": 0.50}
puts prices.fetch("Pears", 0.75)

→ 0.75
```

### `.dig(k1, k2, ...)`

Safe nested access through any combination of hashes and vectors. 

Returns nil instead of crashing at any level.

```ruby
prices = {"fruit": {"apples": 1.20, "banana":0.50}, "vegetables": {"lettuce":0.45}}
puts prices.dig("fruit", "banana")

→ 0.5
```

---

## Inspection

### `.keys`

Vector of all keys. 

```ruby
{a: 1, b: 2}.keys 

→ ["a", "b"].
```

### `.values`

Vector of all values. 

```ruby
{a: 1, b: 2}.values 

→ [1, 2]
```

### `.size`

Number of key-value pairs. 

Alias: .count.

```ruby
puts {"fruit": {"apples": 1.20, "banana":0.50}, "vegetables": {"lettuce":0.45}}.size

→ 2
```

### `.has_key?(k)`

True if key k exists. 

```
puts {"apple": 1.20, "banana": 0.50}.has_key?("banana")

→ true
```

### `.empty?`

True if the hash has no pairs.

```
puts {"apple": 1.20, "banana": 0.50}.empty?

→ false
```

### `.nil?`

Always false for a hash.

useful in nil-safe chains.

```ruby
puts {"apple": 1.20, "banana": 0.50}.nil?

→ false
```

---

## Mutation

### `h[key] = value`

Add or update a key.

```ruby
prices = {"apple": 1.20, "banana": 0.50}
prices["apple"] = 1.50
puts prices["apple"]

→ 1.5
```

### `.store(key, value)`

Identical to h[key] = value — method form of key assignment.

### `.delete(key)`

Remove a key, return the hash. 

Doesn't crash if the key is missing.

```ruby
prices = {"apple": 1.20, "banana": 0.50, "orange": 0.4}.delete("banana")
puts prices

→ {apple: 1.2, orange: 0.4}
```

### `.merge_bang(other)`

Merge other into the hash in place. Right-hand keys win.

```
prices = {"apple": 1.20, "banana": 0.50}.merge_bang({"orange": 0.4})
puts prices

→ {apple: 1.2, banana: 0.5, orange: 0.4}
```

---

## Merging

### `.merge(other)` 

Return a new merged hash. Right-hand keys win on conflict. 

Does not modify the original.

```ruby
hash = {"apple": 1.20, "banana": 0.50}
prices = hash.merge({"orange": 0.4})
puts prices
puts hash

→ {apple: 1.2, banana: 0.5, orange: 0.4}
→ {apple: 1.2, banana: 0.5}
```

### `h1 | h2`

Merge operator — identical to .merge. 

Chains cleanly: base | env | local

```
hash = {"apple": 1.20, "banana": 0.50}
prices = hash | {"orange": 0.4}
puts prices
puts hash

→ {apple: 1.2, banana: 0.5, orange: 0.4}
→ {apple: 1.2, banana: 0.5}
```

---

## Iteration

### `.each do |k, v|`

Iterate over all key-value pairs. 

With two block params gives key and value separately; 

with one param gives a [key, value] pair.

```ruby
{"apple": 1.20, "banana": 0.50}.each do |k, v|
  puts "#{k} : #{v}"
end

→ apple : 1.2
→ banana : 0.5
```

### `.each_pair do |k, v|`

Identical to .each — explicit pair-iteration alias.

### `.each_with_object(init) do |pair, obj|`

Iterate with a shared accumulator. 

The pair arrives as [key, value].

```ruby
puts {"apple": 1.20, "banana": 0.50}.each_with_object({}) do |pair, acc|
  key   = pair[0]
  value = pair[1]
  acc[key] = "$#{value}"
end

→ {apple: $1.2, banana: $0.5}
```

---

## Transformation

### `.map do |k, v|`

Transform each pair, return a vector of results. 

Block receives key and value.

```ruby
puts {"apple": 1.20, "banana": 0.50, "orange": 0.4}.map do |k, v|
  h = {}
  h[k] = v * 2
  h
end

→ [{apple: 2.4}, {banana: 1.0}, {orange: 0.8}]
```

### `.map_hash do |k, v|` 

Transform each pair, return a new hash. 

Block must return [new_key, new_value].

```ruby
puts {"apple": 1.20, "banana": 0.50, "orange": 0.4}.map_hash do |k, v|
  [k, v * 2]
end

→ {apple: 2.4, banana: 1.0, orange: 0.8}
```

### `.select do |k, v|`

Keep pairs where the block returns true. 

Returns a new hash.

```ruby
puts {"apple": 1.20, "banana": 0.50, "orange": 0.4}.select do |k, v|
  v > 0.4
end

→ 
```

### `.reject do |k, v|` 

Keep pairs where the block returns false. 

Returns a new hash.

```ruby
puts {"apple": 1.20, "banana": 0.50, "orange": 0.4}.reject do |k, v|
  v > 0.4
end

→ {orange: 0.4}
```

---

## Conversion

### `.to_a`

Convert to a vector of [key, value] pairs. 

```ruby
{a: 1, b: 2}.to_a 

→ [["a", 1], ["b", 2]]
```

### `.keys`

Already listed above — also a conversion in the sense of extracting one dimension.

```ruby
puts {"apple": 1.20, "banana": 0.50, "orange": 0.4}.keys

→ [apple, banana, orange]
```

### `.values`

Already listed above — also a conversion in the sense of extracting one dimension.

```ruby
puts {"apple": 1.20, "banana": 0.50, "orange": 0.4}.keys

→ [1.2, 0.5, 0.4]
```

---

## Standalone functions that operate on hashes

### `json_write(path, hash)`

Serialise a hash to a JSON file. 

```ruby
puts json_write("PATH/test.json", {"apple": 1.20, "banana": 0.50, "orange": 0.4})

→ true
```

### `json_read(path)`

Read it back.

```ruby
puts json_read("PATH/test.json")

→ {apple: 1.2, banana: 0.5, orange: 0.4}
```

### `template(str, hash)`

{{key}} placeholder replacement.

```ruby
puts template("{{name}} is awesome!", {name: "Frankie"})

→ Frankie is awesome!
```

### `str.format(hash)`

{key} placeholder substitution.

```ruby
template = "Hello, {name}! You are {age} years old."
puts template.format({name: "Frankie", age: 1})

→ Hello, Frankie! You are 1 years old.
```

---

## Advanced Transformation *(introduced in v1.16)*

### `.transform_values do |v| ... end`

Return a new hash with the same keys but each value replaced by the block's return value.

```ruby
prices = {apple: 1.0, banana: 0.5, cherry: 2.5}

doubled = prices.transform_values do |v| v * 2 end
# {apple: 2.0, banana: 1.0, cherry: 5.0}

taxed = prices.transform_values do |v| round(v * 1.08, 2) end
# {apple: 1.08, banana: 0.54, cherry: 2.7}

# Stringify all values
stringified = prices.transform_values do |v| v.to_s end
# {apple: "1.0", banana: "0.5", cherry: "2.5"}
```

---

### `.transform_keys do |k| ... end`

Return a new hash with transformed keys and unchanged values.

```ruby
data = {first_name: "Alice", last_name: "Smith"}

upcased = data.transform_keys do |k| k.upcase end
# {FIRST_NAME: "Alice", LAST_NAME: "Smith"}

# Convert symbol-style keys to string format
normalized = data.transform_keys do |k| k.gsub("_", "-") end
# {"first-name": "Alice", "last-name": "Smith"}
```

---

### `.deep_merge(other)`

Recursively merge `other` into the hash. Where both hashes have the same key and both values are themselves hashes, the merge recurses rather than overwriting. Returns a new hash; the originals are not mutated.

```ruby
defaults = {
  db:    {host: "localhost", port: 5432, pool: 5},
  app:   {debug: false, log_level: "info"},
  cache: {ttl: 300}
}
overrides = {
  db:  {host: "prod-db.example.com"},
  app: {debug: true}
}

config = defaults.deep_merge(overrides)
# {
#   db:    {host: "prod-db.example.com", port: 5432, pool: 5},
#   app:   {debug: true, log_level: "info"},
#   cache: {ttl: 300}
# }

puts config["db"]["host"]       # prod-db.example.com
puts config["db"]["port"]       # 5432  ← kept from defaults
puts config["app"]["debug"]     # true
puts config["app"]["log_level"] # info  ← kept from defaults
```

Contrast with `.merge` (shallow) and `|` (shallow):

```ruby
# shallow merge — nested hash is replaced entirely
a = {db: {host: "localhost", port: 5432}}
b = {db: {host: "prod"}}
puts (a | b)["db"]["port"]        # nil — port was lost

# deep merge — nested hash is merged recursively
puts a.deep_merge(b)["db"]["port"] # 5432 — port preserved
```
