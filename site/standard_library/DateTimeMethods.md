# DateTime Methods

## `now()`

Current date and time as a DateTime object

```ruby
puts now()

→ 2026-04-02 06:21:00.946891 
```

## `today()`

Today's date at midnight

```ruby
puts today()

→ 2026-04-02 00:00:00
```

## `date_parse(str, fmt)`

Parse a string into a DateTime. 

Default format is "%Y-%m-%d". Pass a custom format as the second argument

```ruby
puts date_parse("02/04/2026", "%d/%m/%Y")

→ 2026-04-02 00:00:00
```

## `date_from(year, month, day, hour, minute, second)`

Build a DateTime from components. Hour, minute, and second default to 0.

```ruby
puts date_from(2026, 04, 02, 13, 25, 0)

→  2026-04-02 13:25:00
```

## `.year`

Integer year. 

```ruby
puts now().year 

→ 2026
```

## .month

Integer month.

```ruby
puts now().month 

→ 4
```

## `.day`

Integer day of month. 

```ruby
puts now().day 

→ 2
```

## `.hour`

Integer hour (0–23). 

```ruby
puts now().day 

→ 14
```

## `.minute`

Integer minute (0–23). 

```ruby
puts now().minute 

→ 30
```

## `.second`

Integer second (0–23). 

```ruby
puts now().second 

→ 40
```

## `.format(fmt)` 

Format as a string using strftime codes. 

Default format is "%Y-%m-%d %H:%M:%S".

```ruby
puts now().format("%d/%m/%Y")

→ 02/04/2026
```

## `.add_days(n)` 

Return a new DateTime n days in the future. 

Negative values go backward.

```ruby
puts now().add_days(5)

→ 2026-04-07 22:59:30.275494
```

## `.add_minutes(n)` 

Return a new DateTime n minutes in the future.

```ruby
puts now().add_minutes(5)

→ 2026-04-02 23:08:00.487584
```

## `.diff_days(other)` 

Number of days between two DateTime values. Always positive.

```ruby
puts now().diff_days(date_parse("1977-11-22", "%Y-%m-%d"))

→ 17664
```

## `.diff_seconds(other)` 

Number of seconds between two DateTime values. Always positive.

```ruby
puts now().diff_seconds(date_parse("1977-11-22", "%Y-%m-%d"))

→ 1526252921
```

## `.weekday`

Day of week as integer. 

0 = Monday, 6 = Sunday.

```ruby
puts now().weekday

→ 4 
```

## `.is_before(other)`

True if this DateTime is earlier than other.

```ruby
puts now().is_before(date_parse("1977-11-22", "%Y-%m-%d"))

→ false
```

## `.is_after(other)`

True if this DateTime is later than other.

```ruby
puts now().is_after(date_parse("1977-11-22", "%Y-%m-%d"))

→ true
```

## `.timestamp`

Unix timestamp as a Float. Seconds since 1970-01-01 00:00:00 UTC.

```ruby
puts now().timestamp

→ 1775275966.768478
```

---

## Date Arithmetic *(introduced in v1.15)*

Date objects support `+` and `-` with integers (days) and comparison operators.

### `date + n` / `date - n`

Add or subtract a number of days. Returns a new date object.

```ruby
d = date_from(2025, 6, 1)

puts (d + 30).format("%Y-%m-%d")    # 2025-07-01
puts (d - 7).format("%Y-%m-%d")     # 2025-05-25
```

### `date_a - date_b`

Subtract two dates to get the difference in days as an integer.

```ruby
d1 = date_from(2025, 6, 1)
d2 = date_from(2025, 7, 1)

puts d2 - d1    # 30
```

### Comparison operators

`<`, `>`, `<=`, `>=`, `==` all work between date objects.

```ruby
d = date_from(2025, 6, 1)

puts d < date_from(2025, 12, 31)    # true
puts d == date_from(2025, 6, 1)     # true
puts d > date_from(2024, 1, 1)      # true
```
