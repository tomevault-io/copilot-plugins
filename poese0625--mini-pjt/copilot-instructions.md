## mini-pjt

> - Python, LangChain, LangGraph

# 프로젝트 규칙

## 기술 스택
- Python, LangChain, LangGraph
- 모델은 Amazon Bedrock (ChatBedrockConverse), 임베딩은 Bedrock `amazon.titan-embed-text-v2:0`
- 모델은 MODEL_IDS 목록(Sonnet 3종 → Haiku 2종 → Amazon Nova Pro 1종, 총 6종)으로 관리하고, 토큰 한도(ThrottlingException)에 걸리면 다음 모델로 자동 폴백한다. 체크포인터는 에이전트 밖에 두어 모델이 바뀌어도 승인 대기 상태가 유지된다
- Agent 생성은 langchain.agents 의 create_agent 를 쓴다 (ReAct, bind_tools)
- 벡터 저장소는 ChromaDB(로컬/임베디드), 검색은 BM25+벡터 가중 RRF 앙상블(hybrid retriever)
- 검색기 결합은 LangChain EnsembleRetriever(langchain_classic.retrievers)를 쓴다. 내부적으로 가중 RRF 라 점수가 아닌 순위만 쓰므로 BM25 점수와 벡터 유사도의 스케일 차이를 신경쓰지 않는다. 가중치는 VECTOR_WEIGHT/BM25_WEIGHT, 순위 완충 상수는 RRF_K(=c)로 조정하고, 바꾸면 evaluation/run_rag_eval.py 로 MRR 변화를 확인한다
- 승인(HITL)은 LangGraph `interrupt()` + checkpointer 기반으로 구현한다
- 그래프 시각화와 디버깅은 LangGraph Studio 를 쓴다. 설정은 src/langgraph.json 이고 그래프 진입점은 src/agent.py 의 studio_graph() 다

## 문서
- SERVICE.md          기획·정책·성공 기준. 요구사항과 평가 기준의 기준 문서다
- BUILD_CHECKLIST.md  12개 구현 패턴 점검표(패턴/실제 상태/판정). 기능을 추가하거나 제거하면 이 표의 판정을 함께 갱신한다

## 폴더 구조 (제출 규약. 바꾸지 않는다)
src/agent.py       create_agent 기반 에이전트 그래프. doc_type 판정(양식 선택), bind_tools, QA 검증 강제 후처리(정보량과 무관하게 항상 실행), 승인 대기를 재개하는 resume_agent() 를 포함한다. 체크포인터 유지를 위해 에이전트는 싱글턴으로 보관한다
src/tools.py       도메인 도구: retrieve_docs, get_current_datetime, lookup_template, validate_notice, persist_notice
src/api.py         POST /query API 서버(FastAPI). 신규 질문과 승인/거부 요청을 같은 엔드포인트에서 처리한다
src/langgraph.json LangGraph Studio 설정. graphs 는 src/agent.py:studio_graph 를 가리키고, env 는 리포지토리 루트의 .env 를 쓴다
src/callbacks.py   BaseCallbackHandler 상속 콜백. LLM/도구 이벤트를 data/trace.jsonl 에 JSONL 로 기록한다(입출력·토큰·지연시간)
src/retriever.py   BM25+벡터 하이브리드 RAG 파이프라인. data/notices.jsonl 일괄 적재와 확정 후 저장을 포함한다
run.sh             실행 스크립트(api/cli/eval/eval-rag/eval-faith/studio/install)
data/              notices.jsonl(RAG 원본 데이터), ChromaDB 영속화 디렉터리(예: data/chroma/)
evaluation/rag_queries.csv        RAG 검색 평가용 질의 15건(실사용 말투)과 정답 문서 ID
evaluation/run_rag_eval.py        검색 품질 평가(Recall@k/Precision@k/MRR). LLM 심판 없이 결정론적으로 계산한다
evaluation/run_faithfulness_eval.py  생성 충실도 평가. RAG 근거 기반 서술형 항목만 Bedrock 심판으로 판정한다(기본 5문항)
evaluation/round1_report.md       1차 자체 평가 결과(44/47, 발견한 코드 결함 4건과 실패 3건 분석)
evaluation/round2_report.md       2차 자체 평가 결과(47/47, 개선 내역과 기준 대비 달성 현황)
evaluation/        test_queries.csv(data/notices.jsonl과 무관한 held-out 샘플, 총 46건: positive/negative/edge/guardrail 정성 11건 + quant_format 20 · quant_critical 10 · quant_rag_hit/quant_rag_miss/quant_minor 스모크 5건)와 평가 리포트

## 주고받는 형식 (제출 규약)
- POST /query 로 받고 question 필드를 읽는다
- 답은 answer, contexts, trace 세 키로 돌려준다
  - contexts 는 [{doc_id, text}] 형태로 문서 하나를 원소 하나로 담는다
  - trace 는 [{step, input, output}] 형태로 실행 단계를 담는다(step 은 도구 이름)
- 문구 생성 후 persist_notice 내부의 interrupt() 로 일시정지하므로(HITL 승인 게이트), 이 흐름을 지원하기 위해 응답에 thread_id 와 status(예: "pending_approval")를 추가로 포함한다(기존 세 키 유지, 필드만 확장)
- 승인/거부 요청은 같은 엔드포인트에 { thread_id, resume: true, approved: true|false, final_text } 를 실어 보낸다. approved=true 면 final_text 를 RAG에 저장하고, approved=false 면 저장 없이 종료한다
- CLI(python src/agent.py)는 승인 대기 시 같은 프로세스 안에서 y/e/n 입력을 받아 처리한다. MemorySaver 는 메모리 전용이라 프로세스가 끝나면 체크포인트가 사라지므로, 별도 명령으로 나중에 thread_id 를 재개할 수는 없다
- 사용자가 문구를 직접 수정한 경우 final_text 는 수정본이며, QA 재검증 없이 그대로 저장한다. 이 흐름은 입력 정보량과 무관하게 모든 케이스에 동일하게 적용된다

## LangGraph Studio
- 설정 파일은 src/langgraph.json 이며, 그래프 진입점은 src/agent.py 의 studio_graph() 다
- studio_graph() 는 체크포인터를 달지 않는다(checkpointer=None). Studio 가 스레드와 체크포인트를 자체 저장소로 관리하므로, 우리 _CHECKPOINTER 를 넘기면 승인 재개가 어긋난다
- 반대로 CLI(python src/agent.py)와 API(src/api.py)는 _CHECKPOINTER 를 공유하는 build_agent() 를 그대로 쓴다. 두 경로가 체크포인터를 다르게 쓰는 것이 의도된 설계다
- 실행: `pip install "langgraph-cli[inmem]"` 후 `cd src && langgraph dev`
- Studio 에서도 persist_notice 호출 시 interrupt() 로 멈춘다. 재개하려면 Studio 의 resume 입력에 {"approved": true, "final_text": "..."} 형태로 넣는다
- 모델 폴백(MODEL_IDS)은 invoke_with_fallback() 안에 있어 Studio 실행 경로에는 적용되지 않는다. Studio 에서 토큰 한도에 걸리면 MODEL_IDS 앞쪽 순서를 바꾸거나 한도가 풀린 뒤 다시 시도한다

## 코드 규칙
- 파일 하나에 한 가지 역할만 둔다
- 함수와 도구에는 한국어 docstring 을 쓴다
- 비밀 값은 .env 에서 읽고 코드에 적지 않는다
- 추적은 두 갈래다. 자체 기록은 src/callbacks.py 가 data/trace.jsonl 에 남기고, LangSmith 는 .env 의 LANGSMITH_TRACING/LANGSMITH_API_KEY/LANGSMITH_PROJECT 로 자동 활성화된다(코드 수정 불필요)
- LangSmith 로 프롬프트와 공지 내용이 외부 전송되므로, 내부망 데이터 정책상 문제가 되면 LANGSMITH_TRACING=false 로 끈다
- doc_type(작업/장애)은 양식(7/10항목)을 고르려고 판정한다. 한 요청에 작업과 장애가 동시에 들어오는 경우는 서비스 전제상 없다고 보며, 단서가 희미해도 되묻지 말고 더 가까운 쪽으로 정해 진행한다
- doc_type이 확정되면, 입력의 콘텐츠 필드 개수(작업: 작업목적·작업대상·작업영향 / 장애: 영향서비스·장애현상)와 무관하게 항상 동일한 표준 파이프라인(retrieve_docs → 생성 → validate_notice → 승인 대기)을 실행한다. 콘텐츠 필드가 0개(정보 극히 부족)여도 사용자에게 되묻지 않고 agent가 채울 수 있는 최대한을 채워 반환하며, 채울 수 없는 항목은 빈칸으로 남기고 빈칸에는 어떤 표기도 붙이지 않는다. "극히 부족" 판단은 입력 글자수가 아니라 구조화 추출 결과의 콘텐츠 필드 개수이며(자동채움 대상 필드인 일시·발송자·문의처·장애등급은 집계 제외), 이 판단은 내부 분류일 뿐 별도 분기나 되묻기를 유발하지 않는다
- 문구 생성 직후 validate_notice(QA)는 run_agent 후처리에서 항상 강제 호출한다(LLM이 자율적으로 건너뛰지 않도록 하며, 입력 정보량과 무관하게 항상 실행한다). LangChain 미들웨어(create_agent(middleware=...))는 쓰지 않는다
- 검수 결과는 별도 블록이 아니라 해당 항목 값 뒤에 괄호로 인라인 표기한다. validate_notice 는 {항목명: 표기문구} 매핑(annotations)을 반환하고, agent 가 그걸 문구에 삽입한다. 모델이 빠뜨렸을 때를 대비해 코드에서도 강제 보정한다
- 빈칸 항목에는 어떤 표기도 붙이지 않는다. 오타는 TYPO_DICT 기반으로 빈출 사례만 탐지한다
- 치명적 오류(날짜형식·placeholder·필수항목누락)를 발견해도 생성을 차단하지 않는다. 입력값을 그대로 반영한 원본 문구를 반환하고 임의로 고쳐서 넣지 않는다
- 과거 날짜는 오류가 아니다(사후 공지가 정상 시나리오). 과거 일시에는 어떤 표기도 붙이지 않는다
- 발송자·문의처 등 조직 고정값은 자동 채우고, 사람 고유값(작업자·조치담당 등)과 채울 수 없는 콘텐츠 필드는 추론 불가 시 빈칸으로 남긴다(임의로 지어내지 않는다)
- 장애등급은 조직 기본값로 취급한다. 입력에서 알 수 없으면 빈칸이 아니라 기본값 4등급으로 채우고 제목에도 동일하게 반영한다
- 실제 슬랙/이메일 등 외부 채널 발송 기능은 구현하지 않는다(agent 책임 범위 밖, 항상 사람이 수행)
- RAG 원본 데이터 파일 경로는 항상 `data/notices.jsonl`을 사용한다(리포지토리 루트의 `notices.jsonl`을 참조하지 않는다)

## 하지 말 것
- 요청하지 않은 파일을 새로 만들지 않는다
- 기존 파일을 통째로 다시 쓰지 않는다. 바뀐 부분만 고친다

---
> Source: [poese0625/mini-pjt](https://github.com/poese0625/mini-pjt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
