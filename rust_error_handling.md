# 🦀 Training Session: Error Handling

**Goal:** Master Rust's unique approach to error handling, moving from basic panics to robust, recoverable errors with `Result`.

---

## Panic!
**Unrecoverable Errors**

A `panic!` is for bugs that shouldn't happen. It crashes the program (unwinds the stack) and prints an error message.

```rust
fn main() {
    let v = vec![1, 2, 3];
    // v[100]; // This will PANIC!
    
    panic!("Something went horribly wrong!");
}
```

### When to Panic:
*   Logic bugs (e.g., index out of bounds).
*   Unrecoverable states (e.g., a critical config file is missing).
*   In tests or quick prototypes.

---

## Result<T, E>
**Recoverable Errors**

Most errors aren't bugs; they're expected failures (file not found, network timeout). Use `Result` to force the caller to handle these cases.

```rust
enum Result<T, E> {
    Ok(T),  // The successful value
    Err(E), // The error details
}
```

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
    
    let file = match f {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {error:?}"),
    };
}
```

---

## The ? Operator
**Propagating Errors Elegantly**

The `?` operator is a shortcut for: "If this is an error, return it immediately. If it's a value, give it to me."

```rust
use std::io::{self, Read};
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();
    // If open fails, return the error. If it succeeds, continue.
    File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
```

---

## Custom Error Types
**Defining your own domain errors**

You can define your own error types using an `enum`.

```rust
#[derive(Debug)]
enum MyError {
    Io(std::io::Error),
    Parse(std::num::ParseIntError),
    Validation(String),
}

// Implement From to allow '?' to convert errors automatically
impl From<std::io::Error> for MyError {
    fn from(err: std::io::Error) -> Self { MyError::Io(err) }
}
```

---

## Powerful Crates: `thiserror` and `anyhow`
**Standard tools for error handling**

*   **`thiserror`**: Best for **libraries**. It helps you define custom error enums with less boilerplate.
*   **`anyhow`**: Best for **applications**. It provides a single `anyhow::Error` type that can wrap *any* error and add context.

```rust
// Using anyhow in an application
use anyhow::{Context, Result};

fn run_app() -> Result<()> {
    let content = std::fs::read_to_string("config.json")
        .context("failed to read config file")?;
    Ok(())
}
```

---

## Exercise: Rewriting with Result
**Refactoring for Safety**

**Goal:** Update an expression evaluator to return a `Result` instead of panicking on division by zero.

```rust
#[derive(Debug)]
struct DivideByZeroError;

fn eval(e: Expression) -> Result<i64, DivideByZeroError> {
    match e {
        Expression::Value(v) => Ok(v),
        Expression::Op { op, left, right } => {
            let l = eval(*left)?;
            let r = eval(*right)?;
            if let Operation::Div = op {
                if r == 0 { return Err(DivideByZeroError); }
            }
            // ... perform operation ...
            Ok(result)
        }
    }
}
```

---

## Pro-Tips for the Instructor:
*   **Don't unwrap!**: Encourage students to avoid `.unwrap()` in real code. Use `?` or `match` instead.
*   **Context is King**: Show how `anyhow` allows you to attach messages like "while processing user profile" to an underlying I/O error.
*   **Result vs Option**: Explain that `Option` is for "maybe nothing," while `Result` is for "maybe a mistake."
*   **Main returns Result**: Remind students that `fn main() -> Result<(), anyhow::Error>` is perfectly valid!

