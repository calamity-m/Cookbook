
# Read file contents

```rust
let contents = fs::read_to_string("filepath.txt")?;
```

# Iterate file contents

```rust
contents
    .lines() // get iterator
    .filter(|line| line.contains(query)) // filter
    .collect() // collect into vec
```