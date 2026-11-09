# Vector Methods

## Access

### `.first`

First element. Returns nil if empty.

```ruby
puts [1,2,3].first

→ 1
```

### `.last`

Last element. Returns nil if empty.

```ruby
puts [1,2,3].last

→ 3
```


### `[i]`

Element at index i. 

Negative indices count from the end. 

v[-1] is the last element.

```ruby
puts [1,2,3][1]

→ 2
```


### `[a..b]`

Inclusive slice. 

```ruby
[1,2,3,4][1..2] 

→ [2,3]
```


### `[a...b]`

Exclusive slice — up to but not including b.

```ruby
[1,2,3,4][1...2] 

→ [2]
```


### `.take(n)`

First n elements. 

```ruby
[1,2,3,4].take(2) 

→ [1,2]
```

### `.drop(n)`

All elements after the first n. 

```ruby
[1,2,3,4].drop(2) 

→ [3,4]
```

### `.dig(i1, i2, ...)`

Safe nested access through vectors and hashes. Returns nil instead of crashing.

```ruby
puts [1,2,3].dig(4)

→ nil
```

---

## Inspection

### `.length`

Number of elements.

```ruby
puts [1,2,3].length

→ 3
```

### `.size` 

*Alias for .length*

```ruby
puts [1,2,3].size

→ 3
```

### `.count`

*Alias for .length*

With a block: count elements satisfying the condition.

```ruby
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
puts numbers.count do |x|
  x % 2 == 0
end

→ 5
```

### `.empty?` 

True if the vector has no elements.

```ruby
puts [].empty?

→ true
```

### `.include?(n)`

True if n is an element.

```ruby
puts ["a","b","c"].include?("b")

→ true
```

### `.nil?`

Always false for a vector; useful in nil-safe chains.

```ruby
puts [].nil?

→ false
```

### `.tally`

Count occurrences of each element, return a Hash. 

```ruby
puts ["a","b","a"].tally 

→ {a: 2, b: 1}
```

---

## Iteration

### `.each do |x|`

Iterate over each element. Returns nothing — used for side effects.

```ruby
[1,2,3].each do |x|
    puts x * 2
end

→ 2
  4
  6 
```

### `.each_with_index do |x, i|`

Iterate with element and its index available.

```ruby
[1,2,3].each_with_index do |x, i|
	puts "i: #{i}, x: #{x}"
end

→ i: 0, x: 1
  i: 1, x: 2
  i: 2, x: 3
```

### `.each_with_object(init) do |x, obj|`

Iterate with a shared accumulator. 

Works with any init type — vector, hash, integer.

```ruby
double = [1,2,3].each_with_object([]) do |x, arr|
	arr.push(x * 2)
end

puts double


→ [2, 4, 6]
```

### `.each_slice(n) do |slice|`

Iterate in non-overlapping chunks of n elements

```ruby
[1,2,3,4,5].each_slice(2) do |slice|
    puts slice
end

→ [1, 2]
  [3, 4]
  [5]
```

### `.each_cons(n) do |window|`

Iterate with a sliding window of n elements.

```ruby
[1,2,3,4,5].each_cons(2) do |window|
    puts window
end
```

---

## Transformation

### `.map do |x|`

Transform each element, return a new vector.

```ruby
puts [1,2,3].map do |x|
    x * 3
end

→ [3, 6, 9]
```

### `.map_with_index do |x, i|`

Map with both the element and its index available.


```ruby
puts [1,2,3,4,5].map_with_index do |x, i|
  x * i
end

→ [0, 2, 6, 12, 20]
```

### `.flat_map do |x|`

Map then flatten one level. 

Block returns a vector per element.

```ruby
sentences = ["hello world", "frankie is great"]

puts sentences.flat_map do |s|
  s.split(" ")
end

→ [hello, world, frankie, is, great]
```

### `.select do |x|`

Keep elements where block returns true. 

*Alias: .filter*

```ruby
puts [1,2,3,4,5].select do |x|
   x % 2 == 0
end

→ [2, 4]
```

### `.reject do |x|`

Keep elements where block returns false — opposite of .select.

```ruby
puts [1,2,3,4,5].reject do |x|
   x % 2 == 0
end

→ [1, 3, 5]
```

### `.inject(init) do |acc, x|`

Fold elements into a single value.

### `.reduce(init) do |acc, x|`

Fold elements into a single value. 

*Alias: .inject*

```ruby
puts [1,2,3,4,5].reduce(0) do |acc, x|
  acc + x
end

→ 15
```

### `.sort`

Sorted copy using natural ordering.

```ruby
puts [1,3,8,4,5].sort

→ [1, 3, 4, 5, 8]
```

### `.sort_by do |x|`

Sort by a computed key.

```ruby
puts ["frankie","hello","world"].sort_by do |w| 
    w.length 
end

→ [hello, world, frankie]
```

### `.reverse`

Reversed copy.

```ruby
puts [1,2,3,4,5].reverse

→ [5, 4, 3, 2, 1]
```
 
### `.uniq`

Remove duplicates, preserving first occurrence order. 

```ruby
puts [1,2,3,1,1,4,5,3].uniq

→ [1, 2, 3, 4, 5]
```

### `.union(other)` *(v1.18)*

All elements from both vectors — order-preserving, deduplicating.

```ruby
puts [1, 2, 3].union([3, 4, 5])

→ [1, 2, 3, 4, 5]
```

### `.intersect(other)` *(v1.18)*

Elements present in both vectors.

```ruby
puts [1, 2, 3].intersect([2, 3, 9])

→ [2, 3]
```

### `.difference(other)` *(v1.18)*

Elements of the receiver not present in the argument.

```ruby
puts [1, 2, 3].difference([2])

→ [1, 3]
```

### `.flatten`

Deep flatten — recursively collapses all nested vectors.

```ruby
puts [[1,2,3],["a","b","c"]].flatten

→ [1, 2, 3, a, b, c]
```

### `flatten(n)`

Flatten exactly n levels deep.

```ruby
puts [[[1,2,3],["a","b","c"]]].flatten(2)

→ [1, 2, 3, a, b, c]
```

### `.compact`

Remove all nil values.

```ruby
puts [1,2,nil,4,nil,6].compact

→ [1, 2, 4, 6]
```

### `.zip(other)`

Pair elements with another vector. 

```ruby
[1,2].zip([3,4]) 

→ [[1,3],[2,4]]
```

### `.zip_with(other) do |a, b|`

Zip and transform each pair in one step.

```ruby
puts [1,2].zip_with([3,4]) do |a, b| 
    a * b
end

→ [3, 8]
```

### `.product(other)`

Cartesian product. 

```
[1,2].product([3,4]) 

→ [[1,3],[1,4],[2,3],[2,4]].
```

### `.chunk(n)`

Split into sub-vectors of size n. Last chunk may be smaller.

```ruby
puts [1,2,3,4,5,6].chunk(3)

→ [[1, 2, 3], [4, 5, 6]]
```

### `.group_by do |x|`

Bucket elements into a Hash of arrays keyed by block result.

```ruby
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

result = numbers.group_by do |x|
  (if x % 2 == 0 then "even" else "odd" end)
end

puts result["even"]  →  [2, 4, 6, 8, 10]
puts result["odd"]   →  [1, 3, 5, 7, 9]
```

### `join(sep)`

Concatenate all elements as strings with a separator between them.

```ruby
puts [1,2,3,4,5].join("-")

→ 1-2-3-4-5 
```

### `.encode`

Convert a vector of byte integers back to a string.

```ruby
puts [70, 114, 97, 110, 107, 105, 101].encode

→ Frankie
```

### `* n`

Repeat the vector n times. 

```ruby
[1,2] * 3 → [1,2,1,2,1,2]
```

---

## Search & test

### `.find do |x|`

First element where block is true. 

Returns nil if none. 

*Alias: .detect*

```ruby
puts [1,2,3,4,5].find do |x|
  x > 3
end

→  4
```

### `.any? do |x|` 

True if at least one element satisfies the block.

```ruby
puts [1,2,3,4,5].any? do |x|
  x > 3
end

→  true
```

### `.all? do |x|`

True if every element satisfies the block.

```ruby
puts [1,2,3,4,5].all? do |x|
  x > 3
end

→  false
```

### `.none? do |x|`

True if no element satisfies the block.

```ruby
puts [1,2,3,4,5].none? do |x|
  x > 6
end

→  true
```

### `.count do |x|`

Count of elements satisfying the block.

```ruby
puts [1,2,3,4,5].count do |x|
  x % 2 == 0
end

→  2
```

### `.min_by do |x|`

Element with the smallest key value.

```ruby
puts [1,2,3,4,5].min_by do |x| 
    x
end

→  1
```

### `.max_by do |x|`

Element with the largest key value.

```ruby
puts [1,2,3,4,5].max_by do |x| 
    x
end

→  5
```

### `.sum_by do |x|`

Sum of block results across all elements.

```ruby
puts [1,2,3,4,5].sum_by do |x| 
    x + 1
end

→  20
```

---

## Mutation

### `.push(x)`

Append an element, return the vector.

```ruby
puts [1,2,3].push("a")

→ [1, 2, 3, a]
```

### `.pop()`

Remove and return the last element.

```ruby
puts [1,2,3,"a"].pop()

→ a
```

### `.delete(x)`

Remove first occurrence of x.

```ruby
puts [1,2,3,4].delete(3)

→ [1, 2, 4]
```

---

## Aggregation

### `.sum`

Sum of all elements.

```ruby
puts [1,2,3,4,5].sum

→ 15
```

### `.min`

Smallest element.

```
puts [1,2,3,4,5].min

→ 1
```

### `.max`

Largest element.

```ruby
puts [1,2,3,4,5].min

→ 5
```

### `unzip(vec)`

Standalone function. 

Split a vector of pairs into two vectors. 

```ruby
unzip([[1,"a"],[2,"b"]]) 

→ [[1,2],["a","b"]]
```

## Statistics

### `.mean` 

Arithmetic mean.

```ruby
puts [1,2,3,4,5].mean

→ 3.0
```

### `median(vec)`

Median value — standalone only.

```ruby
puts median([1,2,3,4,5])

→ 3
```

### `stdev(vec)`

Sample standard deviation — standalone only.

```ruby
puts stdev([1,2,3,4,5])

→ 1.5811388300841898
```

### `variance(vec)`

Sample variance — standalone only.

```ruby
puts variance([1,2,3,4,5])

→ 2.5000000000000004
```
