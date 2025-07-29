
# Null safe access with defaults
I wouldn't use this very often, but it's helpful for record-based configuration in spring

```java
public static String safeHost(AppProperties props) {
    return Optional.ofNullable(props)
	    // if db is null
        .map(AppProperties::db)
        // if connection is null
        .map(DatabaseProps::connection)
        // if host is null
        .map(ConnectionProps::host)
        .orElse("localhost");
}
```
