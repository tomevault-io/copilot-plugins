## rules-quarkus-skills

> > **Expert AI Agent Skills for Modern Java + Quarkus + Bazel Development**

# rules-quarkus-skills

> **Expert AI Agent Skills for Modern Java + Quarkus + Bazel Development**
>
> This extension provides specialized knowledge for building high-performance Java applications, covering the entire stack from language primitives to cloud-native deployment.

---

## 📦 Available Skills

### 1. java-expert
**Keyword:** `java`  
**Description:** High-performance Modern Java (21+) expertise

**When to activate:**
- General Java development and logic questions
- Performance tuning (JFR, G1GC)
- Clean Architecture and testing patterns

### 2. maven-expert
**Keyword:** `maven`  
**Description:** Expert knowledge for Apache Maven and dependency management

**When to activate:**
- Managing external dependencies and BOMs
- Resolving transitive dependency conflicts
- Migrating projects from Maven to Bazel

### 3. gradle-expert
**Keyword:** `gradle`  
**Description:** Expert knowledge for Gradle Build Tool and performance

**When to activate:**
- Groovy and Kotlin DSL build configuration
- Multi-project builds and dependency management
- Migrating projects from Gradle to Bazel

### 4. vertx-expert
**Keyword:** `vertx`  
**Description:** Reactive foundation expertise (Quarkus engine)

**When to activate:**
- Deep reactive programming and Event Loop questions
- High-performance non-blocking I/O
- Standalone reactive service development

### 5. graalvm-expert
**Keyword:** `graalvm`  
**Description:** Native Image (AOT) and Polyglot runtime expertise

**When to activate:**
- Native compilation and GraalVM configuration
- Troubleshooting reflection/resource issues in native binaries
- Polyglot multi-language integration

### 6. rules-quarkus
**Keyword:** `rules-quarkus`  
**Description:** Expert knowledge for the rules_quarkus Bazel build system

**When to activate:**
- Building Quarkus apps with Bazel
- Troubleshooting augmentation errors
- Adding new Quarkus extensions to BUILD files

### 7. quarkus-expert
**Keyword:** `quarkus`  
**Description:** High-performance Quarkus framework expertise

**When to activate:**
- General Quarkus development and CDI patterns
- Reactive programming with Mutiny
- Extension development and Dev Services

### 8. quarkus-debug
**Keyword:** `quarkus-debug`
**Description:** Advanced troubleshooting and deep debugging for Quarkus

**When to activate:**
- Debugging reactive/asynchronous code and Mutiny
- Native Image (AOT) troubleshooting and GDB/LLDB
- Investigating build-time augmentation failures
- Fixing ClassLoader and memory leak issues

### 9. refactoring-expert
**Keyword:** `refactoring`  
**Description:** Expert knowledge for Clean Code, Refactoring, and Design Patterns based on Refactoring Guru

**When to activate:**
- Code reviews and quality audits
- Applying Design Patterns (Creational, Structural, Behavioral)
- Identifying and fixing Code Smells
- Improving legacy code maintainability

### 10. code-author
**Keyword:** `author`
**Description:** Professional code change authoring following Google's Engineering Practices

**When to activate:**
- Drafting PR/CL descriptions
- Organizing changes into small, reviewable units
- Responding to code review feedback

### 11. code-reviewer
**Keyword:** `reviewer`
**Description:** High-quality code review following Google's Engineering Practices

**When to activate:**
- Conducting code reviews
- Providing constructive, mentoring-focused feedback
- Assessing if a change is ready for LGTM

### 12. bazel-expert
**Keyword:** `bazel`  
**Description:** Bazel rules and Starlark best practices

**When to activate:**
- Writing custom Bazel rules and macros
- Optimizing build performance and hermeticity
- Bzlmod and external dependency management

---

## 🚀 Installation

### Option 1: Copy to skills directory

```bash
# For Gemini CLI
cp -r .agent-skills/* ~/.gemini/skills/

# For Qwen Code  
cp -r .agent-skills/* ~/.qwen/skills/

# For Claude Code
cp -r .agent-skills/* ~/.claude/skills/
```

### Option 2: Activate directly in session

```bash
# Example: Activate core Java and Quarkus integration
activate_skill .agent-skills/java-expert
activate_skill .agent-skills/rules-quarkus
```

---

## 📁 Structure

```
.agent-skills/
├── java-expert/       # Modern Java (21+)
├── maven-expert/      # Dependency Management (Maven)
├── gradle-expert/     # Dependency Management (Gradle)
├── vertx-expert/      # Reactive Foundation
├── graalvm-expert/    # Native & Polyglot
├── rules-quarkus/     # Quarkus + Bazel
├── quarkus-expert/    # Quarkus Framework
├── quarkus-debug/     # Quarkus Deep Troubleshooting
├── refactoring-expert/# Refactoring & Design Patterns
├── code-author/       # Google-style PR Authoring
├── code-reviewer/     # Google-style Code Review
└── bazel-expert/      # Bazel Build System
```

---

## 🔗 Related Projects

- **rules_quarkus:** https://github.com/kinhluan/rules_quarkus
- **Quarkus:** https://quarkus.io
- **Vert.x:** https://vertx.io
- **Gradle:** https://gradle.org
- **GraalVM:** https://www.graalvm.org
- **Bazel:** https://bazel.build

---

## License

MIT

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kinhluan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kinhluan)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
