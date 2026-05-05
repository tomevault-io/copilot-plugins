## 2025-11-megapot

> - Do not implement core Solidity logic unless directly prompted to

General Rules

- Do not implement core Solidity logic unless directly prompted to
- Whenever you are asked to add something to the contract compile the contract to make sure that it works

Testing

- All test files should be stored in test folder. Test folder structure should match the structure of contracts folder
- Test files should be named using Camel Case follow by `spec.ts` like so: `camelCaseContract.spec.ts`
- All error states and branches should be tested
- Each function being tested should be scaffolded as such with describe blocks used to test different invariants:
```
  describe("#functionName", async () => {
    let subjectParamOne: Address;
    let subjectParamTwo: BigNumber;
    let subjectCaller: Account;

    beforeEach(async () => {
      subjectParamOne = var1;
      subjectParamTwo = var2;
      subjectCaller = onRamper;
    });

    async function subject(): Promise<any> {
      return contract.connect(subjectCaller.wallet).function(subjectParamOne, subjectParamTwo);
    }

    it("should create the correct entry in the intents mapping", async () => {
      await subject();
    });

    describe("when var is set differently", async () => {
      beforeEach(async () => {
        subjectParamOne = var3;
      });

      it("does something", async () => {
        await subject();
      });
    });
  }
);


Development Workflow

- Utilize Hardhat's testing and debugging features.
- Implement a robust CI/CD pipeline for smart contract deployments.
- Use static type checking and linting tools in pre-commit hooks.

Documentation

- Document code thoroughly, focusing on why rather than what.
- Maintain up-to-date API documentation for smart contracts.
- Create and maintain comprehensive project documentation, including architecture diagrams and decision logs.

---
> Source: [code-423n4/2025-11-megapot](https://github.com/code-423n4/2025-11-megapot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
