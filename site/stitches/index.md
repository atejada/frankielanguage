# Stitches

## `What are stitches?`

Frankie's zero-dependency, zero-registry package system. Drop a `.fk` file in the right folder and stitch it in.

| Feature | Category | Summary |
|---|---|---|
| **`stitch "name"`** | language | Load a package by name — resolves from `./stitches/` then `~/.frankie/stitches/` |
| **`frankieforms`** | stitch | Form field validation — required, min/max length, email, numeric, pattern |
| **`frankietable`** | stitch | ASCII table rendering from a vector of hashes |
| **`frankiecolor`** | stitch | ANSI color helpers for terminal output |
| **`frankiepager`** | stitch | Pagination math — total pages, slicing, prev/next |
| **`frankieconfig`** | stitch | Layered config loading from defaults, JSON file, and env vars |
| **`frankiestring`** | stitch | Additional string functions |
| **`frankietemplate`** | stitch | Mustache-compatible template rendering for Frankie |
| **`frankiecookie`** | stitch | Signed cookie helpers for Frankie |
| **`frankieauth`** | stitch | HTTP Basic Auth and Bearer token authentication — introduced in v1.15 |
| **`frankieratelimit`** | stitch | In-memory per-IP sliding-window rate limiting — introduced in v1.15 |
| **`frankiemail`** | stitch | Send email via SMTP — plain text or HTML, CC/BCC — introduced in v1.16 |
| **`frankiecli`** | stitch | Structured CLI argument parsing — flags, options, subcommands — introduced in v1.16 |
| **`frankiecache`** | stitch | In-memory key/value cache with optional TTL — introduced in v1.16 |
| **[`frankiegame`](frankiegame.md)** | stitch | Terminal game engine — raw keys, double-buffered ANSI rendering — introduced in v1.20 |
| **[`frankiecanvas`](frankiecanvas.md)** | stitch | Browser game engine, same API as frankiegame — HTML5 canvas over WebSockets — introduced in v1.20 |
---

## The `stitch` Keyword

`stitch` loads a Frankie package by name. It resolves the file in two locations, in order:

1. `./stitches/<name>.fk` — project-local (checked first)
2. `~/.frankie/stitches/<name>.fk` — user-global

If neither exists, a clear error tells you exactly what to do:

```
╔══ Frankie Runtime Error ══════════════════════════════
║  [Frankie] Stitch not found: "frankieforms"
║    Put frankieforms.fk in ./stitches/ or ~/.frankie/stitches/
╚═══════════════════════════════════════════════════════
```

Each stitch is loaded **at most once** — calling `stitch "name"` multiple times in the same program is safe.

---

## Installing Stitches (v1.17)

The stitch installer fetches stitches straight from the Frankie GitHub repository using the Python stdlib HTTP client — a package manager with no packaging:

```bash
frankiec stitch install frankiecolor            # → ./stitches/
frankiec stitch install frankiecache --global   # → ~/.frankie/stitches/
frankiec stitch list                            # installed + registry
```

### Lockfile — `stitch.lock` (v1.18)

Installing a stitch pins it in `stitch.lock` (sha256, source, size, date). Commit the lockfile and your builds are reproducible:

```bash
frankiec stitch install frankiecolor   # writes/updates stitch.lock
frankiec stitch verify                 # ✓ pinned · ⚠ modified · ✗ missing
frankiec stitch update                 # re-fetch everything + re-pin
frankiec stitch update frankiecolor    # or just one
```

`verify` exits 1 on problems — drop it straight into CI. Global installs (`--global`) don't touch the lockfile.

### Installing from any URL (v1.19)

Not every stitch has to come from the Frankie registry:

```bash
frankiec stitch install https://example.com/stitches/foo.fk
```

It's pinned in `stitch.lock` exactly like a registry stitch, with the URL recorded as its source — `verify` and `update` work the same way.

### Compared to `require`

| | `require` | `stitch` |
|---|---|---|
| Path | Explicit: `require "lib/utils"` | By name: `stitch "frankieforms"` |
| Resolution | Relative to cwd | `./stitches/` then `~/.frankie/stitches/` |
| Signals | Your own code | Third-party packages |
| Convention | `lib/` folder | `stitches/` folder |

They use the same underlying machinery — `stitch` is `require` with a conventional resolution path and a more meaningful name for the job.

### `Project layout with stitches`

```
myapp/
├── main.fk
├── test.fk
├── lib/
│   └── utils.fk        ← your own code, loaded with require
└── stitches/
    ├── frankieforms.fk   ← third-party, loaded with stitch
    └── frankietable.fk
```

Anyone cloning your repo immediately knows what `stitches/` contains.

---

## `Writing Your Own Stitch`

A stitch is any `.fk` file. Drop it in `./stitches/` and `stitch "name"` loads it.

```ruby
# stitches/myutils.fk

def slugify(s)
  s.downcase.gsub("[^a-z0-9]+", "-").strip
end

def truncate(s, n)
  if s.length <= n
    s
  else
    s[0...n] + "..."
  end
end
```

```ruby
# main.fk
stitch "myutils"

puts slugify("Hello World!")    # hello-world
puts truncate("Long string", 4) # Long...
```

## `Conventions`

- Prefix private helper functions with `_` to signal they're internal.
- Use `##` doc-comments for `frankiec docs` compatibility.
- Keep each stitch focused on one concern.
- Avoid redefining stdlib functions.
