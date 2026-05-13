## components

> rules for working with components


## General Component Rules

- Shad-cn Vue is installed and all of it's components are stored in `/components/ui`. They can be for most common UI elements.

## Component Organization and Naming Conventions

- Organize components by feature within `app/components` for example:

```
app/components/
    User
        UserProfile // very important it has prefix of parent directory name (ie User)
            UserProfile.vue  // 👈 the "main" component
            UserProfileImage.vue   // used in UserProfile.vue
            UserProfileEdit.vue   // very important it has full prefix of parent directory name (ie UserProfile)
            UserProfile.nuxt.test.ts // 👈 test for the "main" component
    Post
        PostList
            PostList
            PostListHeader
            PostListRow
        PostCard
            PostCard
            PostCardImage
            PostCardFooter
```

- Organize base/general that are NOT ShadCN components within `app/components/Base`
- You should NOT structure custom project components directory or naming structure based on any ShadCN component

## Component Definition Rules

- Only create a new custom component is shad-cn doesn't already have a solution!! Rely primarily on ShadCN components!
- NEVER try to generate a ShadCN component, all supported components are already installed (you can scan the `app/components/ui` directory to see all available)
- Do NOT create long components with a lot of responsibilities. Instead, break components down into smaller, more focused components
- Create small, focused components (< 50 lines)
- break complex logic into composables or utility functions
- Always use TypeScript to define all components.
- Use Typescript to define component props and emits.
- use JS docs to document component props and emits
- Use script setup
- Provide the component sections in the following order:
  - script
  - template
  - style 

## Component Usage Rules

- Do NOT import components defined within the project source code. They are auto-imported based on the compnent name. For example:
  - `Post/PostList/PostList.vue` is used via <PostList>
  - `Post/PostList/PostListItem.vue` is used via <PostListItem>
  - `Post/PostCard/PostCardImage` is used via <PostCardImage>
- The naming convention usage for ShadCN components are an exception. They are always autoimported and prefixed with `Ui` for example <UiButton>, <UiTexarea>, etc


## Template Ref Rules

- When creating template refs that reference a native DOM element always name them with the `El` suffix. For example:
  -  `const textareaEl = useTemplateRef('textareaEl')`  
- When creating template refs that reference a Vue Component always name them with the `ComponentRef` suffix. For example: 
  - `const myComponentRef = useTemplateRef('myComponentRef')`   
- Always use `useTemplateRef('[ref identifier here]')` to define template refs (not `ref()`)

## Icons

- Nuxt Icon is installed. 
- Display an icon with the icon component. For example:
  - `<Icon name="lucide:refresh">
- Rember to prefix the icon with the packname (like `lucide:`)  
- Any iconify icon can be used

# Style

- Primarily use shadcn classes for color styles. For example:
  - bg-primary
  - text-primary-foreground
  - bg-muted
  - text-muted-foreground

- Pair bg and text-foreground colors appropriately. For example:
  - bg-primary and text-primary-foreground
  - bg-muted and text-muted-foreground

## Data Fetching 
* Prefer to fetch data at the page level most of the time and keep components free from network requests as it makes them easier to test. 

---
> Source: [vueschool/aidd-workhop](https://github.com/vueschool/aidd-workhop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
