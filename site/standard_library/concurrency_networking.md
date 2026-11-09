# Concurrency & Networking

Introduced in **v1.17**, with TLS and UDP added in **v1.19**. Thread-pool mapping, raw TCP/UDP sockets, and WebSockets — all built on the Python standard library, zero external dependencies.

---

## `parallel_map`

Thread-pool map for I/O-bound work: HTTP calls, shell commands, file reads. Runs the block across a pool of worker threads and returns results **in input order**.

```ruby
urls = [
  "https://api.example.com/a",
  "https://api.example.com/b",
  "https://api.example.com/c"
]

pages = parallel_map(urls, workers: 8) do |u|
  http_get(u)
end
```

- `workers:` defaults to `4`.
- Results always come back in the same order as the input vector.
- The first exception raised by any worker is re-raised.
- An empty vector returns `[]` immediately.

Because Frankie threads share one Python interpreter, `parallel_map` shines for **I/O-bound** work (waiting on the network or disk), not CPU-bound number crunching.

```ruby
# Run several shell commands at once
results = parallel_map(["ls", "date", "whoami"], workers: 3) do |cmd|
  shell(cmd)["stdout"]
end
```

---

## TCP Sockets

### `tcp_connect(host, port, timeout: nil, tls: false)` — client

Opens a TCP connection and returns a socket object:

```ruby
sock = tcp_connect("example.com", 7777, timeout: 5)
sock.send_line("ping")
puts sock.recv_line()
puts sock.peer()          # "93.184.216.34:7777"
sock.close()
```

Pass `tls: true` for a certificate-verified TLS connection (v1.19), via Python's stdlib `ssl`:

```ruby
sock = tcp_connect("example.com", 443, tls: true)
sock.send_line("GET / HTTP/1.0\r\n\r\n")
puts sock.recv_line()
sock.close()
```

Server-side TLS stays on the wish list — `tcp_listen`/`tcp_serve` are plaintext only.

### Socket methods

| Method | Description |
|---|---|
| `sock.send(s)` | Send a string — returns bytes sent |
| `sock.send_line(s)` | Send a string followed by a newline |
| `sock.recv(n)` | Receive up to `n` bytes as a string (default 4096) — `nil` when the peer closes |
| `sock.recv_line()` | Receive until newline (exclusive) — `nil` when the peer closes |
| `sock.peer()` | Remote end as `"host:port"` |
| `sock.close()` | Close the connection |

Use `send`/`recv(n)` for raw data, `send_line`/`recv_line` for line-based protocols.

### `tcp_listen(port, host: "0.0.0.0")` — manual server

```ruby
server = tcp_listen(7777)
client = server.accept()       # blocks until a client connects
msg = client.recv_line()
client.send_line("got: #{msg}")
client.close()
server.close()
```

### `tcp_serve(port) do |client| ... end` — threaded server loop

One thread per client, sockets closed automatically when the block returns. Blocks forever (Ctrl+C to stop):

```ruby
tcp_serve(7777) do |client|
  msg = client.recv_line()
  client.send_line("echo: #{msg}")
end
# 🧟 Frankie TCP server listening on 0.0.0.0:7777
```

### Example: a tiny client/server pair

```ruby
# server.fk
tcp_serve(7777) do |client|
  name = client.recv_line()
  client.send_line("Hello, #{name}!")
end
```

```ruby
# client.fk
sock = tcp_connect("localhost", 7777)
sock.send_line("Frankie")
puts sock.recv_line()    # Hello, Frankie!
sock.close()
```

---

## UDP Sockets (v1.19)

Connectionless datagrams — no handshake, no ordering guarantees, fire-and-forget or request/response as you like.

### `udp_listen(port, host: "0.0.0.0")`

```ruby
sock = udp_listen(9999)
msg = sock.recv()                  # blocks for the next datagram
puts msg["data"]                   # the payload as a string
puts msg["host"]                   # sender's address
sock.send_to(msg["host"], msg["port"], "pong")
sock.close()
```

### `udp_send(host, port, message)`

Fire-and-forget — opens a socket, sends one datagram, closes:

```ruby
udp_send("127.0.0.1", 9999, "ping")
```

### UDP socket methods

| Method | Description |
|---|---|
| `sock.recv(n: 4096)` | Block for the next datagram → `{data:, host:, port:}` |
| `sock.send_to(host, port, msg)` | Send a datagram to a specific address |
| `sock.close()` | Close the socket |

---

## WebSockets (v1.18) + TLS (v1.19)

Full WebSocket coverage lives on the [Web Server](webserver.md#websockets-v118) page — `app.websocket` for the server side, `ws_connect` for clients. Since v1.19, `ws_connect` also accepts `wss://` URLs for TLS-secured connections:

```ruby
sock = ws_connect("wss://echo.example.com/socket")
sock.send("hello")
puts sock.recv()
sock.close()
```

---

## Error Handling

Socket operations raise on failure — connection refused, timeouts, etc. Rescue them like any other error:

```ruby
begin
  sock = tcp_connect("localhost", 9999, timeout: 2)
rescue TimeoutError => e
  puts "Connection timed out: #{e}"
rescue e
  puts "Could not connect: #{e}"
end
```

---

## Quick Reference

| Function | Description |
|---|---|
| `parallel_map(vec, workers: 4) do \|x\| ... end` | Thread-pool map, results in input order |
| `tcp_connect(host, port, timeout: nil, tls: false)` | Open a TCP connection — `tls: true` for TLS (v1.19) |
| `tcp_listen(port, host: "0.0.0.0")` | Listen on a port; `.accept()` returns a client socket |
| `tcp_serve(port) do \|client\| ... end` | Threaded accept loop, auto-closes clients |
| `sock.send(s)` / `sock.send_line(s)` | Send a string (newline appended by `send_line`) |
| `sock.recv(n)` / `sock.recv_line()` | Receive a string — `nil` when peer closes |
| `sock.peer()` / `sock.close()` | Remote `"host:port"` / close the connection |
| `udp_listen(port, host: "0.0.0.0")` | Bind a UDP socket (v1.19) |
| `udp_send(host, port, msg)` | Fire-and-forget datagram (v1.19) |
| `sock.recv(n)` / `sock.send_to(host, port, msg)` | UDP receive/send (v1.19) |
| `ws_connect(url, timeout: nil)` | WebSocket client — `ws://` or `wss://` (v1.18 / TLS in v1.19) |
