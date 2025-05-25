# Turn TLS verification off with default client

```go
http.DefaultTransport.(*http.Transport).TLSClientConfig = &tls.Config{InsecureSkipVerify: true}
```

# Turn off TLS verification with custom client

```go
cliient := &http.Client{
	Transport: &http.Transport{
		TLSClientConfig: &tls.Config{
			InsecureSkipVerify: true,
		},
	},
}
```

# HTTP Client with TLS

```go
func newHttpClient(cfg *config.Config) (*http.Client, error) {
    client := &http.Client{}
    tlsClientConfig := &tls.Config{}

    if cfg.Client.CertificatePath != "" {
        cert, err := tls.LoadX509KeyPair(cfg.Client.CertificatePath, cfg.Client.CertificateKeyPath)
        if err != nil {
            return nil, err
        }

        tlsClientConfig.Certificates = []tls.Certificate{cert}
    }

    if cfg.CACertificatePath != "" {
        ca, err := os.ReadFile(cfg.CACertificatePath)
        if err != nil {
            return nil, err
        }

        pool, err := x509.SystemCertPool()
        if err != nil {
            return nil, err
        }

        ok := pool.AppendCertsFromPEM(ca)
        if !ok {
            return nil, fmt.Errorf("failed to append ca to cert pool")
        }

        tlsClientConfig.RootCAs = pool
    }

    tlsClientConfig.InsecureSkipVerify = cfg.Auth.VerificationSkip

    client.Transport = &http.Transport{TLSClientConfig: tlsClientConfig}

    return client, nil
}
```

# URL Encoded Parameter Body

```go
body := url.Values{
    "param":      []string{"value},
}

req, err := http.NewRequest(http.MethodPost, url, strings.NewReader(body.Encode()))
```
