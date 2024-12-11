[[Dreambook/Void/Snippets|Snippets]]

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


# String Formatting

```go
// Format a string and print it to the terminal
fmt.Printf("str: %s\n", "stuff")

// Format a string and assign it to a variable
formatted := fmt.SPrintf("%str: %s\n", "stuff")
```

## Formats

```
%s -> string
%q -> string wrapped with quotes
%d -> base 10 integer
%T -> type of the object
%t -> boolean
%f -> floating point
%p -> pointer
%v -> value in its default format
%+v -> include field names
%#v -> include field names and type
```


# Build and Run One Liner

```bash
go build -o bin/ && bin/hello
```

