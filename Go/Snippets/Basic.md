
# Hello World

```go
package main

import "fmt"

func helloWorld() string {
	return "Hello World"
}

func main() {
	hello := helloWorld()
	fmt.Println(hello + " Wow!? Again! " + hello2)
}
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
