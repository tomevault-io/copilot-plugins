## terrarium-monitor

> - Where possible use existing common and well-tested components and libraries.

### Do's
- Where possible use existing common and well-tested components and libraries.
- Use reliable and easily accessible components for project stack.
- Use clear and concise language when defining agent behaviors.
- Prioritize small modules over large monolithic god modules
- Document the purpose and functionality of every class, method, and function.
- Plan for ease of deployment, use, and maintenance from thes start.
- Where appropriate, containerize services to ensure consistency across different environments.
- Plan for unit tests often to ensure code quality and reliability.
- Prefer modern, clean, and visually appealing user interfaces, and only use sans-serif fonts.

### Don'ts
- Don't reinvent the wheel; avoid creating custom solutions for problems that have established solutions.
- Avoid using obscure or poorly documented libraries.
- Don't overcomplicate agent logic; keep it as simple and straightforward as possible.
- Avoid tight coupling between components; strive for loose coupling to enhance maintainability.
- Don't neglect documentation; ensure all code is well-documented for future reference.

### Project Specific Guidelines
- BME280 and LTR390-uv sensor and DS18B20 are all in scope as well as a small light sensor 
- Sensor sampling frequency should start at ~1min but should be a tunable parameter (and possibly individually tunable for different sensors/modalities 
- Would love to eventually include alerting needs but they should definitely be free utilities/resourses/methods
- No need for auth, trusted local network app

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Loyale)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Loyale)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
