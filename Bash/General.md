
# Mute Return Code

```bash
failed_cmd 2>/dev/null || true
```

# Unzip recursively

```bash
find . -name "*.zip" | while read fn; do unzip -o -d "`dirname "$fn"`" "$fn"; done;
```
