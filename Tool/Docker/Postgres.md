
```yaml
services:
  db:
    container_name: postgres
    image: postgres
    restart: always
    ports:
      - 9999:5432
    volumes:
      - ./scripts/init-db.sh:/docker-entrypoint-initdb.d/init-db.sh:ro
    environment:
      POSTGRES_PASSWORD: password
```

```bash
#!/usr/bin/env bash
set -e

psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname "$POSTGRES_DB" <<-EOSQL
	CREATE DATABASE databasethingtocreate;
EOSQL
```