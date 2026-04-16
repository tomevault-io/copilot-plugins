## swipeclock-scripting-extension

> Swipeclock scripting is a custom scripting language used in Swipeclock's TimeWorksPlus and WorkforceHub software products. This language is case-insensitive and has unique syntax rules that differ significantly from standard programming languages.

# Cursor Rules for Swipeclock Scripting Language

## Language Overview
Swipeclock scripting is a custom scripting language used in Swipeclock's TimeWorksPlus and WorkforceHub software products. This language is case-insensitive and has unique syntax rules that differ significantly from standard programming languages.

## Critical Language Rules

### Case Sensitivity
- **ALL keywords, operators, functions, and object names are CASE-INSENSITIVE**
- `employee.firstname` is the same as `EMPLOYEE.FIRSTNAME` or `Employee.FirstName`
- `if`, `IF`, and `If` are all valid
- `and`, `AND`, and `And` are all valid
- `contains`, `CONTAINS`, and `Contains` are all valid

### Object-Based Properties
- Properties MUST be accessed through objects: `employee.propertyName` or `reportingdate.propertyName`
- **NEVER use properties without the object prefix** (e.g., `firstname` is INVALID, must use `employee.firstname`)
- `date` is INVALID - must use `reportingdate.date`
- `firstname` is INVALID - must use `employee.firstname`

### Employee Object Properties
Valid properties for `employee.*`:
- Basic info: `yearsofservice`, `daysofservice`, `monthsofservice`, `anniversary`, `monthiversary`, `title`, `firstname`, `lastname`, `code` (or `employeecode`), `designation`, `startdate`, `enddate`
- Organization: `department` (default), `department1`, `department2`, etc. (if enabled), `location`, `location1`, `location2`, etc. (if enabled), `supervisor`
- Contact: `home1`, `home2`, `home3`, `home4`, etc. (if enabled)
- Time settings: `autolunchhours`, `lunchminutes`
- Accrual: `accrualfactor`, `employeetype`, `accrualvalidator` (when BasicAccruals rule is activated)
- Pay: `payrate0` (default), `payrate1`, `payrate2`, `payrate3`, etc. (if enabled)

### ReportingDate Object Properties
Valid properties for `reportingdate.*`:
- Date info: `tomorrow`, `yesterday`, `date`, `year`, `month`, `weekday` (returns MTWRFSU), `todaysdate`, `isfirstdayofmonth`, `islastdayofmonth`, `payperiodstart`, `payperiodend`, `isholiday`
- Time calculations: `spread` (timespan), `totalhours`, `totalhoursot`, `hourstodate`, `hourstodateot`, `weekhours`, `pphours`, `islastpunchpp`, `islastpunchweek`
- Functions with arguments: `totalweek("<promptFieldName>")`, `totalpp("<promptFieldName>")`, `totalday("<promptFieldName>")`, `depthours("<FieldName>", "<Value>")`
- Schedule: `punchsets`, `schedhours(<punchset>)`, `schedlines`, `schedin(<punchset>)`, `schedout(<punchset>)`, `schedplace(<punchset>)`, `workweekend`
- Other: `average` (undocumented but valid)

### Global Functions (NOT object-based)
These functions are global and do NOT use object notation:
- **Date functions**: `dateadd("mm", 6, employee.startdate)`, `dateserial(year, month, day)`, `weekday(date)`, `cdate(string)`, `cdatetime(string)`, `ctime(datetime)`, `day(date)`, `month(date)`, `year(date)`
- **Number functions**: `val(string)` (returns 0 for null), `cint(value)` (throws error for null), `cstr(value)`, `abs(number)`
- **String functions**: `translate(field, list1, list2)`, `within(field, list)`, `left(string, count)`, `right(string, count)`, `mid(string, start, count)`
- **Time rounding**: `round(timeString)` or `round(number)`, `roundin(timeString)`, `roundout(timeString)`, `roundends(timeString)`, `roundtoschedule(a1, a2, b1, b2)`
- **Math rounding**: `roundup(number)`, `rounddown(number)` (NOT for time rounding)
- **Additional**: `addalert(message)`, `unpay(hours)`, `touches(startTime, endTime)`, `isedited(property)`, `tomorrow(0)`, `yesterday(0)`, `overlaps(startTime, endTime)`, `overlap(start1, end1, start2, end2)`, `addentry("type", amount, "category")`, `otrules("OT Rule Name")`
- **Accrual**: `accrueup("Bucket", amount, max, vestDate, expDate)`, `accruedown("Bucket", amount, min)`, `getbalance("Bucket")`, `setbalance("Bucket", value)`

### Global Timecard Properties (NOT object-based)
These properties are global and do NOT use object notation:
- `payrate`, `isfirsttoday`, `islasttoday`, `hours`, `minutes`, `seconds`, `breakseconds`, `minutesout` or `minutesout(false)`, `minutestil`, `punchset`, `category`, `otcategory`, `punchdate`, `intime`, `outtime`, `indt`, `outdt`, `inismissing`, `outismissing`, `inispresent`, `outispresent`, `istimes`, `ishours`, `ispayonly`, `inisedited`, `outisedited`, `isedited`, `hourstopunch`, `hourstopunchot`, `linetonow`, `inip`, `outip`

### Variables
- **Local variables**: Start with `$` (e.g., `$myvar = 10`)
- **Global variables**: No prefix, just name it (e.g., `myvar = 10`)
- Variables are case-insensitive

### Operators (NOT functions - no parentheses)
- Logical: `and`, `or`, `&&`, `||`
- Comparison: `==`, `!=`, `<>`, `>`, `>=`, `<`, `<=`
- Arithmetic: `+`, `-`, `*`, `/`, `\` (integer division), `%`, `mod`
- String: `contains`, `startswith`, `endswith` (these are operators, NOT functions)
- Assignment: `=`, `==` (both work for equality, `==` preferred)

### Syntax Rules
- **If statements**: Use curly braces `{}` for code blocks
- **Case insensitive**: Everything is case-insensitive
- **Spacing**: Braces and parentheses can have spaces or not
- **Comments**: `//` for single-line, `/* */` for multi-line
- **Time literals**: Can be strings `"7:00am"` or literals `7:00am` (both valid)
- **Time formats**: `7:00am`, `7:00:00am`, `19:00`, `19:00:00` (24-hour format)
- **Negative numbers**: Must be written as `0-1` or as string `"-1"` (not just `-1`)

### Special If Statement Rules
- **Top-level if-statements** with `else if` MUST have an `else` (script fails without it)
- **Nested if-statements** with `else if` do NOT need an `else`

### Round Functions
- Time rounding: `round("7:45am-8:00am=8:00am")` or `round("N10")` (N=nearest, U=up, D=down, C=company, I=in only, O=out only)
- Math rounding: `round(5.555)` returns 6 (different from time rounding)
- `roundtoschedule(a1, a2, b1, b2)` - rounds to employee's schedule

### Examples of CORRECT Usage
```
// CORRECT - object-based properties
if (employee.department == "OPS") {
  addalert("Department is OPS");
}

// CORRECT - reportingdate object
if (reportingdate.date > "2024-01-01") {
  addalert("Date is after 2024-01-01");
}

// CORRECT - global properties (no object prefix)
if (intime > "8:00am") {
  addalert("Late");
}

// CORRECT - global functions (no object prefix)
$result = dateadd("mm", 6, employee.startdate);

// CORRECT - case insensitive
IF (EMPLOYEE.DEPARTMENT == "OPS") {
  ADDALERT("Works");
}
```

### Examples of INCORRECT Usage
```
// WRONG - missing object prefix
if (firstname == "John") { }  // Should be: employee.firstname

// WRONG - missing object prefix
if (date > "2024-01-01") { }  // Should be: reportingdate.date

// WRONG - using object notation for global functions
employee.dateadd(...)  // Should be: dateadd(...)

// WRONG - using object notation for global properties
employee.intime  // Should be: intime (no object prefix)
```

## When Suggesting Code

1. **ALWAYS use object notation** for employee and reportingdate properties
2. **NEVER suggest properties without object prefix** (e.g., never suggest `firstname`, always `employee.firstname`)
3. **Remember case-insensitivity** - accept any case variation
4. **Distinguish between object properties and global properties/functions**
5. **Use correct operator syntax** - `contains`, `startswith`, `endswith` are operators, not functions
6. **Include `else` clause** for top-level if-statements with `else if`
7. **Use proper time rounding syntax** - distinguish between time rounding and math rounding

## Common Mistakes to Avoid

- ❌ `firstname` → ✅ `employee.firstname`
- ❌ `date` → ✅ `reportingdate.date`
- ❌ `employee.intime` → ✅ `intime`
- ❌ `contains(employee.department, "OPS")` → ✅ `employee.department contains "OPS"`
- ❌ `if (condition) { } else if (condition2) { }` (top-level, missing else) → ✅ Must include `else`
- ❌ `round(5.555)` for time → ✅ Use `round("N10")` for time rounding

## Language Limitations

- No switch statements
- No loops (for, while, etc.)
- No function definitions (only built-in functions)
- Limited to if/else if/else conditionals

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JeremyFail) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
