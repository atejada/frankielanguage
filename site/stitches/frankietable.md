# Frankietable

ASCII Tables

## Functions

### `table(rows)`

Render all columns. Columns inferred from first row's keys.

```ruby
stitch "frankietable"

rows = [
  {name: "Alice", dept: "Engineering", salary: 95000},
  {name: "Bob",   dept: "Marketing",   salary: 72000},
  {name: "Carol", dept: "Finance",     salary: 88000}
]

puts table(rows)

→ +-------+-------------+--------+ 
→ | name  | dept        | salary |
→ +-------+-------------+--------+
→ | Alice | Engineering | 95000  |
→ | Bob   | Marketing   | 72000  |
→ | Carol | Finance     | 88000  |
→ +-------+-------------+--------+
```

### `table(rows, cols)`

Render specific columns in the given order.


```ruby
stitch "frankietable"

rows = [
  {name: "Alice", dept: "Engineering", salary: 95000},
  {name: "Bob",   dept: "Marketing",   salary: 72000},
  {name: "Carol", dept: "Finance",     salary: 88000}
]

puts table(rows, ["name", "salary"])

→ +-------+--------+ 
→ | name  | salary |
→ +-------+--------+
→ | Alice | 95000  |
→ | Bob   | 72000  |
→ | Carol | 88000  |
→ +-------+--------+
```

### `table_print(rows, cols)`

Shorthand for `puts table(rows, cols)`.

```ruby
stitch "frankietable"

rows = [
  {name: "Alice", dept: "Engineering", salary: 95000},
  {name: "Bob",   dept: "Marketing",   salary: 72000},
  {name: "Carol", dept: "Finance",     salary: 88000}
]

table_print(rows, ["name", "salary"])

→ +-------+--------+ 
→ | name  | salary |
→ +-------+--------+
→ | Alice | 95000  |
→ | Bob   | 72000  |
→ | Carol | 88000  |
→ +-------+--------+
```
