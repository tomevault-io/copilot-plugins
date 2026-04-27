## nextjs15-trpc

> This is an instruction for creating trpc files and API endpoints.

This is an instruction for creating trpc files and API endpoints.

Example file references:
TRPC: @init.ts
Service Provider: @index.ts
Controller Factory: @index.ts
Router: @auth.ts
Controller: @auth.ts
Repository: @auth-repo.ts

---
Below is the detailed step-by-step pseudo-code for creating trpc router files

1. Repository
`src/lib/extern/db/supabase/<repo>.ts`

```ts
import db from '@/lib/extern/db'
import { DTO } from '@/lib/core/dtos'
import { TransactionDB } from '@/lib/extern/db/types'

export class Repo {
 
  constructor() {

  }
}
```

2. MyServiceProvider

`src/lib/extern/index.ts`


3. Controller 

```ts
import { Repo } from '@/lib/extern/db/supabase/repo'
export class Controller {
  private repo: Repo
  constructor(private repo: Repo) {}
}
```

4. MyControllerFactory

`src/lib/core/controllers/index.ts`

```ts
export class MyControllerFactory {
  services: MyServiceProvider
  constructor(services: MyServiceProvider) {
    this.services = services
  }

    Feat(){
        return new FeatController(this.services.FeatRepo())
    }
}
```


5. trpc register controller

`src/lib/trpc/init.ts`


6. trpc router

`src/lib/adapters/trpc/routers/feat.ts`
`src/lib/adapters/trpc/routers/index.ts`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/raphaelmans) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
