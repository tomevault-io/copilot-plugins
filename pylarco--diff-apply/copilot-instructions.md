## test

> 1. while addressing fail tests, always have a mindset that there must be main program core error to fix rather than make test less strict!!

1. while addressing fail tests, always have a mindset that there must be main program core error to fix rather than make test less strict!!
2. Test cases should be isolated and clean no left over even on sigterm
3. Test should use bun:test describe,it,afterAll,beforeAll,afterEach,beforeEach without mock
4. Create challenging, thorough test cases that fully verify implementation
5. Test cases should match expected requirements
6. Do not create test of tricks, simulation, stub, mock, etc. you should produce code of real algorithm
7. Do not create any new file for helper,script etc. just do what prompted.
8. test files should only be located outside src at test/unit or test/e2e
9. test should use/modify test/test.util.ts for reusability

---
> Source: [pylarco/diff-apply](https://github.com/pylarco/diff-apply) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
