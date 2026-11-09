# Frankiepager

Pagination

## `Pager hash keys`

| Key | Description |
|---|---|
| `page` | Current page (clamped to valid range) |
| `per_page` | Items per page |
| `total` | Total item count |
| `total_pages` | Total number of pages |
| `from` | First item number on this page |
| `to` | Last item number on this page |
| `has_prev` | True if there is a previous page |
| `has_next` | True if there is a next page |
| `prev_page` | Previous page number or nil |
| `next_page` | Next page number or nil |
| `first_page` | True if on the first page |
| `last_page` | True if on the last page |

## Functions

### `paginate(opts)`

Compute pagination. Opts: `total`, `page`, `per_page`.

```ruby
stitch "frankiepager"

pg = paginate({total: 247, page: 3, per_page: 20})
puts "Page #{pg["page"]} of #{pg["total_pages"]}"
puts "Showing items #{pg["from"]}–#{pg["to"]} of #{pg["total"]}"
puts "Has prev: #{pg["has_prev"]}  Has next: #{pg["has_next"]}"
puts "Prev page: #{pg["prev_page"]}  Next page: #{pg["next_page"]}"

→ Page 3 of 13
→ Showing items 41–60 of 247
→ Has prev: true  Has next: true
→ Prev page: 2  Next page: 4
```

### `page_slice(items, page, per_page)`

Return the slice of a vector for the given page.

```ruby
stitch "frankiepager"

items = seq(1, 50)
puts "Page 2 of seq(1,50) at 10/page:"
puts page_slice(items, 2, 10)

→ Page 2 of seq(1,50) at 10/page:
→ [11, 12, 13, 14, 15, 16, 17, 18, 19, 20]
```

### `page_links(pager, url_template)`

Generate a vector of `{label, url, active}` hashes for navigation links. 

Use `{page}` in the template: `"/posts?page={page}"`.

```ruby
stitch "frankiepager"

pager = paginate({total: 60, page: 3, per_page: 10})

links = page_links(pager, "/posts?page={page}")

links.each do |link|
  if link["active"]
    puts "[#{link["label"]}]  #{link["url"]}"   # current page — marked active
  else
    puts " #{link["label"]}   #{link["url"]}"
  end
end

→  ← Prev   /posts?page=2
→  1   /posts?page=1
→  2   /posts?page=2
→ [3]  /posts?page=3
→  4   /posts?page=4
→  5   /posts?page=5
→  6   /posts?page=6
→  Next →   /posts?page=4
```
