
# Option

```rust
let query = match args.next() {
    Some(arg) => arg,
    None => return Err("Didn't get a query string"),
};
```