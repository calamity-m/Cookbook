
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

# Custom Error

```go
// Anything that implements the following interface is considered an "Error"
// type error interface {
//	Error() string
//}

// Simple explicitly defined error
type ErrCustom struct {
	msg string
}

func (e *ErrCustom) Error() string {
	return e.msg
}
```
