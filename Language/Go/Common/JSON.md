
# Unmarshal

```go
var arg T
err := json.Unmarshal([]byte(inStr), &arg)
if err != nil {
    return nil, err
}
```

# Marshal

```go
j, err := json.Marshal(in)
if err != nil {
    return nil, err
}
```

# Encode

```go
func EncodeJSON[T any](val T) (string, error) {

	w := bytes.Buffer{}
	if err := json.NewEncoder(&w).Encode(val); err != nil {
		return "", fmt.Errorf("failed encoding json: %w", err)
	}

	return w.String(), nil
}
```

# Decode

```go
func DecodeJSON[T any](barray []byte) (T, error) {
	var val T

	if err := json.NewDecoder(bytes.NewReader(barray)).Decode(&val); err != nil {
		return val, fmt.Errorf("failed decoding json: %w", err)
	}

	return val, nil
}
```

# Encode VS Marshal

https://articles.wesionary.team/difference-of-json-encoding-vs-marshaling-and-json-decoding-vs-unmarshaling-1a6baf6a7f5c