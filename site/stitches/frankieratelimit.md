# Frankieratelimit

In-memory per-IP rate limiting middleware for Frankie web apps. No dependencies. Uses a sliding window algorithm — resets on server restart by design.

Introduced in **v1.15**.

---

## Functions

### `rate_limit_check(req, next_fn, max: 60, window: 60)`

Checks the request against the rate limit for the client IP (read from `X-Forwarded-For`). If the limit is exceeded it calls `halt(429, "Too Many Requests")`; otherwise it calls `next_fn.(req)` to continue the middleware chain.

| Parameter | Default | Description |
|---|---|---|
| `req` | — | The incoming request object |
| `next_fn` | — | The next middleware function, or `nil` to skip chaining |
| `max` | `60` | Maximum number of requests allowed in the window |
| `window` | `60` | Sliding window size in seconds |

### `rate_limit_reset(ip, window: 60)`

Clears the rate limit record for a given IP and window. Useful in tests or admin tooling.

---

## Default Limit (60 req/min globally)

```ruby
stitch "frankieratelimit"

app = web_app()

app.use do |req, next_fn|
  rate_limit_check(req, next_fn)
end

app.get("/") do |req|
  json_response({status: "ok"})
end

app.run(3000)
```

---

## Custom Limits per Route

```ruby
stitch "frankieratelimit"

app = web_app()

# Strict limit on the login endpoint: 5 requests per minute
app.post("/login") do |req|
  rate_limit_check(req, nil, max: 5, window: 60)
  # ... login logic here
  json_response({token: "..."})
end

# Relaxed limit on the API: 200 requests per minute
app.use do |req, next_fn|
  rate_limit_check(req, next_fn, max: 200, window: 60)
end

app.get("/api/data") do |req|
  json_response({data: []})
end

app.run(3000)
```

---

## Combining with `frankieauth`

```ruby
stitch "frankieratelimit"
stitch "frankieauth"

SECRET = env("TOKEN_SECRET", "dev-secret")

app = web_app()

# Rate limit all requests
app.use do |req, next_fn|
  rate_limit_check(req, next_fn, max: 100, window: 60)
end

# Auth on protected routes
app.get("/api/me") do |req|
  user = bearer_required(req, SECRET)
  json_response({user: user})
end

app.run(3000)
```

---

## Notes

- IP is read from the `X-Forwarded-For` header. If behind a load balancer or proxy, make sure this header is set correctly.
- State is in-memory — all counts reset when the server restarts. For persistent or distributed rate limiting, store counts in a database or cache instead.
- Passing `nil` as `next_fn` skips middleware chaining — useful when calling `rate_limit_check` directly inside a route handler rather than from `app.use`.
- The sliding window counts only requests made within the last `window` seconds, so limits recover gradually rather than resetting all at once.
