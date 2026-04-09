## okrbest-plugin-boards

> Go 서버 코드 규칙


# Go 서버 코드 규칙

## 아키텍처 레이어

```
Plugin (plugin.go) 
    → BoardsApp (boards/boardsapp.go)
        → Server (server/server.go)
            → API (api/api.go)
                → App (app/app.go)
                    → Store (services/store/)
```

### 각 레이어 역할

| 레이어 | 위치 | 역할 |
|--------|------|------|
| Plugin | `server/plugin.go` | Mattermost 플러그인 인터페이스, 라이프사이클 관리 |
| BoardsApp | `server/boards/boardsapp.go` | 애플리케이션 초기화, 서비스 구성 |
| Server | `server/server/server.go` | HTTP 서버, 라우터 설정 |
| API | `server/api/` | HTTP 요청/응답, 인증, 라우팅 |
| App | `server/app/` | 비즈니스 로직, 권한 검증, 알림 |
| Store | `server/services/store/` | 데이터베이스 CRUD |
| Model | `server/model/` | 도메인 모델 정의 |

### 서비스 구조

| 서비스 | 위치 | 역할 |
|--------|------|------|
| Store | `services/store/` | 데이터 접근 계층 |
| Permissions | `services/permissions/` | 권한 검증 (Mattermost 통합) |
| Notify | `services/notify/` | 알림 서비스 (@멘션, 구독) |
| Metrics | `services/metrics/` | Prometheus 메트릭 |
| Audit | `services/audit/` | 감사 로그 |

## HTTP 요청 처리 흐름

```
1. Client → Plugin.ServeHTTP
2. Plugin → Gorilla Mux Router
3. Router → API Handler
4. API Handler:
   - 인증/권한 검증
   - 요청 파싱
5. API → App (비즈니스 로직)
6. App → Store (데이터 접근)
7. Store → Database
8. 결과 반환 → JSON Response
```

## 코딩 패턴

### 새 API 엔드포인트 추가

1. `server/api/{feature}.go` - 핸들러 작성
2. `server/api/api.go` - 라우트 등록
3. `server/app/{feature}.go` - 비즈니스 로직
4. 필요시 `server/model/` - 모델 추가

```go
// 핸들러 예시 (server/api/{feature}.go)
func (a *API) handleMyFeature(w http.ResponseWriter, r *http.Request) {
    session := a.getSession(r)
    // 권한 검증, 비즈니스 로직 호출
    result, err := a.app.DoMyFeature(session.UserID, params)
    if err != nil {
        a.errorResponse(w, r, err)
        return
    }
    a.jsonResponse(w, r, result)
}

// 라우트 등록 (server/api/api.go)
func (a *API) registerMyFeatureRoutes(r *mux.Router) {
    r.HandleFunc("/myfeature", a.handleMyFeature).Methods("GET")
}
```

### 에러 처리

```go
// 표준 에러 타입 사용 (server/model/errorResponse.go)
model.NewErrBadRequest("잘못된 요청")
model.NewErrUnauthorized("인증 필요")
model.NewErrForbidden("권한 없음")
model.NewErrNotFound("리소스 없음")
```

### 로깅

```go
a.logger.Debug("디버그 메시지", mlog.String("key", "value"))
a.logger.Info("정보 메시지")
a.logger.Warn("경고 메시지")
a.logger.Error("에러 발생", mlog.Err(err))
```

## 데이터베이스

### Store 인터페이스 (`server/services/store/store.go`)

- 모든 DB 작업은 Store 인터페이스를 통해
- `@withTransaction` 주석으로 트랜잭션 자동 래핑

### SQLStore 구현 (`server/services/store/sqlstore/`)

| 파일 | 역할 |
|------|------|
| `sqlstore.go` | Store 인터페이스 구현, DB 연결 관리 |
| `board.go` | Board 관련 쿼리 |
| `blocks.go` | Block 관련 쿼리 |
| `migrate.go` | DB 마이그레이션 |

### 마이그레이션

- 위치: `server/services/store/sqlstore/migrations/`
- 파일명: `000XXX_description.up.sql`, `000XXX_description.down.sql`
- PostgreSQL, MySQL, SQLite 모두 지원

## WebSocket 실시간 동기화

### Plugin Adapter (`server/ws/plugin_adapter.go`)

```go
type PluginAdapterInterface interface {
    OnWebSocketConnect(webConnID, userID string)
    OnWebSocketDisconnect(webConnID, userID string)
    WebSocketMessageHasBeenPosted(webConnID, userID string, req *mm_model.WebSocketRequest)
    BroadcastBlockChange(teamID string, block *model.Block)
    BroadcastBoardChange(teamID string, board *model.Board)
}
```

### 변경 브로드캐스트

```go
// App에서 변경 발생 시
a.wsAdapter.BroadcastBlockChange(teamID, block)
a.wsAdapter.BroadcastBoardChange(teamID, board)
```

## 알림 시스템 (`server/services/notify/`)

| 백엔드 | 위치 | 역할 |
|--------|------|------|
| Mentions | `notifymentions/` | @멘션 알림 (DM 발송) |
| Subscriptions | `notifysubscriptions/` | 구독 알림 (카드/보드 변경) |

## 메트릭스 (`server/services/metrics/`)

| 메트릭 | 설명 |
|--------|------|
| `focalboard_blocks_total` | 총 블록 수 |
| `focalboard_boards_total` | 총 보드 수 |
| `focalboard_api_requests_total` | API 요청 수 |
| `focalboard_api_request_duration` | API 응답 시간 |

## 테스트

```bash
# 서버 테스트
make server-test

# 특정 패키지 테스트
go test ./server/app/...

# 커버리지
make coverage
```

## 주요 모델

### Board (`server/model/board.go`)
```go
type Board struct {
    ID             string                   `json:"id"`
    TeamID         string                   `json:"teamId"`
    ChannelID      string                   `json:"channelId"`
    Type           BoardType                `json:"type"`  // "O" (Open) | "P" (Private)
    Title          string                   `json:"title"`
    CardProperties []map[string]interface{} `json:"cardProperties"`
}
```

### Block (`server/model/block.go`)
```go
type Block struct {
    ID       string                 `json:"id"`
    ParentID string                 `json:"parentId"`
    BoardID  string                 `json:"boardId"`
    Type     BlockType              `json:"type"`  // card, view, text, image 등
    Title    string                 `json:"title"`
    Fields   map[string]interface{} `json:"fields"`
}
```

## 주요 API 엔드포인트

| 엔드포인트 | 메서드 | 설명 |
|-----------|--------|------|
| `/api/v2/boards` | GET, POST | 보드 목록/생성 |
| `/api/v2/boards/{boardId}` | GET, PATCH, DELETE | 보드 CRUD |
| `/api/v2/boards/{boardId}/blocks` | GET, POST | 블록 조회/생성 |
| `/api/v2/cards` | POST | 카드 생성 |
| `/api/v2/teams/{teamId}/categories` | GET, POST | 카테고리 관리 |
| `/api/v2/users/me` | GET | 현재 사용자 |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/okrbest)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/okrbest)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
