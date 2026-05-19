## compoents

> - Always use the typed `<Link>` from `@tanstack/react-router`.


## Links and Navigation
- Always use the typed `<Link>` from `@tanstack/react-router`.
- Pass `to`, `params`, `activeProps`, and Tailwind classes, mirroring:  
  ```tsx
  <Link
    to="/posts/$postId"
    params={{ postId: post.id }}
    activeProps={{ className: 'text-black ...' }}
    className="block ..."
  >
    {post.title}
  </Link>
  ```

---
> Source: [instructa/ai-chat-example](https://github.com/instructa/ai-chat-example) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
