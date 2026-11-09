# Getting Started with Frankie

## Requirements

* Python 3.10 or higher
* No external dependencies — pure Python standard library

## Installation

### Homebrew (macOS/Linux)

The easiest path if you're on a Mac or Linux box with Homebrew. Frankie ships as a third-party tap:

```bash
brew tap atejada/frankie
brew trust atejada/frankie     # Homebrew 6.0+: third-party taps need explicit trust
brew install frankie
frankiec version                # Frankie v1.22.3
```

One-liner alternative (auto-taps and trusts just this formula):

```bash
brew install atejada/frankie/frankie
```

Homebrew installs are pure Python stdlib too — the only dependency is `python@3.12`. Standard stitches resolve from the installed tree automatically, alongside project-local `./stitches` and `~/.frankie/stitches` (which still take precedence).

To upgrade later: `brew upgrade frankie`. To remove: `brew uninstall frankie`.

### Git clone + installer

```bash
git clone https://github.com/atejada/Frankie.git
cd Frankie
python3 install.py

# Add frankie/bin to your PATH (add to ~/.bashrc or ~/.zshrc)
export PATH="/path/to/frankie/bin:$PATH"
```

To uninstall:

```bash
python3 install.py --uninstall
```

### Without installing

```bash
python3 frankiec.py run examples/hello.fk
```

---

## Where the Examples Live

Every install method ships the same bundle of examples and showcase projects (Hello World, Fizzbuzz, the full showcase, and playable projects like Snake and Zombie Chat). `frankiec` always knows where its own install lives, so you don't need to hunt for the source tree:

```bash
frankiec examples              # list everything bundled with this install
frankiec examples snake        # copy a project into the current directory
cd snake && frankiec run main.fk
```

This works identically whether you installed via Homebrew, `git clone` + `install.py`, or an unpacked zip.

* **Homebrew:** examples live at `$(brew --prefix frankie)/libexec/examples/` — typically `/opt/homebrew/opt/frankie/libexec/examples/` on Apple Silicon, or `/usr/local/opt/frankie/libexec/examples/` on Intel Macs. `frankiec examples` (no arguments) also prints this path directly, so you never have to work it out by hand.
* **Git clone / zip:** examples are just `examples/` in the repo root.

Single-file examples (like `fizzbuzz.fk`) copy as one `.fk` file; full projects (like `snake` or `frankie_ledger`) copy as a whole directory, including their pinned `stitches/` and `stitch.lock`.

---

## Quick Start

```bash
frankiec repl                              # interactive REPL
frankiec new myapp                         # scaffold a new project
frankiec new --game mygame                 # scaffold a playable starter game
frankiec run examples/hello.fk             # run a program
```
