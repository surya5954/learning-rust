# 🦀 Training Session: Concurrency and Threads

**Goal:** Understand how Rust enables "Fearless Concurrency" by using its ownership and type system to catch data races at compile time.

---

## Threads
**Executing in Parallel**

Rust uses OS threads. `thread::spawn` creates a new thread that runs a closure.

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("Hi from spawned thread! {i}");
            thread::sleep(Duration::from_millis(1));
        }
    });

    handle.join().unwrap(); // Wait for the thread to finish
}
```

### Key Rules:
*   **program termination**: The program ends when `main` finishes, even if other threads are still running. Use `.join()` to wait.
*   **Move closures**: You usually need to use the `move` keyword to give the new thread ownership of the variables it uses.

---

## Scoped Threads
**Borrowing from the environment**

Normal threads cannot borrow local variables because the compiler can't guarantee how long the thread will live. Scoped threads solve this by guaranteeing all threads finish before the scope ends.

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    thread::scope(|s| {
        s.spawn(|| {
            println!("I can borrow v: {v:?}");
        });
    }); // All threads joined here automatically
}
```

---

## Channels
**Message Passing**

Rust follows the philosophy: "Do not communicate by sharing memory; instead, share memory by communicating."

```rust
use std::sync::mpsc; // Multi-Producer, Single-Consumer
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        tx.send(String::from("Hi")).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {received}");
}
```

### Channel Types:
*   **Unbounded**: `mpsc::channel()` (infinite buffer).
*   **Bounded**: `mpsc::sync_channel(n)` (blocks sender when full).

---

## Shared State: Arc and Mutex
**Thread-Safe Shared Ownership**

When you *must* share memory, use `Arc` (Atomic Reference Counted) and `Mutex` (Mutual Exclusion).

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles { handle.join().unwrap(); }
    println!("Result: {}", *counter.lock().unwrap());
}
```

---

## Send and Sync Traits
**The engine of thread safety**

These are "marker traits" that the compiler uses to track thread safety.

*   **`Send`**: Allows a type to be **moved** to another thread. (Most types are `Send`).
*   **`Sync`**: Allows a type to be **shared** (via `&T`) across threads. (A type `T` is `Sync` if `&T` is `Send`).

### Non-Thread-Safe Types:
*   `Rc<T>`: Not `Send` or `Sync` (use `Arc`).
*   `RefCell<T>`: Not `Sync` (use `Mutex` or `RwLock`).

---

## Exercise: Parallel Processing
**Speeding things up**

**Goal:** Take a large vector of numbers and sum them in parallel using multiple threads.

```rust
fn parallel_sum(v: Vec<i32>) -> i32 {
    let chunk_size = v.len() / 4;
    thread::scope(|s| {
        let mut handles = vec![];
        for chunk in v.chunks(chunk_size) {
            handles.push(s.spawn(|| chunk.iter().sum::<i32>()));
        }
        handles.into_iter().map(|h| h.join().unwrap()).sum()
    })
}
```

---

## Pro-Tips for the Instructor:
*   **Poisoned Mutexes**: Explain that `lock().unwrap()` is common because if a thread panics while holding a lock, the data might be corrupted.
*   **Deadlocks**: Remind students that Rust prevents data races but **does not prevent deadlocks**.
*   **Arc vs Rc**: Emphasize that `Arc` uses atomic operations which have a slight performance cost compared to `Rc`.
*   **Fearless Concurrency**: Point out that most concurrency bugs in other languages are "compile-time errors" in Rust.

