## okrbest-plugin-boards

> 트러블슈팅/문제 해결 모음


# 트러블슈팅 (Troubleshooting)

## 개요

개발 중 발생하는 문제와 해결 방법을 기록합니다.

## 빌드 관련

### 웹앱 빌드 실패

| 문제 | 원인 | 해결 |
|------|------|------|
| `Module not found` | node_modules 오염 | `cd webapp && rm -rf node_modules && npm install` |
| TypeScript 에러 | 타입 불일치 | `npm run check-types`로 확인 |
| 메모리 부족 | 대용량 빌드 | `NODE_OPTIONS=--max-old-space-size=4096` 설정 |

### 서버 빌드 실패

| 문제 | 원인 | 해결 |
|------|------|------|
| Go 버전 오류 | 잘못된 Go 버전 | Go 1.24+ 설치 확인 |
| 의존성 오류 | go.sum 불일치 | `go mod tidy` 실행 |
| CGO 오류 | C 컴파일러 필요 | `CGO_ENABLED=0` 설정 또는 컴파일러 설치 |

### 플러그인 로드 실패

| 문제 | 원인 | 해결 |
|------|------|------|
| 버전 불일치 | MM 버전 낮음 | Mattermost 10.7.0+ 확인 |
| 서명 오류 | 잘못된 빌드 | `make clean && make dist` |
| 권한 오류 | 파일 권한 | `chmod 644 dist/*.tar.gz` |

## 런타임 관련

### API 에러

| 문제 | 원인 | 해결 |
|------|------|------|
| 401 Unauthorized | 세션 만료/미인증 | 로그인 확인, 쿠키 확인 |
| 403 Forbidden | 권한 없음 | 보드 멤버 역할 확인 |
| 404 Not Found | 리소스 없음 | ID 확인, 삭제 여부 확인 |
| 500 Server Error | 서버 에러 | 서버 로그 확인 |

### WebSocket 연결

| 문제 | 원인 | 해결 |
|------|------|------|
| 연결 안됨 | WS URL 오류 | 서버 WS 설정 확인 |
| 자주 끊김 | 네트워크 불안정 | 재연결 로직 확인 |
| 이벤트 미수신 | 구독 안됨 | `subscribeToTeam` 호출 확인 |

### 데이터 동기화

| 문제 | 원인 | 해결 |
|------|------|------|
| 변경 미반영 | WS 이벤트 누락 | 새로고침, WS 연결 확인 |
| 충돌 데이터 | 동시 편집 | 마지막 수정 기준 병합 |
| 캐시 문제 | 오래된 캐시 | 하드 새로고침 (Ctrl+Shift+R) |

## 디버깅 팁

### 서버 디버깅

```bash
# 서버 로그 확인
make logs

# 개발 모드로 빌드 (디버그 심볼 포함)
MM_DEBUG=true make dist

# 특정 테스트만 실행
go test -v ./server/app/... -run TestName

# 레이스 컨디션 검사
go test -race ./server/...
```

### 웹앱 디버깅

```bash
# 개발 모드 빌드 (소스맵 포함)
cd webapp && npm run build:dev

# 타입 에러 확인
npm run check-types

# 린트 실행
npm run check
```

### 브라우저 디버깅

1. **Redux DevTools**: 상태 변경 추적
2. **Network 탭**: API 요청/응답 확인
3. **WebSocket 탭**: WS 메시지 확인
4. **Console**: 에러 로그 확인

### 데이터베이스 디버깅

```sql
-- 블록 조회
SELECT * FROM focalboard_blocks WHERE board_id = 'xxx' LIMIT 10;

-- 블록 이력 조회
SELECT * FROM focalboard_blocks_history WHERE id = 'xxx' ORDER BY insert_at DESC;

-- 멤버 조회
SELECT * FROM focalboard_board_members WHERE board_id = 'xxx';
```

## 일반적인 해결 패턴

### 1. 클린 빌드

```bash
make clean
cd webapp && rm -rf node_modules dist
npm install
make dist
```

### 2. 캐시 초기화

```bash
# Go 캐시
go clean -cache

# npm 캐시
npm cache clean --force
```

### 3. 재배포

```bash
make deploy
# 또는
make watch-plugin
```

## 로그 레벨 설정

```go
// 상세 로그 활성화
logger.SetLevel(mlog.LevelDebug)
```

## 메트릭스 확인

```bash
# Prometheus 메트릭스 (활성화된 경우)
curl http://localhost:8067/plugins/boards/api/v2/metrics
```

## 지원 요청 시 정보

문제 보고 시 포함할 정보:
1. Mattermost 버전
2. Boards 플러그인 버전
3. 에러 메시지 전문
4. 서버 로그 관련 부분
5. 브라우저 콘솔 에러
6. 재현 단계

---

## Q&A 목록

> 개발 중 생긴 문제 해결 사례가 `q-{주제}.mdc` 파일로 이 폴더에 추가됩니다.

| 문제 | 파일 |
|------|------|
| 카드 정렬 시 localeCompare 에러 및 무한 리다이렉션 | `q-sort-localecompare-error.mdc` |
| 카드 아이콘 삭제 후 제목/프로퍼티에서 Backspace/Delete 키가 작동하지 않는 버그 | `q-icon-delete-backspace-bug.mdc` |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/okrbest) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
