
# Opts Configuration

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

# Serializing Nested Structs

```go
type Config struct {
	Env              string `mapstructure:"env" json:"environment,omitempty"`
	Server           server
}

type server struct {
	Host string `mapstructure:"host" json:"host,omitempty"`
	Port int    `mapstructure:"port" json:"port,omitempty"`
}

// End up with a serialization with a key path for
// env, server.host and server.port
```



