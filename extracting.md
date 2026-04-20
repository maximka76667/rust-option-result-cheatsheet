# Extracting Values

## `.unwrap()`

Returns the inner value. **Panics if `None` or `Err`.**

```rust
Some(5).unwrap()              // 5
None::<i32>.unwrap()          // panic: called `Option::unwrap()` on a `None` value

Ok::<i32, &str>(5).unwrap()   // 5
Err::<i32, &str>("oops").unwrap() // panic: called `Result::unwrap()` on an `Err` value: "oops"
```

**When to use:** Only when you are certain the value exists and a panic would indicate a programming error, not a runtime condition. Common in tests and prototypes.

```rust
// fine — you know this can't fail
let re = Regex::new(r"^\d+$").unwrap(); // regex is hardcoded, always valid

// fine in tests
let result = parse_config(valid_input).unwrap();
assert_eq!(result.port, 8080);

// bad — this can legitimately fail at runtime
let user = db.find_user(id).unwrap(); // panics if user doesn't exist
```

---

## `.expect("message")`

Same as `.unwrap()` but with a custom panic message. **Always prefer over `.unwrap()`.**

```rust
None::<i32>.expect("user id should always be present after auth")
// panic: user id should always be present after auth
```

**When to use:** Wherever you'd use `.unwrap()`. The message explains why you expected this to succeed — makes debugging much faster.

```rust
// good — message explains the invariant
let config = load_config().expect("config file must exist — run setup first");
let port = env::var("PORT").expect("PORT env var must be set");

// bad — no information
let config = load_config().unwrap();
```

---

## `.unwrap_or(default)`

Returns inner value if `Some`/`Ok`, returns `default` if `None`/`Err`.

```rust
Some(5).unwrap_or(0)          // 5
None::<i32>.unwrap_or(0)      // 0

Ok::<i32, &str>(5).unwrap_or(0)       // 5
Err::<i32, &str>("oops").unwrap_or(0) // 0 — error silently dropped
```

**When to use:** When you have a sensible default and don't need to handle the error case specially.

```rust
let timeout = env::var("TIMEOUT")
    .ok()
    .and_then(|s| s.parse().ok())
    .unwrap_or(30);  // default 30 seconds

let name = user.nickname.unwrap_or("anonymous".to_string());
```

---

## `.unwrap_or_else(|| ...)`

Same as `unwrap_or` but default is computed lazily.

```rust
None::<String>.unwrap_or_else(|| expensive_default_string())
// closure only called if None
```

**When to use:** When the default is expensive to compute (heap allocation, function call, etc).

```rust
let greeting = find_custom_greeting(user_id)
    .unwrap_or_else(|| format!("Hello, {}!", user.name));
// format! only called if no custom greeting found
```

---

## `.unwrap_or_default()`

Returns inner value if `Some`/`Ok`, returns `Default::default()` for the type if not.

```rust
None::<i32>.unwrap_or_default()     // 0    (i32 default)
None::<String>.unwrap_or_default()  // ""   (String default)
None::<Vec<i32>>.unwrap_or_default() // []  (Vec default)
None::<bool>.unwrap_or_default()    // false (bool default)
```

**When to use:** When the zero value of the type is a sensible default.

```rust
let tags: Vec<String> = user.tags.unwrap_or_default(); // empty vec if no tags
let count: i32 = get_count().unwrap_or_default();      // 0 if failed
```

---

## Checking Without Extracting

```rust
// returns bool — does not consume the value
option.is_some()   // true if Some
option.is_none()   // true if None

result.is_ok()     // true if Ok
result.is_err()    // true if Err
```

**When to use:** When you only need to know if a value exists, not what it is. Common in conditions and assertions.

```rust
if response.headers().get("X-Cache").is_some() {
    record_cache_hit();
}

assert!(parse_config(valid_input).is_ok());
assert!(parse_config("bad").is_err());
```

---

## The `?` Operator — Most Important

Inside a function returning `Result` or `Option` — returns early on `Err`/`None`, unwraps on `Ok`/`Some`.

```rust
// without ?
fn read_and_parse(path: &str) -> Result<Config, AppError> {
    let contents = match std::fs::read_to_string(path) {
        Ok(s) => s,
        Err(e) => return Err(AppError::Io(e)),
    };
    let config = match parse(&contents) {
        Ok(c) => c,
        Err(e) => return Err(AppError::Parse(e)),
    };
    Ok(config)
}

// with ? — identical behaviour
fn read_and_parse(path: &str) -> Result<Config, AppError> {
    let contents = std::fs::read_to_string(path)?;  // returns early if Err
    let config = parse(&contents)?;                  // returns early if Err
    Ok(config)
}
```

`?` also calls `.into()` on the error — so errors convert automatically if `From` is implemented:

```rust
// io::Error automatically converts to AppError
// if AppError implements From<io::Error>
fn read_file(path: &str) -> Result<String, AppError> {
    Ok(std::fs::read_to_string(path)?)  // io::Error → AppError via From
}
```

**When to use:** Almost always, inside any function that returns `Result` or `Option`. It is the idiomatic Rust way to propagate errors.

---

## Quick Decision Guide

```
need to panic if missing?      → .expect("why it should exist")
have a default value?          → .unwrap_or(default)
default is expensive?          → .unwrap_or_else(|| compute())
default is the zero value?     → .unwrap_or_default()
just checking existence?       → .is_some() / .is_ok()
propagating in a function?     → ?
```
