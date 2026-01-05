# 🦀 Training Session: Testing and Lints

**Goal:** Learn how to write robust Rust code using the built-in testing framework and automated linting tools.

---

## Unit Testing
**Testing logic where it lives**

In Rust, unit tests are typically placed in the same file as the code they test, inside a nested `tests` module.

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)] // Only compiled when running tests
mod tests {
    use super::*; // Bring the parent module into scope

    #[test]
    fn test_add() {
        assert_eq!(add(2, 2), 4);
    }
}
```

### Key Features:
*   **`#[test]`**: Marks a function as a test.
*   **`assert_eq!` / `assert_ne!`**: Check for equality or inequality.
*   **`#[should_panic]`**: Verifies that a function crashes as expected.

---

## Integration Testing
**Testing your library as a client**

Integration tests live in a special `tests/` directory at the root of your project. They can only test the **public API** of your crate.

```text
my_project/
├── src/
│   └── lib.rs
└── tests/
    └── integration_test.rs
```

```rust
// tests/integration_test.rs
use my_project::public_function;

#[test]
fn test_external_call() {
    assert!(public_function().is_ok());
}
```

---

## Documentation Tests
**Examples that actually run**

Rust compiles and runs code blocks in your documentation comments (`///`). This ensures your examples never go out of date.

```rust
/// Adds two numbers together.
///
/// ```
/// use my_crate::add;
/// assert_eq!(add(2, 3), 5);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

---

## Compiler Lints and Clippy
**Automated Code Review**

Rust comes with a set of built-in lints to catch common mistakes. **Clippy** is an even more advanced tool with 500+ additional checks.

*   **`#[allow(unused_variables)]`**: Silences a warning.
*   **`#[deny(clippy::all)]`**: Turns warnings into errors.

### Running Tools:
```bash
cargo check   # Fast check for errors
cargo test    # Run all tests
cargo clippy  # Run clippy lints
```

---

## Exercise: Luhn Algorithm
**Test-Driven Development**

**Goal:** Implement and test the Luhn algorithm used for credit card validation.

```rust
pub fn luhn(cc_number: &str) -> bool {
    // 1. Ignore spaces
    // 2. Reject if < 2 digits
    // 3. Double every second digit from right
    // 4. Sum and check if sum % 10 == 0
}

#[cfg(test)]
mod tests {
    use super::*;
    #[test]
    fn test_valid_card() {
        assert!(luhn("4263 9826 4026 9299"));
    }
}
```

---

## Pro-Tips for the Instructor:
*   **Test Private Functions**: Mention that because unit tests are in a child module (`mod tests`), they have access to private functions in the parent.
*   **`#[cfg(test)]`**: Explain that this prevents test code from being included in the final production binary.
*   **Cargo Test Output**: Show how to read the output, including captured stdout (which is hidden by default unless you use `--nocapture`).
*   **Clippy as a Teacher**: Encourage students to run Clippy often; its suggestions are one of the best ways to learn idiomatic Rust.

