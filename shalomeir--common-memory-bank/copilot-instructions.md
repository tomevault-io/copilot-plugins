## sub-folder-rules

> Guidelines for creating and maintaining Cursor rules to ensure consistency and effectiveness.

## Folder Path-Based Sub Rule Management System

### Basic Structure
1. All subfolder rules are stored in the `.cursor/rules/subrules/` directory.
2. The rules of each subfolder are stored according to the directory structure reflecting the folder path.
3. The paths of all rule files are recorded in `.cursor/rules/subrules/rule_list.txt`.

### Rule File Path Rules
1. Rule files for a specific folder reflect the folder path as it is:
   - Example: Rules related to `./src/components/auth.ts` are stored in `.cursor/rules/subrules/src/components/subrule.txt`

2. The rule_list.txt file records the full paths of all subrule files, one per line:
```
./cursor/rules/subrules/src/components/subrule.txt
./cursor/rules/subrules/src/utils/api/subrule.txt
./cursor/rules/subrules/src/auth/subrule.txt
```

### Rule Application Mechanism
1. When modifying or creating a specific file:
   - Read the `.cursor/rules/subrules/rule_list.txt` file first.
   - Check the path of the file you are currently working on. - Check if there is a subfolder rule file related to the path.
   - If there is a related rule file, add its contents to the context.

2. Path matching method:
   - If the file path is a subpath of the rule file path, the rule is applied.
   - Example: When modifying the file `./src/components/auth/login.ts`, the rule `./cursor/rules/subrules/src/components/subrule.txt` is applied.

## How to create and register a rule file

### When special subrules are needed
1. When there is a requirement to record separately in the process of creating, modifying, or deleting the subcode of the corresponding subfolder
2. Special arcitecture pattern applied only under the corresponding subfolder
3. Special technical context applied only under the corresponding subfolder
4. Bug fixes found in the subcode of the corresponding subfolder and handled as an exception
5. Other exception measures applied only under the corresponding subfolder

### Create a new rule file

1. When a special rule for a specific subfolder is needed:
```
If a special rule related to ./src/components/auth.ts is needed:
```

2. Create a rule file reflecting the path:
```
./cursor/rules/subrules/src/components/subrule.txt
```

3. Write the context, architecture, exceptions, etc. required in the rule file.

### Update rule_list.txt

1. After creating a new rule file, add the full path to it in `.cursor/rules/subrules/rule_list.txt`:
```
./cursor/rules/subrules/src/components/subrule.txt
```

2. Write multiple rule files, one per line:
```
./cursor/rules/subrules/src/components/subrule.txt
./cursor/rules/subrules/src/utils/api/subrule.txt
./cursor/rules/subrules/src/models/subrule.txt
```

## Real-world application example

### Scenario: Modifying the Auth component

1. A developer wants to modify the file `./src/components/auth/Login.tsx`

2. The AI ​​assistant:
- Reads the file `.cursor/rules/subrules/rule_list.txt`
- Checks that the file it is currently working on is in the path `./src/components/auth/`
- Finds the relevant rule file in rule_list.txt: `./cursor/rules/subrules/src/components/subrule.txt`
- Adds the contents of that rule file to the context
- Makes a code change suggestion considering this context

### Example Rule File Contents

File: `.cursor/rules/subrules/src/components/subrule.txt`
```
# Auth Component Rules

All authentication-related components in this folder must follow these rules:

1. Store JWT tokens only in HttpOnly cookies. Never store them in localStorage.
2. All authentication logic must be handled through the AuthService. 3. Authentication state changes must be done via AuthContext.
4. All authentication errors must be routed to a centralized error handling system.
5. Do not use React.memo for security-critical components (to prevent side-channel attacks).

Exceptions:
- Social login components follow a separate OAuth flow.
- MockAuthService can be used in development environments.

Related files:
- src/services/AuthService.ts
- src/contexts/AuthContext.tsx
- src/utils/auth/tokenStorage.ts
```

## System Maintenance

1. When adding a new subfolder rule:
- Create a rule file in the appropriate path
- Add the path to rule_list.txt

2. When modifying a rule:
- Edit the rule file directly
- No need to change rule_list.txt

3. When deleting a rule:
- Delete the rule file
- Remove the path from rule_list.txt

With this system, the AI ​​coding assistant can always automatically refer to special rules related to the path of the file you are working on, and generate and modify code while being aware of the special context of each folder.

---
> Source: [shalomeir/common-memory-bank](https://github.com/shalomeir/common-memory-bank) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
