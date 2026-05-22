## project

> This is a shadcnui project, so please look at the components in /src/components/ui before creating your own for something or if there is an existing shadcn component that you know of and think the project needs to be installed please ask me to install it.

This is a shadcnui project, so please look at the components in /src/components/ui before creating your own for something or if there is an existing shadcn component that you know of and think the project needs to be installed please ask me to install it.

When calling convex mutations from the React code you should use the "useApiErrorHandler" from "src/misc/error" and then you should catch errors, for example: 

```ts
createAgent()
    .then((agentId) => routes.agent({ agentId }).push())
    .catch(onApiError)
```

Simmilarly if the logic is simple you should refreain from using try catch finally and instead just use the funcional approach. 

So instead of this:

```ts
try {
 await deleteAgent({ agentId });
 routes.home().push();
}
catch(error) {
    onApiError(error);
}
finally {
    tidyUp();
}

```

You should do this as it uses less lines and is easier to read:

```ts
deleteAgent({ agentId })
    .then(() => routes.home().push())
    .catch(onApiError)
    .finally(tidyUp)
```

---
> Source: [mikecann/agent-inbox](https://github.com/mikecann/agent-inbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
