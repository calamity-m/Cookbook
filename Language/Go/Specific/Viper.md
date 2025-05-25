
# Viper Setup

```go
type Config struct {
	// Logging Configuration
	Environment   string     `mapstructure:"environment" json:"environment,omitempty"`
	LogLevel      slog.Level `mapstructure:"log_level" json:"log_level,omitempty"`
	Spicy string `mapstructure:"spicy" json:"spicy,omitempty"`

	// Not filled out by viper defaults
	GrpcServerOpts []grpc.ServerOption
}

func NewConfig(debug bool) (*Config, error) {
	// Setup
	base := &Config{}
	vip := viper.New()

	// Enable ENV var reading
	vip.AutomaticEnv()
	vip.SetEnvPrefix("PREFIX")
	vip.SetEnvKeyReplacer(strings.NewReplacer("-", "_"))

	// Establish sane defaults before overriding
	vip.SetDefault("environment", "dev")
	vip.SetDefault("log_level", slog.LevelDebug)

	// Bind env var that has no default
	if err := vip.BindEnv("spicy"); err != nil {
		return &Config{}, err
	}
	
	// Magic to unamrshal viper into the config sturct. The decode hook is used to map things like the logging level
	// into the slog logging level type.
	if err := vip.Unmarshal(&base, viper.DecodeHook(mapstructure.TextUnmarshallerHookFunc())); err != nil {
		return &Config{}, err
	}

	// Forecefully override if we're on debug mode
	// Do this without viper/cobra buy-in and just do the simple
	// brute force
	if debug {
		base.LogLevel = slog.LevelDebug
	}

	return base, nil
}
```
