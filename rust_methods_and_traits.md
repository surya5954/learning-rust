# 🦀 Training Session: Methods and Traits

**Goal:** Learn how to associate behavior with types using methods and how to abstract behavior using traits.

---

## Methods
**Associating Logic with Types**

In Rust, you use an `impl` block to define methods on a struct or enum.

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // A constructor (static method)
    fn new(width: u32, height: u32) -> Self {
        Self { width, height }
    }

    // A method that borrows the instance
    fn area(&self) -> u32 {
        self.width * self.height
    }

    // A method that modifies the instance
    fn scale(&mut self, factor: u32) {
        self.width *= factor;
        self.height *= factor;
    }
}
```

### Method Receivers:
*   `&self`: Shared borrow (read-only). Most common.
*   `&mut self`: Exclusive borrow (read-write).
*   `self`: Takes ownership (consumes the object).
*   *No receiver*: Static method (called as `Rectangle::new(...)`).

---

## Traits
**Shared Behavior (Interfaces)**

Traits define a set of methods that multiple types can implement. They are similar to interfaces in other languages.

```rust
trait Summary {
    fn summarize(&self) -> String;
    
    // Default implementation
    fn detailed_summary(&self) -> String {
        format!("Summary: {}", self.summarize())
    }
}

struct Article {
    title: String,
    content: String,
}

impl Summary for Article {
    fn summarize(&self) -> String {
        format!("{}...", &self.content[0..20])
    }
}
```

### Key Traits Concepts:
*   **Explicit Implementation**: You must explicitly `impl Trait for Type`.
*   **Default Methods**: Traits can provide logic that implementations can override or use as-is.
*   **Supertraits**: A trait can require another trait (e.g., `trait Pet: Animal`).

---

## Associated Types
**Output Types for Traits**

Used when a trait's method needs to return a type that is decided by the implementation.

```rust
trait Graph {
    type Node;
    type Edge;

    fn nodes(&self) -> Vec<Self::Node>;
}

struct MyGraph;
impl Graph for MyGraph {
    type Node = u32;
    type Edge = (u32, u32);

    fn nodes(&self) -> Vec<u32> { vec![1, 2, 3] }
}
```

---

## Deriving Traits
**Automatic Implementation**

Rust can automatically implement certain "standard" traits for you using `#[derive]`.

```rust
#[derive(Debug, Clone, Default, PartialEq)]
struct Player {
    name: String,
    level: u32,
}
```

*   `Debug`: Enables `{:?}` printing.
*   `Clone`: Enables `.clone()`.
*   `Default`: Adds `Player::default()`.
*   `PartialEq`: Enables `==` comparison.

---

## Exercise: Generic Logger
**Traits in Action**

**Goal:** Define a `Logger` trait and a `VerbosityFilter` that wraps any logger.

```rust
trait Logger {
    fn log(&self, msg: &str);
}

struct ConsoleLogger;
impl Logger for ConsoleLogger {
    fn log(&self, msg: &str) {
        println!("{msg}");
    }
}

struct VerbosityFilter<L: Logger> {
    max_level: u8,
    inner: L,
}

impl<L: Logger> Logger for VerbosityFilter<L> {
    fn log(&self, msg: &str) {
        // logic to check level...
        self.inner.log(msg);
    }
}
```

---

## Pro-Tips for the Instructor:
*   **Self vs self**: Explain that `Self` (capital S) is the type, while `self` (lowercase s) is the instance.
*   **Method Organization**: Emphasize that `impl` blocks keep code organized by grouping data and behavior together.
*   **Trait Inheritance**: Clarify that `trait A: B` isn't OO inheritance; it's just a prerequisite.
*   **Orphan Rule**: Briefly mention you can only implement a trait if either the trait or the type is local to your crate.

