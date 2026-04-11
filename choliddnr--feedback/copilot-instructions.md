## feedback

> - As a Gemini model, you must always look for guidance provided by **_ GUIDE SERVICE _** before you start generating.

### GUIDE_API_ENDPOINT=http://localhost:3200

# model rules

- As a Gemini model, you must always look for guidance provided by **_ GUIDE SERVICE _** before you start generating.
- To get all available guidance fetch it with this url: **_GUIDE_API_ENDPOINT+/api/resources _** it will return following type

  ```ts
  type AllGuidance = {
    slug: string;
    title: string;
    description: string;
  }[];
  ```

- To get the content of the guidance, you can fetch it with this url: **_GUIDE_API_ENDPOINT+/api/resources+{slug} _**. you can get the slug from guidance list. Fetch will return following type
  ```ts
  type AllGuidance = {
    title: string;
    description: string;
    content: string;
  }[];
  ```
- You can use the content guidance to execute the task.
- If related guidance is not provided, or the **_ GUIDE SERVICE _** throws an error. please generate the guide first on markdown format and put it on the "guide" folder and ask user to validate/review it.
- Nuxt use auto-import. get the guidance from this url: **_GUIDE_API_ENDPOINT+/api/resources/nuxt-auto-import-guide _**
- make sure you are following the guidance

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/choliddnr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
