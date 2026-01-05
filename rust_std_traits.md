# 🦀 Training Session: Standard Library Traits

**Goal:** Understand the "superpower" traits in Rust that enable operator overloading, type conversion, and common abstractions.

---

## Comparison Traits
**How to use ==, <, and >**

Rust splits comparisons into four main traits. Most of the time, you just `#[derive]` them.

*   **`PartialEq` & `Eq`**: Enable `==` and `!=`. 
    *   `Eq` is a "marker trait" for full equivalence (it has no methods). `f32` is `PartialEq` but NOT `Eq` because `NaN != NaN`.
*   **`PartialOrd` & `Ord`**: Enable `<`, `>`, `<=`, `>=`.
    *   `Ord` requires a total ordering. Useful for sorting.

```rust
#[derive(PartialEq, PartialOrd, Eq, Ord)]
struct Player {
    score: u32,
    name: String,
}
```

---

## Operator Overloading
**Customizing + , - , * , etc.**

Rust allows you to define how operators work for your types via traits in `std::ops`.

```rust
use std::ops::Add;

struct Point { x: i32, y: i32 }

impl Add for Point {
    type Output = Self;
    
    fn add(self, other: Self) -> Self {
        Self { x: self.x + other.x, y: self.y + other.y }
    }
}
```

---

## From and Into
**Safe Type Conversions**

`From` and `Into` are the idiomatic way to convert between types in a lossless way.

*   **Rule of Thumb**: Implement `From`. Rust automatically gives you `Into` for free.

```rust
struct MyString(String);

impl From<&str> for MyString {
    fn from(s: &str) -> Self {
        MyString(s.to_string())
    }
}

fn main() {
    // Both work!
    let s1 = MyString::from("hello");
    let s2: MyString = "world".into();
}
```

---

## Read and Write
**Abstracting over I/O**

The `Read` and `Write` traits allow you to write functions that work with anything that can be read from (files, sockets, byte arrays) or written to.

```rust
use std::io::{Read, Write, Result};

fn copy_data<R: Read, W: Write>(mut source: R, mut dest: W) -> Result<()> {
    let mut buffer = [0; 128];
    while let Ok(n) = source.read(&mut buffer) {
        if n == 0 { break; }
        dest.write_all(&buffer[..n])?;
    }
    Ok(())
}
```

---

## The Default Trait
**Creating "Empty" states**

`Default` provides a standard way to initialize a type with default values. It also enables **Struct Update Syntax**.

```rust
#[derive(Default, Debug)]
struct Config {
    port: u16,
    timeout: u32,
    path: String,
}

fn main() {
    // Create a config with custom port, but everything else as default
    let cfg = Config {
        port: 8080,
        ..Config::default() // Struct Update Syntax
    };
}
```

---

## Exercise: ROT13 Decoder
**Implementing Traits**

**Goal:** Implement the `Read` trait for a `RotDecoder` struct that rotates ASCII characters.

```rust
struct RotDecoder<R: Read> {
    inner: R,
    rot: u8,
}

impl<R: Read> Read for RotDecoder<R> {
    fn read(&mut self, buf: &mut [u8]) -> std::io::Result<usize> {
        let size = self.inner.read(buf)?;
        for b in &mut buf[..size] {
            if b.is_ascii_alphabetic() {
                let base = if b.is_ascii_uppercase() { b'A' } else { b'a' };
                *b = (*b - base + self.rot) % 26 + base;
            }
        }
        Ok(size)
    }
}
```

---

## Pro-Tips for the Instructor:
*   **Derive vs Manual**: Most traits should be derived. Only implement manually if you need custom logic (e.g., comparing `Person` by ID only).
*   **Lossy vs Lossless**: Remind students that `From`/`Into` are for **lossless** conversions. For lossy/dangerous conversions, use `as` or `TryFrom`.
*   **The ? Operator**: Mention that `Result` from I/O operations works perfectly with the `?` operator for error propagation.
*   **Infallible**: Explain that `From` is "infallible" (cannot fail). If a conversion can fail, use `TryFrom`.

