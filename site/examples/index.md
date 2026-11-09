# Examples

## Showcase Projects (v1.19-v1.20)

Beyond the single-file snippets below, Frankie ships six complete, playable projects — each a self-contained `.fk` program (or small pinned-stitch project) you can read in one sitting and run with one command. They install with Frankie itself, so there's nothing extra to download.

Grab any of them with `frankiec examples`, which works the same way whether Frankie was installed via Homebrew, `git clone` + `install.py`, or an unpacked zip:

```bash
frankiec examples              # list everything bundled with this install
frankiec examples snake        # copies the project into the current directory
cd snake && frankiec run main.fk
```

| Project | Renderer / stack | What it shows |
|---|---|---|
| 🧟 `zombie_chat` | WebSockets | Multi-room chat — server + web UI in one file |
| ⚡ `word_reanimator` | WebSockets | Multiplayer browser hangman, shared state |
| 💰 `frankie_ledger` | SQLite + stitches | Terminal expense tracker with R-style stats |
| 🐍 `snake` | `frankiegame` (terminal) | The whole game engine API in ~80 lines |
| 🧟 `zombie_invaders` | `frankiecanvas` (browser) | Rects, sprites, waves, beeps |
| 🏓 `pong` | `frankiecanvas` (browser) | Multiplayer via `on_player_key` — two tabs, one ball |

Each project bundles to a single self-contained `.py` with `frankiec bundle` — no Frankie installation required to run the result.

---

## Hello World

```ruby
name = "World"
puts "Hello, #{name}!"
puts "Welcome to Frankie — stitched together from the finest languages."
```

## Fizzbuzz

```ruby
for i in 1..100
  if i % 15 == 0
    puts "FizzBuzz"
  elsif i % 3 == 0
    puts "Fizz"
  elsif i % 5 == 0
    puts "Buzz"
  else
    puts i
  end
end
```

## Fibonacci

```ruby
def fibonacci(n)
  if n <= 1
    return n
  end
  return fibonacci(n - 1) + fibonacci(n - 2)
end

for i in 0..10
  puts "fib(#{i}) = #{fibonacci(i)}"
end
```

## Stats

```ruby
data = [23, 45, 12, 67, 34, 89, 15, 56, 78, 42]

puts "Mean   : #{mean(data)}"
puts "StdDev : #{stdev(data)}"
puts "Median : #{median(data)}"

# Pipe operator
data |> sum |> puts

# Vectorized operations
v = [1, 2, 3, 4, 5]
puts v * 2          # [2, 4, 6, 8, 10]
```

## Sorting

```ruby
def bubble_sort(arr)
  n = length(arr)
  i = 0
  while i < n - 1
    j = 0
    while j < n - i - 1
      if arr[j] > arr[j + 1]
        temp = arr[j]
        arr[j] = arr[j + 1]
        arr[j + 1] = temp
      end
      j = j + 1
    end
    i = i + 1
  end
  return arr
end

def selection_sort(arr)
  n = length(arr)
  i = 0
  while i < n
    min_idx = i
    j = i + 1
    while j < n
      if arr[j] < arr[min_idx]
        min_idx = j
      end
      j = j + 1
    end
    temp = arr[i]
    arr[i] = arr[min_idx]
    arr[min_idx] = temp
    i = i + 1
  end
  return arr
end

puts "=== Frankie Sorting Algorithms ==="
puts ""

data1 = [64, 34, 25, 12, 22, 11, 90]
data2 = [29, 10, 14, 37, 13, 0, 99, 55]

puts "Original (1): #{data1}"
sorted1 = bubble_sort(data1)
puts "Bubble Sort : #{sorted1}"

puts ""

puts "Original (2): #{data2}"
sorted2 = selection_sort(data2)
puts "Select Sort : #{sorted2}"

puts ""
puts "=== Using built-in sort ==="
v = [5, 3, 8, 1, 9, 2, 7, 4, 6]
puts "Unsorted: #{v}"
puts "Sorted  : #{v.sort}"
```

## Leds

```ruby
leds = {
  0: [' _  ', '| | ', '|_| '],
  1: ['  ',   '| ',   '| '  ],
  2: [' _  ', ' _| ', '|_  '],
  3: ['_  ',  '_| ',  '_| ' ],
  4: ['    ', '|_| ', '  | '],
  5: [' _  ', '|_  ', ' _| '],
  6: [' _  ', '|_  ', '|_| '],
  7: ['_   ', ' |  ', ' |  '],
  8: [' _  ', '|_| ', '|_| '],
  9: [' _  ', '|_| ', ' _| ']
}

print "Enter a number: "
number = input("")

for i in 0...3
  for j in 0...length(number)
    print leds[to_int(number[j])][i]
  end
  puts ""
end
```

## Database

```ruby
puts "╔══════════════════════════════════════════╗"
puts "║     Frankie Database Demo (SQLite)       ║"
puts "╚══════════════════════════════════════════╝"
puts ""

# ── 1. Open an in-memory database ────────────────────────────────
db = db_open(":memory:")
puts "── 1. Opened in-memory database ──"
puts db
puts ""

# ── 2. Create tables ─────────────────────────────────────────────
puts "── 2. Create Tables ──"
db.exec("CREATE TABLE departments (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT NOT NULL UNIQUE)")

db.exec("CREATE TABLE employees (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT NOT NULL, age INTEGER NOT NULL, department TEXT NOT NULL, salary REAL NOT NULL)")

puts "Tables created: #{db.tables}"
puts ""

# ── 3. Insert rows ───────────────────────────────────────────────
puts "── 3. Insert Rows ──"

# Low-level exec with params (? placeholders — safe from SQL injection)
db.exec("INSERT INTO departments (name) VALUES (?)", ["Engineering"])
db.exec("INSERT INTO departments (name) VALUES (?)", ["Marketing"])
db.exec("INSERT INTO departments (name) VALUES (?)", ["Finance"])

# Convenience insert with hash
db.insert("employees", {name: "Alice",   age: 32, department: "Engineering", salary: 95000})
db.insert("employees", {name: "Bob",     age: 28, department: "Engineering", salary: 87000})
db.insert("employees", {name: "Carol",   age: 35, department: "Marketing",   salary: 72000})
db.insert("employees", {name: "Dave",    age: 41, department: "Finance",     salary: 105000})
db.insert("employees", {name: "Eve",     age: 29, department: "Engineering", salary: 91000})
db.insert("employees", {name: "Frank",   age: 38, department: "Marketing",   salary: 68000})
db.insert("employees", {name: "Grace",   age: 45, department: "Finance",     salary: 118000})
db.insert("employees", {name: "Hector",  age: 27, department: "Engineering", salary: 82000})

puts "Inserted #{db.count("employees")} employees"
puts "Inserted #{db.count("departments")} departments"
puts ""

# ── 4. Query all rows ────────────────────────────────────────────
puts "── 4. All Employees ──"
all = db.find_all("employees")
all.each do |row|
  puts "  [#{row["id"]}] #{row["name"]} | #{row["department"]} | age #{row["age"]} | $#{row["salary"]}"
end
puts ""

# ── 5. Filtered queries ──────────────────────────────────────────
puts "── 5. Engineering Team ──"
engineers = db.find("employees", {department: "Engineering"})
engineers.each do |e|
  puts "  #{e["name"]} (age #{e["age"]}) — $#{e["salary"]}"
end
puts ""

# ── 6. Raw SQL queries ───────────────────────────────────────────
puts "── 6. Employees Over 35 ──"
seniors = db.query("SELECT name, age, salary FROM employees WHERE age > ? ORDER BY age", [35])
seniors.each do |row|
  puts "  #{row["name"]} | age #{row["age"]} | $#{row["salary"]}"
end
puts ""

# ── 7. Aggregation ──────────────────────────────────────────────
puts "── 7. Department Summary ──"
summary = db.query("SELECT department, COUNT(*) AS headcount, AVG(salary) AS avg_salary, MAX(salary) AS top_salary FROM employees GROUP BY department ORDER BY avg_salary DESC")
summary.each do |row|
  avg = to_int(row["avg_salary"])
  puts "  #{row["department"]}: #{row["headcount"]} people | avg $#{avg} | top $#{row["top_salary"]}"
end
puts ""

# ── 8. find_one ──────────────────────────────────────────────────
puts "── 8. Find One ──"
alice = db.find_one("employees", {name: "Alice"})
puts "Found: #{alice["name"]} in #{alice["department"]}"
missing = db.find_one("employees", {name: "Nobody"})
puts "Missing: #{missing}"
puts ""

# ── 9. Update ────────────────────────────────────────────────────
puts "── 9. Update ──"
updated = db.update("employees", {salary: 100000}, {name: "Bob"})
puts "Updated #{updated} row(s)"
bob = db.find_one("employees", {name: "Bob"})
puts "Bob's new salary: $#{bob["salary"]}"
puts ""

# ── 10. Transaction ──────────────────────────────────────────────
puts "── 10. Transaction ──"
db.transaction do
  db.insert("employees", {name: "Ivan",  age: 31, department: "Finance",     salary: 97000})
  db.insert("employees", {name: "Julia", age: 26, department: "Engineering", salary: 79000})
end
puts "After transaction: #{db.count("employees")} employees"
puts ""

# ── 11. Transaction rollback on error ────────────────────────────
puts "── 11. Transaction Rollback ──"
count_before = db.count("employees")
begin
  db.transaction do
    db.insert("employees", {name: "Karl", age: 33, department: "Marketing", salary: 71000})
    raise "simulated error mid-transaction"
    db.insert("employees", {name: "Lena", age: 29, department: "Marketing", salary: 66000})
  end
rescue e
  puts "Rolled back: #{e}"
end
puts "Count unchanged: #{db.count("employees") == count_before}"
puts ""

# ── 12. Delete ───────────────────────────────────────────────────
puts "── 12. Delete ──"
deleted = db.delete("employees", {name: "Hector"})
puts "Deleted #{deleted} row(s)"
puts "Remaining: #{db.count("employees")} employees"
puts ""

# ── 13. Schema introspection ─────────────────────────────────────
puts "── 13. Schema Introspection ──"
puts "Tables: #{db.tables}"
cols = db.columns("employees")
puts "employees columns:"
cols.each do |col|
  puts "  #{col["name"]} (#{col["type"]})"
end
puts ""

# ── 14. Frankie-style stats on query results ─────────────────────
puts "── 14. Stats on Query Results ──"
salaries = db.query("SELECT salary FROM employees")
salary_list = salaries.map do |row|
  row["salary"]
end
puts "Salaries   : #{salary_list}"
puts "Mean salary: $#{to_int(mean(salary_list))}"
puts "Max salary : $#{max(salary_list)}"
puts "Min salary : $#{min(salary_list)}"
puts "Std dev    : $#{to_int(stdev(salary_list))}"
puts ""

# ── 15. Persistent file database ─────────────────────────────────
puts "── 15. File Database ──"
db2 = db_open("/tmp/frankie_demo.db")
db2.exec("CREATE TABLE IF NOT EXISTS notes (id INTEGER PRIMARY KEY, content TEXT)")
db2.insert("notes", {content: "Frankie has SQLite support!"})
db2.insert("notes", {content: "Zero external dependencies."})
notes = db2.find_all("notes")
notes.each do |n|
  puts "  Note #{n["id"]}: #{n["content"]}"
end
db2.close
file_delete("/tmp/frankie_demo.db")
puts "File DB created, used, and cleaned up."
puts ""

# ── Done ─────────────────────────────────────────────────────────
db.close
puts "Database connection closed."
puts "Frankie + SQLite — alive and querying! 🧟🗄️"
```

## Hashmaps

```ruby
puts "╔══════════════════════════════════╗"
puts "║     Frankie Hashmap Demo         ║"
puts "╚══════════════════════════════════╝"
puts ""

# ─── 1. Creating a Hash ───────────────────────────────────────────────────────
puts "── 1. Creating a Hash ──"
person = {name: "Alice", age: 30, city: "Lima"}
puts person
puts ""

# ─── 2. Reading Values ────────────────────────────────────────────────────────
puts "── 2. Reading Values ──"
puts "Name : #{person[:name]}"
puts "Age  : #{person[:age]}"
puts "City : #{person[:city]}"
puts ""

# ─── 3. Adding & Updating Keys ───────────────────────────────────────────────
puts "── 3. Adding & Updating ──"
person[:job] = "Engineer"
person[:age] = 31
puts "After update: #{person}"
puts ""

# ─── 4. Keys & Values ────────────────────────────────────────────────────────
puts "── 4. Keys & Values ──"
puts "Keys   : #{person.keys}"
puts "Values : #{person.values}"
puts "Size   : #{person.size}"
puts ""

# ─── 5. has_key? ─────────────────────────────────────────────────────────────
puts "── 5. Checking Keys ──"
puts "Has :name?   #{person.has_key?(:name)}"
puts "Has :email?  #{person.has_key?(:email)}"
puts ""

# ─── 6. delete ───────────────────────────────────────────────────────────────
puts "── 6. Deleting a Key ──"
person.delete(:city)
puts "After deleting :city: #{person}"
puts ""

# ─── 7. fetch with default ───────────────────────────────────────────────────
puts "── 7. Fetch with Default ──"
job    = person.fetch(:job, "Unknown")
salary = person.fetch(:salary, 0)
puts "Job    : #{job}"
puts "Salary : #{salary}"
puts ""

# ─── 8. Iterating with each ──────────────────────────────────────────────────
puts "── 8. Iterating (each) ──"
person.each do |key, val|
  puts "  #{key} => #{val}"
end
puts ""

# ─── 9. merge ────────────────────────────────────────────────────────────────
puts "── 9. Merging Hashes ──"
extra = {email: "alice@example.com", country: "Peru"}
full  = person.merge(extra)
puts "Merged: #{full}"
puts ""

# ─── 10. Nested Hashes ───────────────────────────────────────────────────────
puts "── 10. Nested Hashes ──"
company = {
  name: "Frankieworks",
  address: {street: "Av. Larco", city: "Miraflores"},
  employees: 42
}
puts "Company : #{company[:name]}"
puts "Street  : #{company[:address][:street]}"
puts "City    : #{company[:address][:city]}"
puts ""

# ─── 11. Hash of Vectors ─────────────────────────────────────────────────────
puts "── 11. Hash of Vectors ──"
scores = {
  alice: [90, 85, 92],
  bob:   [78, 80, 88],
  carol: [95, 99, 97]
}

scores.each do |student, s|
  avg = mean(s)
  puts "  #{student}: avg = #{avg}"
end
puts ""

# ─── 12. Building a Hash dynamically ─────────────────────────────────────────
puts "── 12. Building Dynamically ──"
word_count = {}
words = ["frankie", "is", "fun", "frankie", "is", "fast", "frankie"]

for word in words
  if word_count.has_key?(word)
    word_count[word] = word_count[word] + 1
  else
    word_count[word] = 1
  end
end

puts "Word counts:"
word_count.each do |word, count|
  puts "  #{word}: #{count}"
end
puts ""

puts "Done! Frankie hashmaps are alive 🧟"
```

## Webapp

```ruby
# Run with:   frankiec run examples/webapp.fk
# Then visit: http://localhost:3000

app = web_app()

# ── Simple HTML response ──────────────────────────────────────────────────────
app.get("/") do |req|
  html_response("<h1>Hello from Frankie! 🧟</h1><p>Try <a href='/greet/world'>/greet/world</a></p>")
end

# ── Path parameters ───────────────────────────────────────────────────────────
app.get("/greet/:name") do |req|
  name = req.params["name"]
  response("Hello, #{name}!")
end

# ── Query parameters ──────────────────────────────────────────────────────────
# GET /search?q=frankie
app.get("/search") do |req|
  q = req.query["q"]
  if q == nil
    return response("Please provide ?q=something")
  end
  response("You searched for: #{q}")
end

# ── JSON response ─────────────────────────────────────────────────────────────
app.get("/api/status") do |req|
  json_response({status: "ok", language: "Frankie", version: "1.4"})
end

# ── In-memory notes store ─────────────────────────────────────────────────────
# Use a wrapper hash so handlers can mutate the list without scoping issues
store = {notes: []}

app.get("/notes") do |req|
  json_response(store["notes"])
end

app.post("/notes") do |req|
  data = req.json
  if data == nil
    return halt(400, "Expected JSON body")
  end
  note = {id: length(store["notes"]) + 1, text: data["text"]}
  store["notes"].push(note)
  json_response(note, 201)
end

app.get("/notes/:id") do |req|
  id = to_int(req.params["id"])
  found = nil
  store["notes"].each do |n|
    if n["id"] == id
      found = n
    end
  end
  if found == nil
    return halt(404, "Note not found")
  end
  json_response(found)
end

app.delete("/notes/:id") do |req|
  id = to_int(req.params["id"])
  store["notes"] = store["notes"].select do |n|
    n["id"] != id
  end
  response("Deleted")
end

# ── Custom 404 ────────────────────────────────────────────────────────────────
app.not_found do |req|
  html_response("<h1>404</h1><p>No route for #{req.path}</p>", 404)
end

# ── Start ─────────────────────────────────────────────────────────────────────
app.run(3000)
```
