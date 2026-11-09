# The Frankie Playground

Run Frankie in the browser, with nothing to install: **[Open the Playground](https://atejada.github.io/frankielanguage/#playground)**.

It is **not a re-implementation**. The page runs the real compiler, standard library, formatter, test runner and bundled stitches — the same files that ship with `frankiec` — inside [Pyodide](https://pyodide.org) (CPython compiled to WebAssembly). Error messages, line mapping and test output match a local install. Nothing is sent to a server: your code runs in a Web Worker in your own tab.

## Buttons

| Button | What it does |
|--------|--------------|
| **Run** (`Ctrl/Cmd + Enter`) | Compile and run `main.fk`; output streams into the Output pane |
| **Test** | Run the `test "…" do … end` blocks with the built-in harness |
| **Format** | `frankiec fmt` — comments and parentheses are preserved (v1.22.3) |
| **Stop** | Kill the worker (programs running longer than 20 s are stopped automatically) |
| **Share link** | Copy a URL whose `#pg=…` fragment contains your compressed code |
| **Input (stdin)** | Lines fed to `input()`, `input_int()` and `input_float()` |

An example picker offers short programs covering hello world, FizzBuzz, hashes, error handling, records, FORTRAN-style features, labeled loops, pipes, tests, stitches and input. Shared links never auto-run: opening one loads the code and waits for you to press Run.

## What does not work in a browser

Pyodide has no sockets, threads or real terminal, so these only work in a real install: `web_app()` / `app.run`, `tcp_*` / `udp_*` / `ws_connect`, `http_*` requests, `spawn` and `parallel_map`, the terminal engine `frankiegame` (use `frankiecanvas` locally), `shell()`, and files that must outlive the tab (the in-browser filesystem resets on reload). A server program runs until the 20-second timeout and is then stopped with an explanatory message.

## First load

The first use downloads Pyodide (about 7 MB, cached afterwards) and the Frankie bundle (about 120 KB gzipped). Later runs start instantly.
