## `frankieconfig`

Layered Config

Sources are merged in priority order: defaults → JSON file → env vars → overrides. Higher priority wins.

Type coercion is automatic — if the default is an Integer, the env var string is converted to Integer. Same for Float and Boolean.

## Functions

### `load_config(opts)`

Load and merge config. Opts: `file`, `env_prefix`, `defaults`, `overrides`.

```ruby
stitch "frankieconfig"

config = load_config({
  file:       "config.json",
  env_prefix: "APP",
  defaults:   {host: "localhost", port: 3000, debug: false}
})

puts config["host"]    # APP_HOST env → config.json → "localhost"
puts config["port"]    # APP_PORT env → config.json → 3000
puts config["debug"]   # APP_DEBUG env → config.json → false
```

### `config_get(config, key, fallback)`

Get a key with a fallback if missing.

```ruby
stitch "frankieconfig"

config = load_config({
  file:       "config.json",
  env_prefix: "APP",
  defaults:   {host: "localhost", port: 3000, debug: false}
})

puts config_get(config, "username", "Frankie")
```
