
# Testing HTTP requests

request calls can be faked/tested through Go by using the net/http/httptest package. 

## Get with no body

```GO
S := ... // omitted

// Request object
req := httptest.NewRequest(http.MethodGet, "/", nil)
// Response object for inspection
resp := httptest.NewRecorder()
// Send the request through our server handler
S.ServeHTTP(resp, req)
// Validate the response through .Result()
if resp.Result().StatusCode != 404 {
	// In this example we want a 404
	t.Errorf("wanted 404")
}
```

## Get with a string body

```go
req := httptest.NewRequest(http.MethodGet, "/", strings.NewReader(`{"json":"yes}`))
```

## Get with a struct body as JSON

```go
mystruct := &MyStruct{}
var buf bytes.Buffer
err := json.NewEncoder(&buf).Encode(mystruct)
if err != nil {
	...
}
request := httptest.NewRequest(..., ..., &buf)

```

## Check response headers

```go
if response.Result().Header.Get(middleware.RequestIDHeader) == "" {
	t.Errorf(...)
}
```

# Test Case Array

A somewhat common paradigm in go is property based tests, below is a generic example of how you can do easily apply this.

```go
cases := []struct {
	Name string
	Check bool
}{
	{Value: "first", Check: true},
	{Value: "second", Check: false},
}

for _, test := range cases {
	t.Run(test.Name, func (t *testing.T {
		if test.Check {
			...
		}
	}))
}
```
