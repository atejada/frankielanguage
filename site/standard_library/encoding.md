# Encoding & Signing

Built-in functions for Base64 encoding and HMAC signing. Zero dependencies — backed by Python's `base64` and `hmac` / `hashlib` standard library modules.

Introduced in **v1.15** (previously available as internal `_fk_hmac_*` functions).

---

## Base64

### `base64_encode(str)`

Encode a string to standard Base64.

```ruby
puts base64_encode("hello:world")    # aGVsbG86d29ybGQ=
puts base64_encode("Alice")          # QWxpY2U=

# Build an HTTP Basic Auth header
creds = base64_encode("#{user}:#{pass}")
puts "Authorization: Basic #{creds}"
```

### `base64_decode(str)`

Decode a Base64 string back to its original value.

```ruby
puts base64_decode("aGVsbG86d29ybGQ=")    # hello:world
puts base64_decode("QWxpY2U=")            # Alice

# Decode an incoming Basic Auth header
encoded = req.headers["Authorization"][6..-1].strip
decoded = base64_decode(encoded)          # "username:password"
```

---

## HMAC Signing

### `hmac_sign(subject, secret)`

Create a tamper-proof signed token by combining a subject string with a secret key using HMAC-SHA256. Returns a string in the form `subject:signature`.

```ruby
SECRET = env("APP_SECRET", "dev-only")

token = hmac_sign("user_id=42", SECRET)
puts token    # user_id=42:be38cbad...
```

### `hmac_verify(token, secret)`

Verify a token produced by `hmac_sign`. Returns the original subject string if the signature is valid, or `nil` if the token is missing, malformed, or tampered with.

```ruby
SECRET = env("APP_SECRET", "dev-only")

subject = hmac_verify("user_id=42:be38cbad...", SECRET)
puts subject    # user_id=42

puts hmac_verify("tampered:abc", SECRET)    # nil
```

---

## Signed Token Flow

```ruby
SECRET = env("APP_SECRET", "dev-only")

# Create
token = hmac_sign("user_id=42", SECRET)

# Verify
subject = hmac_verify(token, SECRET)
if subject == nil
  puts "Invalid token"
else
  puts "Verified: #{subject}"    # Verified: user_id=42
end

# Tampering is detected
bad = token[0..-3] + "XX"
puts hmac_verify(bad, SECRET)    # nil
```

---

## Notes

- `hmac_sign` produces signed strings, not encrypted ones. The subject is visible in the token — store a user ID, not a password or sensitive value.
- Tokens produced by `hmac_sign` do not expire. To add expiry, include a timestamp in the subject and check it after verifying: `hmac_sign("user=42,exp=#{now().timestamp + 3600}", SECRET)`.
- The `frankieauth` stitch builds a full Basic Auth + Bearer token middleware layer on top of these functions. Use it if you need drop-in route protection.

---

## Quick Reference

| Function | Description |
|---|---|
| `base64_encode(str)` | Encode string to Base64 |
| `base64_decode(str)` | Decode Base64 string |
| `hmac_sign(subject, secret)` | Create a signed token |
| `hmac_verify(token, secret)` | Verify token — returns subject or `nil` |
