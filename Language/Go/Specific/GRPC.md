
# Run with Graceful Shutdown

```go
// Runs the GRPC server until notify is pushed to. You can wait
// on the returned channel for an exit code to account for graceful
// shutdowns.
func (s *CentralServiceServer) Run(notify <-chan os.Signal) <-chan error {
	// Channel we'll use to signal for finish
	exit := make(chan error)

	// create the grpc server that we can later serve on
	grpcServer := grpc.NewServer(s.config.GrpcServerOpts...)

	// register ourselves
	centralproto.RegisterCentralServiceServer(grpcServer, s)
	centralproto.RegisterCentralFoodServiceServer(grpcServer, s)

	if s.config.Reflect {
		reflection.Register(grpcServer)
	}

	go func() {
		// At the end of our function. If no errors were otherwise
		// pushed to this channel, it notifies as a successful shutdown
		defer close(exit)

		// Block until we receive a Interrupt or Kill
		<-notify

		grpcServer.GracefulStop()
	}()

	listener, err := net.Listen("tcp", s.config.Address)
	if err != nil {
		s.logger.Error(fmt.Sprintf("failed to listen on: %q", s.config.Address), slog.Any("err", err))
	}

	s.logger.Info(fmt.Sprintf("Starting server on %s", s.config.Address))
	if err := grpcServer.Serve(listener); err != nil {
		exit <- fmt.Errorf("failed to start/close server due to: %w", err)
	}

	return exit
}
```

```go
sig := make(chan os.Signal, 2)
signal.Notify(sig, syscall.SIGINT, syscall.SIGTERM)

//  Run the server
exit := server.Run(sig)
```

# GRPC Custom Error

```go
func ErrGRPCInvalidArgumentFromError(err error) error {
	return &errGRPCInvalidArgument{err.Error()}
}

func ErrGRPCInvalidArgument(msg string) error {
	return &errGRPCInvalidArgument{msg}
}

func ErrGRPCInvalidArgumentf(format string, a ...any) error {
	return &errGRPCInvalidArgument{fmt.Sprintf(format, a...)}
}

type errGRPCInvalidArgument struct {
	msg string
}

func (e *errGRPCInvalidArgument) Error() string {
	return e.msg
}

func (e *errGRPCInvalidArgument) GRPCStatus() *status.Status {
	return status.New(codes.InvalidArgument, fmt.Sprintf("invalid argument - %s", e.msg))
}

// used like...
// return nil, ErrGRPCInvalidArgument("no entry provided")
```