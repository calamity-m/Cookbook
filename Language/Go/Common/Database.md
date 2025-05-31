# Postgres
## PGX Insert Row (With ID returned)

```go
insertSql := `
	INSERT INTO entries
	(uuid, user_id, submitted_at)
	VALUES
	(@uuid, @user_id, @submitted_at)
	RETURNING id;
`
args := pgx.NamedArgs{
	"uuid":         entry.Uuid,
	"user_id":      entry.UserId,
	"submitted_at": pgtype.Timestamp{Time: entry.SubmittedAt, Valid: true},
}

var id int
err := pg.pool.QueryRow(ctx, insertSql, args).Scan(&id)
if err != nil {
	return 0, err
}
```
## PGX Scan One Row

```go
getSql := `
    SELECT id, uuid, name
    FROM entries WHERE uuid = @uuid; 
`

args := pgx.NamedArgs{
    "uuid": uuid,
}

rows, err := pg.pool.Query(ctx, getSql, args)
if err != nil {
    return Entry{}, err
}

entry, err := pgx.CollectExactlyOneRow(rows, pgx.RowToStructByName[Entry])
if err != nil {
    return Entry{}, err
}
```

## PGX Scan Many Rows

```go
getSql := `
	SELECT id, uuid, user_id, description, name, kilojules, grams, submitted_at
	FROM entries WHERE user_id = @user_id; 
`

args := pgx.NamedArgs{
	"user_id": userId,
}

rows, err := pg.pool.Query(ctx, getSql, args)
if err != nil {
	return nil, err
}

entries, err := pgx.CollectRows(rows, pgx.RowToStructByName[Entry])
if err != nil {
	return nil, err
}
```

## PGX Pool

```go
type PGStore struct {
   pool *pgxpool.Pool
}
...
pg := &PGStore{}
pool, err := pgxpool.New(context.Background(), fmt.Sprintf("postgres://%s:%s@%s/db", user, password, address))
if err != nil {
    return nil, err
}
pg.pool = pool
```

## PGX Health Check

```go
func (pg *PGStore) HealthCheck() error {
	if pg.pool == nil {
		return fmt.Errorf("postgres pool is nil - %w", shared.ErrNilNotAllowed)
	}

	if err := pg.pool.Ping(context.Background()); err != nil {
		return fmt.Errorf("failed healtcheck - %w", err)
	}

	return nil
}
```

## PGX Like Query With Named Arg

```go
// (@name::text IS NULL OR name ILIKE @name)
args := pgx.NamedArgs{
	"name": fmt.Sprintf("%%%s%%", filter.Name),
}
```

## PGX Dynamic Query

```go
fSql := `
SELECT
	id,
	uuid,
	user_id,
	submitted_at
FROM
	entries
WHERE
	user_id = @user_id
AND
	(@before::timestamp IS NULL OR submitted_at < @before::timestamp)
AND
	(@after::timestamp IS NULL OR submitted_at > @after::timestamp)
`

args := pgx.NamedArgs{
	"user_id": filter.UserId,
}

if !filter.After.IsZero() {
	args["after"] = pgtype.Timestamp{Time: filter.After, Valid: true}
}

if !filter.Before.IsZero() {
	args["before"] = pgtype.Timestamp{Time: filter.Before, Valid: true}
}

rows, err := pg.pool.Query(ctx, fSql, args)
if err != nil {
	return nil, err
}
```

# Redis

## Client

```go
import (
	"github.com/redis/go-redis/v9"
)
...
client := redis.NewClient(&redis.Options{
	Addr:     conf.RedisAddress,
	Password: conf.FoodRedisPassword,
	DB:       conf.FoodRedisDB,
	Protocol: 2,
})

status := client.Ping(context.Background())

if status.Err() != nil {
	return nil, fmt.Errorf("failed to connect redit client - %w", status.Err())
}
```

## Drop Index

```go
client.FTDropIndex(context.Background(), "idx:food")
```