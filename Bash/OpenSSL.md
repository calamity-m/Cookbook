
# Inspect certificate in pem

```bash
openssl x509 -in cert.pem -noout -text
```

# Inspect all bundled certificates in pem

```bash
while openssl x509 -noout -text; do :; done < cert-bundle.pem
```

