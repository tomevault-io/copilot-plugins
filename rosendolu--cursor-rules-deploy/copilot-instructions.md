## cursor-rules-deploy

> - 🔒 Lock all direct dependency versions to specific versions or exact ranges to ensure reproducible builds

# 📦 Dependency and Version Constraints

## Critical Rules

- 🔒 Lock all direct dependency versions to specific versions or exact ranges to ensure reproducible builds
- 🚫 Never use loose version constraints (e.g., ^, ~, >=, >) in production dependencies
- ⚡ Specify Node.js version constraints in package.json using the engines field
- 🔄 Keep all dependencies up to date with security patches while maintaining version locks
- 🏗️ Separate devDependencies from production dependencies in Node.js projects
- 📝 Document any specific version requirements or compatibility issues in README.md
- 🧪 Test the application with the exact versions specified in lock files
- 🔍 Review all transitive dependencies for security vulnerabilities
- 📊 Maintain compatibility with the following version constraints:
  - Node.js: >= 18.x (LTS)
  - Python: >= 3.8
  - .NET Core: >= 6.0
  - Java: >= 11
  - Ruby: >= 3.0
- 🚨 Flag any dependencies that require specific OS or architecture support

## Examples

<example>
  # package.json
  {
    "name": "my-app",
    "version": "1.0.0",
    "engines": {
      "node": ">=18.0.0"
    },
    "dependencies": {
      "express": "4.18.2",
      "react": "18.2.0",
      "typescript": "5.0.4"
    },
    "devDependencies": {
      "jest": "29.5.0",
      "eslint": "8.40.0"
    }
  }

  # requirements.txt
  fastapi==0.95.2
  sqlalchemy==2.0.15
  pydantic==1.10.7

  # pom.xml
  <properties>
    <java.version>11</java.version>
    <spring.boot.version>2.7.11</spring.boot.version>
  </properties>
</example>

<example type="invalid">
  # package.json
  {
    "dependencies": {
      "express": "^4.18.2",
      "react": "~18.2.0",
      "typescript": ">=5.0.0"
    }
  }

  # requirements.txt
  fastapi>=0.95.2
  sqlalchemy
  pydantic>=1.10.0

  # pom.xml
  <properties>
    <spring.boot.version>LATEST</spring.boot.version>
  </properties>
</example> 

---
> Source: [rosendolu/cursor-rules-deploy](https://github.com/rosendolu/cursor-rules-deploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
