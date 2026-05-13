## database

> rules for working with the database

* This project uses drizzle as a databse ORM. 
* Project Schema - [drizzle-schema.ts](mdc:server/db/drizzle-schema.ts)
* Changes to the schema should be made in the schema file and then run `npx drizzle-kit push` to update the database.
* On the server side, call the auto-imported `useDb()` function
* On the server side, call the auto-imported `useDbSchema()` function
* Use uuids for primary keys (`generateUuid` or `generateUuidv7`)
* Seeds should be created as nitro tasks:

```ts
// server/tasks/seed/posts.ts

export default defineTask({
  meta: {
      name: "seed:posts",
      description: "Seed Posts",
    },
  run({ payload, context }) {
    const db = useDb();
    const schema = useDbSchema();
    const posts = [
      {
        title: "Hello World",
        content: "This is a test post",
      },
      // TODO: Add more here
    ];

    db.insert(schema.posts).values(posts);
    // do seeding
    return { result: "Success" };
  },
});
```

* Seeds can be run by making a request to the `/_nitro/tasks/[seed-name]` endpoint. For example:

```bash
curl -X GET http://localhost:3000/_nitro/tasks/seed:posts
```

---
> Source: [vueschool/aidd-workhop](https://github.com/vueschool/aidd-workhop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
