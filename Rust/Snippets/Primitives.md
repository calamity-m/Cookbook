# Fixed array size initialization

```rust
let xs: [i32; 5] = [1, 2, 3, 4, 5];
```

# Fixed array size with repeated initalization

```rust
let ys: [i32; 500] = [55; 500];
```

# Slicing array

```rust
let a = [1, 2, 3, 4, 5];
let nice_slice = &a[1..4];
```

# Tuple deconstruction

```rust
let cat = ("Furry McFurson", 3.5);
let (name, age) = cat;
```

# Tuple access

```rust
let numbers = (1, 2, 3);
let second = numbers.1;
```