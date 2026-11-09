# Frankietemplate

Mustache-Compatible Templates

`frankietemplate` is a Mustache-compatible template engine for Frankie. Drop HTML (or any text) into a `.html` file, sprinkle in `{{ tags }}`, and call `render_file` from a route handler. No dependencies — the regex engine that powers it is part of Frankie's stdlib.

```ruby
stitch "frankietemplate"

app = web_app()

app.get("/hello/:name") do |req|
  html = render_file("./views/hello.html", {
    name: req.params["name"],
    year: 2026
  })
  html_response(html)
end

app.run(3000)
```

```html
<!-- views/hello.html -->
<h1>Hello, {{ name }}!</h1>
<p>Welcome to Frankie — it's {{ year }}.</p>
```

---

## How It Works

`frankietemplate.fk` is a thin public API wrapper. The actual rendering engine — regex matching, escaping, section expansion — lives in `frankie_stdlib.py` and is available as soon as you `stitch "frankietemplate"`. You never interact with the internals directly; `render`, `render_file`, and `partial` are the only three functions you need.

---

## Template Syntax

### `{{ variable }}`

Interpolation (HTML-escaped)

Replaces the tag with the value from the data hash. 

The value is **HTML-escaped** — `<`, `>`, `&`, `"`, `'` are converted to their safe entity equivalents. 

This is the default and the safe choice for user-supplied content.

```ruby
render("Hello, {{ name }}!", {name: "Alice"})
# → Hello, Alice!

render("<p>{{ bio }}</p>", {bio: "<script>alert(1)</script>"})
# → <p>&lt;script&gt;alert(1)&lt;/script&gt;</p>
```

Missing keys render as an empty string — no error is raised:

```ruby
render("Hello, {{ name }}{{ suffix }}!", {name: "Alice"})
# → Hello, Alice!
```

---

### `{{{ variable }}}`

Raw Output (Unescaped)

Triple braces output the value as-is, with no HTML escaping. 

Use this when the value is already trusted HTML — a rendered sub-template, a rich-text field from your own database, a pre-built HTML snippet.

```ruby
body_html = "<strong>Important</strong> announcement"
render("<div>{{{ body }}}</div>", {body: body_html})
# → <div><strong>Important</strong> announcement</div>

# Contrast with double braces:
render("<div>{{ body }}</div>", {body: body_html})
# → <div>&lt;strong&gt;Important&lt;/strong&gt; announcement</div>
```

**Never use `{{{ }}}` with user-supplied content** — it will pass raw HTML straight through to the browser.

---

### `{{# section }} ... {{/ section }}`

Sections

A section tag renders its inner content based on the value of the key. The behaviour depends on the value's type:

**Boolean `true` — render once:**

```ruby
tmpl = "{{# logged_in }}Welcome back!{{/ logged_in }}"
render(tmpl, {logged_in: true})   # → Welcome back!
render(tmpl, {logged_in: false})  # → (empty)
```

**Vector — iterate:**

Each element becomes the context for one pass of the inner block. 

If the elements are hashes, their keys are available directly inside the block:

```ruby
tmpl = "{{# posts }}<li>{{ title }} by {{ author }}</li>{{/ posts }}"

render(tmpl, {
  posts: [
    {title: "Hello World", author: "Alice"},
    {title: "Day Two",     author: "Bob"}
  ]
})
# → <li>Hello World by Alice</li><li>Day Two by Bob</li>
```

If the elements are plain values (strings, numbers), use the special `.` key to reference the current item:

```ruby
tmpl = "{{# tags }}<span>{{ . }}</span>{{/ tags }}"
render(tmpl, {tags: ["ruby", "frankie", "web"]})
# → <span>ruby</span><span>frankie</span><span>web</span>
```

**`nil` or `false` — render nothing:**

```ruby
render("{{# user }}{{ name }}{{/ user }}", {user: nil})
# → (empty)
```

---

### `{{^ inverted }} ... {{/ inverted }}`

Inverted Sections

The opposite of `{{# }}` — renders when the value is `false`, `nil`, or an empty vector. 

Use it for "no results" states and optional content.

```ruby
tmpl = <<~TMPL
  {{# posts }}<li>{{ title }}</li>{{/ posts }}
  {{^ posts }}<p>No posts yet.</p>{{/ posts }}
TMPL

render(tmpl, {posts: []})
# → <p>No posts yet.</p>

render(tmpl, {posts: [{title: "Hello"}]})
# → <li>Hello</li>
```

---

### `{{> partial_name }}`

Partials

Includes another template file from `./views/partials/<name>.html`. 

Partials receive the same data context as the parent template.

```html
<!-- views/partials/header.html -->
<header>
  <h1>{{ site_name }}</h1>
  <nav><a href="/">Home</a> · <a href="/about">About</a></nav>
</header>
```

```html
<!-- views/index.html -->
{{> header }}
<main>
  <p>Welcome, {{ user_name }}!</p>
</main>
```

```ruby
render_file("./views/index.html", {
  site_name: "My Blog",
  user_name: "Alice"
})
```

If a partial file is not found, the tag renders as an empty string — no error.

---

### `{{! comment }}`

Comments

Stripped entirely from the output. Use them for notes inside template files.

```html
{{! TODO: add pagination here }}
<ul>
  {{# items }}<li>{{ name }}</li>{{/ items }}
</ul>
```

---

## Writing Templates in `.fk` Source

### `Heredoc (recommended for multi-line)`

Use a heredoc to write a multi-line template directly in your `.fk` file:

```ruby
tmpl = <<~HTML
  <h1>{{ title }}</h1>
  <p>{{ body }}</p>
HTML

puts render(tmpl, {title: "Hello", body: "World"})
```

**`^` lexer gotcha:** `{{^ inverted }}` uses the caret character. 

Inside a heredoc, the `^` is passed through correctly — but if you write an inverted section on a line that also contains other `{` characters without a space, the lexer may misread it. 

The safe rule: **always put a space after `{{^`**, which is valid Mustache anyway:

```ruby
# Safe — space after {{^
tmpl = <<~HTML
  {{^ items }}
    <p>Nothing here.</p>
  {{/ items }}
HTML

# Potentially ambiguous — avoid
tmpl = <<~HTML
  {{^items}}<p>Nothing.</p>{{/items}}
HTML
```

### `join` 

for short inline templates.

For short snippets that don't warrant a heredoc, build the template as a vector of lines and join them:

```ruby
tmpl = [
  "<ul>",
  "{{# items }}<li>{{ name }}</li>{{/ items }}",
  "</ul>"
].join("\n")

puts render(tmpl, {items: [{name: "Alice"}, {name: "Bob"}]})
```

### `Template files` 

(recommended for HTML pages)

For anything page-sized, keep templates in `./views/` and use `render_file`. Keeps your `.fk` files clean and lets your editor syntax-highlight the HTML properly:

```ruby
app.get("/posts/:id") do |req|
  post = db.find("posts", {id: req.params["id"]})
  html = render_file("./views/post.html", post)
  html_response(html)
end
```

---

## Functions

### `render(template, data)`

Render a template string. `template` is any string containing Mustache tags. `data` is a hash — keys must be strings (use `"key"` not `key:`).

```ruby
stitch "frankietemplate"

html = render("<h1>{{ title }}</h1>", {"title" => "Hello"})
puts html   # <h1>Hello</h1>
```

Data keys can also be symbol keys (the lookup checks both):

```ruby
html = render("<h1>{{ title }}</h1>", {title: "Hello"})
puts html   # <h1>Hello</h1>
```

---

### `render_file(path, data)`

Load a template file and render it. `path` is relative to the working directory (where `frankiec` is run from — normally the project root). Raises an error if the file does not exist.

```ruby
html = render_file("./views/dashboard.html", {
  user:   db.find("users", {id: session_user_id}),
  posts:  db.find_all("posts", {published: true}),
  title:  "Dashboard"
})
html_response(html)
```

---

### `partial(name)`

Load a partial template by name from `./views/partials/<name>.html`. Returns the raw file contents as a string. Normally you use `{{> name }}` tags inside template files rather than calling this directly — it is available for cases where you need to compose template strings programmatically.

```ruby
header = partial("header")
footer = partial("footer")
page   = header + content + footer
html_response(page)
```

Raises an error if the partial file is not found.

---

## `Project Layout`

```
myapp/
├── main.fk
├── stitches/
│   └── frankietemplate.fk
└── views/
    ├── index.html
    ├── post.html
    ├── dashboard.html
    └── partials/
        ├── header.html
        ├── footer.html
        └── nav.html
```

`frankiec new` creates the `views/` and `views/partials/` directories automatically.

---

## Full Web Example

```ruby
# main.fk
stitch "frankietemplate"

app = web_app()
app.static("./public")

app.get("/") do |req|
  posts = db.find_all("posts", {published: true})
  html  = render_file("./views/index.html", {
    title: "My Blog",
    posts: posts,
    count: posts.length
  })
  html_response(html)
end

app.get("/posts/:id") do |req|
  post = db.find("posts", {id: req.params["id"].to_int})
  if post == nil
    halt(404, "Post not found")
  end
  html = render_file("./views/post.html", post)
  html_response(html)
end

app.run(3000)
```

```html
<!-- views/index.html -->
{{> header }}
<main>
  <h2>{{ count }} posts</h2>

  {{# posts }}
    <article>
      <h3><a href="/posts/{{ id }}">{{ title }}</a></h3>
      <p>{{ summary }}</p>
    </article>
  {{/ posts }}

  {{^ posts }}
    <p>No posts yet. Check back soon!</p>
  {{/ posts }}
</main>
{{> footer }}
```

```html
<!-- views/post.html -->
{{> header }}
<main>
  <article>
    <h1>{{ title }}</h1>
    <p class="meta">By {{ author }} on {{ date }}</p>
    {{{ body_html }}}
  </article>
  <a href="/">← Back</a>
</main>
{{> footer }}
```

```html
<!-- views/partials/header.html -->
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>{{ title }}</title>
  <link rel="stylesheet" href="/style.css">
</head>
<body>
<nav><a href="/">Home</a></nav>
```

```html
<!-- views/partials/footer.html -->
<footer><p>Built with Frankie 🧟</p></footer>
</body>
</html>
```

---

## `Quick Reference`

| Tag | Behaviour |
|---|---|
| `{{ key }}` | Value from data hash, HTML-escaped |
| `{{{ key }}}` | Value from data hash, raw — no escaping |
| `{{# key }} ... {{/ key }}` | Render if truthy; iterate if vector; scope if hash |
| `{{^ key }} ... {{/ key }}` | Render if falsy, nil, or empty vector |
| `{{ . }}` | Current item in a plain-value vector loop |
| `{{> name }}` | Include `./views/partials/<name>.html` |
| `{{! comment }}` | Stripped — not rendered |
