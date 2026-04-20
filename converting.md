# Converting Between Option and Result

## Option → Result

### `.ok_or(err)`

Converts `Some(v)` to `Ok(v)`, `None` to `Err(err)`.

```rust
let x: Option<i32> = Some(5);
x.ok_or("missing")   // Ok(5)

let x: Option<i32> = None;
x.ok_or("missing")   // Err("missing")
```

**When to use:** You have an `Option` but your function returns `Result`. The error value is cheap to construct.

```rust
fn get_user_id(map: &HashMap<&str, i32>, name: &str) -> Result<i32, String> {
    map.get(name)
        .copied()
        .ok_or(format!("user {} not found", name))
}
```

---

### `.ok_or_else(|| err)`

Same as `ok_or` but the error is computed lazily — only constructed if `None`.

```rust
let x: Option<i32> = None;
x.ok_or_else(|| expensive_error_message())  // closure only called if None
```

**When to use:** Same as `ok_or` but when the error value is expensive to construct (heap allocation, formatting, etc). Prefer this over `ok_or` when in doubt.

```rust
fn find_config(path: Option<&str>) -> Result<Config, String> {
    path.ok_or_else(|| format!("no config path set, checked {} locations", count_locations()))
}
```

---

## Result → Option

### `.ok()`

Converts `Ok(v)` to `Some(v)`, `Err(_)` to `None`. **Silently drops the error.**

```rust
let x: Result<i32, &str> = Ok(5);
x.ok()               // Some(5)

let x: Result<i32, &str> = Err("oops");
x.ok()               // None — error gone
```

**When to use:** You don't care why something failed, only whether it succeeded.

```rust
// don't care why parsing failed, just want Some or None
let port: Option<u16> = std::env::var("PORT")
    .ok()                          // don't care if var missing
    .and_then(|s| s.parse().ok()); // don't care if parse fails
```

---

### `.err()`

Converts `Ok(_)` to `None`, `Err(e)` to `Some(e)`. **Silently drops the value.**

```rust
let x: Result<i32, &str> = Ok(5);
x.err()    // None

let x: Result<i32, &str> = Err("oops");
x.err()    // Some("oops")
```

**When to use:** Rare. When you specifically want to work with the error and discard the success value. More common in tests.

```rust
// assert the specific error type in tests
assert_eq!(parse_config("bad input").err(), Some(ConfigError::InvalidFormat));
```
