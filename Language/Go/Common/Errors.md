
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
type CustomError struct {
	string msg
}

func (customErr *CustomError) Error string {
	return customErr.msg
}

// Implictly created error
func ErrRtn() {
	return errors.New("err msg")
} 
```
