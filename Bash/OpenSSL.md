
# Inspect certificate in pem

```bash
openssl x509 -in cert.pem -noout -text
```

# Inspect all bundled certificates in pem

```bash
while openssl x509 -noout -text; do :; done < cert-bundle.pem
```

# PEM to CRT

```bash
openssl x509 -outform der -in your-cert.pem -out your-cert.crt
```

# Remove pass on key

```bash
openssl rsa -in original.key -out new.key
```

# Cert One Liner

```bash
openssl req -new -newkey rsa:4096 -days 365 -nodes -x509 -CA ca.crt -CAkey ca.key     -subj "/CN=cn/O=org/OU=ou" -keyout cert.key  -out cert.crt
```
