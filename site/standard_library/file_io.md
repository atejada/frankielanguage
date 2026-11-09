# File I/O

## Overview

Frankie has a complete file I/O library built on Python's standard `open()` — zero external dependencies. 

All functions are available without any import.

---

## Reading Files

### `file_read(path)`

Read entire file as a string

```ruby
content = file_read("data.txt")
puts content
```

### `file_lines(path)`

Read file as a vector of lines

Strips trailing newlines from each line.

```ruby
lines = file_lines("data.csv")
puts lines.length        → number of lines
puts lines[0]            → first line
lines.each do |line|
  puts line
end
```

### `file_read_base64(path)` *(v1.21)*

Read a binary file, Base64-encoded

`file_read` is text-mode and will corrupt binary data (images, audio, zip files). Use `file_read_base64` for anything that isn't text — [`load_image`](../stitches/frankiecanvas.md#load_image-path-v121) in `frankiegame`/`frankiecanvas` uses it internally for PNG/JPEG/GIF sprites.

```ruby
data = file_read_base64("sprite.png")
puts data.length   # Base64 text, safe to embed in a data: URL or JSON
```

Both `file_read` and `file_lines` (and `file_read_base64`) raise `FileNotFoundError` if the file doesn't exist — use `rescue` to handle it gracefully:

```ruby
begin
  content = file_read("config.json")
rescue FileNotFoundError e
  puts "Config missing — using defaults"
  content = "{}"
end
```

---

## Writing Files

### `file_write(path, content)`

Write string to file (overwrites)

Creates the file if it doesn't exist. Overwrites silently if it does.

```ruby
file_write("output.txt", "Hello, Frankie!\n")
file_write("report.csv", "name,score\nAlice,95\nBob,87\n")
```

### `file_append(path, content)`

Append to file

Adds content to the end of the file without overwriting.

```ruby
file_write("log.txt", "Session started\n")
file_append("log.txt", "User logged in\n")
file_append("log.txt", "User logged out\n")

puts file_read("log.txt")

→ Session started
→ User logged in
→ User logged out
```

---

## File Management

### `file_exists(path)`

Check if a file exists

Returns `true` or `false`. Never raises.

```ruby
if file_exists("config.json")
  config = json_read("config.json")
else
  config = {host: "localhost", port: 3000}
end
```

### `file_delete(path)`

Delete a file

Returns `false` if the file doesn't exist — deleting a missing file is a no-op, not an error.

```ruby
file_delete("temp.txt")          → true if deleted, false if missing
file_delete("/tmp/scratch.json") → safe to call even if missing
```

### `file_copy(src, dst)`

Copy a file

Returns the destination path on success. Raises `FileNotFoundError` if source is missing.

```ruby
file_copy("data.csv", "data.csv.bak")
```

### `file_rename(src, dst)`

Rename or move a file

Raises `FileNotFoundError` if source is missing.

```ruby
file_rename("draft.txt", "final.txt")
file_rename("output.csv", "/tmp/output.csv")   → move to different directory
```

---

## Directory Operations

### `file_mkdir(path)`

Create a directory

Creates all intermediate directories automatically (like `mkdir -p`).

```ruby
file_mkdir("data/reports/2024")   → creates all three levels
```

### `dir_exists(path)`

Check if a directory exists

```ruby
if not dir_exists("output")
  file_mkdir("output")
end
```

### `dir_list(path)`

List directory contents

Returns a sorted vector of filenames (not full paths). Defaults to current directory.

```ruby
entries = dir_list("data")
puts entries   → ["config.json", "users.csv", "report.txt"]

# List current directory
puts dir_list()
```

---

## File Handles

For fine-grained control, open a file as a handle with `file_open`:

```ruby
f = file_open("output.txt", "w")
f.write("line one\n")
f.write("line two\n")
f.close
```

Modes: `"r"` (read), `"w"` (write/overwrite), `"a"` (append).

Always close the handle when done. Use `ensure` to guarantee it:

```ruby
f = file_open("data.txt", "r")
begin
  content = f.read
  puts content
ensure
  f.close
end
```

---

## Consistent Error Behaviour

| Function | Missing file behaviour |
|---|---|
| `file_read(path)` | Raises `FileNotFoundError` |
| `file_read_base64(path)` | Raises `FileNotFoundError` |
| `file_lines(path)` | Raises `FileNotFoundError` |
| `file_copy(src, dst)` | Raises `FileNotFoundError` (src must exist) |
| `file_rename(src, dst)` | Raises `FileNotFoundError` (src must exist) |
| `file_delete(path)` | Returns `false` — idempotent |
| `file_exists(path)` | Returns `false` — that's what it tests |
| `dir_exists(path)` | Returns `false` — same |


---

## Quick Reference

| Function | Description |
|---|---|
| `file_read(path)` | Read entire file as string |
| `file_read_base64(path)` | Read binary file, Base64-encoded (v1.21) |
| `file_lines(path)` | Read file as vector of lines |
| `file_write(path, str)` | Write string to file (overwrites) |
| `file_append(path, str)` | Append string to file |
| `file_exists(path)` | True if file exists |
| `file_delete(path)` | Delete file — returns false if missing |
| `file_copy(src, dst)` | Copy file |
| `file_rename(src, dst)` | Rename or move file |
| `file_open(path, mode)` | Open file handle (`"r"`, `"w"`, `"a"`) |
| `file_mkdir(path)` | Create directory (and parents) |
| `dir_exists(path)` | True if directory exists |
| `dir_list(path)` | Sorted vector of filenames in directory |
| `path_join(*parts)` | Join path segments with the OS separator |
| `path_dirname(path)` | Parent directory of a path |
| `path_basename(path)` | Filename component of a path |
| `path_extname(path)` | File extension including the dot |
| `path_stem(path)` | Filename without the extension |
| `path_absolute(path)` | Resolve to an absolute path |

---

## Path Helpers *(introduced in v1.15)*

Pure path manipulation — no filesystem access required.

### `path_join(*parts)`

Join one or more path segments with the OS separator. Handles trailing slashes correctly.

```ruby
puts path_join("home", "alice", "docs")       # home/alice/docs
puts path_join("/usr", "local", "bin")        # /usr/local/bin
```

### `path_dirname(path)`

Return the parent directory of the given path.

```ruby
puts path_dirname("/home/alice/report.txt")   # /home/alice
puts path_dirname("data/results.csv")         # data
```

### `path_basename(path)`

Return the filename component (last segment) of the path.

```ruby
puts path_basename("/home/alice/report.txt")  # report.txt
puts path_basename("data/results.csv")        # results.csv
```

### `path_extname(path)`

Return the file extension including the leading dot. Returns an empty string if there is none.

```ruby
puts path_extname("/home/alice/report.txt")   # .txt
puts path_extname("archive.tar.gz")           # .gz
puts path_extname("Makefile")                 # (empty string)
```

### `path_stem(path)`

Return the filename without the extension.

```ruby
puts path_stem("/home/alice/report.txt")      # report
puts path_stem("archive.tar.gz")              # archive.tar
```

### `path_absolute(path)`

Resolve a relative path to an absolute path based on the current working directory.

```ruby
puts path_absolute("./data/db.sqlite")        # /home/alice/myapp/data/db.sqlite
puts path_absolute("../config.json")          # /home/alice/config.json
```

---

## Shell & Environment *(introduced in v1.16)*

### `shell(cmd)`

Run an OS shell command and return a structured result hash. Uses `subprocess` internally — no shell injection surprises.

| Field | Type | Description |
|---|---|---|
| `stdout` | string | Standard output from the command |
| `stderr` | string | Standard error from the command |
| `exit_code` | integer | Process exit code (`0` = success) |
| `ok` | boolean | `true` when `exit_code == 0` |

```ruby
result = shell("git log --oneline -5")
if result["ok"]
  puts result["stdout"]
else
  puts "git failed (exit #{result["exit_code"]}): #{result["stderr"]}"
end

# Capture command output into a variable
branch = shell("git rev-parse --abbrev-ref HEAD")["stdout"].strip
puts "Current branch: #{branch}"

# Check for a program's existence
if shell("which python3")["ok"]
  puts "python3 is available"
end
```

---

### `dotenv(path = ".env")`

Load a `.env` file and merge its key/value pairs into the process environment (accessible via `env()`). Returns a hash of the keys that were loaded.

```ruby
# Load the default .env file
dotenv()

# Load a specific file
dotenv(".env.production")

# The loaded vars are now available via env()
puts env("DATABASE_URL")
puts env("SECRET_KEY")
```

The `.env` file format:

```bash
# .env — lines starting with # are comments, blank lines are ignored
DATABASE_URL=postgres://localhost/myapp
SECRET_KEY=changeme
PORT=3000
```

```ruby
# Inspect what was loaded
loaded = dotenv(".env.staging")
puts "Loaded #{loaded.keys.length} variables"
loaded.each do |k, v|
  puts "  #{k}=#{v}"
end
```

Combined with `shell()`:

```ruby
dotenv()   # load .env first

result = shell("pg_dump #{env("DATABASE_URL")} > backup.sql")
if result["ok"]
  puts "Backup complete"
else
  puts "Backup failed: #{result["stderr"]}"
end
```
