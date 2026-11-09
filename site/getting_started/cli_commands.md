# CLI Commands

* `frankiec`                                       - launch the REPL
* `frankiec new   [--game] <project>`              - scaffold a new project (`--game`: playable starter, v1.20)
* `frankiec run   [--debug] <file.fk>`             - run a program (`--debug`: break at line 1, v1.19)
* `frankiec build <file.fk>`                       - compile to Python source
* `frankiec bundle <file.fk> [-o out.py]`          - compile to ONE self-contained .py (v1.18)
* `frankiec check [--strict] <file.fk | dir>`      - syntax check + static analysis (v1.17); directories recurse (v1.18)
* `frankiec test  [file.fk] [--filter <name>] [--tag <tag>] [--coverage]` - run test suite (default: test.fk); `--coverage` since v1.19
* `frankiec fmt   [--write] [--check] <file.fk | dir>` - auto-format; directories recurse (v1.18)
* `frankiec docs  [--html] [--output out.md] <file.fk>` - generate docs; `--html` since v1.19
* `frankiec lsp`                                   - start the Language Server (v1.18)
* `frankiec stitch install <name | url> [--global]` - install a stitch from the registry or any URL (v1.17; URLs v1.19)
* `frankiec stitch list`                           - list installed + registry stitches (v1.17)
* `frankiec stitch verify`                         - check installed stitches against stitch.lock (v1.18)
* `frankiec stitch update [name]`                  - re-fetch + re-pin stitches (v1.18)
* `frankiec examples [name]`                       - list bundled examples, or copy one here (v1.20.1)
* `frankiec repl  [--no-banner]`                   - interactive REPL
* `frankiec watch <file.fk> [--test]`              - re-run on save
* `frankiec version`                               - show version
* `frankiec --help`                                - full usage
* `frankiec <cmd> --help`                          - per-command help

---

## `frankiec lsp` — Language Server (v1.18)

Frankie speaks the Language Server Protocol over stdio, built directly on the static analyzer:

* **Live diagnostics** — undefined names, wrong argument counts, unused variables, lex/parse errors — as you type.
* **Completion** — stdlib functions (with signatures + docs), your own functions, records, enums, and keywords.
* **Hover** — signatures and documentation; `##` doc-comments above your `def`s show up in hover cards.

The VS Code extension in the compiler repo (`editors/vscode/`) launches the server automatically. For Neovim (0.10+):

```lua
vim.filetype.add({ extension = { fk = "frankie" } })
vim.api.nvim_create_autocmd("FileType", {
  pattern = "frankie",
  callback = function()
    vim.lsp.start({ name = "frankie", cmd = { "frankiec", "lsp" } })
  end,
})
```

For Helix (`languages.toml`):

```toml
[language-server.frankie]
command = "frankiec"
args = ["lsp"]

[[language]]
name = "frankie"
scope = "source.frankie"
file-types = ["fk"]
language-servers = ["frankie"]
```

---

## `frankiec examples` — Browse Bundled Examples (v1.20.1)

```bash
frankiec examples              # list everything bundled with this install
frankiec examples snake        # copy a project into the current directory
cd snake && frankiec run main.fk
```

* With no argument, lists every project (in `examples/projects/`) and single-file example bundled with the install, plus the source directory.
* With a name, copies that project directory — or single `.fk` file — into the current directory, ready to run.
* Works identically for Homebrew, git-clone, and `install.py` installs — `frankiec` always knows where its own install lives.
* Won't overwrite an existing file or directory of the same name.

See [Where the Examples Live](installation.md#where-the-examples-live) for details on install-specific paths.

---

## `frankiec new --game` — Playable Starter (v1.20)

```bash
frankiec new --game mygame
cd mygame && frankiec run main.fk
```

Scaffolds a playable starter (a zombie you steer with the arrows) with the `frankiegame` stitch pre-installed and lockfile-pinned. See the [frankiegame](../stitches/frankiegame.md) and [frankiecanvas](../stitches/frankiecanvas.md) stitch pages for the full game engine API.

---

## `frankiec run --debug` — Stepping Debugger (v1.19)

```bash
frankiec run --debug app.fk
```

Breaks at the very first line of the program, dropping you into the `(fkdb)` debug REPL before anything executes — useful for stepping through a program you've never read. See [Debugging & Benchmarking](../standard_library/debugging.md) for the full command reference (`s`, `n`, `stack`, `vars`, `c`).

---

## `frankiec docs --html` (v1.19)

```bash
frankiec docs --html mystitch.fk --output mystitch.html
```

Renders `##` doc-comments (with `@param` / `@return` / `@example`) into a styled single-page HTML document matching the Frankie website's theme, instead of the default Markdown output.

---

## `frankiec bundle` — One-File Distribution (v1.18)

```bash
frankiec bundle app.fk -o app.py
python3 app.py        # anywhere Python 3.8+ exists — no Frankie needed
```

The bundle inlines the entire Frankie stdlib and **pre-compiles every statically referenced file** — `require`, `import` and `stitch` targets, recursively. At runtime they resolve from an embedded registry instead of the filesystem. Dynamic paths (computed at runtime) can't be bundled and produce a warning.

Write a tool, hand someone a single file. That's the whole story.

---

## `frankiec check` — Static Analysis (v1.17)

`check` used to be syntax-only. It now finds real bugs before the program runs:

```
$ frankiec check app.fk
  ✗  app.fk:11 — Undefined function: 'greeet'
  ✗  app.fk:9  — greet() expects 1..2 argument(s), got 3
  ⚠  app.fk:3  — Unused variable 'unused_thing' in function 'greet'

[Frankie] app.fk — 2 error(s), 1 warning(s)
```

* Undefined variables/functions and wrong argument counts are **errors** (exit 1); unused locals are **warnings**.
* `require`, `stitch` and `import` targets are resolved and analyzed, so multi-file programs check cleanly.
* Names inside string interpolation (`"#{typo}"`) are checked too.
* `--strict` makes warnings fail the build (CI mode).
* If a required file can't be resolved (dynamic path), undefined-name errors downgrade to warnings instead of guessing.
* Since v1.18, `check` and `fmt` accept directories and recurse into `.fk` files:

```bash
frankiec check .              # analyze the whole project
frankiec check --strict src   # CI mode — warnings fail too
frankiec fmt --check .        # formatting gate
frankiec fmt --write .        # fix everything in place
```

* Since v1.22.3, `fmt` preserves comments (full-line and trailing) and keeps parentheses that affect precedence.

---

## `frankiec test` — Filtering (v1.17)

```bash
frankiec test                    # run everything
frankiec test --filter math      # only test groups whose name contains "math"
frankiec test --tag slow         # only test groups tagged "slow"
frankiec test --coverage         # v1.19 — report line coverage after the run
```

Works with `test "name", tags: [...] do ... end` groups — skipped groups are reported in the summary. `--coverage` writes `.frankie_coverage.json`, which the LSP reads to show uncovered lines as editor hints. See the [Testing](../standard_library/testing.md) page.

---

## `frankiec stitch` — Stitch Installer (v1.17)

```bash
frankiec stitch install frankiecolor            # → ./stitches/
frankiec stitch install frankiecache --global   # → ~/.frankie/stitches/
frankiec stitch install https://example.com/stitches/foo.fk   # v1.19 — any URL
frankiec stitch list                            # installed + registry
frankiec stitch verify                          # v1.18 — ✓ pinned · ⚠ modified · ✗ missing
frankiec stitch update [name]                   # v1.18 — re-fetch + re-pin
```

Stitches are fetched straight from the Frankie GitHub repository using the Python stdlib HTTP client — a package manager with no packaging. Since v1.19, `install` also accepts arbitrary URLs, not just registry names.

Since v1.18, installs are pinned in `stitch.lock` (sha256, source, size, date). Commit the lockfile for reproducible builds; `verify` exits 1 on problems, so it drops straight into CI. Global installs (`--global`) don't touch the lockfile.

---

## REPL (v1.17 upgrades)

```
fk> 2 + 3
=> 5
fk> _ * 10
=> 50
fk> help parallel_map
  parallel_map(vec, fn, workers=4)
    parallel_map(vec, workers: 4) do |x| ... end
    Runs the block across a thread pool...
```

* Bare expressions echo their value (`=>`), like Ruby's irb.
* `_` always holds the last echoed result.
* `help <function>` shows the signature and documentation of any stdlib or user-defined function.
