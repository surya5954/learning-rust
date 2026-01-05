# 🦀 Training Session: Smart Pointers

**Goal:** Learn how to use `Box`, `Rc`, and Trait Objects to manage memory and enable complex data structures.

---

## What are Smart Pointers?
**More than just a reference**

In Rust, references (`&T`) are simple pointers. **Smart Pointers** are data structures that act like pointers but have extra metadata and capabilities (like reference counting).

*   They usually own the data they point to.
*   They implement `Deref` (to act like a pointer) and `Drop` (to clean up).

---

## Box<T>
**Heap Allocation & Indirection**

`Box` is the simplest smart pointer. It puts data on the heap and gives you a pointer to it.

```rust
fn main() {
    let b = Box::new(5); // 5 is now on the heap
    println!("b = {b}");
}
```

### When to use `Box`:
1.  **Unknown size**: When you have a type whose size can't be known at compile time (like a recursive type).
2.  **Large data**: To transfer ownership of large data without copying it on the stack.
3.  **Trait Objects**: To own a value that implements a certain trait (`Box<dyn Trait>`).

---

## Recursive Data Types
**Breaking the infinite size loop**

The compiler must know the size of a type at compile time. Recursive types (like a Linked List) would be infinitely large without a pointer.

```rust
#[derive(Debug)]
enum List {
    Cons(i32, Box<List>), // Box provides the indirection
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Nil))));
}
```

---

## Rc<T>
**Multiple Owners (Reference Counting)**

Use `Rc` when you need a single piece of data to have **multiple owners** in the same thread.

```rust
use std::rc::Rc;

fn main() {
    let data = Rc::new(String::from("Shared Data"));
    
    let owner1 = Rc::clone(&data); // Increments reference count
    let owner2 = Rc::clone(&data); // Increments reference count
    
    println!("Count: {}", Rc::strong_count(&data)); // 3
}
```

*   **Note**: `Rc` only allows **read-only** access to the shared data.
*   **Thread Safety**: `Rc` is NOT thread-safe. Use `Arc` for multi-threading.

---

## Owned Trait Objects
**Storing different types in one container**

While `&dyn Trait` is a borrowed trait object, `Box<dyn Trait>` is an **owned** trait object.

```rust
trait Pet {
    fn talk(&self) -> String;
}

struct Dog;
struct Cat;

impl Pet for Dog { fn talk(&self) -> String { "Woof!".into() } }
impl Pet for Cat { fn talk(&self) -> String { "Meow!".into() } }

fn main() {
    // A vector of different types that all implement Pet!
    let pets: Vec<Box<dyn Pet>> = vec![
        Box::new(Dog),
        Box::new(Cat),
    ];
}
```

---

## Exercise: Binary Tree
**Building with Box**

**Goal:** Create a binary tree where each node might have a left and right child.

```rust
struct Node<T> {
    value: T,
    left: Option<Box<Node<T>>>,
    right: Option<Box<Node<T>>>,
}

impl<T: Ord> Node<T> {
    fn insert(&mut self, new_val: T) {
        if new_val < self.value {
            // logic to insert into left...
        } else {
            // logic to insert into right...
        }
    }
}
```

---

## Pro-Tips for the Instructor:
*   **The "Fat Pointer"**: Explain that `Box<dyn Trait>` is a "fat pointer" (data pointer + vtable pointer), taking up two `usize` on the stack.
*   **Clone vs Rc::clone**: Emphasize that `Rc::clone` is fast (it just increments a counter), whereas `.clone()` usually deep-copies heap data.
*   **Niche Optimization**: Mention that `Option<Box<T>>` is the same size as a raw pointer because `Box` can't be null.
*   **Memory Leaks**: Briefly mention that while Rust is memory-safe, you can still create memory leaks if you use `Rc` to create a circular reference (use `Weak` to break cycles).

