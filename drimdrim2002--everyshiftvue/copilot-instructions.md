## everyshiftvue

> Vite 6.3.5 개발 환경 설정 가이드


# Vite 개발 환경 설정

- Vite 6.3.5 최신 기능 활용
- auto-import, vue-components 플러그인 활용
- HMR 최적화를 위한 청크 분할
- TypeScript path mapping 설정
- 환경변수는 `VITE_` 접두사 필수

```ts
// vite.config.ts
export default defineConfig({
  plugins: [
    vue(),
    AutoImport({
      imports: ['vue', 'vue-router', 'pinia']
    }),
    Components({
      resolvers: [PrimeVueResolver()]
    })
  ],
  resolve: {
    alias: { '@': path.resolve(__dirname, 'src') }
  }
})
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/drimdrim2002)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/drimdrim2002)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
