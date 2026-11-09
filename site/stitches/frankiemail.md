# Frankiemail

Send email via SMTP from any Frankie script. Zero external dependencies — uses Python's built-in `smtplib` under the hood.

Introduced in **v1.16**.

---

## Functions

### `send_mail(opts)`

Send an email. All parameters are keyword arguments with defaults, so you only pass what you need.

| Option | Default | Description |
|---|---|---|
| `to` | `""` | Recipient address |
| `subject` | `""` | Email subject line |
| `body` | `""` | Plain-text or HTML body |
| `sender` | `"frankie@localhost"` | Sender address (the From: header) |
| `smtp` | `"localhost"` | SMTP server hostname |
| `port` | `587` | SMTP port — `587` = STARTTLS, `465` = SSL, `25` = plain |
| `user` | `""` | SMTP username (leave blank for unauthenticated) |
| `password` | `""` | SMTP password |
| `html` | `false` | Set `true` to send an HTML body |
| `cc` | `""` | CC address |
| `bcc` | `""` | BCC address |

Returns `{ok: true}` on success, or `{ok: false, error: "..."}` on failure.

---

## Basic Example

```ruby
stitch "frankiemail"

result = send_mail(
  to:      "alice@example.com",
  subject: "Hello from Frankie!",
  body:    "This was sent by a Frankie script.",
  sender:  env("MAIL_FROM", "bot@example.com"),
  smtp:    env("SMTP_HOST", "smtp.gmail.com"),
  port:    587,
  user:    env("SMTP_USER"),
  password: env("SMTP_PASS")
)

if result["ok"]
  puts "Email sent!"
else
  puts "Failed: #{result["error"]}"
end
```

---

## HTML Email

```ruby
stitch "frankiemail"

html_body = <<~HTML
  <h1>Frankie Report</h1>
  <p>Your nightly job completed at <strong>#{now().format("%H:%M")}</strong>.</p>
  <p>Records processed: <strong>1,024</strong></p>
HTML

send_mail(
  to:      "reports@example.com",
  subject: "Nightly Report",
  body:    html_body,
  html:    true,
  smtp:    env("SMTP_HOST"),
  user:    env("SMTP_USER"),
  password: env("SMTP_PASS")
)
```

---

## With CC / BCC

```ruby
stitch "frankiemail"

send_mail(
  to:      "alice@example.com",
  cc:      "manager@example.com",
  bcc:     "audit-log@example.com",
  subject: "Deployment complete",
  body:    "v1.16 deployed to production.",
  smtp:    env("SMTP_HOST"),
  user:    env("SMTP_USER"),
  password: env("SMTP_PASS")
)
```

---

## Error Handling

```ruby
stitch "frankiemail"

result = send_mail(
  to:      "ops@example.com",
  subject: "Alert: disk usage > 90%",
  body:    "Please investigate immediately.",
  smtp:    env("SMTP_HOST", "localhost"),
  user:    env("SMTP_USER", ""),
  password: env("SMTP_PASS", "")
)

if not result["ok"]
  puts "Mail error: #{result["error"]}"
  # Fallback: write to a local log instead
  file_append("mail_errors.log", "#{now()}: #{result["error"]}\n")
end
```

---

## Sending from a Script

A common pattern is to run a Frankie script as a cron job and email the output:

```ruby
stitch "frankiemail"

db = db_open("app.db")
rows = db.find_all("events")
db.close

lines = rows.map do |r|
  "#{r["created_at"]}  #{r["type"]}  #{r["user"]}"
end

send_mail(
  to:      env("REPORT_EMAIL"),
  subject: "Daily event log — #{today().format("%Y-%m-%d")}",
  body:    lines.join("\n"),
  smtp:    env("SMTP_HOST"),
  user:    env("SMTP_USER"),
  password: env("SMTP_PASS")
)
```

---

## Notes

- `port: 587` uses STARTTLS, which is compatible with Gmail, SendGrid, Mailgun, Postmark, and most hosted SMTP providers.
- `port: 465` uses SSL/TLS directly. `port: 25` sends with no encryption and no auth.
- For Gmail, use an **App Password** (not your account password) and ensure "Less secure app access" or OAuth2 is configured.
- Set credentials via `env()` rather than hardcoding them. A `.env` file loaded with `dotenv()` works well.
- `frankiemail` is a thin Frankie wrapper around the built-in `smtp_send()` stdlib function, which uses Python's `smtplib`.
