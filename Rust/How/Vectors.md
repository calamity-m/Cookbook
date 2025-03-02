
# Create new vector

```rust
let v: Vec<i32> = Vec::new();
```

# Init new vector

```rust
let v = vec![10, 20, 30, 40];
```

# Add to vector

```rust
let mut v = Vec::new();

v.push(1);

```

# Access vector

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
println!("The third element is {third}");

let third: Option<&i32> = v.get(2);
match third {
    Some(third) => println!("The third element is {third}"),
    None => println!("There is no third element."),
}
```

# Iterate over a vector with mutable

```rust
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

# Vector of enums

```rust
enum SpreadsheetCell { 
	Int(i32), 
	Float(f64), 
	Text(String), 
} 

let row = vec![ 
	SpreadsheetCell::Int(3),
	SpreadsheetCell::Text(String::from("blue")),
	SpreadsheetCell::Float(10.12), 
];
```