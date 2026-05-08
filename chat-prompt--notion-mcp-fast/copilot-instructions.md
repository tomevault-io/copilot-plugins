## notion-mcp-fast

> Notion 데스크톱 앱의 로컬 SQLite 캐시를 읽어 API 호출 없이 Notion 데이터에 접근하는 MCP 서버.

# notion-mcp-fast

Notion 데스크톱 앱의 로컬 SQLite 캐시를 읽어 API 호출 없이 Notion 데이터에 접근하는 MCP 서버.

## 핵심 정보

- **DB 경로**: `~/Library/Application Support/Notion/notion.db`
- **캐시 TTL**: 5분 (메타데이터만 캐시, 블록 내용은 온디맨드)
- **사용자 감지**: `meta_user_id`로 가장 많은 블록을 가진 사용자 자동 감지

## 프로젝트 구조

```
src/notion_mcp_fast/
├── __init__.py          # 패키지 exports
├── __main__.py          # 진입점
├── server.py            # MCP 도구 9개 (FastMCP)
├── reader.py            # NotionLocalReader (SQLite + TTL 캐싱)
└── block_parser.py      # 리치 텍스트 → 플레인 텍스트 변환
```

## MCP 도구

| 도구 | 설명 |
|------|------|
| `list_pages` | 페이지 목록 (workspace, parent_type 필터) |
| `get_page` | 페이지 상세 + 내용 |
| `search_pages` | 제목 검색 |
| `full_text_search` | 페이지 내용까지 검색 |
| `list_databases` | 데이터베이스 목록 |
| `get_database` | DB 상세 + 스키마 |
| `query_database` | DB 레코드 조회 |
| `list_workspaces` | 워크스페이스 목록 |
| `get_summary` | 캐시 요약 정보 |

## 개발 명령어

```bash
# 설치
uv sync

# 테스트 실행
uv run python -c "from notion_mcp_fast.reader import NotionLocalReader; print(NotionLocalReader().get_summary())"

# MCP 서버 실행
uv run notion-mcp-fast
```

## 핵심 테이블 (SQLite)

| 테이블 | 용도 | 주요 컬럼 |
|--------|------|-----------|
| `block` | 모든 블록 (페이지 포함) | id, type, properties, parent_id, meta_user_id, created_by_id, last_edited_by_id |
| `space` | 워크스페이스 | id, name, icon |
| `collection` | 데이터베이스 스키마 | id, name, schema |
| `notion_user` | 사용자 | id, name, email |

## 페이지 메타데이터

각 페이지는 다음 메타데이터를 포함:

| 필드 | 설명 | 예시 |
|------|------|------|
| `id` | 페이지 ID | `2eaa024d-f4dc-...` |
| `title` | 페이지 제목 | `SKT Design Camp` |
| `created_time` | 생성 시간 (timestamp) | `1737001260000` |
| `last_edited_time` | 수정 시간 (timestamp) | `1737877980000` |
| `created_by_id` | 생성자 ID | `7af5870b-...` |
| `created_by_name` | 생성자 이름 | `한가경` |
| `last_edited_by_id` | 최종 수정자 ID | `7af5870b-...` |
| `last_edited_by_name` | 최종 수정자 이름 | `윤누리` |
| `parent_id` | 상위 페이지/DB ID | - |
| `parent_table` | 상위 타입 | `collection`, `space`, `block` |
| `icon` | 아이콘 | 이모지 또는 URL |
| `cover` | 커버 이미지 URL | - |

## 블록 타입별 렌더링

- `header` → `# 제목`
- `sub_header` → `## 제목`
- `bulleted_list` → `• 항목`
- `numbered_list` → `1. 항목`
- `to_do` → `[ ] 할일` / `[x] 완료`
- `quote` → `> 인용`
- `code` → ``` 코드 ```
- `divider` → `---`

## 주의사항

1. **읽기 전용**: 데이터 수정 불가 (API 필요)
2. **동기화 의존**: Notion.app이 마지막으로 동기화한 데이터만 접근 가능
3. **macOS 전용**: Notion.app 로컬 캐시는 macOS에만 존재
4. **권한 필요**: Full Disk Access 권한 필요할 수 있음

## Claude Desktop 설정

```json
{
  "mcpServers": {
    "notion-local": {
      "command": "uv",
      "args": ["run", "--directory", "/path/to/notion-mcp-fast", "notion-mcp-fast"]
    }
  }
}
```

---
> Source: [chat-prompt/notion-mcp-fast](https://github.com/chat-prompt/notion-mcp-fast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
