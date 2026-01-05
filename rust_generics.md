# 🦀 Training Session: Generics

**Goal:** Learn how to write flexible, reusable code that works across different types without sacrificing performance.

---

## Generic Functions
**Abstracting over Types**

Generics allow you to write a function once and use it for many different types.

```rust
fn pick<T>(cond: bool, left: T, right: T) -> T {
    if cond { left } else { right }
}

fn main() {
    println!("picked a number: {:?}", pick(true, 10, 20));
    println!("picked a string: {:?}", pick(false, "left", "right"));
}
```

### How it works (Monomorphization):
Rust generates a specific version of the function for every type you use. This means there is **zero runtime cost** for using generics.

---

## Trait Bounds
**Restricting Generics**

Sometimes you need a generic type to support certain behaviors (like being able to print or compare).

```rust
use std::fmt::Display;

fn print_it<T: Display>(item: T) {
    println!("Item: {item}");
}

// Using 'where' clauses for cleaner signatures
fn complex_function<T, U>(t: T, u: U)
where
    T: Display + Clone,
    U: Clone + PartialEq,
{
    // ... logic ...
}
```

---

## Generic Data Types
**Flexible Structs and Enums**

```rust
struct Pair<T, U> {
    first: T,
    second: U,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

*   `Option<T>` and `Result<T, E>` are the most famous examples of generic enums in Rust.

---

## Static vs Dynamic Dispatch
**Choosing how to call methods**

### 1. Static Dispatch (`impl Trait`)
The compiler knows the exact type at compile time. Fast, but increases binary size.

```rust
fn say_hello(item: &impl Pet) {
    println!("{}", item.talk());
}
```

### 2. Dynamic Dispatch (`dyn Trait`)
The type is erased at compile time. The computer looks up the method in a "vtable" at runtime. Slower, but allows for collections of different types.

```rust
fn say_hello_dynamic(item: &dyn Pet) {
    println!("{}", item.talk());
}

// Allows a list of different animals!
let pets: Vec<Box<dyn Pet>> = vec![Box::new(Dog), Box::new(Cat)];
```

---

## impl Trait in Return Position
**Hiding Concrete Types**

Use `impl Trait` as a return type when you want to return "something that implements this trait" without naming the specific type.

```rust
fn get_debuggable(x: u32) -> impl std::fmt::Debug {
    (x, x + 1)
}
```

---

## Exercise: Generic min
**Applying Trait Bounds**

**Goal:** Implement a `min` function that works for any type that can be compared (`Ord`).

```rust
use std::cmp::Ordering;

fn min<T: Ord>(l: T, r: T) -> T {
    match l.cmp(&r) {
        Ordering::Less | Ordering::Equal => l,
        Ordering::Greater => r,
    }
}

#[test]
fn test_min() {
    assert_eq!(min(10, 20), 10);
    assert_eq!(min("apple", "banana"), "apple");
}
```

---

## Pro-Tips for the Instructor:
*   **Zero-Cost Abstraction**: Reiterate that generics in Rust are as fast as hand-written code for each type.
*   **The Turbofish `::<>`**: Show students how to explicitly provide types when the compiler can't infer them (e.g., `collect::<Vec<_>>()`).
*   **Input vs Output**: Explain that generic parameters are "inputs" (caller decides), while associated types are "outputs" (implementation decides).
*   **Fat Pointers**: Briefly mention that `&dyn Trait` is a "fat pointer" containing both the data pointer and the vtable pointer.

