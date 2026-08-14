## bundis

> > SQLite를 저장 엔진으로 쓰는 RESP 호환 서버.

# CLAUDE.md — bun-resp-sqlite

> SQLite를 저장 엔진으로 쓰는 RESP 호환 서버.
> 클라이언트는 항상 순정 `Bun.RedisClient`로 접속한다.
> 이 문서는 **설계의 단일 진실(SSOT)** 이다. 구현은 Claude Code에서 별도로 진행한다.

---

## 0. 한 줄 정의

`Bun.RedisClient`(RESP3 클라이언트)가 보내는 wire protocol을, Bun TCP 서버가 받아 SQLite로 처리하고 RESP로 응답한다. **서버는 클라이언트를 흉내 내지 않는다 — 프로토콜에 응답할 뿐이다.**

```
┌────────────────────┐   RESP3 over TCP    ┌──────────────────────────────┐
│  Bun.RedisClient   │ ──────────────────▶ │  bun-resp-sqlite (this)      │
│  (순정, 무수정)     │ ◀────────────────── │  Bun.listen() + bun:sqlite   │
└────────────────────┘   RESP3 replies      └──────────────────────────────┘
       애플리케이션                                   같은 머신/별도 프로세스
```

---

## 1. 목적과 비목적 (Scope SSOT)

### 목적
- `Bun.RedisClient`가 **코드 수정 없이** 접속·동작한다. 접속 URL만 이 서버를 가리키면 끝.
- Redis 서버 **설치 없이** Redis를 쓴다. 의존성은 Bun 런타임 하나(`bun:sqlite`, `Bun.listen` 모두 내장).
- 데이터는 SQLite 파일에 영속화된다(프로세스 재시작 후 생존).

### 비목적 (명시적으로 하지 않는 것)
- 인프로세스 라이브러리 모드(클라이언트 시그니처 직접 구현)는 **만들지 않는다.** 접속은 무조건 RESP over TCP.
- Redis Cluster / Sentinel은 지원하지 않는다 (`Bun.RedisClient` 자체가 미지원).
- 다중 노드 HA·자동 failover는 범위 밖.
- 인메모리 전용 Redis의 모든 성능 특성을 동일 재현하지 않는다 — **인터페이스 호환이 목표지 성능 동등이 아니다.**

---

## 2. 호환성 계약 (Compatibility Contract — 가장 중요한 SSOT)

호환성의 기준은 **"클라이언트 메서드를 다 구현했는가"가 아니라 "클라이언트가 보내는 바이트에 올바른 바이트로 답하는가"** 이다. 아래 4개 계층을 모두 만족해야 "완전 호환"이 성립한다.

### 2.1 프로토콜 계층 — RESP3 필수
`Bun.RedisClient`는 RESP3로 말한다(Zig 구현). 따라서:
- 서버는 `HELLO 3` 핸드셰이크에 응답해야 한다. RESP2만 구현하면 응답 타입이 어긋난다.
- RESP3 고유 타입을 낼 수 있어야 한다: Null(`_`), Boolean(`#`), Map(`%`), Set(`~`), Double(`,`), Verbatim 등.
- 클라이언트의 자동 타입 변환 규칙(아래 §2.4)을 서버 응답이 유발해야 한다.

### 2.2 핸드셰이크/연결 계층 — 연결 성립의 전제
클라이언트가 connect 시 호출하는 명령은 **명령 처리 이전에** 반드시 응답해야 한다. 하나라도 빠지면 핸드셰이크에서 막힌다.

| 명령 | 역할 | 1차 구현 |
|---|---|---|
| `HELLO [3] [AUTH ...]` | 프로토콜 협상 + 서버 정보 Map 반환 | **필수** |
| `AUTH` | 인증 (URL에 자격증명 있을 때) | **필수** (no-auth면 OK 반환) |
| `PING` | 헬스체크 / keepalive | **필수** |
| `SELECT` | DB 번호 선택 (URL `/0`) | **필수** (단일 DB여도 OK 반환) |
| `INFO` | 서버 메타데이터 | **필수** (최소 필드) |
| `QUIT` | 연결 종료 | **필수** |
| `CLIENT` | 클라이언트 설정 (`CLIENT SETINFO` 등) | 권장 (OK 반환) |

> 이 명령들은 클라이언트 문서상 **자동 파이프라이닝이 비활성화**되는 명령군에 속한다(`AUTH`/`INFO`/`QUIT`/`SELECT`/`MULTI`/`EXEC`/`WATCH` 등). 서버는 이들을 단건 동기 응답으로 처리한다고 가정해도 된다.

### 2.3 명령 계층 — 커버리지가 곧 호환성
전용 메서드가 있는 명령은 반드시 지원한다. 나머지는 클라이언트가 `send(CMD, args[])`로 raw 전송하므로, **서버 입장에서는 전용/raw 구분이 없다 — 모두 같은 RESP 배열로 도착한다.** 즉 호환성 확장 = 명령 디스패치 테이블에 케이스 추가.

전용 메서드 보유 명령(문서 확인됨, **1차 핵심 대상**):
- String: `GET` `SET` `GETSET`(via send) `DEL` `EXISTS` `GETBUFFER`(=GET, 바이너리 응답)
- Numeric: `INCR` `DECR`
- Expire: `EXPIRE` `TTL`
- Hash: `HSET` `HMSET` `HGET` `HMGET` `HINCRBY` `HINCRBYFLOAT`
- Set: `SADD` `SREM` `SISMEMBER` `SMEMBERS` `SRANDMEMBER` `SPOP`
- Multi-key: `MGET` `MSET` `MSETNX` `SETEX` `SETNX`

### 2.4 응답 타입 변환 계약 — 어긋나면 "호환"이 깨지는 지점
클라이언트는 RESP 응답을 JS 값으로 자동 변환한다. 서버는 **정확히 그 변환을 유발하는 RESP 타입**을 내야 한다. 이게 호환성의 가장 미묘한 부분.

| 클라이언트 기대 JS | 서버가 보낼 RESP 타입 |
|---|---|
| number | Integer (`:`) |
| string | Bulk/Simple String (`$`/`+`) |
| `null` | Null Bulk / RESP3 Null (`_`) |
| array | Array (`*`) |
| boolean | RESP3 Boolean (`#`) |
| object(map) | RESP3 Map (`%`) |
| array(set) | RESP3 Set (`~`) |
| Error throw | Error (`-`) + 코드 |

**명령별 특수 변환(반드시 준수):**
- `EXISTS` → 정수 1/0이 아니라 **boolean**으로 변환됨. 서버는 RESP3 Boolean으로 답해야 클라이언트 기대와 일치. (또는 클라이언트가 정수→boolean 변환을 보장하는지 통합 테스트로 고정)
- `SISMEMBER` → 동일하게 boolean.
- `getBuffer()` → 같은 `GET`이지만 Uint8Array로 받음. 서버는 동일 Bulk String을 내되 **바이너리 안전**해야 한다(임의 바이트 보존).

**에러 코드 계약:** 클라이언트는 `error.code`로 분기한다. 서버 에러 응답은 클라이언트가 아는 코드로 매핑되어야 한다:
- `ERR_REDIS_CONNECTION_CLOSED`, `ERR_REDIS_AUTHENTICATION_FAILED`, `ERR_REDIS_INVALID_RESPONSE`.
- 일반 명령 에러는 표준 RESP 에러 프리픽스(`ERR`, `WRONGTYPE`, `WRONGPASS` 등)로.

---

## 3. 아키텍처 (메타 설계)

### 3.1 레이어 경계
각 레이어는 한 가지 책임만 진다. 위→아래 단방향 의존.

```
┌─────────────────────────────────────────────────────────────┐
│ L1  Transport      Bun.listen() TCP 서버 / 소켓 수명주기      │
│        ▼            (바이트 in/out, 연결당 상태)               │
│ L2  RESP Codec     RESP3 파서(스트리밍) + 직렬화기            │
│        ▼            (바이트 ↔ Command / Reply)                │
│ L3  Connection     연결당 상태기계: handshake→ready→subscribe │
│        ▼            (HELLO/AUTH/SELECT, 모드 전환)             │
│ L4  Dispatcher     명령 라우팅 테이블 (CMD → Handler)         │
│        ▼            (arity 검증, 미지원 명령 에러)            │
│ L5  Command Engine 명령별 의미론 (KV/Hash/Set/Expire...)      │
│        ▼                                                       │
│ L6  Storage        StorageEngine 인터페이스 (추상)            │
│        ▼            └─ SqliteStorage (bun:sqlite, WAL)        │
│ L7  Side-systems   ExpiryReaper / PubSubHub / TxnContext     │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 의존성 역전 — Storage는 추상 경계
`Command Engine`은 SQLite를 모른다. `StorageEngine` 인터페이스에만 의존한다. 이렇게 두면 (1) 테스트 시 인메모리 mock 교체, (2) 후일 다른 저장 엔진 실험이 열린다. **단, 이 추상화에 Redis 고유 개념을 그대로 노출하지 않는다** — KV/필드맵/정렬셋 같은 저장 원형(primitive)만 올린다.

```
StorageEngine (interface = 저장 SSOT)
  ├─ kvGet/kvSet/kvDel/kvExists
  ├─ fieldGet/fieldSet/fieldDel        (hash 계열의 저장 원형)
  ├─ memberAdd/memberRem/memberHas     (set 계열의 저장 원형)
  ├─ expireSet/expireGet/sweepExpired  (TTL)
  └─ withTransaction(fn)               (원자 단위)
        │
        └─ SqliteStorage  (bun:sqlite, WAL 모드)
```

### 3.3 왜 이 경계인가
- **RESP Codec과 Command Engine을 분리**: 프로토콜 버그와 의미론 버그를 독립적으로 잡을 수 있다. RESP 파서는 명령이 뭔지 몰라도 된다.
- **Connection 상태기계를 독립 레이어로**: Pub/Sub 모드 전환, 핸드셰이크 진행을 한 곳에서 관리. subscribe가 연결을 "점유"하는 Redis 의미론을 여기서 강제.
- **Dispatcher를 테이블로**: 명령 추가가 곧 호환성 확장이므로, 확장 지점을 한 파일에 모은다(§7 확장 경로의 핵심).

---

## 4. 네트워킹 설계 (Bun 네트워킹 매핑)

요구된 Bun 네트워킹 API를 이 시스템의 어디에 쓰는지/안 쓰는지 명확히 한다. **남용하지 않는 것도 설계다.**

| Bun API | 이 시스템에서의 역할 | 채택 |
|---|---|---|
| **TCP** (`Bun.listen`/`Bun.connect`) | **주 전송 계층.** RESP는 TCP 위에서 동작. 서버 소켓 = `Bun.listen`. | **핵심** |
| **WebSockets** | 선택적 게이트웨이. 브라우저/엣지에서 RESP-over-WS가 필요할 때 L1에 어댑터로 추가. 1차 비채택. | 확장 옵션 |
| **UDP** | RESP는 순서·신뢰성 보장이 필요해 UDP 부적합. **사용 안 함.** (모니터링 메트릭 push 등 부수 용도만 검토) | 비채택 |
| **DNS** | 클라이언트 접속 호스트 해석에만 관여. 서버는 보통 bind 주소 고정이라 직접 호출 적음. | 부수 |
| **Fetch** | RESP 경로와 무관. 운영용 HTTP 헬스/메트릭 엔드포인트(`/healthz`, Prometheus)에만 선택 사용. | 부수 |

### 4.1 TCP 서버 핵심 요구사항
`Bun.listen({ socket: { data, open, close, error } })` 기반. 연결당 다음을 보장:

1. **스트리밍 파싱**: TCP는 메시지 경계가 없다. `data` 콜백은 임의 청크로 쪼개져 들어온다. RESP 파서는 **연결별 누적 버퍼**를 두고, 완전한 명령이 모일 때까지 보류 → 완성될 때마다 디스패치. (RESP의 길이 프리픽스가 이걸 가능케 함.)
2. **파이프라이닝 순서 보장**: 한 청크에 여러 명령이 있으면 **도착 순서대로 처리하고 응답도 그 순서로** 한다. 클라이언트의 자동 파이프라이닝이 이걸 전제로 하므로, 순서가 뒤바뀌면 호환이 깨진다. → 연결별 직렬 처리 큐.
3. **연결별 상태 격리**: handshake 완료 여부, 선택된 DB, 인증 상태, subscribe 채널 집합을 **소켓별로** 보관(`socket.data`).
4. **백프레셔**: `socket.write` 반환값/`drain` 처리. 대량 응답 시 버퍼 폭주 방지.

---

## 5. 저장소 설계 (SQLite 스키마 SSOT)

### 5.1 설계 원칙
- **타입 통합 vs 분리**: Redis 타입(string/hash/set/zset)을 **키 메타 + 타입별 값 테이블**로 표현. 단일 거대 테이블은 zset 정렬·set 멤버십에서 인덱스가 꼬이므로, **메타는 통합 / 값은 타입별 분리**가 균형점.
- **TTL은 컬럼 + 스위퍼**: 만료 시각을 epoch ms로 저장. Redis의 lazy+active 만료를 모사(§5.3).
- **원자성은 SQLite 트랜잭션**: `INCR`, `SETNX`, `MSET`, `MULTI/EXEC`를 트랜잭션 한 단위로.

### 5.2 스키마 개요 (구현 시 확정)
```
keys        (key PK, type, expire_at_ms NULL)         -- 모든 키의 메타 + TTL
kv          (key PK→keys, value BLOB)                  -- string (바이너리 안전)
hash_fields (key, field, value BLOB, PK(key,field))    -- hash
set_members (key, member, PK(key,member))              -- set
list_items  (key, seq, value BLOB, PK(key,seq))        -- list; seq는 연속 정수 구간(양끝 push/pop 전용)
zset_members(key, member, score, PK(key,member))       -- zset; + INDEX(key,score,member)
                                                       --   동점 score는 member 사전순 → 순서 결정성
```
- `value`는 BLOB — `getBuffer()`의 바이너리 안전성과 임의 바이트 보존을 위해 TEXT가 아닌 BLOB.
- `keys`에서 타입 충돌 시 `WRONGTYPE` 에러(Redis 의미론).
- WAL 모드 필수 — 단일 프로세스 내 동시 읽기/쓰기 처리량 확보.

### 5.3 만료(TTL) 의미론 — 동등 효과의 핵심
Redis 체감과 맞추려면 **두 경로 모두** 구현:
1. **Lazy(읽기 시점)**: `GET`/`EXISTS` 등에서 `expire_at_ms < now`면 즉시 미존재 취급 + 삭제. 만료 키가 응답에 새어나가지 않게.
2. **Active(주기 스위프)**: `ExpiryReaper`가 주기적으로 만료 행 일괄 삭제. 안 하면 `DBSIZE`·디스크가 어긋남.

`bun:sqlite`는 동기 API라 단일 프로세스 내 원자성 추론이 단순하다. **단일 writer 가정**을 계약에 명시(여러 서버 프로세스가 같은 .db 파일을 공유하지 않는다).

---

## 6. 연결 상태기계 & 부가 시스템

### 6.1 Connection 상태
```
        connect
           │
           ▼
     ┌───────────┐  HELLO/AUTH/SELECT ok   ┌────────┐
     │ HANDSHAKE │ ──────────────────────▶ │ READY  │
     └───────────┘                         └────┬───┘
                                                │ SUBSCRIBE
                                                ▼
                                         ┌──────────────┐
                                         │  SUBSCRIBED  │ (명령 제한 모드)
                                         └──────────────┘
```
- READY에서만 일반 명령 허용. HANDSHAKE 미완 상태에서 데이터 명령이 오면 에러 또는 보류 정책 결정(구현 시).
- **SUBSCRIBED 모드**: Redis 의미론상 subscribe된 연결은 (P)SUBSCRIBE/UNSUBSCRIBE/PING/QUIT만 허용. 일반 명령은 에러. 클라이언트가 `.duplicate()`로 별도 연결을 쓰는 것을 전제.

### 6.2 Pub/Sub (1차 포함)
- `PubSubHub`: 채널 → 구독 연결 집합. `PUBLISH`가 해당 채널 구독 연결들에 메시지 푸시.
- **단일 프로세스 메모리 허브**로 1차 구현(같은 서버에 붙은 연결 간 전달). 다중 프로세스 브로드캐스트는 비목적.
- 메시지 푸시는 RESP3 push 타입(`>`)으로. 클라이언트 `subscribe(channel, (msg, ch) => {})` 콜백과 정합.

### 6.3 Transaction (1차 포함, 최소)
- `MULTI` → 연결별 `TxnContext`에 명령 큐잉 시작. `EXEC` → SQLite 트랜잭션으로 일괄 실행 후 결과 배열 반환. `DISCARD` → 큐 폐기.
- `WATCH`는 낙관적 락 — 1차에서는 **단일 writer 가정** 덕에 단순화 가능하나, 정확한 의미론(키 변경 감지 시 EXEC nil)은 확장 단계에서 강화.
- 이 명령군은 자동 파이프라이닝 비활성 대상이므로 연결별 순차 처리와 자연히 맞물린다.

---

## 7. 구현 로드맵 (MVP 우선 → 단계적 확장)

호환성은 **명령 디스패치 테이블에 케이스를 더하는 것**으로 선형 확장된다. 단계마다 "순정 `Bun.RedisClient`로 통합 테스트 통과"가 완료 기준(DoD).

### Phase 0 — 핸드셰이크 + 핵심 KV (MVP, 1차 목표)
- L1 TCP 서버 + L2 RESP3 코덱(파서/직렬화) + 연결별 스트리밍 버퍼·파이프라인 순서.
- 핸드셰이크: `HELLO 3` / `AUTH` / `SELECT` / `PING` / `INFO`(최소) / `QUIT`.
- 핵심 KV: `SET` `GET` `DEL` `EXISTS` `INCR` `DECR` `EXPIRE` `TTL` + `getBuffer` 바이너리 경로.
- SQLite: `keys`+`kv` 테이블, WAL, lazy 만료, `ExpiryReaper` 기본형.
- **DoD**: 순정 클라이언트로 connect→set→get→incr→expire→ttl→del→exists 전 과정 통과. 타입 변환(§2.4) 검증 통과.

### Phase 1 — Hash / Set / Multi-key
- `HSET`/`HMSET`/`HGET`/`HMGET`/`HINCRBY`/`HINCRBYFLOAT`, `SADD`/`SREM`/`SISMEMBER`/`SMEMBERS`/`SRANDMEMBER`/`SPOP`, `MGET`/`MSET`/`MSETNX`/`SETEX`/`SETNX`.
- `hash_fields`/`set_members` 테이블. `SISMEMBER`/`EXISTS` boolean 변환 계약 고정.

### Phase 2 — Pub/Sub + Transaction
- 연결 상태기계에 SUBSCRIBED 모드 + `PubSubHub` + RESP3 push.
- `MULTI`/`EXEC`/`DISCARD`/`WATCH`(기본).

### Phase 3 — 확장 명령 (필요 시)
- list(`LPUSH`/`RPOP`/`LRANGE`), zset(`ZADD`/`ZRANGE`...), `SCAN`/`KEYS`, `INFO` 확장.
- 각 명령군마다 저장 테이블 + 디스패처 케이스 + 통합 테스트.

### 영구적 비목적
- Cluster / Sentinel (클라이언트 미지원).
- 다중 프로세스 공유 .db / HA / failover.
- 인프로세스 라이브러리 모드.

---

## 8. 테스트 전략 (호환성 검증 SSOT)

호환성은 "주장"이 아니라 "통과한 통합 테스트"로만 증명된다.

- **계약 테스트**: 서버를 `Bun.listen`으로 띄우고, **순정 `new RedisClient(thisServerUrl)`** 로 모든 지원 명령을 호출. 클라이언트가 반환하는 **JS 값의 타입과 값**을 단언. (메서드 흉내가 아니라 wire 호환을 검증하는 유일한 방법.)
- **타입 변환 테스트**: `EXISTS`/`SISMEMBER` → boolean, `GET` 미존재 → `null`, `getBuffer` → Uint8Array, hash → 배열/객체 등 §2.4 표를 그대로 테스트 케이스화.
- **파이프라인 테스트**: `Promise.all([...여러 명령])`로 자동 파이프라이닝을 유발하고 순서·정확성 단언.
- **만료 테스트**: `EXPIRE` 후 lazy/active 양 경로에서 미존재 확인.
- **재시작 영속성 테스트**: set → 서버 재시작 → get 생존 확인.
- **차분 테스트(선택)**: 동일 명령 시퀀스를 진짜 Redis와 이 서버에 각각 던져 클라이언트 반환값 비교.

---

## 9. 개발 스택 / 규약

- 런타임: **Bun** (외부 의존성 0 목표 — `bun:sqlite`, `Bun.listen` 모두 내장).
- 언어: **TypeScript** (strict). 명령 핸들러 시그니처는 타입으로 고정.
- 코드 구현은 **Claude Code에서** 진행. 본 문서는 설계·경계·계약만 규정한다.
- 우선순위 충돌 시 판단 기준: **호환성 계약(§2) > 데이터 정합성 > 성능.**

---

## 10. 핵심 의사결정 요약 (왜 이렇게 했나)

| 결정 | 이유 |
|---|---|
| 서버(A)만, 인프로세스(B) 비채택 | 접속은 무조건 순정 `Bun.RedisClient`. 클라이언트 메서드 흉내가 아니라 wire 호환이 SSOT. |
| RESP3 필수 | 클라이언트가 RESP3로 말함. RESP2만 하면 타입 변환이 어긋남. |
| 전송은 TCP만, UDP 비채택 | RESP는 순서·신뢰성 필요. UDP는 부적합. |
| Storage 추상 경계 | 명령 엔진이 SQLite에 직접 의존하지 않도록. 테스트·교체 가능성 확보. |
| 메타 통합 / 값 타입별 분리 | 단일 테이블은 zset·set에서 인덱스가 꼬임. 균형점. |
| TTL lazy+active 양쪽 | Redis 체감 일치. 한쪽만 하면 DBSIZE·디스크 어긋남. |
| 단일 writer 가정 명문화 | SQLite 동시성 모델과 원자성 추론을 단순화. 다중 프로세스 공유는 비목적. |
| 호환성=디스패치 테이블 확장 | 명령 추가가 선형 확장이 되도록 확장 지점을 한 곳에 집중. |

---
> Source: [Munsunty/bundis](https://github.com/Munsunty/bundis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
