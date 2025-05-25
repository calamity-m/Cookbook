
```go
// This is the function we use to create a configured
// something. In this case, a LoggingHandler.
func New(opts ...func(*LoggingHandler)) *LoggingHandler {
	handler := &LoggingHandler{}

	// Apply the functions we supply to the handler
	for _, fn := range opts {
		fn(handler)
	}

	return handler
}

// Configure a LoggingHandler with a specific environment
func WithEnvironment(env string) func(*LoggingHandler) {
	return func(handler *LoggingHandler) {
		handler.Environment = env
	}
}


func A() *LoggingHandler {
	handler := New(
		// Pass the function itself, which mutates the
		// handler
		WithEnvironment("dev"),
	)

	return handler
}
```