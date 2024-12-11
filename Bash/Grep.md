
# Delete lines in a file containing a string

```bash
grep -vi "pattern" file.json > file2.json

# Inline replace
grep -vi "pattern" file1.json > tmp && mv tmp file1.json
```

# Find 