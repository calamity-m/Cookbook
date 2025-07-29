
# Integration Application Properties

```text
// 1. Application properties for integration tests
// src/test/resources/application-integration-test.properties

spring.datasource.url=jdbc:oracle:thin:@${ORACLE_HOST:localhost}:${ORACLE_PORT:1521}:${ORACLE_SID:xe}
spring.datasource.username=system
spring.datasource.password=password
spring.datasource.driver-class-name=oracle.jdbc.OracleDriver

# Liquibase configuration
spring.liquibase.enabled=true
spring.liquibase.change-log=classpath:db/changelog/db.changelog-master.xml
spring.liquibase.default-schema=${test.schema:TEST_SCHEMA}

# JPA/Hibernate configuration
spring.jpa.hibernate.ddl-auto=none
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.Oracle12cDialect
spring.jpa.properties.hibernate.default_schema=${test.schema:TEST_SCHEMA}

# Connection pool settings for tests
spring.datasource.hikari.maximum-pool-size=5
spring.datasource.hikari.minimum-idle=2
```

# Integration Gradle Config

```
dependencies {
    testImplementation 'com.oracle.database.jdbc:ojdbc8:21.9.0.0'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    implementation 'org.liquibase:liquibase-core'
}

test {
    useJUnitPlatform {
        excludeTags 'integration'
    }
}

task integrationTest(type: Test) {
    useJUnitPlatform {
        includeTags 'integration'
    }
    shouldRunAfter test
}

check.dependsOn integrationTest
```

# Integration Test Config Class

```java
package com.example.test.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Primary;
import org.springframework.test.context.TestPropertySource;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.UUID;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

/**
 * Test configuration for Oracle integration tests with schema isolation
 */
@TestConfiguration
@TestPropertySource("classpath:application-integration-test.properties")
public class OracleTestConfig {

    @Value("${spring.datasource.url}")
    private String jdbcUrl;

    @Value("${spring.datasource.username}")
    private String username;

    @Value("${spring.datasource.password}")
    private String password;

    @Value("${spring.datasource.driver-class-name}")
    private String driverClassName;

    @Bean
    @Primary
    public DataSource testDataSource() throws SQLException {
        // Generate unique schema name for this test run
        String schemaName = "TEST_" + UUID.randomUUID().toString().replace("-", "").substring(0, 20);
        
        // Create a temporary connection to create the schema/user
        HikariConfig tempConfig = new HikariConfig();
        tempConfig.setJdbcUrl(jdbcUrl);
        tempConfig.setUsername(username);
        tempConfig.setPassword(password);
        tempConfig.setDriverClassName(driverClassName);
        tempConfig.setMaximumPoolSize(1);
        
        try (HikariDataSource tempDs = new HikariDataSource(tempConfig);
             Connection conn = tempDs.getConnection();
             Statement stmt = conn.createStatement()) {
            
            // Create a new user/schema for this test
            String userPassword = "test_pass_" + UUID.randomUUID().toString().substring(0, 8);
            
            // Drop user if exists (cascade will drop all objects)
            try {
                stmt.execute("DROP USER " + schemaName + " CASCADE");
            } catch (SQLException e) {
                // User doesn't exist, which is fine
            }
            
            // Create new user with necessary privileges
            stmt.execute("CREATE USER " + schemaName + " IDENTIFIED BY \"" + userPassword + "\"");
            stmt.execute("GRANT CONNECT, RESOURCE TO " + schemaName);
            stmt.execute("GRANT CREATE SESSION TO " + schemaName);
            stmt.execute("GRANT CREATE TABLE TO " + schemaName);
            stmt.execute("GRANT CREATE SEQUENCE TO " + schemaName);
            stmt.execute("GRANT CREATE TRIGGER TO " + schemaName);
            stmt.execute("GRANT UNLIMITED TABLESPACE TO " + schemaName);
            
            // Set system properties for Liquibase and Hibernate to use
            System.setProperty("test.schema", schemaName);
            
            // Create the actual test datasource connected as the new user
            HikariConfig config = new HikariConfig();
            config.setJdbcUrl(jdbcUrl);
            config.setUsername(schemaName);
            config.setPassword(userPassword);
            config.setDriverClassName(driverClassName);
            config.setMaximumPoolSize(5);
            config.setMinimumIdle(2);
            config.setConnectionTestQuery("SELECT 1 FROM DUAL");
            
            // Store schema info for cleanup
            config.addDataSourceProperty("testSchemaName", schemaName);
            config.addDataSourceProperty("adminUsername", username);
            config.addDataSourceProperty("adminPassword", password);
            
            return new HikariDataSource(config);
        }
    }
}
```

# Integration Test Base Class

```java
package com.example.test;

import com.example.test.config.OracleTestConfig;
import com.zaxxer.hikari.HikariDataSource;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.annotation.Import;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.transaction.annotation.Transactional;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.Statement;

/**
 * Base class for Oracle integration tests with automatic schema isolation
 */
@ExtendWith(SpringExtension.class)
@SpringBootTest
@ActiveProfiles("integration-test")
@Import(OracleTestConfig.class)
@Tag("integration")
@Transactional
public abstract class OracleIntegrationTestBase {

    @Autowired
    protected DataSource dataSource;

    @AfterEach
    void cleanupSchema() {
        if (dataSource instanceof HikariDataSource) {
            HikariDataSource hikariDs = (HikariDataSource) dataSource;
            String schemaName = (String) hikariDs.getDataSourceProperties().get("testSchemaName");
            String adminUsername = (String) hikariDs.getDataSourceProperties().get("adminUsername");
            String adminPassword = (String) hikariDs.getDataSourceProperties().get("adminPassword");
            
            if (schemaName != null) {
                // Close the test datasource first
                hikariDs.close();
                
                // Connect as admin to drop the test schema
                HikariDataSource adminDs = new HikariDataSource();
                adminDs.setJdbcUrl(hikariDs.getJdbcUrl());
                adminDs.setUsername(adminUsername);
                adminDs.setPassword(adminPassword);
                adminDs.setDriverClassName(hikariDs.getDriverClassName());
                
                try (Connection conn = adminDs.getConnection();
                     Statement stmt = conn.createStatement()) {
                    stmt.execute("DROP USER " + schemaName + " CASCADE");
                } catch (Exception e) {
                    // Log error but don't fail the test
                    System.err.println("Failed to cleanup schema " + schemaName + ": " + e.getMessage());
                } finally {
                    adminDs.close();
                }
            }
        }
    }
}
```

# Integration Test Example Class

```java
package com.example.test;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;

import static org.junit.jupiter.api.Assertions.*;

/**
 * Example integration test using Oracle database
 */
public class ExampleOracleIntegrationTest extends OracleIntegrationTestBase {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Test
    void testDatabaseConnection() {
        // Verify we can connect and query the database
        String result = jdbcTemplate.queryForObject("SELECT 'Hello Oracle' FROM DUAL", String.class);
        assertEquals("Hello Oracle", result);
    }

    @Test
    void testLiquibaseMigrationsRan() {
        // This test assumes you have a Liquibase migration that creates a table
        // Verify that Liquibase migrations have run
        
        // Example: Check if a table created by Liquibase exists
        Integer tableCount = jdbcTemplate.queryForObject(
            "SELECT COUNT(*) FROM USER_TABLES WHERE TABLE_NAME = 'DATABASECHANGELOG'",
            Integer.class
        );
        
        assertEquals(1, tableCount, "Liquibase changelog table should exist");
        
        // Fill in here - your business logic tests
    }

    @Test
    void testBusinessLogic() {
        // Fill in here - test your repositories, services, etc.
        
        // Example structure:
        // Given: Set up test data
        // jdbcTemplate.execute("INSERT INTO your_table ...");
        
        // When: Execute business logic
        // YourService service = ...;
        // var result = service.doSomething();
        
        // Then: Verify results
        // assertEquals(expected, result);
    }

    @Test
    void testSchemaIsolation() {
        // Create a test table
        jdbcTemplate.execute("CREATE TABLE test_isolation (id NUMBER PRIMARY KEY, name VARCHAR2(100))");
        jdbcTemplate.execute("INSERT INTO test_isolation (id, name) VALUES (1, 'Test Data')");
        
        // Verify data exists
        Integer count = jdbcTemplate.queryForObject(
            "SELECT COUNT(*) FROM test_isolation", 
            Integer.class
        );
        assertEquals(1, count);
        
        // This table will be automatically cleaned up after the test
        // due to the schema being dropped in @AfterEach
    }
}
```

# Integration Gitlab CI

```yaml
# .gitlab-ci.yml
stages:
  - test
  - integration-test

# Regular unit tests
test:
  stage: test
  script:
    - mvn test
    # or for Gradle: ./gradlew test

# Integration tests with Oracle
integration-test:
  stage: integration-test
  services:
    - name: container-registry.oracle.com/database/express:21.3.0-xe
      alias: oracle
  variables:
    # Oracle container environment variables
    ORACLE_PWD: password
    ORACLE_CHARACTERSET: AL32UTF8
    # Test environment variables
    ORACLE_HOST: oracle
    ORACLE_PORT: 1521
    ORACLE_SID: XE
  script:
    # Wait for Oracle to be ready
    - |
      echo "Waiting for Oracle to start..."
      for i in {1..60}; do
        if nc -z oracle 1521; then
          echo "Oracle is ready!"
          break
        fi
        echo "Waiting... ($i/60)"
        sleep 5
      done
    # Run integration tests
    - ./gradlew integrationTest
  only:
    - merge_requests
    - main
```
