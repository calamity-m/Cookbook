
# Get first capture group

```go
cat file | awk '/search:stuff/{print $0; exit}'
```

# Easy Helm Version Extraction

```bash
cat Chart.yaml | awk '/version:/{print $2; exit}'
```