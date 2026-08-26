## llm-planner

> This document provides guidance for LLM agents working on the Polydoc project, a Clojure-based tool for building documentation systems using Pandoc filters.

# Agents Guide for Polydoc Development

This document provides guidance for LLM agents working on the Polydoc project, a Clojure-based tool for building documentation systems using Pandoc filters.

## Project Overview

**Polydoc** is a documentation processing tool that brings Pandoc's powerful filtering capabilities to the JVM/Clojure ecosystem. It aims to provide:

- Advanced Pandoc filters compiled with GraalVM for reduced latency
- JVM-native documentation tooling (no Python/Node.js dependencies)
- Support for multiple document formats via Pandoc
- Interactive documentation viewing with SQLite-powered search
- LLM-driven document processing capabilities

### Key Capabilities

The project implements several types of Pandoc filters:

- **Code execution filters**: Run Clojure, SQLite, JavaScript, Python code blocks
- **Rendering filters**: Process PlantUML diagrams
- **Linting filters**: Check Clojure code with clj-kondo
- **Include filters**: Compose documents from multiple sources
- **Build system**: Generate books with table of contents and full-text search
- **Interactive viewer**: HTTP-based document browser with search

## Allowed Command-Line Tools

As an LLM agent, you are restricted to using **only** these command-line tools:

1. **clj-nrepl-eval** - For evaluating Clojure code via nREPL
2. **clojure-skills** - For searching skills, managing plans, and tracking tasks
3. **clj-paren-repair** - For automatically fixing delimiter errors in Clojure files

All other interactions must happen through the **REPL** using the connected nREPL server (port 7889).

### clj-paren-repair Tool

The `clj-paren-repair` tool automatically detects and fixes delimiter (parentheses, brackets, braces) errors in Clojure files. It's particularly useful after making edits that may have introduced unbalanced delimiters.

**Usage:**
```bash
# Fix delimiter errors in a single file
clj-paren-repair path/to/file.clj

# Fix multiple files at once
clj-paren-repair file1.clj file2.clj file3.clj

# Fix all Clojure files in a directory
clj-paren-repair src/**/*.clj
```

**When to use:**
- After making edits that result in "EOF while reading" errors
- When encountering "Unmatched delimiter" compiler errors
- As a quick fix for parenthesis-related syntax errors
- Before committing changes to ensure proper formatting

**Example:**
```bash
$ clj-paren-repair src/polydoc/viewer/server.clj
Reformatted src/polydoc/viewer/server.clj

clj-paren-repair Results
========================

  src/polydoc/viewer/server.clj: Delimiter errors fixed and formatted [delimiter-fixed, formatted]

Summary:
  Success: 1
  Failed:  0
```

**Note:** The tool uses parinfer-rust (if available) or parinferish as a fallback to intelligently repair delimiters based on indentation and context.

## Fish Shell Usage

This project uses the **fish shell**. Fish has different syntax than bash/zsh, especially for multi-line strings and heredocs.

### ❌ WRONG (bash heredoc syntax doesn't work in fish):

```bash
cat > file.txt << 'EOF'
Line 1
Line 2
EOF
```

### ✅ CORRECT Fish shell patterns:

**1. Multi-line strings in echo:**
```fish
echo "Line 1
Line 2
Line 3"
```

**2. Use printf for complex output:**
```fish
printf '%s\n' "Line 1" "Line 2" "Line 3" > file.txt
```

**3. Use command substitution with echo:**
```fish
echo (echo "Line 1"; echo "Line 2"; echo "Line 3") > file.txt
```

**4. For creating files, use the write tool instead:**
- Prefer the `write` tool for creating files with content
- This avoids shell syntax issues entirely

**5. For clj-nrepl-eval with multi-line Clojure code:**
```fish
# WRONG - heredoc doesn't work
clj-nrepl-eval -p 7889 <<'EOF'
(defn foo [x] x)
EOF

# CORRECT - use double quotes and escape as needed
clj-nrepl-eval -p 7889 "(defn foo [x] (println x) x)"

# CORRECT - for complex code, write to file first, then eval
# Or use parentheses to wrap multi-line Clojure
```

**Key differences from bash:**
- No `<<EOF` heredoc syntax
- Multi-line strings work directly in double quotes
- Newlines in quoted strings are preserved
- Use `printf` for formatted output
- Command substitution uses `()` not `$()`

**Best practice:** For any complex file content creation, use the `write` tool instead of shell commands.

## Dependencies Analysis

### Core Dependencies (deps.edn)

```clojure
{:deps
 {org.clojure/clojure {:mvn/version "1.12.3"}           ; Latest Clojure
  
  ;; CLI & Terminal
  cli-matic/cli-matic {:mvn/version "0.5.4"}           ; CLI interface
  io.github.paintparty/bling {:mvn/version "0.8.8"}    ; Terminal formatting
  
  ;; Data Processing
  org.clojure/data.json {:mvn/version "2.5.1"}         ; JSON handling (Pandoc AST)
  clj-commons/clj-yaml {:mvn/version "1.0.29"}         ; YAML support
  org.clj-commons/pretty {:mvn/version "3.6.7"}        ; Pretty printing
  mvxcvi/puget {:mvn/version "1.3.4"}                  ; Pretty printing
  
  ;; Data Structures & Validation
  metosin/malli {:mvn/version "0.19.2"}                ; Schema validation
  funcool/lentes {:mvn/version "1.3.3"}                ; Functional lenses
  
  ;; Database
  com.github.seancorfield/honeysql {:mvn/version "2.7.1350"}   ; SQL DSL
  com.github.seancorfield/next.jdbc {:mvn/version "1.3.1070"}  ; JDBC wrapper
  org.xerial/sqlite-jdbc {:mvn/version "3.47.1.0"}             ; SQLite driver
  
  ;; Development Tools
  dev.weavejester/hashp {:mvn/version "0.5.1"}         ; Debug printing
  dev.glossa/metazoa {:mvn/version "0.2.298"}          ; Metadata tooling
  com.github.clojure-lsp/clojure-lsp {:mvn/version "2025.08.25-14.21.46"}}}
```

### Development Dependencies (:dev alias)

```clojure
:dev {:extra-deps
      {;; Testing Framework
       lambdaisland/kaocha {:mvn/version "1.91.1392"}
       lambdaisland/kaocha-cloverage {:mvn/version "1.1.89"}
       lambdaisland/kaocha-junit-xml {:mvn/version "1.17.101"}
       nubank/matcher-combinators {:mvn/version "3.9.1"}
       org.clojure/test.check {:mvn/version "1.1.1"}
       com.gfredericks/test.chuck {:mvn/version "0.2.13"}
       
       ;; Development Tools
       io.github.tonsky/clj-reload {:mvn/version "0.9.8"}
       com.stuartsierra.component.repl {:mvn/version "0.2.0"}
       
       ;; Analysis & Linting
       com.github.clojure-lsp/clojure-lsp {:mvn/version "2025.08.25-14.21.46"}
       clj-kondo/clj-kondo {:mvn/version "2022.09.08"}
       
       ;; nREPL
       org.clojure/tools.nrepl {:mvn/version "0.2.11"}}
      :extra-paths ["test" "dev"]}
```

### Project Structure

```
polydoc/
├── src/polydoc/           # Main source code
│   └── main.clj           # Entry point
├── dev/                   # Development namespace
│   ├── user.clj          # User namespace (REPL startup)
│   └── dev.clj           # Dev namespace (tools & helpers)
├── test/                  # Test directory
├── resources/             # Resources
├── classes/               # Compiled classes
├── deps.edn              # Dependencies
└── README.org            # Project documentation
```

## Agent Development Workflow

When working on Polydoc, follow the **Gather-Action-Verify** loop pattern using **only the REPL** and allowed command-line tools.

### Important: Use clj-reload for Namespace Reloading

**Always use `clj-reload` instead of `:reload`** when reloading namespaces after making changes. This project uses [clj-reload](https://github.com/tonsky/clj-reload) for clean, efficient namespace reloading.

```clojure
;; Load clj-reload (already in dev dependencies)
(require '[clj-reload.core :as reload])

;; Reload all changed namespaces (default behavior)
(reload/reload)
;; => {:unloaded [ns1 ns2] :loaded [ns1 ns2]}

;; Options:
(reload/reload {:only :changed})  ; Default: only changed namespaces
(reload/reload {:only :loaded})   ; Reload all loaded namespaces
(reload/reload {:only :all})      ; Reload everything
(reload/reload {:only #"polydoc.*"})  ; Pattern-based reload
```

**Why clj-reload over `:reload`?**
- **Cleaner reloads**: Properly unloads old definitions before reloading
- **Dependency tracking**: Reloads dependent namespaces automatically
- **Better error handling**: Reports what was unloaded/loaded
- **Efficient**: Only reloads what changed by default

**Using with clj-nrepl-eval:**
```bash
clj-nrepl-eval -p 7889 "(require '[clj-reload.core :as reload])"
clj-nrepl-eval -p 7889 "(reload/reload)"
```

### 1. Gather Context

**Before making any changes, explore via REPL:**

```clojure
;; The nREPL server is running on port 7889
;; Use clj-nrepl-eval to interact with it

;; Explore the codebase
(require '[clojure-lsp.api :as lsp-api])

;; Find diagnostics (errors, warnings)
(lsp-api/diagnostics {:namespace '[polydoc.main]})

;; List all loaded namespaces
(all-ns)
(map ns-name (all-ns))

;; Examine a specific namespace
(require '[polydoc.main])
(dir polydoc.main)

;; Get documentation
(doc polydoc.main/some-function)

;; View source code
(source polydoc.main/some-function)

;; Search for symbols
(apropos "filter")
```

**Using clj-nrepl-eval:**

```bash
# Evaluate a single expression
clj-nrepl-eval -p 7889 "(require '[polydoc.main])"

# List namespaces
clj-nrepl-eval -p 7889 "(map ns-name (all-ns))"

# Check function documentation
clj-nrepl-eval -p 7889 "(doc polydoc.main/some-function)"

# Test a function
clj-nrepl-eval -p 7889 "(polydoc.main/some-function test-data)"
```

**Read files strategically:**

- Use the `read` tool to examine source files
- Start with high-level overview (scan defn names)
- Dive into specific functions as needed
- Check related namespaces for dependencies

**Key questions to answer:**

- What is the current implementation?
- What are the dependencies?
- Are there existing tests?
- What's the expected behavior?
- Are there any diagnostics/errors?

### 2. Take Action

**Make focused, incremental changes:**

```clojure
;; Use standard editing tools to modify files
;; - Read the file first
;; - Make targeted edits
;; - Save the file

;; Clean up after editing (via REPL)
(require '[clojure-lsp.api :as lsp-api])
(lsp-api/clean-ns! {:namespace '[polydoc.main]})
(lsp-api/format! {:namespace '[polydoc.main]})

;; For safe renaming
(lsp-api/rename! {:from 'old-fn :to 'new-fn})
```

**Using clj-nrepl-eval for cleanup:**

```bash
# Clean namespace imports
clj-nrepl-eval -p 7889 "(require '[clojure-lsp.api :as lsp-api])"
clj-nrepl-eval -p 7889 "(lsp-api/clean-ns! {:namespace '[polydoc.main]})"

# Format code
clj-nrepl-eval -p 7889 "(lsp-api/format! {:namespace '[polydoc.main]})"
```

**Tool selection guide:**

| Task | Tool | Example |
|------|------|---------|
| Read code | `read` | Read source files |
| Edit code | `edit` or `write` | Modify functions |
| Eval code | `clj-nrepl-eval` | Test via nREPL |
| Rename safely | `lsp-api/rename!` (via REPL) | Updates all references |
| Clean imports | `lsp-api/clean-ns!` (via REPL) | Removes unused requires |
| Format code | `lsp-api/format!` (via REPL) | Applies formatting rules |

### 3. Verify Output

**Always verify changes via REPL:**

```clojure
;; 1. Reload namespaces (IMPORTANT: Use clj-reload for clean reloads!)
(require '[clj-reload.core :as reload])
(reload/reload)  ; Reloads all changed namespaces cleanly

;; clj-reload only reloads :changed namespaces by default (efficient!)
;; Returns: {:unloaded [...] :loaded [...]}

;; Alternative: Reload specific namespace (may have stale state)
(require 'polydoc.main :reload)

;; 2. Test basic functionality
(polydoc.main/your-function test-input)

;; 3. Test edge cases
(polydoc.main/your-function nil)
(polydoc.main/your-function [])
(polydoc.main/your-function "invalid-input")

;; 4. Check for new diagnostics
(lsp-api/diagnostics {:namespace '[polydoc.main]})

;; 5. Run tests (if they exist)
(require 'polydoc.main-test :reload)
(clojure.test/run-tests 'polydoc.main-test)
```

**Using clj-nrepl-eval for verification:**

```bash
# BEST: Use clj-reload for clean namespace reloading
clj-nrepl-eval -p 7889 "(require '[clj-reload.core :as reload])"
clj-nrepl-eval -p 7889 "(reload/reload)"

# clj-reload options:
# (reload/reload) - Default: only changed namespaces
# (reload/reload {:only :loaded}) - Reload all loaded namespaces
# (reload/reload {:only :all}) - Reload everything
# (reload/reload {:only #"polydoc.*"}) - Reload namespaces matching pattern

# Alternative: Reload specific namespace
clj-nrepl-eval -p 7889 "(require 'polydoc.main :reload)"

# Test the function
clj-nrepl-eval -p 7889 "(polydoc.main/your-function test-input)"

# Test edge cases
clj-nrepl-eval -p 7889 "(polydoc.main/your-function nil)"

# Run tests
clj-nrepl-eval -p 7889 "(require 'polydoc.main-test :reload)"
clj-nrepl-eval -p 7889 "(clojure.test/run-tests 'polydoc.main-test)"
```

**Verification checklist:**

- [ ] Namespace reloaded successfully
- [ ] Function works with valid input
- [ ] Edge cases handled (nil, empty, invalid)
- [ ] No new diagnostics/warnings
- [ ] Existing tests still pass
- [ ] New behavior tested

## Common Development Tasks

### Adding a New Pandoc Filter

**Context needed:**
- What input format does the filter process? (Pandoc AST)
- What transformation should it perform?
- What external tools are needed? (e.g., PlantUML, SQLite)
- How should errors be handled?

**Implementation pattern (via REPL):**

```clojure
;; 1. Read existing filter for reference (use read tool)

;; 2. Prototype the filter function in REPL
(defn process-my-filter
  "Processes [specific element type] in Pandoc AST."
  [ast]
  (require '[clojure.walk :as walk])
  (walk/postwalk
    (fn [node]
      (if (matches? node)
        (transform node)
        node))
    ast))

;; 3. Test with sample Pandoc AST
(def sample-ast {:t "Document" :c [...sample data...]})
(process-my-filter sample-ast)

;; 4. Once tested, edit main.clj to add the function
;; 5. Add CLI command using cli-matic structure
```

**Using clj-nrepl-eval:**

```bash
# Test the prototype
clj-nrepl-eval -p 7889 <<'EOF'
(defn process-my-filter [ast]
  (require '[clojure.walk :as walk])
  (walk/postwalk
    (fn [node]
      (if (matches? node)
        (transform node)
        node))
    ast))
EOF

# Test with sample data
clj-nrepl-eval -p 7889 "(process-my-filter sample-ast)"
```

### Working with Pandoc AST

Pandoc represents documents as JSON AST. Test transformations via REPL:

```clojure
;; Parse JSON AST
(require '[clojure.data.json :as json])
(def ast (json/read-str (slurp "input.json") :key-fn keyword))

;; Walk and transform AST
(require '[clojure.walk :as walk])

;; Test transformations
(walk/postwalk
  (fn [node]
    (if (map? node)
      (do
        (println "Node type:" (:t node))
        node)
      node))
  ast)
```

**Common AST patterns:**

- `{:t "CodeBlock" :c [attrs code]}` - Code blocks
- `{:t "Para" :c [...]}` - Paragraphs  
- `{:t "Header" :c [level attrs content]}` - Headers

### Database Operations

Polydoc uses SQLite for search indexing. Test queries via REPL:

```clojure
(require '[next.jdbc :as jdbc])
(require '[honey.sql :as sql])

;; Create connection
(def db {:dbtype "sqlite" :dbname "polydoc.db"})
(def ds (jdbc/get-datasource db))

;; Execute query with HoneySQL
(jdbc/execute! ds
  (sql/format {:select [:*]
               :from [:sections]
               :where [:= :book_id 1]}))

;; Insert data
(jdbc/execute! ds
  (sql/format {:insert-into :sections
               :values [{:book_id 1
                        :content "..."
                        :hash "..."}]}))
```

**Using clj-nrepl-eval:**

```bash
# Test database connection
clj-nrepl-eval -p 7889 <<'EOF'
(require '[next.jdbc :as jdbc])
(def db {:dbtype "sqlite" :dbname "polydoc.db"})
(def ds (jdbc/get-datasource db))
EOF

# Run query
clj-nrepl-eval -p 7889 <<'EOF'
(require '[honey.sql :as sql])
(jdbc/execute! ds
  (sql/format {:select [:*]
               :from [:sections]
               :limit 5}))
EOF
```

### Adding CLI Commands

Polydoc uses cli-matic for command-line interface. Prototype in REPL first:

```clojure
;; 1. Test the command function in REPL
(defn my-command-fn
  [{:keys [input]}]
  (println "Processing:" input)
  ;; ... implementation ...
  )

;; 2. Test it
(my-command-fn {:input "test.md"})

;; 3. Once working, edit main.clj to add command configuration
;; {:command "my-command"
;;  :description "Does something useful"
;;  :opts [{:option "input"
;;          :short "i"
;;          :as "Input file"
;;          :type :string
;;          :required true}]
;;  :runs my-command-fn}
```

### Using Debugging Tools

Use hashp for debugging during REPL development:

```clojure
;; Install hashp in REPL session
(require '[hashp.install :as hashp])
(hashp/install!)

;; Use #p to debug any expression
(defn calculate [x y]
  (+ #p (* x 2) #p (/ y 3)))

(calculate 10 9)
;; Output to STDERR:
;; #p[user/calculate:2] (* x 2) => 20
;; #p[user/calculate:2] (/ y 3) => 3
;; => 23

;; Debug threading macros
(-> data
    (assoc :x 10)
    #p
    (update :y inc)
    #p)

;; Debug let bindings
(let [a #p (* x 2)
      b #p (+ y 3)
      c #p (- a b)]
  (/ c 2))
```

**Using clj-nrepl-eval:**

```bash
# Install hashp
clj-nrepl-eval -p 7889 "(require '[hashp.install :as hashp])"
clj-nrepl-eval -p 7889 "(hashp/install!)"

# Test with debugging
clj-nrepl-eval -p 7889 <<'EOF'
(defn calculate [x y]
  (+ #p (* x 2) #p (/ y 3)))
(calculate 10 9)
EOF
```

### Working with Metadata

Use Metazoa for metadata tooling via REPL:

```clojure
(require '[glossa.metazoa :as meta])

;; View examples from metadata
(meta/view #'polydoc.main/some-function :glossa.metazoa/example)

;; Check that examples still work
(meta/check #'polydoc.main/some-function :glossa.metazoa/example)

;; Search for functions
(meta/search "name:filter*")

;; Query with Datalog
(meta/query
  '[:find [?name ...]
    :where
    [?e :ns ?ns]
    [?e :name ?name]]
  (the-ns 'polydoc.main))
```

### Using clojure-skills

The clojure-skills CLI tool is one of the allowed command-line tools. Use it for managing skills, prompts, and implementation plans:

```bash
# Initialize database (first time only)
clojure-skills db init

# Sync skills from filesystem
clojure-skills db sync

# Search for relevant skills
clojure-skills skill search "database"

# Show specific skill
clojure-skills skill show "next_jdbc"

# List all skills
clojure-skills skill list

# Create implementation plan
clojure-skills plan create \
  --name "Add SQLite filter" \
  --description "Implement SQLite execution filter for code blocks"

# List plans
clojure-skills plan list

# Show plan details
clojure-skills plan show 1

# Associate skills with plan
clojure-skills plan skill add 1 "next_jdbc"
clojure-skills plan skill add 1 "honeysql"

# Create task list
clojure-skills plan task-list create 1 \
  --name "Implementation tasks"

# Add tasks
clojure-skills task-list task create 1 \
  --description "Create database schema" \
  --order 1

# Mark task completed
clojure-skills task complete 1

# Update plan status
clojure-skills plan update 1 --status in-progress

# Complete plan
clojure-skills plan complete 1

# Set result
clojure-skills plan result set 1 \
  --result "Successfully implemented SQLite filter"
```

## Best Practices for Polydoc Development

### DO:

1. **Use REPL for everything**
   - All code evaluation via nREPL (port 7889)
   - Use clj-nrepl-eval for command-line evaluation
   - Test incrementally as you develop

2. **Read before writing**
   - Use `read` tool to examine files
   - Understand existing patterns
   - Check for similar implementations

3. **Test incrementally**
   - Reload namespace after changes
   - Test happy path first
   - Then test edge cases (nil, empty, invalid)

4. **Use appropriate tools**
   - Static analysis with clojure-lsp (via REPL)
   - Runtime testing with REPL
   - clojure-skills for planning and skill discovery

5. **Follow Clojure idioms**
   - Pure functions where possible
   - Data transformation over mutation
   - Clear naming conventions
   - **NEVER use `!` suffix in function names** (this project doesn't follow that convention)
   - **ALWAYS put `require` in the `ns` form, NEVER inside functions**

6. **Leverage existing libraries**
   - HoneySQL for SQL generation
   - Malli for validation
   - cli-matic for CLI structure

7. **Use hashp for debugging**
   - Add `#p` liberally during development
   - Remove before committing
   - Shows context (ns/fn/line) with values

### DON'T:

1. **Use disallowed command-line tools**
   - NO `clojure`, `clj` commands
   - NO `bb` (babashka) commands
   - NO `lein` commands
   - ONLY use: `clj-nrepl-eval` and `clojure-skills`

2. **Skip verification**
   - Always test changes via REPL
   - Check diagnostics
   - Run tests if they exist

3. **Make large, unfocused changes**
   - One task at a time
   - Small, testable increments
   - Easy to verify and rollback

4. **Ignore errors**
   - Fix diagnostics immediately
   - Handle edge cases
   - Provide helpful error messages

5. **Forget to reload**
   - Always `:reload` after editing
   - Check that changes took effect
   - Re-run tests after reload

6. **Hardcode values**
   - Use configuration
   - Environment variables for paths
   - Make code reusable

7. **Assume without testing**
   - Verify behavior at REPL
   - Test edge cases explicitly
   - Don't rely on "it should work"

8. **Commit debug statements**
   - Remove `#p` statements before committing
   - Or use conditional enabling for development

## Common Issues & Solutions

### Issue: "Unable to resolve symbol after editing"

**Cause:** Forgot to reload namespace

```clojure
;; Solution (via REPL or clj-nrepl-eval)
(require 'polydoc.main :reload)
(polydoc.main/new-function args)  ; Now works
```

```bash
# Via clj-nrepl-eval
clj-nrepl-eval -p 7889 "(require 'polydoc.main :reload)"
clj-nrepl-eval -p 7889 "(polydoc.main/new-function args)"
```

### Issue: "Changes don't take effect"

**Cause:** Old namespace still loaded

```clojure
;; Solution: Force complete reload
(require 'polydoc.main :reload-all)
```

```bash
# Via clj-nrepl-eval
clj-nrepl-eval -p 7889 "(require 'polydoc.main :reload-all)"
```

### Issue: "Tests fail after refactoring"

**Cause:** Test namespace not reloaded

```clojure
;; Solution
(require 'polydoc.main-test :reload)
(clojure.test/run-tests 'polydoc.main-test)
```

```bash
# Via clj-nrepl-eval
clj-nrepl-eval -p 7889 "(require 'polydoc.main-test :reload)"
clj-nrepl-eval -p 7889 "(clojure.test/run-tests 'polydoc.main-test)"
```

### Issue: "Can't find function/namespace"

**Cause:** Exploring unfamiliar codebase

```clojure
;; Solution: Use discovery tools via REPL
(map ns-name (all-ns))
(apropos "filter")
(find-doc "pandoc")

;; Or search with metazoa
(require '[glossa.metazoa :as meta])
(meta/search "name:process*")
```

### Issue: "Pandoc AST structure unclear"

**Cause:** JSON structure is complex

```clojure
;; Solution: Pretty-print and explore via REPL
(require '[puget.printer :as puget])
(puget/cprint sample-ast)

;; Or use bling for terminal formatting
(require '[bling.core :as b])
(b/pprint sample-ast)
```

### Issue: "Need to trace execution flow"

**Cause:** Complex logic hard to follow

```clojure
;; Solution: Use hashp throughout
(defn complex-process [data]
  (-> data
      (step-1)
      #p
      (step-2)
      #p
      (step-3)
      #p))

;; Shows value at each step with context
```

## REPL Workflow Pattern

All development should follow this pattern:

```
1. GATHER (via REPL)
   ├─ Explore namespaces: (all-ns), (dir ns)
   ├─ Check diagnostics: (lsp-api/diagnostics ...)
   ├─ Read documentation: (doc fn), (source fn)
   └─ Search: (apropos "..."), (meta/search "...")

2. ACTION (edit files, then reload)
   ├─ Edit files with read/edit/write tools
   ├─ Reload: (require 'ns :reload)
   ├─ Clean: (lsp-api/clean-ns! ...)
   └─ Format: (lsp-api/format! ...)

3. VERIFY (via REPL)
   ├─ Test function: (my-fn test-data)
   ├─ Test edge cases: (my-fn nil), (my-fn [])
   ├─ Run tests: (clojure.test/run-tests 'ns-test)
   └─ Check diagnostics: (lsp-api/diagnostics ...)

4. REPEAT until satisfied
```

**Using clj-nrepl-eval for the same pattern:**

```bash
# 1. GATHER
clj-nrepl-eval -p 7889 "(all-ns)"
clj-nrepl-eval -p 7889 "(dir polydoc.main)"
clj-nrepl-eval -p 7889 "(doc polydoc.main/some-fn)"

# 2. ACTION (after editing files)
clj-nrepl-eval -p 7889 "(require 'polydoc.main :reload)"
clj-nrepl-eval -p 7889 "(lsp-api/clean-ns! {:namespace '[polydoc.main]})"

# 3. VERIFY
clj-nrepl-eval -p 7889 "(polydoc.main/my-fn test-data)"
clj-nrepl-eval -p 7889 "(polydoc.main/my-fn nil)"
clj-nrepl-eval -p 7889 "(clojure.test/run-tests 'polydoc.main-test)"

# 4. REPEAT
```

## Critical: Error Handling Protocol

**⚠️ STOP AND ASK FOR USER HELP IMMEDIATELY when:**

### 1. MCP Tool Failures

If ANY MCP tool call fails:

```bash
# Example: Tool call fails
# Error: Unable to execute bash command
# Error: File read permission denied
# Error: Edit operation failed
```

**Required Response:**
1. **STOP all work immediately**
2. **Show the user the complete error message**
3. **Explain what you were trying to do**
4. **Ask the user for guidance**

**DO NOT:**
- Try alternative approaches without user input
- Skip the failed step
- Continue as if nothing happened
- Guess at solutions

### 2. Clojure/JVM Exceptions

If ANY Clojure exception or JVM error occurs via `clj-nrepl-eval`:

```clojure
;; Example exceptions:
;; - ClassNotFoundException
;; - IllegalArgumentException
;; - NullPointerException
;; - ArithmeticException
;; - ExceptionInfo
;; - CompilerException
;; - Any stacktrace output
```

**Required Response:**

1. **STOP all work immediately**

2. **Pretty-print the error using clojure.pprint/pretty**:

```bash
clj-nrepl-eval -p 7889 <<'EOF'
(require '[clojure.pprint :refer [pprint]])
(try
  ;; The failing code that produced the error
  (your-failing-function args)
  (catch Exception e
    (pprint {:error-type (type e)
             :message (.getMessage e)
             :cause (.getCause e)
             :data (ex-data e)
             :stacktrace (take 10 (.getStackTrace e))})))
EOF
```

3. **Show the user:**
   - The pretty-printed error details
   - The exact code that failed
   - What you were trying to accomplish
   - The context (which function, which namespace)

4. **Ask the user:**
   - "I encountered this error. How should I proceed?"
   - Provide the full error context
   - Wait for user guidance

**DO NOT:**
- Try to fix the error without understanding it
- Continue with other tasks
- Assume what the error means
- Skip error handling

### 3. nREPL Connection Issues

If `clj-nrepl-eval` cannot connect or times out:

```bash
# Examples:
# Connection refused
# Read timeout
# Server not responding
```

**Required Response:**
1. **STOP immediately**
2. **Show the connection error to user**
3. **Ask: "The nREPL server seems unavailable. Should I wait, or would you like to restart it?"**
4. **DO NOT proceed until connection is restored**

### 4. Unexpected or Confusing Behavior

If code produces unexpected results, even without explicit errors:

```clojure
;; Example: Function returns nil when you expected a value
;; Example: Empty collection when you expected data
;; Example: Wrong type returned
```

**Required Response:**
1. **STOP and investigate**
2. **Show the user what you expected vs what you got**
3. **Ask: "This behavior is unexpected. Should we debug this together?"**
4. **Wait for guidance before proceeding**

## Error Handling Examples

### Example 1: Handling a Clojure Exception

**❌ WRONG (Don't do this):**
```
[Agent tries code, gets exception, tries different approach without telling user]
```

**✅ CORRECT (Do this):**
```
I attempted to reload the namespace and encountered an exception:

clj-nrepl-eval -p 7889 <<'EOF'
(require '[clojure.pprint :refer [pprint]])
(try
  (require 'polydoc.main :reload)
  (catch Exception e
    (pprint {:error-type (type e)
             :message (.getMessage e)
             :cause (.getCause e)
             :data (ex-data e)
             :stacktrace (take 10 (.getStackTrace e))})))
EOF

Error Output:
{:error-type java.lang.RuntimeException
 :message "Unable to resolve symbol: unknown-fn"
 :cause nil
 :data {:file "polydoc/main.clj" :line 42 :column 5}
 :stacktrace [...]}

The error indicates that polydoc/main.clj line 42 references 
an undefined symbol 'unknown-fn'. 

How would you like me to proceed? Should I:
1. Show you the code around line 42?
2. Search for where this symbol should be defined?
3. Something else?
```

### Example 2: Handling MCP Tool Failure

**❌ WRONG:**
```
[Agent gets "file not found" error, tries different file path without asking]
```

**✅ CORRECT:**
```
I tried to read the file src/polydoc/filter.clj but got this error:

Error: File not found: src/polydoc/filter.clj

I was trying to examine the existing filter implementation 
to understand the pattern before implementing the new filter.

Could you help me locate the correct file? Should I:
1. List the files in src/polydoc/ to find the right one?
2. Is the file named differently?
3. Should I look somewhere else?
```

### Example 3: Handling Unexpected Behavior

**❌ WRONG:**
```
[Function returns nil, agent assumes it's correct and continues]
```

**✅ CORRECT:**
```
I tested the function but got an unexpected result:

clj-nrepl-eval -p 7889 "(polydoc.filters/process-code-block sample-block)"
=> nil

I expected this to return a transformed AST node, but it's 
returning nil instead. This could indicate:
1. The function isn't finding matching blocks
2. There's an error being silently caught
3. The sample data format is wrong

Would you like me to:
1. Add debug output to trace the execution?
2. Check the input data format?
3. Examine the function implementation more carefully?

Please advise how to proceed.
```

## LLM Agent Limitations & Error Protocol

As an LLM agent working on this project, remember:

1. **You can only use specific command-line tools**
   - `clj-nrepl-eval` for all Clojure code evaluation
   - `clojure-skills` for skill/plan management
   - NO other command-line tools (no `clojure`, `bb`, `lein`, etc.)

2. **All development happens via REPL**
   - Connect to nREPL on port 7889
   - Use clj-nrepl-eval to interact from command line
   - Test everything incrementally

3. **CRITICAL: You MUST stop and ask for help when errors occur**
   - **ANY MCP tool failure** → STOP, show error, ask user
   - **ANY Clojure/JVM exception** → STOP, pretty-print with `clojure.pprint`, ask user
   - **ANY nREPL connection issue** → STOP, report issue, ask user
   - **ANY unexpected behavior** → STOP, explain discrepancy, ask user
   - **NEVER continue past errors without explicit user guidance**

4. **Pretty-print ALL Clojure errors**
   - Use `clojure.pprint/pprint` to format error details
   - Show error type, message, cause, data, and stacktrace
   - Make errors readable and actionable for the user

5. **You need user input for decisions**
   - Architecture choices
   - API design
   - Trade-offs between approaches
   - **How to handle ANY error or unexpected behavior**

6. **Test assumptions immediately**
   - Don't assume code works
   - Verify at the REPL
   - Use clj-nrepl-eval to test
   - **Stop and ask if results are unexpected**

7. **Communicate progress AND problems**
   - Explain what you're doing
   - Show verification steps
   - **Report errors IMMEDIATELY with full context**
   - **Wait for user guidance before proceeding past any issue**

## Resources

### Relevant Clojure Skills

Use clojure-skills to search for:

- **Language**: `clojure_intro`, `clojure_repl`
- **Data**: `data_json`, `clj_yaml`, `malli`, `lentes`
- **Database**: `next_jdbc`, `honeysql`, `sqlite_jdbc`
- **CLI**: `cli_matic`, `bling`
- **Testing**: `kaocha`, `matcher_combinators`, `test_check`
- **Tools**: `clojure_lsp_api`, `metazoa`, `hashp-debugging`

### External Documentation

- [Pandoc Filters](https://pandoc.org/filters.html) - Filter system documentation
- [Pandoc AST](https://pandoc.org/using-the-pandoc-api.html) - AST structure reference
- [cli-matic](https://github.com/l3nz/cli-matic) - CLI framework
- [HoneySQL](https://github.com/seancorfield/honeysql) - SQL DSL
- [Malli](https://github.com/metosin/malli) - Data validation
- [hashp](https://github.com/weavejester/hashp) - Debug printing

## Browser Testing & Debugging

### Etaoin Browser Automation

Polydoc uses [Etaoin](https://github.com/clj-commons/etaoin) for browser automation testing with Firefox/GeckoDriver (migrated from Chrome/ChromeDriver).

**Key lessons learned:**

#### 1. Database Schema Mismatches

**Problem Pattern:** Functions query by text identifiers but compare against numeric foreign keys.

```clojure
;; WRONG: Comparing text book_id against numeric sections.book_id FK
(jdbc/execute! ds
  (sql/format {:select [:*]
               :from [:sections]
               :where [:= :book_id (str book-id)]}))  ; book-id is "test-book" but sections.book_id is 1

;; RIGHT: JOIN to resolve text ID to numeric ID
(jdbc/execute! ds
  (sql/format {:select [:sections.*]
               :from [:sections]
               :join [:books [:= :sections.book_id :books.id]]
               :where [:= :books.book_id book-id]}))  ; Now compares text to text
```

**Debugging approach:**
- Test queries directly via REPL with known data
- Check schema for ID types (INTEGER vs TEXT)
- Verify foreign key relationships
- Add JOINs when crossing table boundaries

#### 2. Firefox vs Chrome Selector Differences

**Problem:** Text matching behaves differently between browsers.

```clojure
;; Button HTML: <a role="button">Next →</a>

;; WRONG: Exact text match fails due to arrow character
(e/click driver [{:tag :a :role :button} {:fn/text "Next"}])
;; Error: Unable to locate element with text "Next"

;; RIGHT: Use partial text match with class selector
(e/click driver [{:tag :a :class :nav-button :fn/has-text "Next"}])
;; Works! Matches "Next" within "Next →"
```

**Best practices:**
- Use `:fn/has-text` for partial text matching
- Combine with class/role selectors for specificity
- Test selectors in both headless and non-headless modes
- Consider special characters (arrows, emojis) in UI text

#### 3. Form Submission in Firefox

**Problem:** `e/submit` on form elements not supported in Firefox.

```clojure
;; WRONG: Firefox says form "not reachable by keyboard"
(e/fill driver {:tag :input :type :search} "query")
(e/submit driver {:tag :form :class :search-form})
;; Error: Element not reachable by keyboard

;; RIGHT: Submit by pressing Enter in input field
(e/fill driver {:tag :input :type :search} "query")
(e/fill driver {:tag :input :type :search} etaoin.keys/enter)
;; Works! Simulates user pressing Enter
```

**Alternative patterns:**
- Click submit button if one exists
- Use `e/fill-human` for human-like typing + Enter
- Press Enter key explicitly after filling input

#### 4. Conditional Rendering Bugs

**Problem:** Inverted if/else logic hiding results.

```clojure
;; WRONG: Results rendered in "empty" branch
(if (empty? results)
  [:p "No results."
   [:div (for [r results] ...)]]  ; Never rendered when results exist!
  nil)

;; RIGHT: Results in else branch
(if (empty? results)
  [:p "No results."]
  [:div (for [r results] ...)])  ; Rendered when results exist
```

**Debugging approach:**
- Test page rendering via REPL with known data
- Generate HTML and inspect structure
- Check conditional logic carefully
- Verify both empty and non-empty cases

#### 5. Search Result URLs

**Problem:** Missing context in search results links.

```clojure
;; WRONG: Link missing book_id context
[:a {:href (str "/section/" section-id)} ...]
;; Doesn't match routing: /book/:book-id/section/:section-id

;; RIGHT: Include book_id in search results and URLs
;; 1. Add to SQL query
"SELECT s.section_id, b.book_id, ... FROM sections_fts JOIN books ..."

;; 2. Use in URL generation
[:a {:href (str "/book/" book-id "/section/" section-id)} ...]
```

**Pattern:** When building links, ensure you have all context needed for routing.

#### 6. FTS5 Trigger Verification

**Problem:** Search returns no results despite data existing.

**Debugging checklist:**
1. Verify FTS5 table is populated (check triggers fire on INSERT)
2. Test search function directly via REPL
3. Check query syntax (FTS5 uses special syntax)
4. Inspect actual SQL being generated
5. Verify JOIN conditions between FTS5 and main tables

```clojure
;; Test FTS5 population
(jdbc/execute! ds ["SELECT * FROM sections_fts"])
;; Should show entries after inserting into sections table

;; Test search directly
(search/search ds "test-query" {:limit 10})
;; Should return results if FTS5 is working
```

#### 7. Browser Testing Best Practices

**DO:**
- Start simple: test page loading first, then interactions
- Use REPL to generate and inspect HTML before browser tests
- Add explicit waits for dynamic content: `(e/wait-visible driver selector)`
- Test with visible browser first, then headless after working
- Verify database state via REPL before running browser tests
- Use class-based selectors over text matching when possible

**DON'T:**
- Assume selectors work the same across browsers
- Skip REPL validation of page rendering
- Forget to reload namespaces after fixing server code
- Use brittle selectors (exact text, complex XPath)
- Mix test data setup issues with browser interaction issues

#### 8. CI/CD Browser Testing

**GitHub Actions Firefox setup:**

```yaml
- name: Setup Firefox and GeckoDriver
  run: |
    # Firefox is pre-installed on ubuntu-latest
    # Install GeckoDriver
    GECKODRIVER_VERSION=$(curl -s https://api.github.com/repos/mozilla/geckodriver/releases/latest | grep -oP '"tag_name": "\K(.*)(?=")')
    wget https://github.com/mozilla/geckodriver/releases/download/${GECKODRIVER_VERSION}/geckodriver-${GECKODRIVER_VERSION}-linux64.tar.gz
    tar -xzf geckodriver-${GECKODRIVER_VERSION}-linux64.tar.gz
    sudo mv geckodriver /usr/local/bin/
    sudo chmod +x /usr/local/bin/geckodriver

- name: Verify Firefox and GeckoDriver
  run: |
    firefox --version
    geckodriver --version
```

**Key points:**
- Firefox pre-installed on `ubuntu-latest` runners
- GeckoDriver must be installed separately
- Always verify versions before running tests
- Use latest GeckoDriver release for compatibility

## Summary

When working on Polydoc:

1. **REPL-First Development**
   - All code evaluation via nREPL (port 7889)
   - Use clj-nrepl-eval from command line
   - Only allowed CLI tools: clj-nrepl-eval, clojure-skills

2. **Gather-Action-Verify Loop**
   - Gather: Explore via REPL tools
   - Action: Edit files, reload namespaces
   - Verify: Test via REPL, check diagnostics

3. **Key Principles**
   - Read before writing
   - Test after editing
   - Use REPL for everything
   - Fix errors immediately
   - Follow Clojure idioms
   - Leverage existing libraries

4. **Tool Usage**
   - `clj-nrepl-eval` - All Clojure evaluation
   - `clojure-skills` - Skill search, plan management
   - `read/edit/write` - File operations
   - `clojure-lsp` API - Via REPL only
   - `hashp` - Debug printing via REPL

This ensures reliable, REPL-driven development that works correctly in the Polydoc documentation processing system.

---
> Source: [iwillig/llm-planner](https://github.com/iwillig/llm-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
