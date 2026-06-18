## ariadne

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 한 줄 요약

Linux sysfs/procfs를 직접 파싱하여 **PC 호스트 내 모든 디바이스의 정보와 토폴로지(CPU–메모리–PCIe–디바이스)** 를 그래프로 모델링하고, 임의의 두 디바이스 간 E2E 경로의 BW/latency를 추적·시각화하는 도구. CLI(Typer/Rich) + Web UI(FastAPI + Vanilla JS) 모두 동일한 Engine API를 호출한다.

## 프로젝트 방향성 (중요)

Ariadne의 두 가지 핵심 확장 방향. 새 기능을 추가할 때 두 방향을 모두 의식해야 한다.

### 방향 1: 단일 호스트 → 다중 호스트

현재는 **단일 호스트** 분석이지만, 최종 목표는 **호스트 접속 정보(예: SSH)를 받아 여러 호스트를 한 번에 분석·시각화**하는 도구다.

- **수집 위치는 어디든 될 수 있다** — collector 로직은 sysfs/procfs를 "원격에서 실행"되거나 "원격 결과를 JSON으로 받아 처리"되는 두 방식 모두를 지원해야 한다. `docs/DESIGN.md`의 "원격 수집" 항목 참조. `snapshot`/`load` 명령과 `SystemTopology`의 JSON 직렬화는 이 확장의 출발점이다 — 이미 구현된 이 직렬화 경계를 깨지 말 것.
- **`hostname` 필드는 호스트 식별자다** — `SystemTopology.hostname`은 단일 호스트 표시용 문자열이지만, 다중 호스트 모델로 확장 시 이 필드가 키가 될 가능성이 높다. component ID는 현재 글로벌 unique지만, 다중 호스트에서는 host-scoped namespace가 필요할 수 있음을 새 ID 규칙 설계 시 고려.
- **`api/server.py`의 `_cached_topo`는 단일 인스턴스 캐시** — 다중 호스트 시 `host_id → topology` 맵으로 바뀔 자리. 새 캐싱 코드를 짤 때 이 변경을 염두에 둔 인터페이스로.

### 방향 2: 다른 프로젝트의 데이터 공급원 (consumer-ready API)

Ariadne는 **자체 도구(CLI/Web UI)**일 뿐 아니라, **다른 프로젝트가 import/HTTP로 사용하는 토폴로지 정보 공급원**이 되어야 한다. 시각화 도구가 보여주는 모든 정보(디바이스 간 + **디바이스 그룹 간** 통신 비용/대역폭)는 인터페이스/API로 노출되어야 한다.

**1차 소비자: lmtune** (`~/ml_ai/benchmark/`, package `lmtune`) — LLM endpoint 벤치마크 자동화 시스템. 현재 endpoint를 `slug` + `notes`로만 HW 태깅하고 있어, Ariadne가 그 자리에 들어간다. lmtune은 LLM serving 인프라 정보(GPU/NPU/NIC 배치, NUMA 거리, NVLink/PCIe 토폴로지, 디바이스/디바이스 그룹 간 통신 비용)를 필요로 한다.

**의존성은 단방향: `lmtune → ariadne`**. ariadne 코드가 `lmtune`을 import하거나 lmtune의 도메인 용어(TP/PP/DP, all-reduce 등 collective 명칭)를 채택하는 일은 없다. ariadne는 일반화된 그래프/디바이스 그룹/통신 패턴(`one_to_many`, `many_to_one`, `all_to_all` 등) 형태로만 결과를 노출하고, lmtune이 **소비자 측에서** 자체 도메인(TP/PP 그룹, all-reduce 비용)에 매핑한다.

이 방향이 새 코드에 부과하는 제약:

- **새 분석 기능은 Engine API에 먼저 추가하고, CLI/Web UI는 그것을 호출하기만 한다** — UI에만 존재하는 로직이 있으면 외부 소비자는 못 쓴다. 시각화 라벨 계산조차도 Engine 함수가 반환한 값을 표시하는 형태여야 한다.
- **모든 응답은 JSON-serializable Pydantic 모델** — `TraceResult`처럼 평범한 클래스 말고 `BaseModel` 기반으로 통일해 `model_dump()`/`model_dump_json()` 직렬화 보장. lmtune이 DuckDB/Parquet에 그대로 적재할 수 있어야 한다.
- **그룹 통신 분석이 1차 시민 기능** — 현재 `trace_path()`는 1:1만 다룬다. M:N 그룹 trace(예: GPU TP-group 4개 ↔ NIC, GPU 그룹 ↔ 다른 GPU 그룹의 all-reduce 비용)는 다른 프로젝트 요구사항이므로 별도 함수로 노출되어야 한다 — UI 동작에만 묶지 말 것.
- **REST API는 안정 계약** — `api/server.py` 엔드포인트 변경은 lmtune 같은 외부 소비자를 깨뜨릴 수 있다. 새 필드 추가는 자유, 기존 필드 의미 변경/제거는 신중히. 새 엔드포인트로 분리하는 편이 안전.
- **Snapshot JSON은 안정 포맷** — lmtune의 `runs` 메타에 그대로 첨부될 수 있어야 한다. 외부 소비자가 같은 토폴로지 ID로 여러 endpoint 측정 결과를 묶을 수 있도록 `hostname` + 타임스탬프 + (다중 호스트 시) host_id 같은 식별자를 안정적으로 유지.
- **공개 Python API는 `ariadne.__init__`에서 명시적으로 export** — 현재 `__init__.py`는 비어 있다. consumer-ready로 가려면 `from ariadne import build_topology, trace_path, SystemTopology, ...` 형태의 진입점을 정리할 시점.

## 명령어

모든 명령은 `ariadne-core/`에서 실행한다. 가상환경 활성화 후:

```bash
cd ariadne-core
source .venv/bin/activate
pip install -e ".[dev]"        # 최초 설치 (entry: ariadne)
```

| 작업 | 명령 |
|------|------|
| 전체 토폴로지 트리 출력 | `ariadne show` (요약: `--summary`) |
| 상세 정보 포함 | `sudo $(which ariadne) show` — PCIe config space, DIMM 등은 root 필요 |
| E2E trace (CLI) | `ariadne trace` (인터랙티브 fuzzy) / `ariadne trace gpu:0 memory` / `ariadne trace 0000:01:00.0 nvme:0` |
| Snapshot 저장/로드 | `sudo $(which ariadne) snapshot out.json` / `ariadne load out.json` |
| Web UI | `ariadne serve --port 8000` → http://localhost:8000 |
| 전체 테스트 | `pytest` |
| 단일 테스트 | `pytest tests/test_collector_cpu.py::test_collect_cpu_cores_smt -v` |

`pyproject.toml`은 `ariadne-core/`에만 있다 — 루트가 아니라 서브디렉터리가 패키지 루트다.

## 아키텍처 — 데이터 흐름 한 사이클

요청 한 번이 어떻게 흐르는지를 머리에 넣으면 어떤 파일을 읽어야 할지 빠르게 결정할 수 있다.

```
sysfs/procfs (/sys/bus/pci, /sys/devices/system/cpu, /proc/meminfo, /proc/cmdline)
   │
   ▼  collector/  (cpu, numa, memory, pcie, iommu)
raw dicts / Pydantic 모델 (numa_nodes, cpu_cores, pci_devices, iommu_groups …)
   │
   ▼  model/topology.py :: build_topology()
SystemTopology (Pydantic) — 핵심 두 리스트:
  • components: list[Component]   # NUMA, Socket, Core, L3, MC, DRAM, RC, RP, GPU, NPU, NIC, NVMe
  • links:      list[Link]        # internal / pcie / memory / upi / nvlink …
   │
   ▼  model/topology.py :: to_networkx()  →  NetworkX DiGraph (undirected로 변환해 사용)
   │
   ├─→ analyzer/trace.py :: trace_path()  →  shortest_path + 구간별 BW/latency 누적
   │     • PCIe segment: edge bandwidth × pcie_efficiency(0.90)
   │     • Memory: bandwidth × 0.75
   │     • UPI: numa_remote_latency_ns
   │     • bottleneck = min(effective_bw)
   │     • DEFAULT_PARAMS dict가 모든 모델 파라미터의 단일 진입점
   │
   ├─→ viz/terminal.py            (CLI 출력)
   └─→ api/server.py              (FastAPI)
            ├─ GET /api/topology         전체 SystemTopology JSON
            ├─ GET /api/topology/graph   {nodes, edges, iommu_groups, iommu_context}
            ├─ GET /api/trace            TraceResult JSON
            └─ POST /api/topology/reload _cached_topo 무효화 후 재수집
                                          ↓
                                    web/static/{core.js, ui.js, style.css}
                                    web/templates/index.html
```

### 모듈 책임 경계

- **collector/** — sysfs/procfs 직접 파싱. `subprocess` 호출은 `pcie.get_vendor_name`(lspci)이 거의 유일. **외부 도구(hwloc 등) 의존 없음**이 명시적 설계 원칙. 모든 collector 함수는 `sysfs_base: Path` 인자를 받아 테스트 시 임시 디렉터리로 주입 가능 (`tests/test_collector_cpu.py` 참조).
- **model/types.py** — 모든 데이터 클래스(`Component`, `Link`, `SystemTopology` 등)와 Enum. `ComponentPrefix`(P)는 모든 component ID 생성의 단일 출처 — 직접 `f"numa_{n}"` 같은 문자열 조합 금지, 반드시 `f"{P.NUMA}{n}"` 사용.
- **model/topology.py** — `build_topology()`가 collector 결과를 NetworkX 그래프 노드/링크로 변환하는 유일한 진입점. PCIe 트리 조립은 `_build_pcie_components()` → `_add_host_bridges/_root_ports/_endpoints`로 분할되어 있다.
- **analyzer/trace.py** — Engine의 핵심 시뮬레이션 로직. `DEFAULT_PARAMS`가 모든 모델 상수의 source-of-truth.
- **api/server.py** — `_cached_topo` 글로벌로 토폴로지 캐싱. UPI(NUMA↔NUMA) 링크는 트리에서 의도적으로 제외 (peer 관계, 부모-자식 아님).
- **cli/main.py** — 무거운 모듈은 명령 핸들러 안에서 lazy import (CLI 시작 속도). `_resolve_target`이 `gpu:0`/`memory`/`0000:01:00.0` 같은 다양한 표기를 component ID로 변환.
- **web/static/** — `core.js`(상수, 상태, 데이터 로딩, edge SVG 렌더링) + `ui.js`(인터랙션, trace, sidebar). 의도적 분할 — 한 파일에 묶지 말 것.

### 토폴로지 구조 규칙 (자주 헷갈림)

- **물리 부모-자식 관계만 트리 링크**. NUMA→NUMA UPI는 peer이므로 `LinkType.UPI`로 별도 표현하고 graph API에서 트리 응답 시 제외한다.
- Memory Controller(MC), PCIe Root Complex(RC)는 **물리적으로 Socket 내부**. 따라서 부모는 `Socket`, CPU 없는 NUMA 노드일 때만 `NUMA`로 fallback (`_find_socket_for_numa`).
- `traceable` 노드 타입은 web UI(`core.js`)와 CLI에서 동일 정의: `gpu, npu, nvme, nic, memory_controller, pcie_endpoint`. 이 6개 외 타입은 trace 시작점/끝점이 될 수 없다.

## 코드 컨벤션 (이 repo 특유)

- **Python 들여쓰기는 2 spaces** — PEP8(4 spaces) 아님. 글로벌 가이드의 PEP8 규칙보다 이 repo의 기존 컨벤션이 우선한다.
- 한국어 주석 우세 — 새 코드도 한국어로 통일.
- Component ID는 항상 `ComponentPrefix` 상수로 생성. 새 ComponentType을 추가할 땐 `types.py`의 enum과 `ComponentPrefix`, `web/static/core.js`의 `COLORS`/`TRACEABLE`/`FILTER_MAP` 모두 동기화.
- 새 모델 파라미터는 `analyzer/trace.py`의 `DEFAULT_PARAMS`에 추가하고 함수 인자 `params`로 override 가능하게 유지.
- collector 함수는 sysfs 경로를 인자로 받아 테스트 가능하게 작성 (`sysfs_base: Path = SYSFS_PCI_BASE` 패턴).

## 확장 시 참고

- **새 디바이스 벤더 지원**: `collector/pcie.py`의 `KNOWN_VENDORS` + (필요시) 제품명 매핑 dict 추가. Rebellions NPU(`0x1eff`)가 참조 구현.
- **새 LinkType**: `model/types.py`에 enum 추가 → `analyzer/trace.py`의 segment 분기에 latency/BW 처리 추가 → `web/static/style.css`에 `.edge-line.<type>` 스타일.
- **새 API 엔드포인트**: `api/server.py`에만 추가. 캐싱 필요하면 `_cached_topo` 패턴 따름. JSON-serializable 원칙 — 응답은 모두 dict/list/scalar.
- **Web UI 변경**: `docs/UI_GUIDE.md`(색상 체계, 레이아웃 상수)와 `docs/UI_FEATURES.md`(현재 기능 + 알려진 이슈) 먼저 읽기. CSS 변수(`--child-indent` 등)와 JS `LAYOUT` 객체가 레이아웃 상수의 단일 출처.

## 실행 환경 주의

- **Linux 전용**. sysfs/procfs 경로 하드코딩 — macOS/Windows는 collector가 빈 결과를 반환한다.
- `sudo` 없이도 기본 토폴로지는 수집되지만, PCIe BAR resource, DIMM 상세, IOMMU 그룹 등은 누락될 수 있다. 디버깅 시 권한 차이를 먼저 의심.
- `current_link_speed/width`는 디바이스가 D3 상태이면 0/빈 문자열일 수 있음 — `calc_pcie_bandwidth`는 이 경우 0.0 반환.

## 문서 구조

설계/요구사항 변경은 `docs/`의 해당 문서 먼저 업데이트:

| 문서 | 다룰 때 참조 |
|------|--------------|
| `docs/REQUIREMENTS.md` | 기능/비기능 요구사항, 사용자 시나리오 |
| `docs/DESIGN.md` | Engine/UI 분리 원칙, 입출력 모델, 시뮬레이션 레벨 |
| `docs/DATA_MODEL.md` | 7개 레이어별 측정 가능 항목, 모델 파라미터 |
| `docs/UI_GUIDE.md` | UI 색상/레이아웃 상수, 인터랙션 패턴 |
| `docs/UI_FEATURES.md` | 현재 구현된 기능 + 알려진 이슈 (리팩토링 기준) |
| `docs/COMPARISON.md` | hwloc, pcicrawler, gem5 등 7개 도구와 비교 |

---
> Source: [JinmooSeok-Dev/ariadne](https://github.com/JinmooSeok-Dev/ariadne) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
