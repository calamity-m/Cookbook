
# Echo container with mTLS

```bash
docker run -p 8080:8080 -p 8443:8443 -v /path-to-server.pem:/app/fullchain.pem:z -v /path-to-server-key-unencrypted.key:/app/privkey.pem:z --rm -t mendhak/http-https-echo:36
```
