# JSON & CSV

## Overview

Frankie has built-in support for reading and writing both JSON and CSV — no external libraries, no imports required. J

SON is handled with Python's `json` module; CSV with the standard `csv` module. Both are zero-dependency.

---

## JSON

### `json_write(path, data)`

Write data to a JSON file

Accepts any Frankie value — hash, vector, string, number. Returns `true` on success.

```ruby
data = {name: "Alice", age: 30, scores: [95, 87, 92]}
json_write("data.json", data)
```

Pass `true` as a third argument for pretty-printed output:

```ruby
json_write("data.json", data, true)

→ produces:
→ {
→   "name": "Alice",
→   "age": 30,
→   "scores": [95, 87, 92]
→ }
```

### `json_read(path)`

Read a JSON file

Returns the parsed value — a Hash, Vector, String, or number depending on the file content. Hash keys come back as strings regardless of how you wrote them.

```ruby
data = json_read("data.json")
puts data["name"]      → Alice
puts data["age"]       → 30
puts data["scores"]    → [95, 87, 92]
```

### `json_parse(str)`

Parse a JSON string

Parses a JSON string directly without touching the filesystem.

```ruby
raw = '{"host": "localhost", "port": 5432}'
config = json_parse(raw)
puts config["host"]   → localhost
puts config["port"]   → 5432
```

### `json_encode(data)`

Serialise to a JSON string *(preferred — introduced in v1.15)*

The preferred name for `json_dump`. Accepts `pretty: true` for formatted output.

```ruby
data = {name: "Alice", scores: [95, 87, 92]}
puts json_encode(data)
# {"name": "Alice", "scores": [95, 87, 92]}

puts json_encode(data, pretty: true)
# {
#   "name": "Alice",
#   "scores": [95, 87, 92]
# }

# Round-trip
raw  = json_encode([1, 2, 3])
back = json_parse(raw)
puts back[1]    # 2
```

### `json_dump(data)`

Serialise to a JSON string

Alias for `json_encode` — kept for backwards compatibility. Pass `true` for pretty output.

```ruby
h = {name: "Alice", scores: [95, 87]}
puts json_dump(h)
# {"name": "Alice", "scores": [95, 87]}

puts json_dump(h, true)
# {
#   "name": "Alice",
#   "scores": [95, 87]
# }
```

### Round-trip Example

```ruby
original = {
  user:   "Alice",
  prefs:  {theme: "dark", lang: "en"},
  scores: [95, 87, 92]
}

json_write("/tmp/user.json", original, true)

loaded = json_read("/tmp/user.json")
puts loaded["user"]                  # Alice
puts loaded.dig("prefs", "theme")    # dark
puts loaded["scores"][1]             # 87

file_delete("/tmp/user.json")
```

### Handling Missing Files

```ruby
begin
  config = json_read("config.json")
rescue FileNotFoundError e
  puts "No config found — using defaults"
  config = {host: "localhost", port: 3000}
end
```

---

## CSV

### `csv_write(path, data)`

Write data to a CSV file

Accepts a vector of hashes (one hash per row) or a vector of vectors. Column headers are inferred from the first hash's keys.

```ruby
employees = [
  {name: "Alice", dept: "Engineering", salary: 95000},
  {name: "Bob",   dept: "Marketing",   salary: 72000},
  {name: "Carol", dept: "Finance",     salary: 88000}
]
csv_write("employees.csv", employees)
```

Produces:

```
name,dept,salary
Alice,Engineering,95000
Bob,Marketing,72000
Carol,Finance,88000
```

Pass explicit headers to control column order:

```ruby
csv_write("employees.csv", employees, ["name", "salary", "dept"])
```

### `csv_read(path)`

Read a CSV file

Returns a vector of hashes. The first row is treated as headers by default.

```ruby
rows = csv_read("employees.csv")
puts rows.length           # 3
puts rows[0]["name"]       # Alice
puts rows[0]["salary"]     # 95000

rows.each do |r|
  puts "#{r["name"]} — #{r["dept"]}"
end
```

Pass `false` to skip header interpretation — returns a vector of vectors instead:

```ruby
rows = csv_read("data.csv", false)
puts rows[0]   → first row as a vector
```

### `csv_parse(text)`

Parse a CSV string

Parses CSV content directly from a string.

```ruby
raw = "name,score\nAlice,95\nBob,87\nCarol,92\n"
rows = csv_parse(raw)
puts rows[0]["name"]    → Alice
puts rows[1]["score"]   → 87
```

### `csv_dump(data)`

Serialise to a CSV string

```ruby
data = [
  {city: "Lima",   pop: 10000000},
  {city: "Cusco",  pop: 430000}
]
puts csv_dump(data)
# city,pop
# Lima,10000000
# Cusco,430000
```

---

## Quick Reference

| Function | Description |
|---|---|
| `json_write(path, data)` | Write data to JSON file |
| `json_write(path, data, true)` | Write pretty-printed JSON |
| `json_read(path)` | Read and parse JSON file |
| `json_parse(str)` | Parse JSON string → value |
| `json_encode(data)` | Serialise value → JSON string (preferred) |
| `json_encode(data, pretty: true)` | Pretty-printed JSON string |
| `json_dump(data)` | Serialise value → JSON string (alias for `json_encode`) |
| `json_dump(data, true)` | Pretty-printed JSON string |
| `csv_write(path, data)` | Write vector of hashes to CSV |
| `csv_write(path, data, headers)` | Write with explicit column order |
| `csv_read(path)` | Read CSV → vector of hashes |
| `csv_read(path, false)` | Read CSV → vector of vectors (no headers) |
| `csv_parse(str)` | Parse CSV string → vector of hashes |
| `csv_dump(data)` | Serialise to CSV string |
