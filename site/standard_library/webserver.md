# Frankie Web Server

Frankie ships a built-in HTTP web server in the style of Sinatra / Camping.
Zero external dependencies — it runs on Python's standard library `http.server`.

---

## `Quick Start`

```ruby
app = web_app()

app.get("/") do |req|
  html_response("<h1>Hello from Frankie! 🧟</h1>")
end

app.run(3000)
```

```bash
frankiec run myapp.fk
# 🧟 Frankie web server running on http://0.0.0.0:3000
```

---

## Routes

Register routes with `.get`, `.post`, `.put`, `.delete`, `.patch`.
Each takes a path pattern and a `do |req| ... end` block.
The block receives a `FrankieRequest` and must return a response value.

```ruby
app.get("/hello") do |req|
  response("Hello!")
end

app.post("/echo") do |req|
  response(req.body)
end

app.delete("/items/:id") do |req|
  response("Deleted #{req.params["id"]}")
end
```

### `Path Parameters`

Use `:name` segments to capture parts of the URL:

```ruby
app.get("/users/:id/posts/:slug") do |req|
  id   = req.params["id"]
  slug = req.params["slug"]
  response("User #{id}, post #{slug}")
end
```

### `Query Parameters`

```ruby
# GET /search?q=frankie&page=2
app.get("/search") do |req|
  q    = req.query["q"]
  page = req.query["page"]
  response("q=#{q} page=#{page}")
end
```

---

## `Request Object`

| Property    | Type   | Description                                          |
|-------------|--------|------------------------------------------------------|
| `method`    | String | `"GET"`, `"POST"`, `"PUT"`, `"DELETE"`, `"PATCH"`    |
| `path`      | String | URL path, e.g. `"/users/42"`                         |
| `params`    | Hash   | Path parameters, e.g. `{id: "42"}`                   |
| `query`     | Hash   | Query-string parameters, e.g. `{page: "2"}`          |
| `headers`   | Hash   | HTTP request headers                                 |
| `body`      | String | Raw request body                                     |
| `json`      | Hash   | Body parsed as JSON, or `nil` if not valid JSON      |
| `form`      | Hash   | Body parsed as `application/x-www-form-urlencoded`   |
| `cookies`   | Hash   | Incoming cookies                                     |

### Query Parameter Helpers *(introduced in v1.15)*

`req.query` values are always strings. These helpers parse and coerce them with a fallback default so route handlers stay clean.

| Method | Returns | Notes |
|---|---|---|
| `req.query_int(key, default)` | Integer | Returns `default` if key is missing or not a valid integer |
| `req.query_float(key, default)` | Float | Returns `default` if key is missing or not a valid float |
| `req.query_bool(key, default)` | Boolean | `"true"`, `"1"`, `"yes"` → `true`; everything else → `false` |

```ruby
app.get("/items") do |req|
  page      = req.query_int("page", 1)
  per_page  = req.query_int("per_page", 20)
  min_score = req.query_float("min_score", 0.0)
  show_all  = req.query_bool("show_all", false)

  items = db.find_all("items")
  json_response({page: page, per_page: per_page, items: items})
end
```


---

## Response Helpers

| Function                          | Status | Content-Type         |
|-----------------------------------|--------|----------------------|
| `response(body)`                  | 200    | text/plain           |
| `response(body, status)`          | custom | text/plain           |
| `response(body, status, headers)` | custom | text/plain           |
| `html_response(html)`             | 200    | text/html            |
| `html_response(html, status)`     | custom | text/html            |
| `json_response(hash)`             | 200    | application/json     |
| `json_response(hash, status)`     | custom | application/json     |
| `redirect(location)`              | 302    | —                    |
| `redirect(location, status)`      | custom | —                    |
| `halt(status, body)`              | custom | text/plain           |

Returning a plain string from the handler is also fine — it becomes a `200 text/plain` response automatically.
Returning a hash or vector auto-converts to JSON.

---

## `JSON API`

```ruby
# Receive JSON
app.post("/api/users") do |req|
  data = req.json           # parsed hash or nil
  if data == nil
    halt(400, "Expected JSON")
  else
    name = data["name"]
    json_response({id: 1, name: name}, 201)
  end
end

# Send JSON
app.get("/api/info") do |req|
  json_response({version: "1.4", alive: true})
end
```

---

## `Before / After Filters`

Filters run before or after every matched route.

```ruby
app.before do |req|
  puts "#{req.method} #{req.path}"
end

app.after do |req, res|
  puts "  -> #{res.status}"
end
```

`before` receives the request. `after` receives the request and the response.

---

## `Custom 404`

```ruby
app.not_found do |req|
  html_response("<h1>404</h1><p>Nothing at #{req.path}</p>", 404)
end
```

---

## `Starting the Server`

```ruby
app.run(3000)           # listen on 0.0.0.0:3000 (default)
app.run(8080)           # custom port
app.run(3000, "127.0.0.1")  # localhost only
```

The server is multi-threaded (one thread per request) and blocks until Ctrl+C.

---

## Cookies and Sessions *(v1.13.1)*

### `Reading and Writing Cookies`

Every request exposes its cookies as a hash via `req.cookies`. Every response can set a cookie via `resp.set_cookie(name, value, opts)`.

```ruby
app.get("/prefs") do |req|
  theme = req.cookies["theme"] or "light"
  resp = html_response("<p>Theme: #{theme}</p>")
  resp.set_cookie("theme", theme, {max_age: 86400})
  resp
end
```

`set_cookie` options (all optional):

| Option | Default | Description |
|---|---|---|
| `path` | `"/"` | Cookie scope path |
| `http_only` | `true` | Hide from JavaScript |
| `max_age` | `nil` | Expiry in seconds (omit for session cookie) |
| `same_site` | `"Lax"` | CSRF protection (`"Strict"`, `"Lax"`, `"None"`) |

### `Cookie-Backed Sessions`

`session(req, resp)` returns a `FrankieSession` — a hash-like object backed by a single JSON cookie (`_fk_session`). Read it, mutate it, and call `.save()` before returning the response. No server-side state, no database, no configuration.

```ruby
app = web_app()

app.get("/counter") do |req|
  resp = response("")
  s = session(req, resp)

  count = (s["count"] or 0) + 1
  s["count"] = count
  s.save()

  html_response("You have visited #{count} time(s).")
end

app.post("/login") do |req|
  user = req.json["username"]
  resp = response("")
  s = session(req, resp)
  s["user"] = user
  s["logged_in_at"] = now()
  s.save()
  redirect("/dashboard")
end

app.get("/logout") do |req|
  resp = response("")
  s = session(req, resp)
  s.clear()
  s.save()
  redirect("/")
end

app.run()
```

**Session API:**

| Method | Description |
|---|---|
| `s["key"]` | Read value (`nil` if missing) |
| `s["key"] = value` | Write value |
| `s.has_key?(key)` | Check existence |
| `s.keys` | All session keys |
| `s.delete(key)` | Remove one key |
| `s.clear()` | Remove all keys |
| `s.save()` | Write cookie to response — must be called before returning |

**Important:** the session cookie is `HttpOnly` and `SameSite=Lax`. It is **not encrypted** — store user IDs, not passwords or secrets.

---

## Concurrency *(v1.14)*

### `spawn { }`

Background Blocks

`spawn` runs a block in a background thread and returns immediately — the rest of the handler keeps going without waiting for the block to finish. Use it for work that shouldn't delay the response: sending emails, writing logs, hitting a slow external API, processing an uploaded file.

```ruby
app.post("/register") do |req|
  user = req.json

  # Respond right away — email sends in the background
  spawn do
    send_welcome_email(user["email"])
  end

  json_response({status: "registered"}, 201)
end
```

`spawn` works anywhere in a Frankie program, not just inside route handlers:

```ruby
spawn do
  report = build_monthly_report(data)
  file_write("report.json", json_dump(report))
end

puts "Report building in background..."
```

Spawned blocks capture variables by value at spawn time. Mutations inside the block do not affect the outer scope.

---

### `timeout(n) { }`

Time-Bounded Execution

`timeout(seconds)` runs a block and raises `TimeoutError` if it takes longer than `n` seconds. Wrap it in `begin/rescue` to handle the failure gracefully. Essential any time you call an external service that might hang.

```ruby
app.get("/weather") do |req|
  begin
    data = timeout(5) do
      http_get("https://api.weather.example.com/current")
    end
    json_response(data)
  rescue TimeoutError
    json_response({error: "weather service unavailable"}, 503)
  end
end
```

Use a short timeout and a sensible fallback — a hung external call should never take your whole server down with it:

```ruby
app.get("/profile/:id") do |req|
  user = db.find("users", {id: req.params["id"]})

  # Try to enrich with remote data, fall back to basics if slow
  begin
    extra = timeout(2) do
      http_get("https://profile-api.example.com/#{user["id"]}")
    end
    user["bio"] = extra["bio"]
  rescue TimeoutError
    user["bio"] = ""
  end

  json_response(user)
end
```

---

### `Async Routes`

app.get_async

Regular routes are synchronous — one slow handler blocks others waiting in queue. Async routes are non-blocking: while one handler is waiting on I/O, another request can run. Use `await` inside the block to pause without blocking.

```ruby
app = web_app()

# Both fetches run while other requests are served freely
app.get_async("/dashboard") do |req|
  posts  = await http_get("https://api.example.com/posts")
  alerts = await http_get("https://api.example.com/alerts")
  json_response({posts: posts, alerts: alerts})
end

app.run(3000)
```

All five HTTP methods have async variants:

```ruby
app.get_async("/items")      do |req| ... end
app.post_async("/items")     do |req| ... end
app.put_async("/items/:id")  do |req| ... end
app.delete_async("/items/:id") do |req| ... end
app.patch_async("/items/:id") do |req| ... end
```

Use async routes when your handler spends most of its time waiting — on a database, an HTTP call, a file read. For CPU-heavy work, `spawn` is the better fit.

---

## Middleware *(v1.14)*

`app.use` registers a middleware layer that wraps every request. Middleware receives the request and a `next_fn` — call `next_fn.(req)` to pass control to the next layer (and eventually the route handler). Return a response directly to short-circuit the chain without reaching the route.

Middleware layers run in registration order, so register them before your routes.

### `Logging`

```ruby
app.use do |req, next_fn|
  puts "→ #{req.method} #{req.path}"
  resp = next_fn.(req)
  puts "← #{resp.status}"
  resp
end
```

### `Auth guard`

```ruby
app.use do |req, next_fn|
  if req.path.start_with?("/admin")
    key = req.headers["X-Api-Key"]
    if key != env("API_KEY")
      halt(401, "Unauthorized")
    end
  end
  next_fn.(req)
end

app.get("/admin/users") do |req|
  json_response(db.find_all("users"))
end
```

### `Combining middleware`

```ruby
app = web_app()

# 1. Log every request
app.use do |req, next_fn|
  puts "#{req.method} #{req.path}"
  next_fn.(req)
end

# 2. Require a session for anything under /app/*
app.use do |req, next_fn|
  if req.path.start_with?("/app")
    resp = response("")
    s = session(req, resp)
    if s["user_id"] == nil
      redirect("/login")
    else
      next_fn.(req)
    end
  else
    next_fn.(req)
  end
end

app.get("/app/dashboard") do |req|
  html_response("<h1>Dashboard</h1>")
end

app.get("/login") do |req|
  html_response("<form method='post'><input name='user'><button>Log in</button></form>")
end

app.run(3000)
```

`before` and `after` filters still work alongside `use` — they run inside the middleware chain.

---

## `Static File Serving *(v1.14)*`

`app.static` serves all files in a directory. The path is relative to where `frankiec` is run from (normally the project root).

```ruby
app = web_app()

# Serve ./public/ at /  — http://localhost:3000/style.css → ./public/style.css
app.static("./public")

app.run(3000)
```

Serve at a URL prefix to keep static assets at a predictable path:

```ruby
# http://localhost:3000/static/app.js → ./assets/app.js
app.static("/static", "./assets")
```

Static files are matched before routes — if both a static file and a route exist at the same path, the file wins. File types served automatically: `.html`, `.css`, `.js`, `.json`, `.png`, `.jpg`, `.gif`, `.svg`, `.webp`, `.ico`, `.woff`, `.woff2`. Directory listing is disabled — unknown paths return 404.

---

## WebSockets *(v1.18)*

Hand-rolled RFC 6455 on the built-in web server — handshake, framing, ping/pong, clean close. Zero dependencies, as always.

### Server — `app.websocket`

```ruby
app = web_app()

app.websocket("/ws/:room") do |ws|
  ws.send("welcome to #{ws.params["room"]}")
  loop do
    msg = ws.recv()
    break if msg == nil      # client closed
    ws.send("echo: #{msg}")
  end
end

app.run(3000)
```

Each connection runs the handler in its own thread and closes automatically when the handler returns. `recv()` answers pings for you and returns `nil` when the peer disconnects.

### Client — `ws_connect`

```ruby
sock = ws_connect("ws://localhost:3000/ws/lobby", timeout: 5)
sock.send("hello")
puts sock.recv()
sock.close()
```

Only `ws://` URLs are supported (no TLS).

### WebSocket object

| Method / property | Description |
|---|---|
| `ws.send(msg)` | Send a text message |
| `ws.recv()` | Receive a text message — `nil` when the peer closes |
| `ws.close()` | Close the connection |
| `ws.params` | Route parameters (e.g. `:room`) — server side |
| `ws.path` | Request path — server side |
| `ws.peer` | Remote `"host:port"` |

---

## `Web API Summary`

| Function / Method                      | Description                               |
|----------------------------------------|-------------------------------------------|
| `web_app()`                            | Create a new application                  |
| `app.get(path) do \|req\| end`         | Register a GET route                      |
| `app.post(path) do \|req\| end`        | Register a POST route                     |
| `app.put(path) do \|req\| end`         | Register a PUT route                      |
| `app.delete(path) do \|req\| end`      | Register a DELETE route                   |
| `app.patch(path) do \|req\| end`       | Register a PATCH route                    |
| `app.get_async(path) do \|req\| end`   | Register a non-blocking GET route         |
| `app.post_async(path) do \|req\| end`  | Register a non-blocking POST route        |
| `app.use do \|req, next_fn\| end`      | Register a middleware layer               |
| `app.static(dir)`                      | Serve a directory at `/`                  |
| `app.static(prefix, dir)`              | Serve a directory at a URL prefix         |
| `app.websocket(path) do \|ws\| end`    | Register a WebSocket route (v1.18)        |
| `ws_connect(url, timeout: nil)`        | WebSocket client connection (v1.18)       |
| `app.before do \|req\| end`            | Register a before-filter                  |
| `app.after do \|req, res\| end`        | Register an after-filter                  |
| `app.not_found do \|req\| end`         | Register a custom 404 handler             |
| `app.run(port)`                        | Start the server (blocking)               |
| `response(body, status, headers)`      | Plain-text response                       |
| `html_response(body, status)`          | HTML response                             |
| `json_response(data, status)`          | JSON response                             |
| `redirect(location, status)`           | Redirect response                         |
| `halt(status, body)`                   | Error response                            |
| `req.cookies`                          | Parsed Cookie header as a hash            |
| `resp.set_cookie(name, val, opts)`     | Append a Set-Cookie header                |
| `session(req, resp)`                   | Cookie-backed session hash                |
| `spawn { }`                            | Run a block in a background thread        |
| `timeout(n) { }`                       | Run a block with a time limit in seconds  |
| `await expr`                           | Non-blocking wait inside async routes     |
