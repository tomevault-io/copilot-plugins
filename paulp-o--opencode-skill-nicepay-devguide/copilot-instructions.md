## opencode-skill-nicepay-devguide

> NICEPAY 개발자 매뉴얼 검색·API·샘플·JS SDK — MCP 번들(cli+data)이 vendor/에 포함, 경로 설정 불필요


# nicepay-devguide

나이스페이 **공식 개발자 가이드(마크다운)** MCP입니다. `vendor/nicepay-devguide-mcp/`에 **번들 JS + `data/manual`** 이 함께 있어 별도 클론·경로 편집이 필요 없습니다. `launch-devguide.cjs`가 패키지 루트를 `cwd`로 잡습니다.

## Available Tools

- `search_nicepay_docs`, `get_api_endpoint`, `get_code_sample`, `get_sdk_method`

`SKILL_DIR` = 이 디렉터리.

```bash
npx -y mcp-to-skill@0.2.2 exec --config "$SKILL_DIR/mcp-config.json" --list
npx -y mcp-to-skill@0.2.2 exec --config "$SKILL_DIR/mcp-config.json" --call '{"tool": "search_nicepay_docs", "arguments": {"query": "웹훅"}}'
```

## 매뉴얼 갱신

운영자는 `scripts/refresh-vendor.sh`로 `nicepay-devguide-mcp` 소스 트리에서 `dist`+`data/manual`을 다시 복사할 수 있습니다.

---

*Generator: [mcp-to-skill](https://www.npmjs.com/package/mcp-to-skill).*

---
> Source: [paulp-o/opencode-skill-nicepay-devguide](https://github.com/paulp-o/opencode-skill-nicepay-devguide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
