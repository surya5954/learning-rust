# 🦀 Training Session: Standard Library Types

**Goal:** Master the essential types provided by the Rust standard library that every developer uses daily.

---

## The Big Picture: core, alloc, and std
**Rust's layered library**

*   **`core`**: The absolute basics. No OS required. Used in embedded systems.
*   **`alloc`**: Types that need a heap (like `Vec`, `String`, `Box`).
*   **`std`**: The full library. Includes `core`, `alloc`, plus OS features (files, networking, threads).

---

## Option<T>
**Handling "Nothing" safely**

`Option` represents a value that might or might not be there. It eliminates the "null pointer" category of bugs.

```rust
fn main() {
    let name = "Rustacean";
    let first_char = name.chars().next(); // Returns Option<char>

    match first_char {
        Some(c) => println!("First char: {c}"),
        None    => println!("String is empty!"),
    }
}
```

### Common Helpers:
*   `.unwrap()`: Get the value or **panic** (use only in tests/prototypes).
*   `.expect("msg")`: Like unwrap, but with a custom error message.
*   `.unwrap_or(default)`: Provide a fallback value.

---

## Result<T, E>
**Recoverable Error Handling**

`Result` is used for operations that can fail (like opening a file or parsing a number).

```rust
use std::fs::File;

fn main() {
    let file_result = File::open("hello.txt");

    match file_result {
        Ok(file) => println!("File opened!"),
        Err(e)   => println!("Failed to open: {e}"),
    }
}
```

---

## String
**Growable, UTF-8 Text**

`String` is stored on the heap and can change size.

```rust
fn main() {
    let mut s = String::from("Hello");
    s.push_str(" World"); // Grow the string
    s.push('!');
    
    println!("{s}"); // Hello World!
    println!("Bytes: {}, Chars: {}", s.len(), s.chars().count());
}
```

### Note on Indexing:
You **cannot** index a string like `s[0]` because UTF-8 characters can be multiple bytes long. Use `.chars().nth(i)` instead.

---

## Vec<T>
**The Resizable Array**

`Vec` is the go-to collection for storing a list of items on the heap.

```rust
fn main() {
    let mut v = vec![1, 2, 3]; // Macro for easy init
    v.push(4);
    
    println!("Third element: {:?}", v.get(2)); // Returns Option<&T>
}
```

*   `Vec` implements `Deref<Target = [T]>`, so you can use all slice methods on it.

---

## HashMap<K, V>
**Key-Value Pairs**

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    
    // The .entry() API is the most efficient way to update values
    scores.entry(String::from("Yellow")).or_insert(50);
}
```

---

## Exercise: Counter
**Practicing Collections and Generics**

**Goal:** Create a `Counter<T>` that tracks how many times each value of type `T` has been seen.

```rust
use std::collections::HashMap;
use std::hash::Hash;

struct Counter<T> {
    counts: HashMap<T, u64>,
}

impl<T: Eq + Hash> Counter<T> {
    fn new() -> Self {
        Self { counts: HashMap::new() }
    }

    fn count(&mut self, item: T) {
        *self.counts.entry(item).or_insert(0) += 1;
    }

    fn get_count(&self, item: &T) -> u64 {
        *self.counts.get(item).unwrap_or(&0)
    }
}
```

---

## Pro-Tips for the Instructor:
*   **Performance**: Explain that `String` and `Vec` have a `capacity` and `len`. Pre-allocating with `with_capacity()` prevents expensive re-allocations.
*   **The Prelude**: Mention that many types (like `Option`, `Result`, `Vec`, `String`) are so common they are automatically imported (the "Prelude"). `HashMap` is not in the prelude!
*   **`expect` vs `unwrap`**: Encourage students to use `expect` with descriptive messages to make debugging easier when a panic occurs.
*   **Documentation**: Show them `rustup doc --std`—it's a lifesaver for offline coding.

