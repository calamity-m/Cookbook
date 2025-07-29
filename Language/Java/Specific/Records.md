
# Compact Constructor

```java
@ConfigurationProperties(prefix = "app")
public record AppProperties(DatabaseProps db) {
    public AppProperties {
        if (db == null) {
            db = new DatabaseProps(null); // default to null connection
        }
    }
```
# Null safe access

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
