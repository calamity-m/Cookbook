
# Bat Alias
```bash
alias bathelp='bat --plain --language=help'
help() {
    "$@" --help 2>&1 | bathelp
}
# use like k9s help
```

# jwt func decode

```bash
decode-jwt() {
  echo $1 | jq -R 'gsub("-";"+") | gsub("_";"/") | split(".") | .[1] | @base64d | fromjson'
}
# use like, decode-jwt $token
```
