# Rust Cheatsheet

## Installing Rust

Visit rustup.rs
Rustup is a toolchain that includes:
- compilation (and cross-compilation) tool
- package manager (cargo)

## Cargo

Cargo is rust package manager. We can use cargo to create a new project.

    cargo new myproject (--bin)

Additional package can be browsed at: [crates.io](http://crates.io).

Cargo can also be used to compile or run the project:

    // compiling the project
    cargo build

    // running the project
    cargo run

## Variables

```rust
    // normal variable declaration & initialization
    // the default is **immutable**
    let a = 1;

    // if we want it to be immutable, we should say so
    let mut b = 3;

    // most of the time, data type can be inferred
    let c = 10; // int32
    let d = 0.75; // f64

    // but can also be explicitly specified
    let e: f32 = 0.75;

    // Variables scope only live within a block
    let x = 1;
    {
        let x = 2;
        println!("{}", x);
    }
    println!("{}", x);

    // rust is very strongly-typed so sometimes we need to cast
    1 as f32;
```

## Strings

``` rust
    // Rust keeps 2 format of string, str and String
    // str is a string slice that allows no allocation costs
    // and zero copying of memory when passing it around.
    // fixed size and content cannot be changed
    let string_slice = "Hello";

    // lifetime of string slice can be explicitly specified
    let string_slice: &'static str = "Hello";

    // String is a high-level format of string and not zero cost in memory
    // Content can be changed, mutable and is guaranteed Unicode
    let mut string: String = string_slice.to_string();
```

## Conditional Statements

```rust
    let a = 5;

    if a < 1 {
        println!("a is smaller than 1");
    } else {
        println!("a is bigger than 1");
    }
```

## Range

```rust
    // Range in rust is 'half-open'
    // Inclusive below but exclusive above
    1..101
```

## Loop

```rust
    // Loop with explicit conditional
    for i in 1..101 {
        //
    }

    // Loop with implicit conditional
    loop {
        //
    }
```

## Pattern Matching

```rust
    // Rust have functional-style pattern matching
    let x = 1;

    match x {
        1 => println!("one"),
        2 => println!("two"),
        3 => println!("three"),
        _ => println!("anything"),
    }

    // You can introduce match guards as part of a match arm
    // by specifying an additional if conditional after the pattern.
    for i in 1..101 {
      match i {
        i if (i % 15 == 0) => println!("FizzBuzz"),
        i if (i % 3 == 0) => println!("Fizz"),
        i if (i % 5 == 0) => println!("Buzz"),
        _ => println!("{}", i),
      }
    }
```

## References & Borrowing

```rust
  // & in front of str means we need a borrowed (referenced) value
  // Instead of the content, we need the pointer
  fn say_hello(to_whom: &str) {
    println!("Hey {}", to_whom)
  }

  // & means borrowing / referenced
  let mut a = 1;
  let b = &mut a;
  *b += 1; // * let's dereference b to content of 'a'
  println!("{}", b);

  // We cannot use 'a' while the content is mutably borrowed
  // Lifetime governs when b return content to 'a'
  println!("{}", a); // this will result in error because a is mutably borrowed

  // 'a' variable can only mutably borrowed once at a time
  let c = &mut a; // this will result in error, because 'a' is still borrowed

  // But borrowing immutably can be done infinitely
  let x = 1;
  let y = &x;
  let z = &x;
  println!("{} {} {}", x, y, z)
```

## Struct

```rust
    struct FootballPlayer {
        name: String,
        age: i32,
        goals: i32,
    }

    // Creating instance of struct
    let shevchenko = FootballPlayer {name: "Shevchenko".to_string(), umur: 30, gol: 999};

    // Implementing functions specifically for struct
    impl FootballPlayer {
        fn new(name: &str, age: i32, goals: i32) -> FootballPlayer {
            FootballPlayer {
                name: name.to_string(),
                age: age,
                goals: goals
            }
        }

        fn get_goals(&self) -> i32 {
            self.goals
        }
    }
```

## Macro

```rust
    // function ended with a bang is macro
    println!("Hello, world!");

```

### Vector

```rust
    // vector is array that can grow or shrink
    // we can create vector using vec! macro
    let mut seq = vec![1, 2, 3];

    // new content can be pushed
    seq.push(5);
```

## References

- https://github.com/froyoframework/rust-intro
