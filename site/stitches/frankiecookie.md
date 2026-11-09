# Frankiecookie

## Functions

### `set_signed_cookie(resp, name, value, secret, opts)`

Write a signed cookie

### `get_signed_cookie(req, name, secret)`

Read + verify a signed cookie (nil if invalid/missing)

### `delete_cookie(resp, name)`

Expire a cookie immediately

### `cookie_set?(req, name)`

Check if a cookie exists

## `opts`

The opts hash at the end of set_signed_cookie accepts these keys:

| Key | Description |
|---|---|
| `max_age` | Lifetime in seconds — 86400 = 1 day, omit for session cookie |
| `same_site` | "Strict" / "Lax" / "None" — CSRF protection, default "Lax" |
| `http_only` | Hide from JavaScript — default true |
| `secure` | HTTPS only — set true in production |
| `path` | Cookie scope — default "/" |

## `Full example`

```ruby
stitch "frankiecookie"

SECRET = "my-secret-key"

app = web_app()

# Write a signed cookie on login
app.post("/login") do |req|
  username = req.json["username"]
  resp = html_response("<p>Logged in as #{username}</p>")
  set_signed_cookie(resp, "user", username, SECRET, {max_age: 86400})
  resp
end

# Read and verify it on subsequent requests
app.get("/me") do |req|
  username = get_signed_cookie(req, "user", SECRET)
  if username == nil
    html_response("<p>Not logged in</p>", 401)
  else
    html_response("<p>Hello, #{username}!</p>")
  end
end

# Expire it on logout
app.get("/logout") do |req|
  resp = html_response("<p>Logged out</p>")
  delete_cookie(resp, "user")
  resp
end

app.run(3000)
```

```bash
## `Test it`

# Test it
curl -X POST http://localhost:3000/login \
     -H "Content-Type: application/json" \
     -d '{"username": "Alice"}' -c cookies.txt

curl http://localhost:3000/me -b cookies.txt
# Hello, Alice!

curl http://localhost:3000/logout -b cookies.txt -c cookies.txt

curl http://localhost:3000/me -b cookies.txt
# Not logged in
```

**`Remember — the value is signed not encrypted, so store a user ID and look up the rest from the database, never a password or token.`**
