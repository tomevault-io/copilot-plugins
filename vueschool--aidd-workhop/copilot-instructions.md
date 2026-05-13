## pages

> * Prefer to fetch data at the page level most of the time. Ask me if you think you should do differently, based on the use case.

* Prefer to fetch data at the page level most of the time. Ask me if you think you should do differently, based on the use case.
* When possible, rely on the type safe return (typed by Nuxt) of useFetch and don't manually type the response. For example:
  * DO THIS: `const {data} = await useFetch('/api/posts') // data is automatically typed`
  * NOT THIS: `const {data} = await useFetch<Post>('/api/posts') // manually typing response is bad and leads to maintainence issues`
* Break down pages into smaller components.

---
> Source: [vueschool/aidd-workhop](https://github.com/vueschool/aidd-workhop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
