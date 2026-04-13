## toolbox

> This document outlines coding standards and best practices for developing Automation Anywhere 360 Bot Commands in this project.

# AA360 Bot Command Development Standards

This document outlines coding standards and best practices for developing Automation Anywhere 360 Bot Commands in this project.

---

## Table of Contents

1. [Return Label Standards](#return-label-standards)
2. [Boolean Input Standards](#boolean-input-standards)
3. [Annotation Guidelines](#annotation-guidelines)
4. [Parameter Ordering](#parameter-ordering)
5. [Class Naming Standards](#class-naming-standards)
6. [Complete Examples](#complete-examples)
7. [Testing Standards](#testing-standards)
8. [Refactoring Checklist](#refactoring-checklist)
9. [Common Mistakes](#common-mistakes)
10. [Quick Reference Tables](#quick-reference-tables)

---

## Return Label Standards

### Required for All Actions/Properties

All `@BotCommand` actions and properties that return values MUST include a `return_label` attribute.

### Pattern

Use the pattern: **"Assign \<specific_output\> to"**

The `<specific_output>` should clearly describe what is being returned, not just generic terms like "value" or "result".

### Examples

#### ✅ Good Examples
```java
return_label = "Assign encoded URL to"
return_label = "Assign sorted list to"
return_label = "Assign maximum number to"
return_label = "Assign modified table to"
return_label = "Assign file content to"
```

#### ❌ Bad Examples
```java
return_label = "Assign the output to variable"  // Too generic
return_label = "Assign value to variable"       // Too generic
return_label = ""                                // Missing entirely
```

### Full Example
```java
@CommandPkg(
    label = "Get sanitized filename",
    name = "getSanitizedFileName",
    group_label = "File",
    return_description = "Sanitized filename",
    return_required = true,
    return_label = "Assign sanitized filename to",  // Clear and specific
    return_type = DataType.STRING
)
```

---

## Boolean Input Standards

### Use BOOLEAN Type, Not Checkboxes

For binary true/false choices, use **`AttributeType.BOOLEAN`** instead of `AttributeType.CHECKBOX`.

### Why BOOLEAN Type?

1. **Clarity**: Boolean type with explicit options makes both choices visible to the user
2. **Discoverability**: Users can see what both choices mean without trial and error
3. **Better UX**: Users understand the impact of their selection immediately
4. **Consistency**: Maintains a uniform pattern across all boolean inputs

### Exceptions

**Keep checkboxes** only for:
- **Independent feature flags** where multiple options can be selected simultaneously
- **Regex pattern flags** (case-insensitive, multiline, dotall, etc.)
- **HTML generation options** where users may want combinations of features

### Pattern

```java
@Idx(index = "3", type = AttributeType.BOOLEAN)
@Pkg(label = "Clear descriptive label (what happens when true)",
    description = "Detailed explanation starting with 'Controls whether...' " +
        "When false (default): Describe false state behavior. " +
        "When true: Describe true state behavior. " +
        "Include concrete examples showing the difference.",
    default_value = "false",
    default_value_type = DataType.BOOLEAN)
@NotEmpty
Boolean parameterName
```

**Key Points:**
- **Label**: Should describe what happens when the parameter is **true** (the true state)
- **Description**: Must explain both states using "When false:" and "When true:" format
- **No options array**: Unlike RADIO/SELECT, BOOLEAN type doesn't use options
- **Always include examples**: Show concrete values for both states

### Examples by Category

#### Date Actions
```java
@Idx(index = "3", type = AttributeType.BOOLEAN)
@Pkg(label = "Include same day if it matches",
    description = "Controls whether to include the input date if it already matches the target day of week. " +
        "When false (default): Returns strictly the next occurrence (e.g., Input is Monday, target is Monday → Returns next Monday, 7 days later). " +
        "When true: Returns the same day if it matches (e.g., Input is Monday, target is Monday → Returns the same Monday).",
    default_value = "false",
    default_value_type = DataType.BOOLEAN)
@NotEmpty
Boolean includeInputIfMatch
```

#### URL Actions
```java
@Idx(index = "2", type = AttributeType.BOOLEAN)
@Pkg(label = "Decode parameter names",
    description = "Decodes URL-encoded parameter names. " +
        "When false (default): Keeps names as-is (e.g., 'user%20name' remains 'user%20name'). " +
        "When true: Decodes names (e.g., 'user%20name' → 'user name').",
    default_value = "false",
    default_value_type = DataType.BOOLEAN)
@NotEmpty
Boolean decodeName
```

---

## Annotation Guidelines

### Description Field

**Rules:**
- ✅ **Capitalization**: Always start with a capital letter
- ✅ **Tense**: Use third-person singular present tense ("Finds", "Gets", "Returns", "Calculates")
- ✅ **Structure**: Use descriptive fragments (no period at end)
- ✅ **Detail**: Be specific and accurate

```java
✅ GOOD:
description = "Finds the last day of the year for a given date"
description = "Returns all keys from a dictionary as a list"
description = "Calculates duration between two datetime values"

❌ BAD:
description = "finds the last day" // Lowercase
description = "Find the last day" // Imperative mood
description = "Change the text case" // Not specific enough
```

### Label Field

**Rules:**
- ✅ **Capitalization**: Use Title Case
- ✅ **Acronyms**: ALL CAPS for well-known acronyms (URL, HTML, XML, CSV, JSON, TOTP)
- ✅ **Pattern**: Action + Object or Descriptive Phrase

```java
✅ GOOD:
label = "Last Day Of Year"
label = "Get URL Protocol"
label = "Encode For HTML"

❌ BAD:
label = "Last day of year" // Not title case
label = "Get Url Protocol" // Acronym not all caps
```

### Node Label Field

The `node_label` is automatically prefixed with the `label`, so it must flow naturally.

**Rules:**
- ✅ **Prepositions**:
  - `from` → extraction/source: `"from {{dict}}"`, `"from {{file}}"`
  - `of` → properties/parts: `"of {{number}}"`, `"of {{date}}"`
  - `for` → operating on behalf: `"for {{inputDate}}"`
  - `in` → location/container: `"in {{list}}"`, `"in {{table}}"`
- ✅ **Ending**: Always `"and assign to {{returnTo}}"` (NOT "and assign **the result** to")
- ✅ **Capitalization**: Lowercase when following variables
- ✅ **Spacing**: No leading/trailing spaces

```java
✅ GOOD:
node_label = "from {{dictionary}} and assign to {{returnTo}}"
node_label = "of {{inputNumber}} and assign to {{returnTo}}"
node_label = "Get keys from {{dictionary}} and assign to {{returnTo}}"

❌ BAD:
node_label = "from {{dict}} and assign the result to {{returnTo}}" // Wrong ending
node_label = " of {{inputDate}} and assign to {{returnTo}}" // Leading space
node_label = "From {{dictionary}} and assign to {{returnTo}}" // Capital after prefix
```

### Description Pattern for BOOLEAN Parameters

```java
description = "Controls whether <what is being controlled>. " +
    "When false (default): <describe false state with example>. " +
    "When true: <describe true state with example>."
```

Or for simpler cases:

```java
description = "<Action description>. " +
    "When false (default): <false state behavior>. " +
    "When true: <true state behavior>."
```

**Examples:**

```java
// ✅ GOOD - Clearly explains both states with examples
description = "Decodes URL-encoded parameter names. " +
    "When false (default): Keeps names as-is (e.g., 'user%20name' remains 'user%20name'). " +
    "When true: Decodes names (e.g., 'user%20name' → 'user name')."

// ❌ BAD - Only describes one state
description = "If true, parameter values will be URL-encoded"

// ❌ BAD - No examples
description = "Match same day"
```

### Parameter Ordering

**HELP Parameters Must Be Last:**

When a bot command includes a HELP parameter (using `@Idx(type = HELP)` and `@Inject`), it **MUST** always be the last parameter in the method signature.

**Rules:**
- ✅ **HELP parameters**: Always use the highest index number
- ✅ **Functional parameters**: Come before HELP
- ✅ **Ordering**: Business logic parameters → HELP

**Rationale:**
- HELP parameters are informational only (injected, not user-provided)
- Keeping them last maintains clean separation between functional and informational parameters
- Consistent positioning improves code readability

```java
✅ GOOD:
public static RecordValue action(
    @Idx(index = "1", type = AttributeType.VARIABLE)
    Record inputRecord,

    @Idx(index = "2", type = ENTRYLIST)
    List<Value> values,

    @Idx(index = "3", type = AttributeType.BOOLEAN)
    Boolean addIfMissing,

    @Idx(index = "4", type = AttributeType.BOOLEAN)
    Boolean caseSensitive,

    @Idx(index = "5", type = HELP)  // HELP is last
    @Inject
    String help
)

❌ BAD:
public static RecordValue action(
    @Idx(index = "1", type = AttributeType.VARIABLE)
    Record inputRecord,

    @Idx(index = "2", type = ENTRYLIST)
    List<Value> values,

    @Idx(index = "3", type = HELP)  // HELP in middle - WRONG
    @Inject
    String help,

    @Idx(index = "4", type = AttributeType.BOOLEAN)
    Boolean addIfMissing
)
```

---

## Class Naming Standards

### General Principles

1. **PascalCase**: All class names (e.g., `GetMaxNumber`, `IsRecordEmpty`)
2. **No Underscores**: Never use underscores
3. **Descriptive**: Names should clearly indicate purpose
4. **Consistent**: Follow patterns within each category
5. **Acronyms**: ALL CAPS for well-known ones (URL, HTML, UUID, etc.)

---

### Conditionals (`conditionals` package)

Conditionals check conditions and return boolean values.

#### Patterns

**State/Property Checks → `Is...`**

For checking state or property of a single item:

```java
✅ GOOD:
IsRecordEmpty
IsRecordBlank
IsTableSchemaEmpty
IsFileWritable

❌ BAD:
RecordIsEmpty // Wrong word order
EmptyRecord // Missing "Is"
CheckIfRecordEmpty // Redundant
```

**Existence/Containment → `Has...`**

For checking if something exists or contains:

```java
✅ GOOD:
HasRecordHeader
HasTableHeader
HasValue

❌ BAD:
HasHeaderInRecordSchema // Too verbose
RecordHasHeader // Wrong word order
```

**Multi-Item Comparisons → `Are...`**

For checking conditions across multiple items:

```java
✅ GOOD:
AreValuesEqual
AreItemsSorted

❌ BAD:
IsValuesEqual // "Is" is singular
ValuesAreEqual // Wrong word order
```

**String Pattern Matching**

```java
✅ GOOD:
StringBeginsWith
StringEndsWith
StringContains
MatchesPattern

❌ BAD:
BeginsWithValidation // "Validation" suffix redundant
EndsWithValidation // "Validation" suffix redundant
```

---

### Actions (`actions` package)

**All action class names MUST be verb-based.**

#### Verb Selection Guide

| Operation | Verb | Examples |
|-----------|------|----------|
| Retrieve/Extract | `Get` | `GetMaxNumber`, `GetFirstDayOfMonth` |
| Search/Filter | `Find` | `FindCommonKeys`, `FindRegexMatch` |
| Compute | `Calculate` | `CalculateDateTimeDuration`, `CalculateSumSubset` |
| Transform Type | `Convert` | `ConvertDateTimeToEpoch`, `ConvertTableToHTML` |
| Create New | `Generate` | `GenerateTOTP`, `GenerateUUID` |
| Modify | Specific | `Add`, `Remove`, `Update`, `Set`, `Merge` |
| Match/Replace | Specific | `MatchRegex`, `ReplaceTemplate` |
| Format | Specific | `RoundNumber`, `PadText` |

#### Examples

```java
✅ GOOD:
GetMaxNumber
GetMinNumber
GetNextDayOfWeek
GetPreviousWorkDate
CalculateDateTimeDuration
CalculateSumSubset
ConvertDateTimeToEpoch
ConvertTableToHTML
RoundNumber
MatchRegex
ReplaceTemplate
PadText

❌ BAD:
MaxNumber // Missing verb
NextDayOfWeek // Adjective form, needs "Get"
Rounding // Gerund/noun form, should be "RoundNumber"
DateTimeDuration // Missing verb, should be "CalculateDateTimeDuration"
RegexMatch // Noun form, should be "MatchRegex"
TemplateReplacer // Noun form, should be "ReplaceTemplate"
TextPad // Noun form, should be "PadText"
ColumnToList // Missing verb, should be "ConvertColumnToList"
```

---

### Properties (`properties` package)

Properties transform data and return modified values.

#### Patterns

**Transformations → Direct Verb Form**

```java
✅ GOOD:
Base64Encode
Base64Decode
EncodeURL
DecodeURL
EscapeHTML
UnescapeXML
Strip
StripLeading
NormalizeSpace

❌ BAD:
Base64Encoder // Noun form
HTMLEscape // Type should come before verb
```

**Extractions → `Get...` (Optional)**

```java
✅ BOTH ACCEPTABLE:
GetFileExtension  OR  FileExtension
GetProtocol       OR  Protocol
GetHostname       OR  Hostname

// Recommendation: Use "Get" prefix for clarity
```

---

### Variables (`variables` package)

Variables provide values. **Noun forms only.**

#### Patterns

**Simple Values → Descriptive Nouns**

```java
✅ GOOD:
Username
AuthToken
UUID
EmptyString
EmptyRecord
EmptyTable

❌ BAD:
GetUsername // No verbs for variables
EmptyDataTable // Redundant "Data"
UUIDValue // Redundant "Value"
```

**Paths/Directories → Descriptive Nouns**

```java
✅ GOOD:
UserHomeDirectory
UserDownloadDirectory

❌ BAD:
GetUserHome // No verbs
HomeDir // No abbreviations
```

**State Variables → Adjective + Noun**

```java
✅ GOOD:
EmptyString
EmptyRecord
DefaultValue
CurrentUser

❌ BAD:
EmptyDataTable // Redundant "Data"
NullString // Use "Empty"
```

---

## Complete Examples

### Action Example

```java
@BotCommand
@CommandPkg(
    label = "Get Maximum Number",
    description = "Finds the maximum number in a list of numbers",
    icon = "Number.svg",
    name = "getMaxNumber",
    group_label = "Number",
    node_label = "in list {{numbers}} and assign to {{returnTo}}",
    return_label = "Assign maximum number to",
    return_description = "The largest number from the list",
    return_required = true,
    return_type = DataType.NUMBER
)
public class GetMaxNumber {
    // Implementation
}
```

### Conditional Example

```java
@BotCommand(commandType = BotCommand.CommandType.Condition)
@CommandPkg(
    description = "Checks if all cell values in the Record are empty (null or zero-length string)",
    name = "isRecordEmpty",
    label = "Record: All cell values are empty",
    node_label = "If all cell values in {{rowRecord}} are empty"
)
public class IsRecordEmpty {
    // Implementation
}
```

### Conditional with Boolean Parameter Example

```java
@BotCommand(commandType = Condition)
@CommandPkg(
    description = "Checks if table has all specified column headers in schema",
    name = "hasTableColumns",
    label = "Table: Has all specified columns",
    icon = "Table.svg",
    node_label = "If table {{inputTable}} {{conditiontype}} all columns {{columnNames}}"
)
public class HasTableColumns {

  @ConditionTest
  public static Boolean validate(
      @Idx(index = "1", type = AttributeType.VARIABLE)
      @Pkg(label = "Table")
      @NotEmpty
      @VariableType(value = DataType.TABLE)
      Table inputTable,

      @Idx(index = "2", type = AttributeType.SELECT, options = {
          @Idx.Option(index = "2.1", pkg = @Pkg(label = "Has", value = "CONTAINS")),
          @Idx.Option(index = "2.2", pkg = @Pkg(label = "Does not have", value = "NOTCONTAINS"))
      })
      @Pkg(label = "Condition", default_value = "CONTAINS", default_value_type = STRING)
      @NotEmpty
      String conditiontype,

      @Idx(index = "3", type = AttributeType.LIST)
      @Pkg(label = "Column names", description = "List of column names to check for existence in the table")
      @NotEmpty
      @ListType(STRING)
      List<StringValue> columnNames,

      @Idx(index = "4", type = AttributeType.BOOLEAN)
      @Pkg(label = "Case-sensitive comparison",
          description = "Controls whether column name comparison should be case-sensitive. " +
              "When false (default): Case-insensitive (e.g., 'Name' matches 'name', 'NAME'). " +
              "When true: Case-sensitive (e.g., 'Name' does not match 'name').",
          default_value = "false",
          default_value_type = DataType.BOOLEAN)
      @NotEmpty
      Boolean caseSensitive
  ) {
      // Implementation
  }
}
```

### Property Example

```java
@BotCommand
@CommandPkg(
    label = "Escape For HTML",
    name = "escapeHTML",
    description = "Escapes string for HTML entity encoding",
    group_label = "String",
    icon = "String.svg",
    node_label = "Escape {{sourceString}} for HTML and assign to {{returnTo}}",
    return_type = DataType.STRING,
    return_required = true,
    return_label = "Assign escaped HTML to",
    property_name = "escapeHTML",
    property_description = "Escapes string for HTML entity encoding",
    property_type = DataType.STRING,
    property_return_type = DataType.STRING
)
public class EscapeHTML {
    // Implementation
}
```

### Variable Example

```java
@BotCommand(commandType = BotCommand.CommandType.Variable)
@CommandPkg(
    description = "Empty string value",
    name = "EmptyString",
    label = "Empty String Value",
    variable_return_type = DataType.STRING
)
public class EmptyString {
    // Implementation
}
```

---

## Testing Standards

### Test Artifact Management

All tests that create files or directories must properly manage cleanup to avoid polluting the file system.

#### Pattern: Use `@BeforeMethod` and `@AfterMethod`

**Most tests should use `@BeforeMethod/@AfterMethod`** to ensure each test runs with fresh test data and proper isolation.

```java
@BeforeMethod
public void setUp() throws IOException {
    // Create temporary test directory
    tempDir = Files.createTempDirectory("file-tests");

    // Create test files/data
    testFile = tempDir.resolve("test.txt").toFile();
    // ...
}

@AfterMethod
public void tearDown() throws IOException {
    // Clean up test files and directories
    if (tempDir != null && Files.exists(tempDir)) {
        Files.walk(tempDir)
            .sorted((a, b) -> b.compareTo(a)) // Delete files before directories
            .forEach(path -> {
                try {
                    Files.delete(path);
                } catch (IOException e) {
                    // Ignore cleanup errors
                }
            });
    }
}
```

#### Pattern: Use `@BeforeClass` and `@AfterClass` (Rare)

**Only use `@BeforeClass/@AfterClass`** when test data can be safely shared across ALL tests in the class.

- Methods must be `static`
- More efficient but less isolation
- Use sparingly - only when appropriate

```java
private static Path tempDir;

@BeforeClass
public static void setup() throws IOException {
    tempDir = Files.createTempDirectory("file-tests");
    // Create shared test data
}

@AfterClass
public static void cleanup() throws IOException {
    if (tempDir != null && Files.exists(tempDir)) {
        Files.walk(tempDir)
            .sorted((a, b) -> b.compareTo(a)) // Delete files before directories
            .forEach(path -> {
                try {
                    Files.delete(path);
                } catch (IOException e) {
                    // Ignore cleanup errors
                }
            });
    }
}
```

#### Cleanup Guidelines

1. **Directory Cleanup**: Use `Files.walk()` with reverse sorting
   ```java
   Files.walk(tempDir)
       .sorted((a, b) -> b.compareTo(a)) // Delete files before directories
       .forEach(path -> {
           try {
               Files.delete(path);
           } catch (IOException e) {
               // Ignore cleanup errors
           }
       });
   ```

2. **Single File Cleanup**: Use `Files.deleteIfExists()` or `file.delete()`
   ```java
   Files.deleteIfExists(testFile.toPath());
   // or
   if (testFile != null && testFile.exists()) {
       testFile.delete();
   }
   ```

3. **Temporary Directory Naming**: Use consistent prefix `"file-tests"` for file operations
   ```java
   tempDir = Files.createTempDirectory("file-tests");
   ```

4. **Temporary File Naming**: Use `File.createTempFile()` with descriptive prefix
   ```java
   testFile = File.createTempFile("testfile", ".txt");
   testFile.deleteOnExit(); // Fallback cleanup
   ```

5. **Error Handling**: Always catch and ignore cleanup exceptions
   - Cleanup failures should not fail tests
   - Use comments: `// Ignore cleanup errors`

#### Complete Example

```java
package file;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import org.testng.Assert;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.Test;

public class ExampleTest {

  private Path tempDir;

  @BeforeMethod
  public void setUp() throws IOException {
    // Create temporary test directory structure
    tempDir = Files.createTempDirectory("file-tests");

    // Create test files
    Files.createFile(tempDir.resolve("test.txt"));
    Files.createFile(tempDir.resolve("data.csv"));
  }

  @AfterMethod
  public void tearDown() throws IOException {
    // Clean up test files and directories
    if (tempDir != null && Files.exists(tempDir)) {
      Files.walk(tempDir)
          .sorted((a, b) -> b.compareTo(a)) // Delete files before directories
          .forEach(path -> {
            try {
              Files.delete(path);
            } catch (IOException e) {
              // Ignore cleanup errors
            }
          });
    }
  }

  @Test
  public void testSomething() {
    // Test implementation
  }
}
```

---

## Refactoring Checklist

### Annotation Checklist

- [ ] `description` starts with capital letter
- [ ] `description` uses present tense third-person ("Finds", "Gets", "Returns")
- [ ] `description` is specific and accurate
- [ ] `label` uses Title Case
- [ ] `label` has ALL CAPS for acronyms (URL, HTML, XML, JSON, CSV, TOTP)
- [ ] `node_label` uses correct preposition (`from`, `of`, `for`, `in`)
- [ ] `node_label` ends with `"and assign to {{returnTo}}"`
- [ ] `node_label` has no leading/trailing spaces
- [ ] `node_label` starts lowercase when following variables
- [ ] `return_label` uses pattern "Assign \<specific_output\> to"
- [ ] `property_description` follows same rules as `description`

### Class Naming Checklist

**Conditionals:**
- [ ] Uses `Is...` for state checks
- [ ] Uses `Has...` for existence checks
- [ ] Uses `Are...` for multi-item comparisons
- [ ] Avoids "...Validation" suffix

**Actions:**
- [ ] Uses verb-based naming (not nouns/adjectives)
- [ ] Uses `Get` for retrieval
- [ ] Uses `Find` for search/filter
- [ ] Uses `Calculate` for computations
- [ ] Uses `ConvertXToY` for type conversions

**Properties:**
- [ ] Uses direct verb form for transformations
- [ ] Optionally uses `Get` for extractions
- [ ] Includes type suffix when applicable

**Variables:**
- [ ] Uses noun forms only
- [ ] Uses descriptive adjectives when needed
- [ ] Uses ALL CAPS for well-known acronyms

### Boolean Parameter Checklist

- [ ] Use `AttributeType.BOOLEAN` for boolean choices (unless they're independent feature flags)
- [ ] **BOOLEAN type does NOT use options array** - just `@Idx(index = "X", type = AttributeType.BOOLEAN)`
- [ ] Label should describe what happens when parameter is **true**
- [ ] Include detailed description with "When false:" and "When true:" format
- [ ] Add concrete examples showing the difference between both states
- [ ] Mark the default state with "(default)" in the description
- [ ] Add `@NotEmpty` annotation for required parameters
- [ ] Set appropriate `default_value` and `default_value_type`

### Parameter Ordering Checklist

- [ ] HELP parameters (`@Idx(type = HELP)`) are always last
- [ ] HELP parameters have the highest index number
- [ ] All functional parameters come before HELP

---

## Common Mistakes

| ❌ Mistake | ✅ Correct | Reason |
|-----------|----------|--------|
| `description = "finds the max"` | `description = "Finds the maximum"` | Lowercase start |
| `node_label = "...the result to {{returnTo}}"` | `node_label = "...to {{returnTo}}"` | Wrong ending |
| `node_label = " of {{date}}"` | `node_label = "of {{date}}"` | Leading space |
| `"Get extension of {{file}}"` | `"Get extension from {{file}}"` | Wrong preposition |
| `class MaxNumber` | `class GetMaxNumber` | Action needs verb |
| `class BeginsWithValidation` | `class StringBeginsWith` | Remove "Validation" suffix |
| `label = "Get Url Protocol"` | `label = "Get URL Protocol"` | Acronym not capitalized |
| `description = "Update Cell"` | `description = "Updates a cell value..."` | Too vague |
| `class NextDayOfWeek` | `class GetNextDayOfWeek` | Action needs verb |
| `class TemplateReplacer` | `class ReplaceTemplate` | Use verb form |
| `class EmptyDataTable` | `class EmptyTable` | Remove redundant "Data" |
| `return_label = "Assign the output to variable"` | `return_label = "Assign sorted list to"` | Too generic |
| Using `AttributeType.CHECKBOX` | Using `AttributeType.BOOLEAN` | Boolean inputs should use BOOLEAN type |
| HELP parameter at index 3 (middle) | HELP parameter at highest index (last) | HELP must be last parameter |

---

## Quick Reference Tables

### Class Naming Quick Lookup

| Category | Pattern | Examples | Avoid |
|----------|---------|----------|-------|
| **Conditionals** | | | |
| State check | `Is...` | `IsRecordEmpty`, `IsFileWritable` | `RecordIsEmpty`, `EmptyRecord` |
| Existence | `Has...` | `HasTableHeader`, `HasValue` | `TableHasHeader` |
| Comparison | `Are...` | `AreValuesEqual` | `IsValuesEqual` |
| String match | `String...` | `StringBeginsWith` | `BeginsWithValidation` |
| **Actions** | | | |
| Retrieve | `Get...` | `GetMaxNumber`, `GetNextDayOfWeek` | `MaxNumber`, `NextDayOfWeek` |
| Search | `Find...` | `FindCommonKeys` | `CommonKeys` |
| Calculate | `Calculate...` | `CalculateDuration` | `Duration` |
| Convert | `Convert...` | `ConvertTableToHTML` | `TableToHTML` |
| Modify | Specific | `AddItem`, `UpdateCell` | `ItemAdder` |
| Match/Replace | Specific | `MatchRegex`, `ReplaceTemplate` | `RegexMatch`, `TemplateReplacer` |
| Format | Specific | `RoundNumber`, `PadText` | `Rounding`, `TextPad` |
| **Properties** | | | |
| Transform | Direct verb | `Base64Encode`, `EscapeHTML` | `Base64Encoder` |
| Extract | `Get...` (opt) | `GetFileExtension` | `FileExtensionGetter` |
| **Variables** | | | |
| Value | Noun | `Username`, `UUID` | `GetUsername` |
| State | Adj+Noun | `EmptyString`, `EmptyTable` | `EmptyDataTable` |

### Common Refactoring Patterns

| Old Name (❌) | New Name (✅) | Category |
|--------------|--------------|----------|
| `BeginsWithValidation` | `StringBeginsWith` | Conditional |
| `EndsWithValidation` | `StringEndsWith` | Conditional |
| `HasHeaderInRecordSchema` | `HasRecordHeader` | Conditional |
| `MaxNumber` | `GetMaxNumber` | Action |
| `MinNumber` | `GetMinNumber` | Action |
| `Rounding` | `RoundNumber` | Action |
| `DateTimeDuration` | `CalculateDateTimeDuration` | Action |
| `SumSubset` | `CalculateSumSubset` | Action |
| `NextDayOfWeek` | `GetNextDayOfWeek` | Action |
| `PreviousWorkDate` | `GetPreviousWorkDate` | Action |
| `RegexMatch` | `MatchRegex` | Action |
| `TemplateReplacer` | `ReplaceTemplate` | Action |
| `TextPad` | `PadText` | Action |
| `ZonedDateTimeToEpoch` | `ConvertDateTimeToEpoch` | Action |
| `ColumnToList` | `ConvertColumnToList` | Action |
| `TableToHTML` | `ConvertTableToHTML` | Action |
| `EmptyDataTable` | `EmptyTable` | Variable |

---

## Compliance

**All new bot commands must follow these guidelines.**

Existing commands should be updated to comply during refactoring or maintenance cycles.

---

## Version History

- **2025-01-XX**: Comprehensive standards document created
  - Established return_label pattern (78/78 classes compliant)
  - Migrated from CHECKBOX to BOOLEAN for binary inputs (12 files updated)
  - Standardized description format with "When unchecked/checked" pattern
  - Merged annotation guidelines and class naming standards
  - Added complete examples, refactoring checklists, and quick reference tables

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/A360-Tools)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/A360-Tools)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
