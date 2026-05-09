## mobx

> MobX ViewModel patterns for LokalMind v2.


# MobX Rules

Full reference: `docs/development/CONVENTIONS.md` section 4.

## ViewModel template

```typescript
import { makeAutoObservable, runInAction } from 'mobx';

export class ExampleViewModel {
  isLoading = false;
  errorMessage: string | null = null;

  constructor(private readonly someUseCase: SomeUseCase) {
    makeAutoObservable(this); // always first
  }

  async doAction() {
    this.isLoading = true;
    const result = await this.someUseCase.execute();
    runInAction(() => {
      this.isLoading = false;
      if (!result.success) this.errorMessage = result.error.message;
    });
  }

  get isReady() { return !this.isLoading; }
}
```

## Hard rules

1. `makeAutoObservable(this)` - first statement in constructor, always
2. Async state mutations - always in `runInAction()`
3. Computed - getter syntax only, no `@computed`
4. All screens/components reading VM state - must be wrapped with `observer()`
5. MobX only in `presentation/` - never in `domain/`, `data/`, `infrastructure/`
6. No `useState` / `useReducer` for state that belongs in a ViewModel

---
> Source: [Sandipan006/lokalmind-app](https://github.com/Sandipan006/lokalmind-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
