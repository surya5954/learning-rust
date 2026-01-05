# 🦀 Training Session: Borrowing and the Borrow Checker

**Goal:** Understand the rules of borrowing that allow multiple parts of your code to access data without ownership issues or data races.

---

## Borrowing a Value
**Access without Ownership**

Instead of moving ownership, you can "borrow" a value using a reference.

```rust
struct Point(i32, i32);

fn add(p1: &Point, p2: &Point) -> Point {
    Point(p1.0 + p2.0, p1.1 + p2.1)
}

fn main() {
    let p1 = Point(3, 4);
    let p2 = Point(10, 20);
    
    // Borrowing p1 and p2
    let p3 = add(&p1, &p2);
    
    // p1 and p2 are still valid here!
    println!("{p1:?} + {p2:?} = {p3:?}");
}
```

---

## The Borrow Checker's Rules
**The "Aliasing" Rule**

Rust enforces a strict set of rules to ensure memory safety. At any given time, for a specific value, you can have:

1.  **Any number of shared references (`&T`)**, OR
2.  **Exactly one exclusive reference (`&mut T`)**.

*You cannot have both at the same time.*

```rust
fn main() {
    let mut a = 10;
    let b = &a;      // Shared borrow
    
    // let c = &mut a; // ERROR! Cannot borrow as mutable while shared borrow 'b' exists
    
    println!("{b}");
}
```

---

## Non-Lexical Lifetimes (NLL)
**Smart Scope Detection**

The compiler is smart enough to see exactly when a reference is last used.

```rust
fn main() {
    let mut a = 10;
    let b = &a;
    println!("{b}"); // 'b' is last used here
    
    let c = &mut a;  // This is OK now, because 'b' is no longer needed
    *c = 20;
}
```

---

## Interior Mutability
**Mutation behind a shared reference**

Sometimes you need to mutate data even when you only have a shared reference (e.g., for internal caches or shared state).

### 1. Cell<T>
Works for small, `Copy` types. You can `.get()` and `.set()` the value. No references to the inner value allowed.

### 2. RefCell<T>
Enforces borrowing rules at **runtime** instead of compile time.
*   `.borrow()`: Returns a shared reference (panics if already borrowed exclusively).
*   `.borrow_mut()`: Returns an exclusive reference (panics if already borrowed).

```rust
use std::cell::RefCell;

let cell = RefCell::new(5);
{
    let mut val = cell.borrow_mut();
    *val += 1;
} // val goes out of scope, borrow ends
println!("{:?}", cell.borrow());
```

---

## Exercise: Health Statistics
**Managing Mutable State**

**Goal:** Implement a `visit_doctor` method that updates a user's stats and returns a report.

```rust
pub fn visit_doctor(&mut self, measurements: Measurements) -> HealthReport<'_> {
    self.visit_count += 1;
    let report = HealthReport {
        patient_name: &self.name,
        visit_count: self.visit_count,
        height_change: measurements.height - self.height,
        // ... logic for blood pressure change ...
    };
    self.height = measurements.height;
    report
}
```

---

## Pro-Tips for the Instructor:
*   **Safety vs. Convenience**: Explain that the borrow checker is "strict but fair." It prevents bugs like iterator invalidation that are common in C++ and Java.
*   **The "Writer's Lock"**: Use a database analogy. You can have many readers, OR one writer, but never both at once.
*   **Runtime Cost of RefCell**: Mention that `RefCell` has a small overhead for the runtime check. Use it only when compile-time borrowing is impossible.
*   **Re-borrowing**: Briefly explain that passing a `&mut T` to a function is actually a "re-borrow," which is why you can still use the reference after the function returns.

