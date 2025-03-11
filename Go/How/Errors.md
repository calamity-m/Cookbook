
# Check Error Type

```go
if errors.Is(err, errs.ErrInvalidRequest) {
	fmt.Println("hello! :)")
}
```

# Create New Error

```go
var (
	ErrThing = errors.New("thing")
)
```
