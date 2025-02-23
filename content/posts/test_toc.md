---
title: testing TOC
id: toc
aliases: []
tags: []
---

Rust, unlike other languages (using exceptions), handles errors using
`Result<T, E>` enum.

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

The `Result` enum has two variants: `Ok` which has a generic type of `T`, and
`Err` carrying a generic type of E. For instance, the `Result` enum below
returns a boolean type value in a success scenario, while a `String`
type value is returned when error happens.

```rust
enum Result<bool, String> {
    Ok(bool),
    Err(String),
}
```

## H2

If we try to open a file, the `std::fs::File::open` function returns a
`Result<std::fs::File, std::io::Error>`.

```rust
use std::fs::File;

let f = match File::open("somefile.txt") {
    Ok(file) => file,
    Err(error) => panic!("Failed to open the file with error {}", error),
}
```

I can specify what type of value to return for my own function.

`Result<T, E>` enum is a generalization of `Option<T>` enum, and the latter is
a special case of the former. Instead of return a generic error type, it sends
back a `None` type when error happens.

```rust
enum Option<T> {
    Some(T),
    None,
}
```

## Another H1

- Errors that would terminate my program
  - Accessing out-of-bound index of an array
  - Although some may argue file `NotFound` error is recoverable, it depnds on
    the program. I think I can still terminate if a required input file is
    missing.
- Use the `panic!` macro to handle unrecoverable error

    ```rust
    fn main() {
        panic!("I panic for no good reason");
    }
    ```

- Post panic event, my program starts to unwinding by cleaning up any data
  in the stack, a process that could take some time. Rust offers an alternative
  way to immediately abort my program. The memory my program used will have to
  be cleaned up by OS. Add the following to my `Cargo.toml` file:

  ```toml
  [profile.release]
  panic = "abort"
  ```

- I can set `RUST_BACKTRACE=1` to see the backtrack up to the panic point. I
  can also run `cargo run` with backtrace enabled:

  ```bash
  RUST_BACKTRACE=1 cargo run
  ```

## Recoverable error

- Capture errors using `Result<T, E>` enum

- Propagate error from function being called to caller
  - Requires the function return either a `Result<T, E>` or `Option` enum.
    The code below fails:
  
  ```rust
    use std::fs::File;

    fn do_something() -> i32 {
        let f = File::open("somefile.txt")?;

        let mut s = String::new();

        f.read_to_string(&mut s)?;

        s.parse::<i32>()?
    }
  ```

    Simply modify return type of the function, or use `match` expression to
    handle both `Ok` and `Err` arms. I can use `?` operator in `main` function.
    I just need to return a `Result` type enum: `Result<(), Box<dyn, Error>>`,
    which it is beyond my understanding atm.

    I can use `?` operator on a `Result` in a function that returns `Result`
    type, and same for `Option`. However, I cannot mix them by calling `?`
    on an `Option` in a function that returns `Result`. In those cases, I can
    use `ok`, `ok_or`, `ok_or_else` to convert between `Result` and `Option`.

### Handling Error

- It is preferable to send back errors to the caller to handle it
- `?` operator
  - Using the `?` operator, we can extract the value `T` from `Ok(T)`
      or `Option<T>`, and propagate `Err(E)` and `Option<None>` value to the
      caller, respectively
  - On error, the `?` operator implicitly calls the `from` function in the
      `From` trait to convert the type of error value received to the one
      defined in the return type of the function. This is crucial because
      I need to think about what type of error I would like to return from
      my function, and the defined return type can be converted from type of
      error received in the `?` operator. This opens the necessity to define
      my error type and implement the `From` trait.

      The Rust compiler will not allow the following code to pass due the
      reason stated above:

      ```rust
      fn parse_num_from_str(num_in_str: &str) -> Result<u16, String> {
          num_in_str.parse::<u16>()?
      }
      ```

      On error, the `parse` function receives a `ParseIntError` type. However,
      the function returns a `String` type error. The `?` operator will fail
      to convert `ParseIntError` type into `String`, because the conversion is
      not supported in the implementation of the `from` function for the latter

      Simply changing the return error type in the function signature fixes
      the error.

      ```rust

      use std::num;

      fn parse_num_from_str(num_in_str: &str) -> Result<u16, num::ParseIntError> {
          num_in_str.parse::<u16>()?
      }
      ```

### When to use `Result<T, E>` enum

- When you want to quickly prototyping something, use the latter. I can
    then go back to places where we use `unwrap` and `expect` and capture
    errors using the former properly
- In test code, using `unwrap` and `expect` to call panic when test fail
- when you know something will not fail, a.k.a you know a bit more info than
    the compiler:

  ```rust
      use std::net::IpAddr;

      // literal '127.0.0.1'.parse() will be a success
      // unless we have a variable that asks input from users
      let ip: IpAddr = "127.0.0.1".parse().unwrap();

  ```
