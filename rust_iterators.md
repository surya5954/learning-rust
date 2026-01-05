# 🦀 Training Session: Iterators

**Goal:** Understand how to use Rust's lazy, powerful, and efficient iterator system to process collections.

---

## What is an Iterator?
**Bundling State and Logic**

An iterator is an object that knows how to produce a sequence of values one by one.

```rust
fn main() {
    let v = vec![1, 2, 3];
    let mut iter = v.iter(); // Create an iterator
    
    assert_eq!(iter.next(), Some(&1));
    assert_eq!(iter.next(), Some(&2));
    assert_eq!(iter.next(), Some(&3));
    assert_eq!(iter.next(), None); // Iteration finished
}
```

### Key Properties:
*   **Lazy**: Iterators do nothing until you call `next()` or a consuming method.
*   **Zero-Cost**: The compiler optimizes iterator chains into code as fast as a manual `while` loop.

---

## The Iterator Trait
**The core of iteration**

To be an iterator, a type must implement the `Iterator` trait, which requires a `next` method.

```rust
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```

---

## Iterator Adapters
**Chaining Logic**

Adapters take one iterator and return a new one with modified behavior.

```rust
fn main() {
    let result: i32 = (1..=10)
        .filter(|x| x % 2 == 0) // Keep only evens
        .map(|x| x * x)         // Square them
        .sum();                 // Consume and sum
    
    println!("Sum of squares of evens: {result}");
}
```

### Common Adapters:
*   `map`: Transform each element.
*   `filter`: Keep elements that match a condition.
*   `take(n)`: Only process the first `n` elements.
*   `zip`: Combine two iterators into one.
*   `cycle`: Repeat the iterator forever.

---

## Consuming Iterators
**Getting the final result**

Methods that call `next()` until the iterator is exhausted are called "consumers."

*   `sum()`, `product()`
*   `count()`
*   `max()`, `min()`
*   **`collect()`**: Transforms an iterator into a collection (like `Vec` or `HashMap`).

```rust
// Using the "turbofish" ::<> to specify the collection type
let v: Vec<_> = (1..5).map(|x| x * 10).collect::<Vec<_>>();
```

---

## IntoIterator
**Enabling `for` loops**

The `for` loop automatically calls `.into_iter()` on the collection you give it.

```rust
let v = vec![1, 2, 3];

// This is actually shorthand for:
// for x in v.into_iter() { ... }
for x in v {
    println!("{x}");
}
```

---

## Exercise: Iterator Method Chaining
**Functional Power**

**Goal:** Calculate the differences between elements offset by a certain amount, wrapping around at the end.

```rust
fn offset_differences(offset: usize, values: Vec<i32>) -> Vec<i32> {
    let a = values.iter();
    let b = values.iter().cycle().skip(offset);
    
    // zip combines the two, map subtracts them, collect builds the result
    a.zip(b).map(|(a, b)| *b - *a).collect()
}
```

---

## Pro-Tips for the Instructor:
*   **Lazy vs Eager**: Emphasize that `map` and `filter` do *nothing* on their own. You must call a consumer like `collect` or `sum` to trigger the work.
*   **Item Types**: Explain that `.iter()` gives you references (`&T`), while `.into_iter()` gives you owned values (`T`).
*   **Performance**: Remind students that Rust's iterators are "abstractions without overhead." They are often faster than manual loops because the compiler can eliminate bounds checks.
*   **The Turbofish**: Show `collect::<Vec<i32>>()` vs `collect::<HashSet<_>>()` to demonstrate how `collect` is generic over its return type.

