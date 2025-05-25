
# Update Go Modules

```go
go get -u
go mod tidy
```

# Create A New Module

```bash
go mod init github.com/calamity-m/my/repo/pkg/path/for/module
```

# Private Modules

```bash
go env -w GOPRIVATE=github.com/path/to/private/repo
```

# Simple Monorepo Structure

```
├── go.mod
├── services
│   ├── sow
│   │   └── main.go
│   └── till
│       └── main.go
└── shared
    └── go
        └── auth
            └── main.go
```

# Importing Monorepo package contents

```go
// Contents of go.mod:
module github.com/calamity-m/reap

go 1.23.3
```

```go
// File wanting to use the shared code
import "github.com/calamity-m/reap/shared/go/auth"
```

```go
// File with shared code:
package auth

func Auth() string {
	return ""
}
```