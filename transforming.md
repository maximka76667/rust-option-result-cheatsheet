# Transforming Values Inside Option and Result

## `.map(|v| ...)`

Transforms the inner value if `Some`/`Ok`, passes `None`/`Err` through unchanged. Never changes the outer type.

```rust
// Option
Some(5).map(|x| x * 2)           // Some(10)
None::<i32>.map(|x| x * 2)       // None — closure never called

// Result
Ok(5).map(|x| x * 2)             // Ok(10)
Err("oops").map(|x: i32| x * 2)  // Err("oops") — closure never called
```

**When to use:** You want to transform a success value without touching the error/None case.

```rust
// transform the value if it exists
fn get_username(id: i32) -> Option<String> {
    find_user(id).map(|user| user.name.to_uppercase())
}

// parse then transform
let doubled: Result<i32, _> = "5".parse::<i32>().map(|n| n * 2);  // Ok(10)
```

---

## `.map_err(|e| ...)`

Transforms the error value if `Err`, passes `Ok` through unchanged. **Result only.**

```rust
Ok::<i32, &str>(5).map_err(|e| format!("error: {}", e))       // Ok(5) — untouched
Err::<i32, &str>("oops").map_err(|e| format!("error: {}", e)) // Err("error: oops")
```

**When to use:** Converting between error types, or adding context to an error.

```rust
// convert library error to your own error type
fn read_config(path: &str) -> Result<Config, AppError> {
    std::fs::read_to_string(path)
        .map_err(|e| AppError::IoError(e))  // io::Error → AppError
        .and_then(|s| parse_config(&s))
}

// add context to an error
fn connect(url: &str) -> Result<Connection, String> {
    raw_connect(url)
        .map_err(|e| format!("failed to connect to {}: {}", url, e))
}
```

---

## Combining `.map` and `.map_err`

These two together let you transform both sides independently:

```rust
parse_input(s)
    .map(|v| v * 2)                          // transform success
    .map_err(|e| AppError::ParseError(e))    // transform error
```

---

## `.map` vs `.and_then`

Easy to confuse — the difference is what the closure returns:

```rust
// .map — closure returns a plain value, gets wrapped automatically
Some(5).map(|x| x * 2)           // closure returns i32, result is Some(i32)

// .and_then — closure returns Option/Result, no extra wrapping
Some(5).and_then(|x| Some(x * 2)) // closure returns Option<i32>
// avoids Some(Some(10))

// if you use .map when you should use .and_then:
Some("5").map(|s| s.parse::<i32>().ok())  // Some(Some(5)) — nested!
Some("5").and_then(|s| s.parse::<i32>().ok())  // Some(5) — flat
```

**Rule:** If your closure returns a plain value, use `.map`. If it returns `Option` or `Result`, use `.and_then`.
