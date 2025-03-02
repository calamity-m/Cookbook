# Custom HTTP Server Struct

```go
type Server struct {
	srv           http.Server
}

func (s *Server) ListenAndServe() error {
	return s.srv.ListenAndServe()
}
```

# Middleware Wrappers

## Wrapper

```go
func Wrap(middlewares ...func(http.Handler) http.Handler) func(http.Handler) http.Handler {

	// Return the expected outer signature, which is the function that can be applied to a handler
	return func(h http.Handler) http.Handler {
		for i := len(middlewares) - 1; i >= 0; i-- {
			// Wrap our middleware like a stack
			h = middlewares[i](h)
		}

		// Finally return our wrapped handler
		return h
	}
}
```

## Middleware

```go 
package middleware

import (
	"log/slog"
	"net/http"
	"time"
)

type loggingWriter struct {
	http.ResponseWriter
	statusCode int
}

// Supplant the inner http.ResponseWriter so we can spy on
// the status code being sent out with the response.
func (l *loggingWriter) WriteHeader(statusCode int) {
	l.ResponseWriter.WriteHeader(statusCode)
	l.statusCode = statusCode
}

// Logs the end of request action with tracing information, including
// the duration the request took.
func LoggingMiddleware(logger *slog.Logger) func(http.Handler) http.Handler {

	return func(h http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			start := time.Now()

			wrappedWriter := loggingWriter{ResponseWriter: w}

			h.ServeHTTP(&wrappedWriter, r)

			end := time.Since(start)

			logger.LogAttrs(
				r.Context(),
				slog.LevelInfo.Level(),
				"processed request",
				slog.Int("status", wrappedWriter.statusCode),
				slog.String("method", r.Method),
				slog.String("host", r.Host),
				slog.String("url", r.URL.Path),
				slog.String("agent", r.UserAgent()),
				slog.String("duration", end.String()),
			)
		})
	}
}
```

## Using Wrapper/Middleware

```go
baseMiddleware := middleware.Wrap(middleware.LoggingMiddleware(log))

// route specific
mux.Handle(fmt.Sprintf("POST %s", WEB_SOW_PATH), baseMiddleware(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		router.HandleSow(w, r)
	})))

// route agnostic
mux := http.NewServeMux()
mux = baseMiddleware(mux)
```

# Decode/Encode for REST API

```go

func EncodeJSON[T any](w http.ResponseWriter, status int, val T) error {
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(status)

	if err := json.NewEncoder(w).Encode(val); err != nil {
		return fmt.Errorf("failed encoding json: %w", err)
	}

	return nil
}

func DecodeJSON[T any](r *http.Request) (T, error) {
	var val T

	if err := json.NewDecoder(r.Body).Decode(&val); err != nil {
		return val, fmt.Errorf("failed decoding json: %w", err)
	}

	return val, nil
}
```

# Graceful Shutdown

```go
func (s *Server) Shutdown() error {

	ctx, cancel := context.WithTimeout(context.Background(), s.shutdownGrace*time.Second)
	defer cancel()

	if err := s.srv.Shutdown(ctx); err != nil {
		return fmt.Errorf("failed to shutdown gracefully: %w", err)
	}

	return nil
}

func (s *Server) Run() error {
	exit := make(chan error)

	go func() {
		// At the end of our function. If no errors were otherwise
		// pushed to this channel, it notifies as a successful shutdown
		defer close(exit)

		sig := make(chan os.Signal, 2)
		signal.Notify(sig, syscall.SIGINT, syscall.SIGTERM)

		// Block until we receive a Interrupt or Kill
		<-sig

		if err := s.Shutdown(); err != nil {
			// Pass error through for return
			exit <- err
			return
		}

	}()

	if err := s.ListenAndServe(); !errors.Is(err, http.ErrServerClosed) {
		return fmt.Errorf("failed to start/close server due to: %w", err)
	}


```