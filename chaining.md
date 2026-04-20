# Chaining Operations

## `.and_then(|v| ...)`

If `Some`/`Ok` — runs closure and returns its result. If `None`/`Err` — passes through unchanged. Closure must return `Option` or `Result`.

```rust
// Option
Some(5).and_then(|x| if x > 3 { Some(x) } else { None })  // Some(5)
Some(2).and_then(|x| if x > 3 { Some(x) } else { None })  // None
None::<i32>.and_then(|x| Some(x * 2))                     // None — never called

// Result
Ok(5).and_then(|x| if x > 3 { Ok(x) } else { Err("too small") })  // Ok(5)
Err::<i32, &str>("oops").and_then(|x| Ok(x * 2))                  // Err("oops")
```

**When to use:** Chaining operations where each step might fail. This is the core of error propagation chains.

```rust
// each step might return None — chain stops at first failure
fn get_user_city(db: &Db, user_id: i32) -> Option<String> {
    db.find_user(user_id)          // Option<User>
        .and_then(|u| u.address)   // Option<Address>
        .and_then(|a| a.city)      // Option<String>
}

// real world — your header parsing
request
    .headers()
    .get("X-API-Key")              // Option<&HeaderValue>
    .and_then(|v| v.to_str().ok()) // Option<&str>
```

---

## `.or(other)`

If `Some`/`Ok` — returns self unchanged. If `None`/`Err` — returns `other`.

```rust
Some(5).or(Some(99))          // Some(5) — already Some
None::<i32>.or(Some(99))      // Some(99) — fallback used

Ok::<i32, &str>(5).or(Ok(99))      // Ok(5)
Err::<i32, &str>("oops").or(Ok(99)) // Ok(99)
```

**When to use:** Providing a fallback value at the same type level.

```rust
// try env var, fall back to default
let log_level = std::env::var("LOG_LEVEL").ok()
    .or(Some("info".to_string()))
    .unwrap();

// try primary config, fall back to secondary
let config = load_config("config.toml")
    .or(load_config("config.default.toml"));
```

---

## `.or_else(|| ...)`

Same as `.or` but the fallback is computed lazily.

```rust
None::<i32>.or_else(|| Some(expensive_default()))  // closure only called if None
Some(5).or_else(|| Some(expensive_default()))      // Some(5) — closure never called
```

**When to use:** Same as `.or` but when the fallback is expensive to compute.

```rust
fn get_api_key() -> Option<String> {
    // try in order — each only evaluated if previous is None
    read_from_env()
        .or_else(|| read_from_file())
        .or_else(|| read_from_keychain())
}
```

---

## `.and(other)`

If `Some`/`Ok` — returns `other`. If `None`/`Err` — returns self. Less common.

```rust
Some(5).and(Some(99))     // Some(99) — discards 5, returns other
None::<i32>.and(Some(99)) // None — short circuits
```

**When to use:** When you care that something succeeded but want to return a different value.

```rust
// validate then return something else
validate_input(&data)   // Result<(), Error>
    .and(Ok(processed)) // if valid, return processed value
```

---

## Chaining Summary

```
.and_then  — next step that might fail (most common)
.or_else   — fallback that might fail
.or        — simple fallback value
.and       — discard value, return something else
```

The most important pattern — a chain where each step might fail:

```rust
// reads like: get header, parse it as string, check it matches
let is_valid = request
    .headers()
    .get("X-API-Key")              // might be None
    .and_then(|v| v.to_str().ok()) // might be None
    .map(|key| key == API_KEY)     // transform if Some
    .unwrap_or(false);             // default if any step was None
```
