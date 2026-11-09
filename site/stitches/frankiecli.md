# Frankiecli

Structured command-line argument parsing for Frankie scripts. Zero dependencies — pure `.fk`, built on `argv()`.

Introduced in **v1.16**.

---

## Functions

### `cli_parse(args)`

Parse a raw `argv()` vector into a structured hash.

```
{
  args:    [...],     # positional arguments (non-flag values)
  flags:   {...},     # boolean flags — {verbose: true, dry_run: true}
  options: {...},     # named values   — {output: "file.txt", port: "8080"}
  command: "..."      # first positional arg (or nil), for subcommand CLIs
}
```

Supported input formats:

| Input | Result |
|---|---|
| `--flag` | `flags["flag"] = true` |
| `--no-flag` | `flags["flag"] = false` |
| `--key value` | `options["key"] = "value"` |
| `--key=value` | `options["key"] = "value"` |
| `-f` | `flags["f"] = true` |
| `positional` | pushed to `args` |

Dashes in flag/option names are converted to underscores: `--dry-run` → `flags["dry_run"]`.

---

### `cli_get(cli, key, default)`

Get a named option value, or `default` if the option was not passed.

```ruby
output = cli_get(cli, "output", "result.txt")
port   = cli_get(cli, "port",   "3000")
```

---

### `cli_flag(cli, name)`

Check if a boolean flag was passed. Returns `false` if absent.

```ruby
verbose = cli_flag(cli, "verbose")
dry_run = cli_flag(cli, "dry-run")   # normalized automatically
```

---

### `cli_arg(cli, index, default)`

Get a positional argument by index, or `default` if not enough args were given.

```ruby
filename = cli_arg(cli, 0, "input.txt")
target   = cli_arg(cli, 1, "output.txt")
```

---

### `cli_require(cli, key, msg)`

Like `cli_get` but halts with an error message and `exit(1)` if the option is missing.

```ruby
db_url = cli_require(cli, "db", "Please provide --db <url>")
```

---

## Quick Start

```ruby
stitch "frankiecli"

cli = cli_parse(argv())

name    = cli_get(cli,  "name",    "World")
debug   = cli_flag(cli, "verbose")
outfile = cli_get(cli,  "output",  "out.txt")

puts "Hello, #{name}!"
puts "Debug mode: #{debug}"
puts "Output: #{outfile}"
```

```bash
frankiec run hello.fk --name Alice --verbose --output report.txt
# Hello, Alice!
# Debug mode: true
# Output: report.txt
```

---

## Subcommand Pattern

```ruby
stitch "frankiecli"

cli = cli_parse(argv())
cmd = cli["command"]

if cmd == "build"
  target = cli_get(cli, "target", "debug")
  puts "Building for #{target}..."

elsif cmd == "deploy"
  env_name = cli_require(cli, "env", "Usage: deploy --env <name>")
  dry_run  = cli_flag(cli, "dry-run")
  puts "Deploying to #{env_name}#{dry_run ? " (dry run)" : ""}..."

elsif cmd == "help" or cmd == nil
  puts "Usage: myscript.fk <command> [options]"
  puts "  build  --target debug|release"
  puts "  deploy --env <name> [--dry-run]"

else
  puts "Unknown command: #{cmd}"
  exit(1)
end
```

```bash
frankiec run myscript.fk deploy --env production
# Deploying to production...

frankiec run myscript.fk deploy --env staging --dry-run
# Deploying to staging (dry run)...
```

---

## --no-flag Negation

```ruby
stitch "frankiecli"

# Pass --no-color to disable colors
cli     = cli_parse(argv())
colored = cli_flag(cli, "color")   # false if --no-color was passed

if colored
  puts green("All good!")
else
  puts "All good!"
end
```

---

## Full Example

```ruby
stitch "frankiecli"

cli = cli_parse(argv())

# Required — exits with error if missing
input = cli_require(cli, "input", "Usage: report.fk --input <file.csv> [--output <file>] [--verbose]")

# Optional with defaults
output  = cli_get(cli,  "output",  "report.txt")
verbose = cli_flag(cli, "verbose")

if verbose
  puts "Reading: #{input}"
end

rows = csv_parse(file_read(input))
puts "Rows: #{rows.length}"

file_write(output, rows.map do |r| r.to_s end.join("\n"))
puts "Written to #{output}"
```

---

## Notes

- All option names are normalized to `snake_case`: `--dry-run` → `options["dry_run"]`.
- `cli_parse` does not call `argv()` automatically — pass `argv()` explicitly so you can also pass test arrays.
- Positional arguments come before or between flags; `--key value` consumes the next token only when it doesn't start with `-`.
