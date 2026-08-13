## obsidian-dropbox-sync

> Obsidian vault를 Dropbox로 동기화하는 플러그인.

# obsidian-dropbox-sync

Obsidian vault를 Dropbox로 동기화하는 플러그인.

@PROGRESS.md

## 빌드 & 테스트

```bash
bun run typecheck   # 타입 검사
bun test            # bun:test 실행
bun run build       # Bun.build → dist/
```

## 아키텍처 요약

```
UI Layer → SyncEngine(Planner → Executor) → Adapters(FileSystem, RemoteStorage, SyncStateStore)
```

- **Planner**: 순수 함수. (local, remote, base) → SyncPlan
- **Executor**: SyncPlan을 adapter로 실행
- **Adapters**: 인터페이스 기반 DI. 테스트 시 Memory mock 주입

### 핵심 설계 결정

1. **content_hash 우선**: mtime은 보조. Dropbox content_hash(4MB 블록 SHA-256)로 변경 판단
2. **rev 기반 낙관적 잠금**: upload(mode: update(rev))로 서버 측 충돌 감지
3. **원본 보존 보장**: 충돌/실패 시 양쪽 원본 반드시 유지
4. **path_lower 키**: Dropbox는 case-insensitive. path_lower를 비교 키로 일관 사용

## 디렉토리 구조

```
src/
  types.ts              ← 공유 타입
  hash.ts               ← Dropbox content_hash (node:crypto)
  adapters/
    interfaces.ts       ← FileSystem, RemoteStorage, SyncStateStore
    memory.ts           ← 테스트용 in-memory mock
  sync/
    planner.ts          ← classifyChange, createPlan (순수 함수)
    executor.ts         ← SyncPlan 실행
    engine.ts           ← SyncEngine.runCycle()
test/
  hash.test.ts
  memory-adapters.test.ts
  planner.test.ts
  executor.test.ts
  simulation/           ← 다기기/실패 시나리오
  support/              ← SyncSimulator, FailingRemoteStorage
```

## 개발 워크플로우

### 로컬 플러그인 업데이트

```bash
# 빌드 + 테스트 vault의 플러그인 폴더에 복사
bun run build && cp dist/* ~/Documents/sync-tester/.obsidian/plugins/dropbox-sync/
```

### 개발 사이클

빌드 → 복사 → 리로드를 한 번에 수행한다:

```bash
bun run build && cp dist/main.js ~/Documents/sync-tester/.obsidian/plugins/dropbox-sync/
```

Hot Reload 플러그인이 `.hotreload` 마커를 감지하여 자동 리로드한다. 사용자에게 수동 리로드를 요청하지 않는다.

### 로그 확인

플러그인은 vault 루트에 `.sync-debug.log` 파일을 생성한다.

```bash
# 실시간 로그 확인
tail -f ~/Documents/sync-tester/.sync-debug.log

# 로그 파일 초기화
rm ~/Documents/sync-tester/.sync-debug.log
```

Obsidian 개발자 콘솔(Cmd+Option+I)에서도 `[Dropbox Sync]` 접두사로 로그가 출력된다.

### 테스트 vault

- 경로: `~/Documents/sync-tester/`
- 플러그인 경로: `~/Documents/sync-tester/.obsidian/plugins/dropbox-sync/`
- 실제 Dropbox 연동 테스트용. 프로덕션 vault가 아님.

### .env

빌드 시 Dropbox App Key를 임베드한다. `.gitignore`에 포함됨.

```
DROPBOX_APP_KEY=your_key_here
```

빌트인 키가 있으면 사용자는 설정 없이 바로 연결 가능.
Advanced에서 커스텀 키로 오버라이드할 수 있음.

## 참조 (vault)

- 아키텍처 결정서: `agents/brill/decisions/dropbox-sync-architecture`
- 문제 공간 분석: `agents/brill/research/dropbox-sync-risks`

---
> Source: [zeakd/obsidian-dropbox-sync](https://github.com/zeakd/obsidian-dropbox-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
