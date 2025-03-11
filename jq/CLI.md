# Decode JWT

```bash
cat $TOKEN | jq -r .access_token | jq -R 'split(".") | .[1] | @base64d | fromjson'
```

# Tail a file and run JQ on each line

```bash
tail -f file.json | jq .
```

# JSON to single line

```bash
jq -c
```

# JSON URL escape/quoted

``` bash
jq -R
```