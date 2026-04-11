## copilot

> Okay, here is a full, runnable example of a Spring Boot application that acts as a Model Context Protocol (MCP) server with tools to fetch information from an Oracle database.

Okay, here is a full, runnable example of a Spring Boot application that acts as a Model Context Protocol (MCP) server with tools to fetch information from an Oracle database.

This example uses the `stdio` transport, which is commonly used when running the MCP server as a local process that an AI client (like Claude Desktop or VS Code with Copilot) launches and communicates with over standard input/output streams.

**Important Considerations:**

* **Oracle JDBC Driver:** The Oracle JDBC driver (`ojdbcX.jar`) is typically **not** available in public Maven repositories due to licensing. You usually need to download it directly from the Oracle website (or obtain it internally if you are in an organization) and manually install it into your local Maven repository or use a corporate repository. Instructions for manual installation are provided below.
* **Database Credentials:** **Never hardcode credentials** in your code. Use `application.properties` and ideally environment variables or a secrets management system in production. This example uses `application.properties` placeholders.
* **Security:** The `executeQuery` tool (commented out) is very risky. The provided database metadata tools (`getTableSchema`, `getForeignKeys`, etc.) are safer as they use `DatabaseMetaData`, but exposing any database access to an AI model requires careful consideration of which information is exposed and to whom.
* **Error Handling:** The error handling in the code is basic (`printStackTrace`). In a production application, you would implement more robust logging and potentially return structured error messages via the MCP protocol.
* **Schema/Catalog:** The `DatabaseMetaData` calls use `null` for catalog and schema, which typically works for the current user's default schema in Oracle. Adjust if you need to access tables in a different schema.

---

**Project Setup (Maven):**

1.  **Create a Spring Boot Project:** Use Spring Initializr ([https://start.spring.io/](https://start.spring.io/)) or your IDE to create a new Maven project. Add the following dependencies:
    * Spring Web (or Spring WebFlux if you prefer reactive) - *Actually, for STDIO transport, you don't strictly need web starters, but they are common.* Let's stick to essentials for STDIO.
    * Spring JDBC
    * Spring AI Model Context Protocol Server (org.springframework.ai:spring-ai-starter-mcp-server)

    Alternatively, you can create a directory manually and add the `pom.xml` below.

2.  **Manual Oracle JDBC Driver Installation:**
    * Download the appropriate `ojdbcX.jar` file from the Oracle JDBC Driver Downloads page (e.g., `ojdbc11.jar` for Java 11+).
    * Open your terminal or command prompt.
    * Run the following Maven command, replacing `/path/to/your/ojdbcX.jar` with the actual path to the downloaded file, and adjusting `ojdbcX` and version if needed:

        ```bash
        mvn install:install-file -Dfile=/path/to/your/ojdbcX.jar -DgroupId=com.oracle.database.jdbc -DartifactId=ojdbcX -Dversion=19.8 -Dpackaging=jar
        ```

    * This command installs the JAR into your local Maven repository (`~/.m2/repository`).

3.  **Create Project Structure:** Ensure your project has a standard Maven structure:

    ```
    your-mcp-oracle-server/
    ├── pom.xml
    └── src/
        └── main/
            └── java/
                └── com/example/mcporacleserver/
                    ├── McpOracleServerApplication.java
                    └── OracleDatabaseTools.java
            └── resources/
                └── application.properties
    ```
    Replace `com/example/mcporacleserver` with your desired package structure.

---

**`pom.xml`**

Place this in the root of your project directory.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.5</version> <relativePath/> </parent>

    <groupId>com.example</groupId>
    <artifactId>mcp-oracle-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>mcp-oracle-server</name>
    <description>Demo MCP server for Oracle database access</description>

    <properties>
        <java.version>17</java.version> <spring-ai.version>0.8.0</spring-ai.version> </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-starter-mcp-server</artifactId>
            <version>${spring-ai.version}</version>
        </dependency>

        <dependency>
            <groupId>com.oracle.database.jdbc</groupId>
            <artifactId>ojdbcX</artifactId> <version>19.8</version> </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.ai</groupId>
                <artifactId>spring-ai-bom</artifactId>
                <version>${spring-ai.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```
*Remember to replace `ojdbcX` and the version (`19.8`) in the Oracle dependency with the actual artifact ID and version you manually installed.*

---

**`src/main/resources/application.properties`**

Create this file and add your Oracle connection details.

```properties
# Oracle Database Connection Configuration
spring.datasource.url=jdbc:oracle:thin:@//<your-oracle-host>:<your-oracle-port>/<your-service-name>
spring.datasource.username=<your-oracle-username>
spring.datasource.password=<your-oracle-password>
spring.datasource.driver-class-name=oracle.jdbc.OracleDriver

# Spring AI MCP Server Configuration
# Name and Version are required for the MCP protocol handshake
spring.ai.mcp.server.name=OracleDbInfoServer
spring.ai.mcp.server.version=1.0.0
# Transport type (stdio is for running as a child process)
spring.ai.mcp.server.transport-type=stdio
# Type can be SYNC or ASYNC - SYNC is simpler for initial tools
spring.ai.mcp.server.type=SYNC

# Optional: Enable logging to a file if running with stdio
# This helps debug as stdout/stderr are used for the protocol
# logging.file.name=./mcp-server.log
# logging.level.root=INFO
# logging.level.org.springframework.ai=DEBUG # Useful for debugging MCP
```
*Replace the placeholder values `<your-oracle-host>`, `<your-oracle-port>`, `<your-service-name>`, `<your-oracle-username>`, and `<your-oracle-password>` with your actual Oracle database connection details.*

---

**`src/main/java/com/example/mcporacleserver/McpOracleServerApplication.java`**

This is the main Spring Boot application class.

```java
package com.example.mcporacleserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class McpOracleServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(McpOracleServerApplication.class, args);
        System.out.println("MCP Oracle Server started..."); // Optional: print a message
        // The application will keep running, listening for MCP messages on stdin/stdout
        // when transport-type is stdio.
    }
}
```

---

**`src/main/java/com/example/mcporacleserver/OracleDatabaseTools.java`**

This is the service class containing the database tools. Copy the full code from the previous response and place it in this file within your package structure.

```java
// ... (Copy the full OracleDatabaseTools class code from the previous response here)
package com.example.mcporacleserver;

import org.springframework.ai.tool.Tool;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;

import javax.sql.DataSource;
import java.sql.DatabaseMetaData;
import java.sql.ResultSet;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Service
public class OracleDatabaseTools {

    private final JdbcTemplate jdbcTemplate;

    public OracleDatabaseTools(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    /**
     * Gets a list of table names in the Oracle database for the current user's schema.
     */
    @Tool(name = "getOracleTableNames", description = "Gets a list of table names in the Oracle database for the current user's schema.")
    public List<String> getTableNames() {
        List<String> tableNames = new ArrayList<>();
        try {
            DatabaseMetaData metaData = jdbcTemplate.getDataSource().getConnection().getMetaData();
            // Use null for catalog and schema to get tables for the current user
            ResultSet tables = metaData.getTables(null, null, "%", new String[]{"TABLE"});
            while (tables.next()) {
                tableNames.add(tables.getString("TABLE_NAME"));
            }
        } catch (Exception e) {
            System.err.println("Error fetching table names: " + e.getMessage());
            e.printStackTrace();
            // Consider more robust error handling
        }
        return tableNames;
    }

    /**
     * Gets the schema (columns, data types, nullability, size) for a specific table.
     * @param tableName The name of the table.
     */
    @Tool(name = "getOracleTableSchema", description = "Gets the schema (columns, data types, nullability, size) for a specific table in the Oracle database. Takes the table name as input.")
    public List<Map<String, String>> getTableSchema(String tableName) {
        List<Map<String, String>> schema = new ArrayList<>();
        if (tableName == null || tableName.trim().isEmpty()) {
             System.err.println("Table name cannot be null or empty for getOracleTableSchema.");
             return schema;
        }
        try {
             DatabaseMetaData metaData = jdbcTemplate.getDataSource().getConnection().getMetaData();
             // Use null for catalog and schema for the current user
             // The table name pattern should match the exact table name (Oracle usually stores names in uppercase)
             ResultSet columns = metaData.getColumns(null, null, tableName.toUpperCase(), null);

             while(columns.next()){
                 Map<String, String> columnInfo = new HashMap<>();
                 columnInfo.put("columnName", columns.getString("COLUMN_NAME"));
                 columnInfo.put("dataType", columns.getString("TYPE_NAME"));
                 columnInfo.put("columnSize", columns.getString("COLUMN_SIZE"));
                 columnInfo.put("isNullable", columns.getString("IS_NULLABLE")); // YES or NO
                 columnInfo.put("remarks", columns.getString("REMARKS")); // Might contain comments if available via metadata
                 schema.add(columnInfo);
             }
        } catch (Exception e) {
             System.err.println("Error fetching schema for table " + tableName + ": " + e.getMessage());
             e.printStackTrace();
             // Consider more robust error handling
        }
        return schema;
    }

    /**
     * Gets primary key information for a specific table.
     * @param tableName The name of the table.
     */
    @Tool(name = "getOraclePrimaryKey", description = "Gets primary key information (columns involved) for a specific table. Takes the table name as input.")
    public List<Map<String, String>> getPrimaryKey(String tableName) {
        List<Map<String, String>> primaryKeys = new ArrayList<>();
         if (tableName == null || tableName.trim().isEmpty()) {
             System.err.println("Table name cannot be null or empty for getOraclePrimaryKey.");
             return primaryKeys;
        }
        try {
            DatabaseMetaData metaData = jdbcTemplate.getDataSource().getConnection().getMetaData();
             // Use null for catalog and schema for the current user
            ResultSet keys = metaData.getPrimaryKeys(null, null, tableName.toUpperCase());
            while (keys.next()) {
                Map<String, String> keyInfo = new HashMap<>();
                keyInfo.put("columnName", keys.getString("COLUMN_NAME"));
                keyInfo.put("keySeq", String.valueOf(keys.getShort("KEY_SEQ"))); // Sequence number within the primary key
                keyInfo.put("pkName", keys.getString("PK_NAME"));
                primaryKeys.add(keyInfo);
            }
        } catch (Exception e) {
             System.err.println("Error fetching primary key for table " + tableName + ": " + e.getMessage());
            e.printStackTrace();
             // Consider more robust error handling
        }
        return primaryKeys;
    }

     /**
      * Gets foreign key information (relationships) for a specific table.
      * Lists tables that reference this table (imported keys) and tables this table references (exported keys).
      * @param tableName The name of the table.
      */
    @Tool(name = "getOracleForeignKeys", description = "Gets foreign key relationships for a specific table, showing which tables reference this one and which tables this one references. Takes the table name as input.")
    public Map<String, List<Map<String, String>>> getForeignKeys(String tableName) {
        Map<String, List<Map<String, String>>> foreignKeys = new HashMap<>();
        List<Map<String, String>> importedKeys = new ArrayList<>(); // Foreign keys in *this* table referencing *other* tables
        List<Map<String, String>> exportedKeys = new ArrayList<>(); // Foreign keys in *other* tables referencing *this* table

         if (tableName == null || tableName.trim().isEmpty()) {
             System.err.println("Table name cannot be null or empty for getOracleForeignKeys.");
             return foreignKeys;
        }

        try {
            DatabaseMetaData metaData = jdbcTemplate.getDataSource().getConnection().getMetaData();

            // Get foreign keys *in this table* that reference other tables
            ResultSet impKeys = metaData.getImportedKeys(null, null, tableName.toUpperCase());
            while (impKeys.next()) {
                Map<String, String> keyInfo = new HashMap<>();
                keyInfo.put("pkTableName", impKeys.getString("PKTABLE_NAME")); // Table being referenced
                keyInfo.put("pkColumnName", impKeys.getString("PKCOLUMN_NAME")); // Column in the referenced table
                keyInfo.put("fkTableName", impKeys.getString("FKTABLE_NAME")); // This table
                keyInfo.put("fkColumnName", impKeys.getString("FKCOLUMN_NAME")); // Column in this table
                keyInfo.put("keySeq", String.valueOf(impKeys.getShort("KEY_SEQ")));
                keyInfo.put("fkName", impKeys.getString("FK_NAME")); // Foreign key constraint name
                keyInfo.put("updateRule", getForeignKeyRuleDescription(impKeys.getShort("UPDATE_RULE")));
                keyInfo.put("deleteRule", getForeignKeyRuleDescription(impKeys.getShort("DELETE_RULE")));
                importedKeys.add(keyInfo);
            }

            // Get foreign keys *in other tables* that reference *this* table
            ResultSet expKeys = metaData.getExportedKeys(null, null, tableName.toUpperCase());
             while (expKeys.next()) {
                Map<String, String> keyInfo = new HashMap<>();
                keyInfo.put("pkTableName", expKeys.getString("PKTABLE_NAME")); // This table
                keyInfo.put("pkColumnName", expKeys.getString("PKCOLUMN_NAME")); // Column in this table
                keyInfo.put("fkTableName", expKeys.getString("FKTABLE_NAME")); // Table referencing this one
                keyInfo.put("fkColumnName", expKeys.getString("FKCOLUMN_NAME")); // Column in the referencing table
                 keyInfo.put("keySeq", String.valueOf(expKeys.getShort("KEY_SEQ")));
                 keyInfo.put("fkName", expKeys.getString("FK_NAME")); // Foreign key constraint name
                 keyInfo.put("updateRule", getForeignKeyRuleDescription(expKeys.getShort("UPDATE_RULE")));
                 keyInfo.put("deleteRule", getForeignKeyRuleDescription(expKeys.getShort("DELETE_RULE")));
                exportedKeys.add(keyInfo);
            }

        } catch (Exception e) {
             System.err.println("Error fetching foreign keys for table " + tableName + ": " + e.getMessage());
            e.printStackTrace();
             // Consider more robust error handling
        }

        foreignKeys.put("importedKeys", importedKeys); // Keys *in* this table
        foreignKeys.put("exportedKeys", exportedKeys); // Keys *referencing* this table

        return foreignKeys;
    }

     // Helper to convert FK rule codes to descriptions
     private String getForeignKeyRuleDescription(short rule) {
         return switch (rule) {
             case DatabaseMetaData.importedKeyNoAction -> "NO ACTION";
             case DatabaseMetaData.importedKeyCascade -> "CASCADE";
             case DatabaseMetaData.importedKeySetNull -> "SET NULL";
             case DatabaseMetaData.importedKeySetDefault -> "SET DEFAULT";
             case DatabaseMetaData.importedKeyRestrict -> "RESTRICT";
             default -> "UNKNOWN";
         };
     }


    /**
     * Gets index information for a specific table.
     * @param tableName The name of the table.
     */
    @Tool(name = "getOracleIndexes", description = "Gets index information (index name, columns, uniqueness) for a specific table. Takes the table name as input.")
    public List<Map<String, String>> getIndexes(String tableName) {
        List<Map<String, String>> indexes = new ArrayList<>();
         if (tableName == null || tableName.trim().isEmpty()) {
             System.err.println("Table name cannot be null or empty for getOracleIndexes.");
             return indexes;
        }
        try {
            DatabaseMetaData metaData = jdbcTemplate.getDataSource().getConnection().getMetaData();
             // Use null for catalog and schema, false for unique (get all), false for approximate
            ResultSet indexInfo = metaData.getIndexInfo(null, null, tableName.toUpperCase(), false, false);

            while (indexInfo.next()) {
                 // Filter out index stats, focus on index definitions
                 if (indexInfo.getShort("TYPE") == DatabaseMetaData.tableIndexStatistic) {
                     continue;
                 }
                Map<String, String> idxInfo = new HashMap<>();
                idxInfo.put("indexName", indexInfo.getString("INDEX_NAME"));
                idxInfo.put("columnName", indexInfo.getString("COLUMN_NAME"));
                idxInfo.put("ordinalPosition", String.valueOf(indexInfo.getInt("ORDINAL_POSITION")));
                idxInfo.put("cardinality", String.valueOf(indexInfo.getLong("CARDINALITY"))); // Number of unique values
                idxInfo.put("pages", String.valueOf(indexInfo.getLong("PAGES"))); // Pages used by the index
                idxInfo.put("filterCondition", indexInfo.getString("FILTER_CONDITION")); // For partial indexes
                idxInfo.put("isUnique", !indexInfo.getBoolean("NON_UNIQUE") ? "Yes" : "No");
                idxInfo.put("indexQualifier", indexInfo.getString("INDEX_QUALIFIER")); // Index catalog
                indexes.add(idxInfo);
            }
        } catch (Exception e) {
             System.err.println("Error fetching indexes for table " + tableName + ": " + e.getMessage());
            e.printStackTrace();
             // Consider more robust error handling
        }
        return indexes;
    }

    /**
     * Gets constraint information (e.g., CHECK, UNIQUE, excluding PK/FK which have dedicated tools) for a specific table.
     * Note: JDBC DatabaseMetaData does not have a standard way to get CHECK constraints.
     * You would typically query Oracle's data dictionary views (like USER_CONSTRAINTS, USER_CONS_COLUMNS) for these.
     * This method will return UNIQUE constraints via getIndexInfo if UNIQUE indexes exist, but not CHECK.
     * A real-world tool might execute a specific SQL query for CHECK constraints.
     *
     * @param tableName The name of the table.
     */
    @Tool(name = "getOracleConstraints", description = "Gets constraint information (like UNIQUE constraints, exclude PK/FK which have separate tools) for a specific table. Note: Does not reliably fetch CHECK constraints via standard metadata. Takes the table name as input.")
    public List<Map<String, String>> getConstraints(String tableName) {
         List<Map<String, String>> constraints = new ArrayList<>();
          if (tableName == null || tableName.trim().isEmpty()) {
             System.err.println("Table name cannot be null or empty for getOracleConstraints.");
             return constraints;
        }
         // DatabaseMetaData doesn't easily expose CHECK constraints.
         // It can give us UNIQUE constraints IF they are enforced with a unique index.
         try {
             DatabaseMetaData metaData = jdbcTemplate.getDataSource().getConnection().getMetaData();
             // Look for unique indexes which often correspond to UNIQUE constraints
             ResultSet indexInfo = metaData.getIndexInfo(null, null, tableName.toUpperCase(), true, false); // true for unique

             while (indexInfo.next()) {
                  if (indexInfo.getShort("TYPE") == DatabaseMetaData.tableIndexStatistic) {
                     continue; // Skip index stats
                  }
                 Map<String, String> constraintInfo = new HashMap<>();
                 constraintInfo.put("constraintName", indexInfo.getString("INDEX_NAME")); // Often index name is constraint name
                 constraintInfo.put("type", "UNIQUE (via Index)"); // Label it as a unique constraint derived from index
                 constraintInfo.put("columnName", indexInfo.getString("COLUMN_NAME"));
                 constraintInfo.put("ordinalPosition", String.valueOf(indexInfo.getInt("ORDINAL_POSITION")));
                 constraints.add(constraintInfo);
             }

             // To get CHECK constraints, you would need a query like:
             /*
              ResultSet checkConstraints = jdbcTemplate.getDataSource().getConnection().createStatement().executeQuery(
                  "SELECT constraint_name, search_condition " +
                  "FROM user_constraints " +
                  "WHERE table_name = '" + tableName.toUpperCase() + "' " +
                  "AND constraint_type = 'C'" // 'C' is for CHECK constraints in Oracle
              );
              while(checkConstraints.next()) {
                   Map<String, String> checkInfo = new HashMap<>();
                   checkInfo.put("constraintName", checkConstraints.getString("CONSTRAINT_NAME"));
                   checkInfo.put("type", "CHECK");
                   checkInfo.put("condition", checkConstraints.getString("SEARCH_CONDITION"));
                   constraints.add(checkInfo);
              }
              */
             // NOTE: Executing arbitrary SQL like the above needs careful security consideration
             // (e.g., preventing SQL injection if tableName came from an untrusted source).
             // Using DatabaseMetaData is safer where possible.

         } catch (Exception e) {
             System.err.println("Error fetching constraints for table " + tableName + ": " + e.getMessage());
             e.printStackTrace();
             // Consider more robust error handling
         }

         return constraints;
    }


     /**
      * Gets comments on the table and its columns.
      * Note: Getting comments requires querying Oracle's data dictionary views (USER_TAB_COMMENTS, USER_COL_COMMENTS).
      * DatabaseMetaData.getColumns might return comments, but it's not guaranteed across all drivers/versions.
      * This method uses direct SQL queries against Oracle views.
      *
      * @param tableName The name of the table.
      */
    @Tool(name = "getOracleComments", description = "Gets comments defined on the table and its columns in the Oracle database. Takes the table name as input.")
    public Map<String, Object> getComments(String tableName) {
        Map<String, Object> comments = new HashMap<>();
         if (tableName == null || tableName.trim().isEmpty()) {
             System.err.println("Table name cannot be null or empty for getOracleComments.");
             return comments;
        }

        try {
            // Get table comment
            List<Map<String, Object>> tableCommentResult = jdbcTemplate.queryForList(
                 "SELECT comments FROM user_tab_comments WHERE table_name = ?",
                 tableName.toUpperCase()
            );
            if (!tableCommentResult.isEmpty() && tableCommentResult.get(0).get("COMMENTS") != null) {
                 comments.put("tableComment", tableCommentResult.get(0).get("COMMENTS"));
            } else {
                 comments.put("tableComment", "No comment defined for table " + tableName);
            }


            // Get column comments
            List<Map<String, String>> columnCommentsList = new ArrayList<>();
             List<Map<String, Object>> columnCommentsResult = jdbcTemplate.queryForList(
                 "SELECT column_name, comments FROM user_col_comments WHERE table_name = ?",
                 tableName.toUpperCase()
            );
             for (Map<String, Object> row : columnCommentsResult) {
                 Map<String, String> colComment = new HashMap<>();
                 colComment.put("columnName", (String) row.get("COLUMN_NAME"));
                 colComment.put("comment", row.get("COMMENTS") != null ? (String) row.get("COMMENTS") : "No comment");
                 columnCommentsList.add(colComment);
             }
             comments.put("columnComments", columnCommentsList);

        } catch (Exception e) {
             System.err.println("Error fetching comments for table " + tableName + ": " + e.getMessage());
             e.printStackTrace();
             // Consider more robust error handling
             comments.put("error", "Could not fetch comments: " + e.getMessage());
        }

        return comments;
    }

    /**
     * Provides a comprehensive overview of a table including schema, primary key,
     * foreign keys, and index information.
     * @param tableName The name of the table.
     */
    @Tool(name = "getOracleTableOverview", description = "Provides a comprehensive overview of a table, including its schema (columns), primary key, foreign keys (relationships), and index information. Takes the table name as input.")
    public Map<String, Object> getTableOverview(String tableName) {
        Map<String, Object> overview = new HashMap<>();
        if (tableName == null || tableName.trim().isEmpty()) {
             System.err.println("Table name cannot be null or empty for getOracleTableOverview.");
             overview.put("error", "Table name cannot be null or empty.");
             return overview;
        }

        System.out.println("Fetching overview for table: " + tableName); // Log which tool is called
        overview.put("tableName", tableName);
        overview.put("schema", getTableSchema(tableName));
        overview.put("primaryKey", getPrimaryKey(tableName));
        overview.put("foreignKeys", getForeignKeys(tableName));
        overview.put("indexes", getIndexes(tableName));
        overview.put("comments", getComments(tableName)); // Include comments in overview

        // Note: getConstraints currently only gets UNIQUE via index.
        // If you implement fetching CHECK constraints via SQL, you might add them here too.
        // overview.put("constraints", getConstraints(tableName));

        // You could add more here, e.g., row count (SELECT COUNT(*) FROM table) - be mindful of performance on large tables

        return overview;
    }


    // Optional: Tool to execute a read-only query - Use with caution and consider security (SQL Injection)
     /*
    @Tool(name = "executeOracleQuery", description = "Executes a read-only SQL query against the database and returns results. Use with caution! Takes the SQL query string as input.")
    public List<Map<String, Object>> executeQuery(String sqlQuery) {
        if (sqlQuery == null || sqlQuery.trim().isEmpty()) {
             System.err.println("SQL query cannot be null or empty.");
             return List.of(Map.of("error", "SQL query cannot be null or empty."));
        }
        // Basic security check (prevent common write/delete commands) - NOT robust, requires proper parsing/validation
        String upperQuery = sqlQuery.trim().toUpperCase();
        if (upperQuery.startsWith("INSERT") || upperQuery.startsWith("UPDATE") || upperQuery.startsWith("DELETE") || upperQuery.startsWith("DROP") || upperQuery.startsWith("ALTER") || upperQuery.startsWith("CREATE")) {
             System.err.println("Write/DDL operations are not allowed via this tool.");
             return List.of(Map.of("error", "Write/DDL operations are not allowed."));
        }

        try {
            // Execute the query and return results as a list of maps
            return jdbcTemplate.queryForList(sqlQuery);
        } catch (Exception e) {
            System.err.println("Error executing query: " + sqlQuery + ": " + e.getMessage());
            e.printStackTrace();
             return List.of(Map.of("error", "Error executing query: " + e.getMessage()));
        }
    }
    */
}
```

---

**How to Run:**

1.  **Ensure Java and Maven are installed.**
2.  **Download and manually install the Oracle JDBC driver** into your local Maven repository (as described above).
3.  **Replace placeholders** in `application.properties` with your Oracle database details.
4.  **Open a terminal** in the root directory of your project (where `pom.xml` is located).
5.  **Build the project:**
    ```bash
    mvn clean package
    ```
6.  **Run the application:**
    ```bash
    java -jar target/mcp-oracle-server-0.0.1-SNAPSHOT.jar
    ```
    (Adjust the JAR name if your version is different).

The server will start and listen on standard input/output. You won't see a typical web server starting message, as it's using the `stdio` transport. Any console output from `System.out` or `System.err` in your Java code might interfere with the MCP protocol, so using a dedicated logging framework configured to write to a file (as suggested in the commented properties) is recommended for debugging.

---

**How to Connect an MCP Client (Example using Claude Desktop):**

The exact steps vary depending on the MCP client you use. For clients that support running a local command as an MCP server (like Claude Desktop or potentially VS Code's Copilot in Agent mode), you typically configure it to execute your compiled JAR.

1.  **Find the path to your JAR file.** After running `mvn clean package`, it will be in the `target/` directory.
2.  **Configure the client:**
    * Find the MCP server configuration file for your client (e.g., `claude_desktop_config.json` for Claude Desktop).
    * Add an entry for your server, specifying the command to run your JAR:

        ```json
        {
          "mcpServers": {
            "oracle-db-tool": {
              "command": "java",
              "args": [
                "-jar",
                "/path/to/your/mcp-oracle-server/target/mcp-oracle-server-0.0.1-SNAPSHOT.jar"
              ]
              // Optional: Add environment variables for sensitive config if not using application.properties directly
              // "env": {
              //    "SPRING_DATASOURCE_URL": "...",
              //    "SPRING_DATASOURCE_USERNAME": "...",
              //    "SPRING_DATASOURCE_PASSWORD": "..."
              // }
            }
          }
        }
        ```
    * Replace `/path/to/your/mcp-oracle-server/target/mcp-oracle-server-0.0.1-SNAPSHOT.jar` with the actual path to your JAR file.
    * Save the client's configuration file.
3.  **Restart the MCP client.** It should detect and connect to your new server.
4.  **Use the tools:** In the client's AI chat interface (often in Agent mode), you should now be able to ask questions that leverage your tools. For example:
    * "List all tables in the database." (Should trigger `getOracleTableNames`)
    * "Show me the schema for the EMPLOYEES table." (Should trigger `getOracleTableSchema`)
    * "What are the primary keys for the PRODUCTS table?" (Should trigger `getOraclePrimaryKey`)
    * "Tell me everything about the ORDERS table." (Should trigger `getOracleTableOverview`)

This provides a complete starting point. Remember to adapt the database connection details and handle the Oracle JDBC driver dependency correctly. Good luck!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/akhilnaidu1994)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/akhilnaidu1994)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
