# 🦀 Training Session: Lifetimes

**Goal:** Learn how to use lifetime annotations to tell the compiler how long references should stay valid.

---

## What is a Lifetime?
**The "Valid From/Until" of a reference**

A lifetime is a construct the compiler uses to ensure all borrows are valid. Most of the time, they are invisible (elided), but sometimes you need to be explicit.

```rust
// 'a is a lifetime parameter
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

### The Meaning of `<'a>`:
*   It does **not** change how long a value lives.
*   It **describes** the relationship between the lifetimes of multiple references.
*   In `longest`, it says: "The returned reference will live as long as the **shortest** of the two inputs."

---

## Multiple Lifetimes
**Independent Borrows**

If a function takes multiple references but only returns one of them, you can use different lifetimes to be more precise.

```rust
fn find_nearest<'a, 'b>(points: &'a [Point], query: &'b Point) -> &'a Point {
    // Logic that only returns a reference from 'points'
    &points[0]
}
```
*   Here, the returned reference is tied **only** to `points`. The `query` reference can be dropped immediately after the function returns.

---

## Lifetime Elision
**The compiler's shorthand**

You don't always have to write `'a`. Rust has three rules to guess them for you:

1.  **Input Rule**: Each input parameter gets its own lifetime.
2.  **Output Rule (Single Input)**: If there is exactly one input lifetime, that lifetime is assigned to all output lifetimes.
3.  **Method Rule**: If there are multiple input lifetimes but one is `&self`, the lifetime of `self` is assigned to all output lifetimes.

```rust
fn identity(x: &i32) -> &i32 { x } 
// Becomes: fn identity<'a>(x: &'a i32) -> &'a i32 { x }
```

---

## Lifetimes in Data Structures
**Structs that "borrow"**

If a struct holds a reference, it **must** have a lifetime annotation. The struct cannot outlive the data it points to.

```rust
struct Highlight<'document> {
    slice: &'document str,
}

fn main() {
    let doc = String::from("The quick brown fox");
    let h = Highlight { slice: &doc[0..3] };
    
    // drop(doc); // ERROR! h still points to doc
    println!("{:?}", h.slice);
}
```

---

## The `'static` Lifetime
**Living Forever**

The `'static` lifetime means the reference is valid for the entire duration of the program.

*   **String Literals**: `"Hello"` has the type `&'static str`.
*   **Global Constants**: `static MAX: i32 = 10;`.

---

## Exercise: Protobuf Parsing
**Zero-copy Parsing**

**Goal:** Build a parser that returns "views" into a byte slice without copying any data.

```rust
struct Message<'a> {
    data: &'a [u8],
}

impl<'a> Message<'a> {
    fn parse_field(&self) -> &'a [u8] {
        // Returns a slice of the original data
        &self.data[0..4]
    }
}
```

---

## Pro-Tips for the Instructor:
*   **Descriptive names**: Use `'doc` or `'input` instead of just `'a` to make code more readable.
*   **"Generic over Lifetimes"**: Explain that just as `Vec<T>` is generic over a type, `&'a str` is generic over a lifetime.
*   **The Borrow Checker's Job**: Reiterate that lifetimes don't *extend* how long things live; they just *verify* that you're not using something after it's gone.
*   **Owned vs. Borrowed**: Encourage using `String` in structs initially to avoid lifetime complexity, then switch to `&str` only when performance requires zero-copy.

