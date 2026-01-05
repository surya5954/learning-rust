# 🦀 Training Session: Unsafe Rust

**Goal:** Understand what "Unsafe" means in Rust, when to use it, and how to wrap it in safe abstractions.

---

## What is Unsafe Rust?
**Stepping outside the safety net**

Safe Rust prevents memory errors at compile time. **Unsafe Rust** allows you to perform operations that the compiler cannot guarantee are safe.

### The Five "Superpowers":
1.  Dereference a **raw pointer**.
2.  Access or modify a **mutable static** variable.
3.  Access fields of a **union**.
4.  Call an **unsafe function** (including FFI).
5.  Implement an **unsafe trait**.

```rust
fn main() {
    let mut num = 5;
    let r1 = &num as *const i32; // Raw pointer
    
    unsafe {
        // Dereferencing is unsafe!
        println!("r1 is: {}", *r1);
    }
}
```

---

## Raw Pointers
**\*const T and \*mut T**

Unlike references, raw pointers:
*   Can be null.
*   Don't have guaranteed lifetimes.
*   Don't have borrow checking (can have multiple mutable pointers to one place).

---

## Unsafe Functions
**Contract-based safety**

An unsafe function is one that has "preconditions"—requirements that the caller must follow to avoid undefined behavior.

```rust
unsafe fn dangerous_operation() {
    // ... logic that assumes something about memory ...
}

fn main() {
    unsafe {
        dangerous_operation();
    }
}
```

---

## Safe Abstractions
**The Rust Way**

The standard way to use `unsafe` is to wrap it inside a **Safe API**.

```rust
// A safe function that uses unsafe internally
fn my_split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();

    assert!(mid <= len);

    unsafe {
        (
            std::slice::from_raw_parts_mut(ptr, mid),
            std::slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}
```

---

## Foreign Function Interface (FFI)
**Talking to C**

Rust can call C functions directly. Because C has no safety guarantees, these calls are always `unsafe`.

```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

---

## Exercise: FFI Wrapper
**Building a Safe Bridge**

**Goal:** Create a safe Rust wrapper around a C library function.

```rust
// Assume we have a C function 'int snappy_compress(...)'
// Your goal is to provide a Rust function that takes a &[u8]
// and returns a Result<Vec<u8>, Error>.
```

---

## Pro-Tips for the Instructor:
*   **Unsafe != Bad**: Emphasize that `unsafe` is necessary for things like operating systems, device drivers, and low-level optimizations.
*   **Trust but Verify**: Explain that the "safety" of a Rust program is the sum of its safe code + the correctness of its `unsafe` blocks.
*   **Miri**: Mention the tool `cargo miri`, which can detect undefined behavior in `unsafe` code at runtime during tests.
*   **The "Unsafe" Keyword**: Remind students that `unsafe` doesn't turn off the borrow checker; it just enables the 5 superpowers.

