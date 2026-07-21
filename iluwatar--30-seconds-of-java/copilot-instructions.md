## 30-seconds-of-java

> 30 Seconds of Java is a curated collection of reusable, copy-pasteable Java 21 snippets designed to be understood in 30 seconds or less. These snippets cover a wide range of common tasks and are automatically published to the project's website via the `README.md` file.

# 30 Seconds of Java - Priming Context for AI Agents

## Quick Overview
30 Seconds of Java is a curated collection of reusable, copy-pasteable Java 21 snippets designed to be understood in 30 seconds or less. These snippets cover a wide range of common tasks and are automatically published to the project's website via the `README.md` file.

## Stack
- **Language**: Java 21
- **Build System**: Gradle 8.x
- **Testing**: JUnit 5 (Jupiter), Hamcrest
- **Quality Assurance**: Checkstyle, SonarCloud

## Trusted Sources
- [Java 21 SE API Documentation](https://docs.oracle.com/en/java/javase/21/docs/api/index.html)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Gradle User Manual](https://docs.gradle.org/current/userguide/userguide.html)
- [Hamcrest Matchers Reference](http://hamcrest.org/JavaHamcrest/javadoc/2.2/org/hamcrest/Matchers.html)

## Structure
- `/src/main/java/{category}/`: Source code for the snippets, organized by category (e.g., `string`, `math`, `network`).
- `/src/test/java/{category}/`: Unit tests for each snippet.
- `README.md`: The main documentation file containing all snippets, used for website generation.
- `build.gradle`: Project dependencies and build configuration.

## Patterns
- **Stateless Utility Methods**: Implement snippets as `public static` methods within a dedicated class.
- **Locale Independence**: Always use `Locale.ENGLISH` when formatting strings (e.g., `String.format`) to ensure consistent behavior across different environments.
- **Modern Java Features**: Leverage Java 21 features (e.g., `var`, `Stream` API, modern `HttpClient`) where appropriate.
- **Focused Scope**: Each snippet class should perform exactly one task or a small set of highly related tasks.

## Anti-patterns
- **External Dependencies**: Do not add third-party libraries; rely exclusively on the Java Standard Library.
- **Global State**: Snippets must not maintain or depend on mutable global state.
- **Interactive Logic**: Avoid snippets that require terminal input or user interaction.
- **Untested Code**: Every snippet must be accompanied by a comprehensive JUnit test.

## Example Snippet
A complete snippet implementation consists of three parts:

1. **Source Code** (`src/main/java/string/FormatBytesSnippet.java`):
```java
public class FormatBytesSnippet {
  public static String formatBytes(long bytes) {
    double kb = 1024;
    // ... logic ...
    return String.format(Locale.ENGLISH, "%.2f MB", bytes / mb);
  }
}
```

2. **Test File** (`src/test/java/string/FormatBytesSnippetTest.java`):
```java
class FormatBytesSnippetTest {
  @Test
  void formatBytes() {
    assertEquals("1.46 MB", FormatBytesSnippet.formatBytes(1536000));
  }
}
```

3. **README.md Entry**:
```markdown
### Format Bytes

```java
// [Full code of the snippet]
```
```

---
> Source: [iluwatar/30-seconds-of-java](https://github.com/iluwatar/30-seconds-of-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
