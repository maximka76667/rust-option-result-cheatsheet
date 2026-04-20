# Rust Error Handling — Quick Reference

## At A Glance

```
CONVERTING
Option → Result    .ok_or(e)           None     → Err(e)
                   .ok_or_else(|| e)   None     → Err(computed lazily)
Result → Option    .ok()               Err(_)   → None  (drops error)
                   .err()              Ok(_)    → None  (drops value)

TRANSFORMING
value inside       .map(|v| ...)       Some/Ok  → transform value
error inside       .map_err(|e| ...)   Err      → transform error   (Result only)

CHAINING
next step          .and_then(|v| ...)  Some/Ok  → run closure (returns Option/Result)
fallback           .or(other)          None/Err → return other
fallback lazy      .or_else(|| ...)    None/Err → run closure

EXTRACTING
panic if missing   .unwrap()           None/Err → panic
panic + message    .expect("why")      None/Err → panic with message
use default        .unwrap_or(v)       None/Err → return v
default lazy       .unwrap_or_else(||) None/Err → compute default
zero value         .unwrap_or_default() None/Err → Default::default()
early return       ?                   None/Err → return from function

CHECKING
bool only          .is_some()  .is_none()
                   .is_ok()    .is_err()
```

---

## map vs and_then

```rust
// closure returns plain value → use .map
Some("hello").map(|s| s.len())           // Some(5)

// closure returns Option/Result → use .and_then
Some("5").and_then(|s| s.parse::<i32>().ok())  // Some(5), not Some(Some(5))
```

---

## unwrap vs expect vs ?

```rust
// prototype / certain it exists / tests
thing.unwrap()

// certain it exists — always add a message
thing.expect("thing must exist because X")

// propagating errors in real code — idiomatic
fn my_fn() -> Result<T, E> {
    let val = thing?;  // returns early if Err
    Ok(val)
}
```

---

## Common Patterns

**Chain that stops at first failure:**

```rust
request
    .headers()
    .get("X-API-Key")              // Option<&HeaderValue>
    .and_then(|v| v.to_str().ok()) // Option<&str>
    .map(|key| key == API_KEY)     // Option<bool>
    .unwrap_or(false)              // bool
```

**Convert error type:**

```rust
std::fs::read_to_string(path)
    .map_err(|e| AppError::Io(e))
```

**Default if env var missing or unparseable:**

```rust
std::env::var("PORT")
    .ok()
    .and_then(|s| s.parse().ok())
    .unwrap_or(8080)
```

**Early return chain with ?:**

```rust
fn handler() -> Result<Response, AppError> {
    let body = read_body()?;
    let parsed = parse_json(&body)?;
    let result = process(parsed)?;
    Ok(result.into_response())
}
```

---

## Files In This Directory

- `converting.md` — Option ↔ Result conversion
- `transforming.md` — map, map_err
- `chaining.md` — and_then, or, or_else
- `extracting.md` — unwrap family, ?, is_ok/is_some
