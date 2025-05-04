
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
