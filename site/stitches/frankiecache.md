# Frankiecache

In-memory key/value cache with optional TTL (time-to-live). Zero dependencies — pure `.fk`, no external libraries. State resets on process restart — use the database for persistence.

Introduced in **v1.16**.

---

## Functions

### `cache_set(key, value, ttl: 0)`

Store a value under `key`. If `ttl > 0`, the entry expires after that many seconds. Returns the stored value.

```ruby
cache_set("user:42", {name: "Alice"})           # no expiry
cache_set("session:abc", {user: "Alice"}, ttl: 300)  # expires in 5 min
```

---

### `cache_get(key)`

Retrieve a cached value. Returns `nil` if the key does not exist or has expired.

```ruby
user = cache_get("user:42")
if user == nil
  # cache miss — load from database
end
```

---

### `cache_set?(key)`

Returns `true` if the key exists and has not expired.

```ruby
if cache_set?("config")
  cfg = cache_get("config")
end
```

---

### `cache_delete(key)`

Remove a single key from the cache.

```ruby
cache_delete("user:42")
```

---

### `cache_clear()`

Remove all entries from the cache.

```ruby
cache_clear()
```

---

### `cache_size()`

Return the number of live (non-expired) entries.

```ruby
puts "#{cache_size()} entries in cache"
```

---

### `cache_fetch(key, ttl: 0)`

> **Note:** Block-based `cache_fetch` is not yet supported in this version of Frankie's stitch system. Use the explicit set/get pattern below instead.

---

## Quick Start

```ruby
stitch "frankiecache"

# Store user data for 5 minutes
cache_set("user:42", {name: "Alice", role: "admin"}, ttl: 300)

user = cache_get("user:42")
if user != nil
  puts "Cache hit: #{user["name"]}"
else
  puts "Cache miss"
end

puts cache_size()   # 1
```

---

## Cache-Aside Pattern

The most common caching pattern — check the cache first, load from the database on a miss, then cache the result:

```ruby
stitch "frankiecache"

def get_user(db, id)
  key    = "user:#{id}"
  cached = cache_get(key)
  if cached != nil
    return cached
  end
  user = db.find_one("users", {id: id})
  if user != nil
    cache_set(key, user, ttl: 60)
  end
  return user
end

db = db_open("app.db")
user = get_user(db, 42)
puts user["name"]
```

---

## TTL-Based Rate Limiting (simple)

```ruby
stitch "frankiecache"

def allow_request?(ip)
  key   = "rate:#{ip}"
  count = cache_get(key)
  if count == nil
    cache_set(key, 1, ttl: 60)
    return true
  end
  if count >= 30
    return false
  end
  cache_set(key, count + 1, ttl: 60)
  return true
end

puts allow_request?("10.0.0.1")   # true
```

---

## Config Caching

```ruby
stitch "frankiecache"

def load_config()
  cached = cache_get("app_config")
  if cached != nil
    return cached
  end
  cfg = json_parse(file_read("config.json"))
  cache_set("app_config", cfg, ttl: 300)
  return cfg
end

config = load_config()
puts config["database"]["host"]
```

---

## Cache Stats and Inspection

```ruby
stitch "frankiecache"

cache_set("a", 1)
cache_set("b", 2, ttl: 10)
cache_set("c", 3, ttl: 10)

puts "size: #{cache_size()}"           # 3
puts "a exists: #{cache_set?("a")}"    # true
puts "x exists: #{cache_set?("x")}"    # false

cache_delete("b")
puts "after delete: #{cache_size()}"   # 2

cache_clear()
puts "after clear: #{cache_size()}"    # 0
```

---

## Notes

- The cache is **process-local and in-memory** — it does not persist across restarts and is not shared between processes.
- TTL is checked lazily on `cache_get` — expired entries are evicted on access, not by a background timer.
- `cache_size()` counts only live (non-expired) entries by iterating and checking TTLs.
- For persistent or distributed caching, use an external store (Redis, SQLite, etc.) and call it from your Frankie script directly.
