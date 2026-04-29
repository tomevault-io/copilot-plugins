## cursor-chiro-app

> Use Bun instead of npm/node for package management, script execution, and development workflows


- **Package Management**
  - Use Bun instead of npm for all package management operations
  - Leverage Bun's speed advantages for faster installations and builds
  - Use Bun's lockfile (bun.lockb) instead of package-lock.json
  ```bash
  # ✅ DO: Use Bun commands
  bun install                 # Install all dependencies
  bun add react-native-admob  # Add a dependency
  bun add -d jest             # Add a dev dependency
  bun remove lodash           # Remove a dependency
  bun update                  # Update dependencies
  
  # ❌ DON'T: Use npm commands
  npm install
  npm install react-native-admob
  npm install --save-dev jest
  npm uninstall lodash
  npm update
  ```

- **Script Execution**
  - Use Bun to run scripts defined in package.json
  - Take advantage of Bun's faster startup time
  ```bash
  # ✅ DO: Use Bun for script execution
  bun run dev
  bun run build
  bun run test
  bun run lint
  
  # ❌ DON'T: Use npm run
  npm run dev
  npm run build
  npm run test
  npm run lint
  ```

- **Project Setup**
  - Initialize new projects with Bun
  - Configure TypeScript correctly for Bun
  ```bash
  # ✅ DO: Initialize project with Bun
  bun init -y            # Create a new project
  bun add -d @types/node # Add TypeScript types
  
  # ❌ DON'T: Use npm init
  npm init -y
  npm install --save-dev @types/node
  ```

- **Testing**
  - Use Bun's built-in test runner when possible
  - Fall back to Jest for React Native-specific tests
  ```javascript
  // ✅ DO: Use Bun's test API when possible
  import { test, expect } from "bun:test";
  
  test("addition works", () => {
    expect(1 + 1).toBe(2);
  });
  
  // For React Native components, continue using Jest
  import { render } from '@testing-library/react-native';
  import Component from './Component';
  
  test('renders correctly', () => {
    const { getByText } = render(<Component />);
    expect(getByText('Hello')).toBeTruthy();
  });
  ```

- **Environment Variables**
  - Use Bun's built-in .env support
  - Access environment variables via Bun.env
  ```typescript
  // ✅ DO: Use Bun's environment variable support
  // .env file is automatically loaded
  const apiKey = Bun.env.API_KEY;
  
  // ❌ DON'T: Use Node's process.env without dotenv
  import dotenv from 'dotenv';
  dotenv.config();
  const apiKey = process.env.API_KEY;
  ```

- **Formatting in Package Scripts**
  - Update package.json scripts to use Bun
  ```json
  // ✅ DO: Define scripts using Bun
  {
    "scripts": {
      "start": "bun run index.ts",
      "dev": "bun --hot run index.ts",
      "build": "bun build ./src/index.ts --outdir ./dist",
      "test": "bun test",
      "lint": "bun run eslint"
    }
  }
  
  // ❌ DON'T: Use Node or npm in scripts
  {
    "scripts": {
      "start": "node index.js",
      "dev": "nodemon index.js",
      "build": "tsc",
      "test": "jest",
      "lint": "eslint ."
    }
  }
  ```

- **CI/CD Configuration**
  - Configure CI/CD pipelines to use Bun
  - Set up Bun caching for faster builds
  ```yaml
  # ✅ DO: Use Bun in GitHub Actions
  name: CI
  on: [push, pull_request]
  jobs:
    test:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v3
        - uses: oven-sh/setup-bun@v1
          with:
            bun-version: latest
        - run: bun install
        - run: bun test
        - run: bun run build
  
  # ❌ DON'T: Use npm in CI/CD
  name: CI
  on: [push, pull_request]
  jobs:
    test:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v3
        - uses: actions/setup-node@v3
          with:
            node-version: 18
        - run: npm ci
        - run: npm test
        - run: npm run build
  ```

- **Using task-master with Bun**
  - When using task-master, prefer global installation with Bun
  - Run task-master commands with Bun
  ```bash
  # ✅ DO: Install and use task-master with Bun
  bun install -g claude-task-master  # Install globally
  bun x task-master list             # Use bun x for npx equivalent
  bun x task-master next
  bun x task-master parse-prd docs/prd.md
  
  # ❌ DON'T: Use npm for task-master
  npm install -g claude-task-master
  npx task-master list
  node scripts/dev.js list
  ``` 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wrtirado) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
