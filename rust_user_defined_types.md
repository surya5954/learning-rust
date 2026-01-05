# 🦀 Training Session: User-Defined Types

**Goal:** Learn how to create custom structures using Named Structs, Tuple Structs, and Enums.

---

## Named Structs
**Data with Labels**

Named structs are the most common way to group related data.

```rust
struct Person {
    name: String,
    age: u8,
}

fn main() {
    let mut peter = Person {
        name: String::from("Peter"),
        age: 27,
    };
    
    peter.age = 28; // Modification requires 'mut'
    println!("{} is {} years old", peter.name, peter.age);
}
```

### Key Features:
*   **Field Shorthand**: If you have a variable `name`, you can just write `Person { name, age }`.
*   **Update Syntax**: `let jackie = Person { name: "Jackie".into(), ..peter };` copies other fields from `peter`.

---

## Tuple Structs
**Data without Labels**

Used when the field names are less important than the type itself, or for "Newtypes".

```rust
struct Color(i32, i32, i32);
struct Newtons(f64); // Newtype pattern for type safety

fn main() {
    let black = Color(0, 0, 0);
    let force = Newtons(9.81);
    
    println!("Force value: {}", force.0);
}
```

### The Newtype Pattern:
Prevents accidental mixing of units (e.g., adding `Pounds` to `Newtons`) by wrapping them in different types.

---

## Enums
**Choosing between Variants**

Enums allow a type to be one of several different variants.

```rust
enum WebEvent {
    PageLoad,                 // Unit variant
    KeyPress(char),           // Tuple variant
    Paste(String),            // Tuple variant
    Click { x: i64, y: i64 }, // Struct variant
}
```

### Why Rust Enums are special:
Unlike C/C++ enums, Rust enums (Sum Types) can **store data** inside each variant. This is incredibly powerful for representing state.

---

## Type Aliases & Constants
**Code Clarity**

*   **Type Aliases**: `type Kilometers = i32;` (just a nickname, not a new type).
*   **Constants**: `const MAX_POINTS: u32 = 100_000;` (inlined at compile time).
*   **Statics**: `static mut TOTAL_REQUESTS: u32 = 0;` (fixed memory location, usually requires `unsafe`).

---

## Exercise: Elevator Events
**Modeling with Enums**

**Goal:** Represent the possible states and commands of an elevator.

```rust
enum Direction {
    Up,
    Down,
}

enum ElevatorEvent {
    ButtonPress(u32),       // Floor number
    DoorOpened,
    DoorClosed,
    Moving(Direction),
}

fn handle_event(event: ElevatorEvent) {
    match event {
        ElevatorEvent::ButtonPress(f) => println!("Button pressed for floor {f}"),
        ElevatorEvent::Moving(dir) => match dir {
            Direction::Up => println!("Elevator is going Up"),
            Direction::Down => println!("Elevator is going Down"),
        },
        _ => println!("Other event triggered"),
    }
}
```

---

## Pro-Tips for the Instructor:
*   **Struct Memory**: Mention that Rust structs are "packed" for efficiency, but you can't assume a specific memory layout like in C unless you use `#[repr(C)]`.
*   **Enum Exhaustiveness**: Show that `match` on an enum requires handling every variant.
*   **Newtypes**: Emphasize how `Newtons(f64)` prevents logical errors that a plain `f64` would allow.
*   **Zero-Sized Types**: Briefly mention `struct Foo;` can be used to implement traits without storing data.

