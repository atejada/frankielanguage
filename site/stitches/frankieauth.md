# Frankieauth

HTTP Basic Auth and Bearer token authentication for Frankie web apps. Zero dependencies — uses Frankie's built-in `hmac_sign` / `hmac_verify` and `base64_decode`.

Introduced in **v1.15**.

---

## Functions

### `basic_auth_ok?(req, expected_user, expected_pass)`

Validates an HTTP Basic Auth header against a known username/password. Returns `true` if the credentials match, `false` otherwise (including when the header is absent).

### `auth_token_create(subject, secret)`

Creates a signed Bearer token for a given subject string (e.g. a username or user ID). Returns the token as a string.

### `auth_token_verify(req, secret)`

Reads and verifies a Bearer token from the `Authorization` header. Returns the original subject string if valid, or `nil` if the header is missing, malformed, or the signature doesn't match.

### `bearer_required(req, secret)`

Like `auth_token_verify` but short-circuits with `halt(401, "Unauthorized")` if the token is absent or invalid. Returns the subject on success — useful for protected routes.

---

## Basic Auth Example

```ruby
stitch "frankieauth"

app = web_app()

app.use do |req, next_fn|
  if not basic_auth_ok?(req, "admin", env("ADMIN_PASS", "secret"))
    halt(401, "Unauthorized")
  end
  next_fn.(req)
end

app.get("/dashboard") do |req|
  html_response("<h1>Dashboard</h1>")
end

app.run(3000)
```

```bash
# Test it
curl http://localhost:3000/dashboard -u admin:secret
# Dashboard

curl http://localhost:3000/dashboard -u admin:wrong
# 401 Unauthorized
```

---

## Bearer Token Example

```ruby
stitch "frankieauth"

SECRET = env("TOKEN_SECRET", "dev-secret")

app = web_app()

# Login — exchange credentials for a token
app.post("/login") do |req|
  user = req.json["username"]
  # In production: verify user + password against your database first
  token = auth_token_create(user, SECRET)
  json_response({token: token})
end

# Protected route — token required
app.get("/me") do |req|
  user = bearer_required(req, SECRET)
  json_response({user: user})
end

# Optional verification (no auto-halt)
app.get("/profile") do |req|
  user = auth_token_verify(req, SECRET)
  if user == nil
    json_response({error: "not authenticated"}, 401)
  else
    json_response({user: user, plan: "free"})
  end
end

app.run(3000)
```

```bash
# Get a token
curl -X POST http://localhost:3000/login \
     -H "Content-Type: application/json" \
     -d '{"username": "alice"}'
# {"token": "alice:be38cbad..."}

# Use the token
curl http://localhost:3000/me \
     -H "Authorization: Bearer alice:be38cbad..."
# {"user": "alice"}

# Missing token
curl http://localhost:3000/me
# 401 Unauthorized
```

---

## Notes

- Tokens are HMAC-SHA256 signed strings — tamper-proof but **not encrypted**. Store a user ID as the subject, not a password or sensitive value.
- Tokens do not expire by default. For expiry, include a timestamp in the subject and validate it in your route handler.
- `basic_auth_ok?` accepts both `Authorization` and `authorization` headers (case-insensitive).
- `basic_auth_ok?` handles passwords that contain colons (splits only on the first one).
