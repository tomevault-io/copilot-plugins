## agentic-salesforce

> This document provides best practices for counting records in CSV files and Salesforce scratch orgs.

# Salesforce Data Counting Reference

This document provides best practices for counting records in CSV files and Salesforce scratch orgs.

## Counting Records in CSV Files

### Command
```bash
count=$(($(wc -l < "file.csv") - 1))
```

### How It Works
1. `wc -l < "file.csv"` - Counts total lines in the file (including header)
2. `- 1` - Subtracts the header row
3. Result = actual data record count

### Example
```bash
# Single file
count=$(($(wc -l < accounts.csv) - 1))
echo "accounts.csv: $count records"

# Multiple files
for file in *.csv; do
  count=$(($(wc -l < "$file") - 1))
  echo "$file: $count records"
done
```

### Verification
```bash
# Show first 3 lines to verify header exists
head -3 accounts.csv

# Expected output:
# Line 1: AccountExtId__c,Name,Type,Industry,...  (header)
# Line 2: RC-ACCT-0001,Acme Health Systems,...    (data row 1)
# Line 3: RC-ACCT-0002,Northern Lights Bank,...   (data row 2)
```

## Counting Records in Salesforce Scratch Org

### Command (Latest Salesforce CLI Best Practice)
```bash
sf data query --query "SELECT COUNT() FROM <Object>" --json
```

### How It Works
1. Uses Salesforce CLI v2+ (`sf` command, not `sfdx`)
2. `SELECT COUNT()` - SOQL aggregate function (server-side count)
3. `--json` - Returns structured JSON output
4. Count is in `result.totalSize` field

### Example Queries

#### Basic Count
```bash
# Count all Accounts
sf data query --query "SELECT COUNT() FROM Account" --json

# Output structure:
{
  "status": 0,
  "result": {
    "records": [],
    "totalSize": 8,     ← The count
    "done": true
  }
}
```

#### Extract Count Value
```bash
# Using Python JSON parser
sf data query --query "SELECT COUNT() FROM Task" --json | \
  python3 -c "import sys, json; print(json.load(sys.stdin)['result']['totalSize'])"

# Output: 225
```

#### Filtered Counts
```bash
# Count with WHERE clause
sf data query --query "SELECT COUNT() FROM PricebookEntry WHERE Pricebook2.IsStandard = true" --json

# Count with relationship filter
sf data query --query "SELECT COUNT() FROM PricebookEntry WHERE Pricebook2.IsStandard = false" --json
```

### Common Object Queries

```bash
# Standard Objects
sf data query --query "SELECT COUNT() FROM Account" --json
sf data query --query "SELECT COUNT() FROM Contact" --json
sf data query --query "SELECT COUNT() FROM Opportunity" --json
sf data query --query "SELECT COUNT() FROM Case" --json
sf data query --query "SELECT COUNT() FROM Task" --json

# Custom Objects
sf data query --query "SELECT COUNT() FROM Product2" --json
sf data query --query "SELECT COUNT() FROM Pricebook2" --json
sf data query --query "SELECT COUNT() FROM PricebookEntry" --json

# Activity Objects
sf data query --query "SELECT COUNT() FROM FeedItem" --json
sf data query --query "SELECT COUNT() FROM EmailMessage" --json

# Content Objects
sf data query --query "SELECT COUNT() FROM ContentDocument" --json
sf data query --query "SELECT COUNT() FROM ContentVersion" --json
```

## Comparing CSV vs Scratch Org Counts

### Quick Comparison Script
```bash
#!/bin/bash
# Compare intended (CSV) vs actual (Salesforce) counts

echo "Object | CSV | Actual | Difference"
echo "-------|-----|--------|------------"

# Accounts
csv_count=$(($(wc -l < data/accounts.csv) - 1))
sf_count=$(sf data query --query "SELECT COUNT() FROM Account" --json | \
  python3 -c "import sys, json; print(json.load(sys.stdin)['result']['totalSize'])")
diff=$((sf_count - csv_count))
echo "Account | $csv_count | $sf_count | $diff"

# Tasks
csv_count=$(($(wc -l < data/tasks.csv) - 1))
sf_count=$(sf data query --query "SELECT COUNT() FROM Task" --json | \
  python3 -c "import sys, json; print(json.load(sys.stdin)['result']['totalSize'])")
diff=$((sf_count - csv_count))
echo "Task | $csv_count | $sf_count | $diff"
```

### Example Output
```
Object | CSV | Actual | Difference
-------|-----|--------|------------
Account | 8 | 8 | 0
Task | 30 | 225 | 195
```

## Best Practices

### CSV Counting
- ✅ Always subtract 1 for the header row
- ✅ Use `wc -l < file.csv` (redirect input, cleaner output)
- ✅ Verify header exists with `head -n 1 file.csv`
- ❌ Don't use `cat file.csv | wc -l` (unnecessary pipe)

### Salesforce Counting
- ✅ Use `sf` CLI (not deprecated `sfdx`)
- ✅ Use `SELECT COUNT()` for exact server-side counts
- ✅ Add `--json` for parseable output
- ✅ Filter with WHERE clauses when needed
- ❌ Don't query all records and count client-side (inefficient)
- ❌ Don't use `SELECT Id FROM Object` then count (wastes data transfer)

### Accuracy Verification
- COUNT() is a database aggregate - returns exact totals
- No client-side filtering or data transfer needed
- Works with any org size (no governor limits on COUNT)
- Returns 0 for objects with no records (not an error)

## Troubleshooting

### CSV Count Issues
```bash
# Check for missing newline at end of file
tail -c 1 file.csv | od -An -tx1
# If output is not "0a" (newline), count may be off by 1

# Check for empty lines
grep -c "^$" file.csv
# Should return 0 (no empty lines)
```

### Salesforce Query Issues

```bash
# Check if object exists
sf data query --query "SELECT COUNT() FROM NonExistentObject" --json
# Will return error if object doesn't exist

# Verify connection
sf org display --json
# Shows current connected org
```

# Salesforce Bulk API Line Endings

## The Requirement
Salesforce Bulk API 2.0 accepts both LF and CRLF line endings, but **you must tell the CLI which one your CSV uses** via the `--line-ending` flag.

## The Issue on macOS
- macOS default: LF (`\n`)
- Salesforce Bulk API standard: CRLF (`\r\n`)
- If they don't match: `ClientInputError: LineEnding is invalid`

## The Solution
**Always use CRLF for Salesforce Bulk API operations:**

```python
# Python: Force CRLF line endings (not macOS default)
with open('file.csv', 'w', newline='') as f:
    writer = csv.writer(f, lineterminator='\r\n')  # CRLF required
    writer.writerow(['Id'])
```

```bash
# CLI: Match the CSV line ending
sf data delete bulk --sobject Account --file file.csv --line-ending CRLF
```

**Key point:** CSV must use CRLF (`\r\n`) AND CLI must specify `--line-ending CRLF`

## Related Documentation
- [Salesforce CLI Command Reference](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/)
- [SOQL Aggregate Functions](https://developer.salesforce.com/docs/atlas.en-us.soql_sosl.meta/soql_sosl/sforce_api_calls_soql_select_agg_functions.htm)

---
> Source: [benjipeng/agentic-salesforce](https://github.com/benjipeng/agentic-salesforce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
