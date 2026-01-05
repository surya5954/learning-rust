# 🦀 Training Session: Pattern Matching

**Goal:** Learn how to use patterns to destructure data, handle control flow, and write concise logic.

---

## Irrefutable Patterns
**Patterns that cannot fail**

Irrefutable patterns always match the value they are applied to. You see these in `let` statements and function parameters.

```rust
fn main() {
    let tuple = ('a', 7, true);
    
    // Destructuring with an irrefutable pattern
    let (letter, number, _) = tuple;
    
    // Using '..' to ignore the rest
    let (first, ..) = tuple;
}
```

### Common Irrefutable Patterns:
*   **Variable names**: `let x = 5;`
*   **Wildcards**: `let _ = 5;`
*   **Tuples/Structs of irrefutable patterns**: `let (x, y) = (1, 2);`

---

## The `match` Keyword
**Powerful Branching**

`match` allows you to compare a value against a series of patterns and execute code based on the first match.

```rust
fn main() {
    let input = 'w';
    
    match input {
        'q' => println!("Quitting"),
        'a' | 's' | 'w' | 'd' => println!("Moving around"), // Multiple values
        '0'..='9' => println!("Digit input"),               // Ranges
        key if key.is_uppercase() => println!("Upper: {key}"), // Match Guards
        _ => println!("Something else"),                     // Catch-all
    }
}
```

---

## Destructuring Structures
**Reaching inside**

Patterns can "reach into" structs and enums to pull out specific fields.

### Destructuring Structs:
```rust
struct Point { x: i32, y: i32 }

let p = Point { x: 0, y: 7 };
match p {
    Point { x, y: 0 } => println!("On the X axis at {x}"),
    Point { x: 0, y } => println!("On the Y axis at {y}"),
    Point { x, y }    => println!("At ({x}, {y})"),
}
```

### Destructuring Enums:
```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
}

match msg {
    Message::Quit => println!("Quit"),
    Message::Move { x, y } => println!("Move to {x}, {y}"),
    Message::Write(text) => println!("Text: {text}"),
}
```

---

## `if let` and `while let`
**Concise Control Flow**

When you only care about **one** specific variant, `if let` is cleaner than a full `match`.

```rust
// Instead of this:
match some_option {
    Some(value) => println!("Got {value}"),
    _ => (), // Mandatory catch-all
}

// Do this:
if let Some(value) = some_option {
    println!("Got {value}");
}
```

### `while let`:
Continues looping as long as a pattern matches.
```rust
while let Some(item) = queue.pop() {
    println!("Processing {item}");
}
```

---

## Exercise: Expression Evaluation
**Recursion and Patterns**

**Goal:** Build a simple calculator that evaluates a tree of operations.

```rust
enum Expr {
    Number(i32),
    Add(Box<Expr>, Box<Expr>),
    Sub(Box<Expr>, Box<Expr>),
}

fn eval(expr: Expr) -> i32 {
    match expr {
        Expr::Number(n) => n,
        Expr::Add(l, r) => eval(*l) + eval(*r),
        Expr::Sub(l, r) => eval(*l) - eval(*r),
    }
}
```

### Evaluation Logic:
```mermaid
graph TD
    Add[Add] --> L[Sub]
    Add --> R[Number: 10]
    L --> L1[Number: 5]
    L --> L2[Number: 2]
    
    subgraph "Calculation"
    Calc[ (5 - 2) + 10 = 13 ]
    end
```

---

## Pro-Tips for the Instructor:
*   **Variable Shadowing**: Mention that `if let Some(x) = x` shadows the outer `x` with the inner value.
*   **Match Guards**: Explain that `if` conditions in match arms are checked *after* the pattern matches.
*   **Exhaustiveness**: Reiterate that the compiler catches missing cases in `match`.
*   **The `_` variable**: Explain the difference between `_` (discard immediately) and `_name` (variable exists but triggers no "unused" warning).

