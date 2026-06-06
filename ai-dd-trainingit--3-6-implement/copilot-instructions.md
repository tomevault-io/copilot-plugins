## sql-commands-json

> Rules for generating SQL commands in JSON configuration files


# SQL Commands JSON Configuration Files

These files define SQL operations for a specific database table in a structured JSON format.

## File Structure

- Place files in the `src/sql/` directory
- Name files according to the table name: `tablename.sql.json`
- Use `UPER_CASE` for SQL operation keys, as they will be used as constants in code.

## Required Keys

- `TABLE`: String with the table name
- `CREATE_TABLE`: Full SQL statement for table creation with all columns and constraints
- `SELECT_ALL`: Query to fetch all records
- `SELECT_BY_ID`: Query to fetch a record by its primary key
- `SELECT_BY_FIELD`: Query to fetch records by a field value pair
- `SELECT_BY_USER_ID`: Query to fetch a record by user foreing key
- `INSERT`: Statement to insert a new record with all fields
- `UPDATE`: Statement to update all fields for a record
- `DELETE`: Statement to delete a record by its primary key
- `SEED`: An array of raw entotoes needed before start the app (config, initial data...)

## Parameter Conventions

- Use `$paramName` syntax for query parameters
- Common parameters:
  - `$id`: For primary key references
  - `$created_at` and `$updated_at`: For timestamp fields
  - Other parameters should match column names
  - Prefix all parameters with `$`
  - Name them in `snake_case`

## SEED Data (Optional)

- Include a `SEED` array if the table needs initial data
- Each seed item should be an object with the table's column values
- Omit auto-generated fields (id, timestamps) from seed objects

## Example

```json
{
	"TABLE": "tools",
	"CREATE_TABLE": "CREATE TABLE tools (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT, description TEXT, url TEXT, created_at DATETIME, updated_at DATETIME)",
	"SELECT_ALL": "SELECT * FROM tools",
	"SELECT_BY_ID": "SELECT * FROM tools WHERE id = $id",
	"INSERT": "INSERT INTO tools (name, description, url, created_at, updated_at) VALUES ($name, $description, $url, $created_at, $updated_at)",
	"UPDATE": "UPDATE tools SET name = $name, description = $description, url = $url, updated_at = $updated_at WHERE id = $id",
	"DELETE": "DELETE FROM tools WHERE id = $id",
	"SEED": [
		{
			"name": "Bun",
			"description": "Bun is a fast, modern, all-in-one toolkit for JavaScript.",
			"url": "https://bun.sh/"
		},
		{
			"name": "Pico CSS",
			"description": "Pico CSS is a minimal CSS framework for semantic HTML.",
			"url": "https://picocss.com/"
		},
		{
			"name": "Playwright",
			"description": "Playwright is a framework for browser automation and testing.",
			"url": "https://playwright.dev/"
		},
		{
			"name": "Vite",
			"description": "Vite is a build tool for modern web development.",
			"url": "https://vitejs.dev/"
		}
	]
}
``` 

---
> Source: [AI-DD-TrainingIT/3_6-implement](https://github.com/AI-DD-TrainingIT/3_6-implement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
