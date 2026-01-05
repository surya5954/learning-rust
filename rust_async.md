# 🦀 Training Session: Async Rust

**Goal:** Understand how to write non-blocking, asynchronous code in Rust using `async/await` and the `Tokio` runtime.

---

## What is Async Rust?
**Efficient Concurrency**

Async allows you to run many tasks concurrently on a small number of OS threads. It's ideal for I/O-bound work (web servers, database clients).

```rust
async fn count_to(n: i32) {
    for i in 0..n {
        println!("Count: {i}");
    }
}

#[tokio::main]
async fn main() {
    // Calling an async function returns a 'Future'
    let future = count_to(10);
    
    // Futures do nothing until they are .awaited
    future.await;
}
```

---

## Futures
**Inert Operations**

A `Future` is a trait that represents a value that will be available later.

```rust
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

*   **Lazy**: In Rust, futures are "inert." They don't start running until you poll them (usually via `.await`).
*   **State Machine**: The compiler transforms your `async fn` into a state machine that tracks progress between `.await` points.

---

## The Runtime (Tokio)
**The Executor**

Rust doesn't have a built-in async runtime. **Tokio** is the most popular choice for production apps.

```rust
#[tokio::main]
async fn main() {
    // tokio::spawn creates a concurrent task (like thread::spawn)
    let handle = tokio::spawn(async {
        // ... some async work ...
        "Done!"
    });

    let result = handle.await.unwrap();
}
```

---

## Asynchronous Control Flow
**Join and Select**

### 1. `join_all`: Wait for everyone
Runs multiple futures in parallel and waits for all of them to finish.

### 2. `select!`: Wait for the first one
Waits for multiple futures and responds to whichever one finishes first. Great for timeouts!

```rust
tokio::select! {
    val = rx.recv() => println!("Got: {:?}", val),
    _ = tokio::time::sleep(Duration::from_millis(100)) => println!("Timeout!"),
}
```

---

## Pitfalls: Blocking the Executor
**The "Never Sleep" Rule**

If you call a blocking operation (like `std::thread::sleep` or a heavy CPU calculation) inside an `async fn`, you block the entire runtime thread, preventing other tasks from running.

*   **BAD**: `std::thread::sleep(Duration::from_secs(1));`
*   **GOOD**: `tokio::time::sleep(Duration::from_secs(1)).await;`
*   **For heavy CPU**: Use `tokio::task::spawn_blocking`.

---

## Pin and Unpin
**Ensuring Memory Safety**

Because async state machines can contain references to their own data, they must stay in one place in memory. `Pin` is a wrapper that guarantees an object will not be moved.

---

## Exercise: Async Web Scraper
**Racing against Time**

**Goal:** Fetch the size of multiple web pages in parallel, but cancel the operation if it takes more than 2 seconds.

```rust
async fn get_page_size(url: &str) -> Result<usize, reqwest::Error> {
    let resp = reqwest::get(url).await?;
    let body = resp.text().await?;
    Ok(body.len())
}

#[tokio::main]
async fn main() {
    let url = "https://www.rust-lang.org";
    
    tokio::select! {
        res = get_page_size(url) => println!("Size: {:?}", res),
        _ = tokio::time::sleep(Duration::from_secs(2)) => println!("Took too long!"),
    }
}
```

---

## Pro-Tips for the Instructor:
*   **Async vs Threads**: Explain that while threads are "preemptive" (the OS switches between them), async is "cooperative" (tasks yield control at `.await` points).
*   **The Ecosystem**: Mention that you need async versions of libraries (e.g., `reqwest` for HTTP, `sqlx` for databases).
*   **Zero-Cost**: Emphasize that Rust's async has no overhead when not in use and generates very efficient machine code.
*   **Cancellation**: Explain that dropping a Future effectively cancels the operation.

