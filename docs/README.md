# 공식 제출 문서 안내

> Status: Phase 1 문서 작성·감사 완료 / 제품 최종 검증은 항목별 Pending
> Implementation Baseline: `gyuniverse-hq/bid-change-validator` `develop@36f1afba8e1a025006b0bfa1876502e6232b3d28`
> 개발 HEAD 재확인: 2026-09-14 21:32:55 KST · 작업 중 기준 SHA 변경 없음
> 공식 문서 작업 대상: `SKNETWORKS-FAMILY-AICAMP/SKN34-3rd-4Team` / `docs` 브랜치

나라장터 공고와 첨부문서에서 참가자격 Requirement와 원문 Evidence를 연결하고, **Changed Notice → Requirement Diff → Affected Requirement → affected-only Revalidation**을 수행하는 프로젝트의 제출 문서입니다.

이 Index의 “완료”는 Phase 1 문서 작성·감사 범위입니다. 실공고 Ground Truth, Human Click E2E, 운영 배포, 공식 저장소 코드·원본 데이터 동기화 완료와 구분합니다.

## 1. 추천 읽기 순서

1. [요구사항](01_product/requirements.md) → [구현·검증 Traceability](02_architecture/feature-traceability.md): 무엇을 만들었고 무엇이 남았는가.
2. [기능 흐름](02_architecture/functional-flow.md) → [시스템](02_architecture/system-architecture.md) → [ERD](04_contracts/db-erd-current.md): 사용자 흐름·컴포넌트·저장 관계.
3. [수집·전처리](04_contracts/data-preprocessing.md) → [AI / Rule / RAG](03_ai/llm-rag-pipeline.md): 원문부터 요건·판정·설명까지의 책임.
4. [시나리오](08_qa_reports/test-scenarios.md) → [실제 테스트 결과](08_qa_reports/test-plan-and-results.md) → [Dataset](08_qa_reports/golden-set.md) → [평가](08_qa_reports/llm-rag-evaluation.md): 어떻게 확인했고 수치는 무엇을 뜻하는가.
5. [문제 해결](06_decisions/troubleshooting.md) → [협업](07_handoff/collaboration.md) → [실제 WBS](09_roadmap/wbs.md) → [향후 개선](09_roadmap/future-roadmap.md): 의사결정·진행·남은 일.

## 2. 기준선과 Source of Truth

1. 고정 SHA의 실제 **Code / Test / Alembic Migration**.
2. 개발 저장소 Current GitHub docs. 코드와 어긋난 과거 문구는 코드로 정정.
3. Current Notion Reconciliation / Control Center.
4. 초기 PRD·일정·Historical 문서는 당시 의도·ID·계획 확인용.

사용자가 마지막으로 확인한 `cafd5dba82a54e467eb59e9578995d3362c5ca9b` 이후 [PR #135](https://github.com/gyuniverse-hq/bid-change-validator/pull/135)가 추가됐습니다. Rule v0.3 및 화면·Copilot의 stale 판정 재사용 차단을 반영해 모든 상세 문서의 기준선을 위 SHA로 통일했습니다. PR #137의 같은 차수 중복 실행 방지도 포함합니다.

[루트 README](../README.md)는 요청 범위에 따라 수정하지 않았습니다. README v0.3의 `c26cdca` 기준보다 이 문서 세트가 최신이며, 후속 변경 목록은 6장에 둡니다.

## 3. 문서 Index와 Source Mapping

`Current`는 고정 구현·근거에 맞춘 문서입니다. `Draft`는 승인되지 않은 라벨/정답이며 `Validation Pending`은 독립·사람·실환경 완료 근거가 없다는 뜻입니다. Current 문서에도 Draft Dataset과 미완료 제품 항목이 포함될 수 있습니다.

아래 소스 경로는 [고정 개발 저장소](https://github.com/gyuniverse-hq/bid-change-validator/tree/36f1afba8e1a025006b0bfa1876502e6232b3d28) 기준입니다. 각 상세 문서에 실제 파일·PR·CI 링크를 제공합니다.

| 제출 문서 | 주 Source Mapping | 문서 상태 | 별도 검증·산출 상태 |
| --- | --- | --- | --- |
| [Requirements](01_product/requirements.md) | API/Rule/화면 코드 + Notion PRD Gap Map | Current | Partial/Future와 수용 기준 구분 |
| [Traceability](02_architecture/feature-traceability.md) | `apps/api/app`, API/Web tests, Golden | Current | 자동 Test와 Human 결과 분리 |
| [System Architecture](02_architecture/system-architecture.md) | Compose, package/requirements, main, Copilot/RAG | Current | 최종 배포·이미지 Pending |
| [Functional Flow](02_architecture/functional-flow.md) | Case Workspace, Analysis/Judgment/Ask-back/Revalidation | Current | Human Click Pending |
| [LLM/RAG Pipeline](03_ai/llm-rag-pipeline.md) | AI extraction/provider, Rule, document_rag, Copilot | Current | 독립 모델 품질·prompt 비교 Pending |
| [ERD](04_contracts/db-erd-current.md) | SQLAlchemy models + migration 001~022 | Current | 운영 DB schema·backup 확인 Pending |
| [Data Preprocessing](04_contracts/data-preprocessing.md) | G2B, notices/storage/extraction services, Poller | Current | live 수집 전수·원본 재파싱 Pending |
| [UX Flow](05_ui_ux/ux-flow.md) | Web routes, case-workspace, status-copy | Current | 최종 사용자 검수 Pending |
| [Screenshots](05_ui_ux/screenshots.md) | 실제 화면 코드·Human 시나리오 | Current capture plan | 실제 캡처 Pending; 이미지 링크 없음 |
| [Test Scenarios](08_qa_reports/test-scenarios.md) | A~D, API/DB Golden, Copilot tests | Current | 실제 사용자 실행 Pending |
| [Test Plan / Results](08_qa_reports/test-plan-and-results.md) | Actions Run 로그·workflow·테스트 코드 | Current evidence register | lint 실패와 Human/배포 Pending 명시 |
| [Golden Set](08_qa_reports/golden-set.md) | `samples/golden`, fixture hash, real adapter | Current inventory | Rule/G2/E3 라벨 Draft |
| [LLM/RAG Evaluation](08_qa_reports/llm-rag-evaluation.md) | Rule CI, E3 보고서, PR #128 | Current evidence summary | Extraction F1·G2·Task 평가 Pending |
| [Troubleshooting](06_decisions/troubleshooting.md) | PR #128/129/135/137, Askability, E3 | Current | 해결 검증과 잔여 한계 분리 |
| [Collaboration](07_handoff/collaboration.md) | Issue/PR/Actions, ownership, Notion | Current | formal review와 도구 활용 수준 한정 |
| [WBS](09_roadmap/wbs.md) | Git merge history + Notion Master Plan | Current retrospective | 향후 마감은 계획 |
| [Future Roadmap](09_roadmap/future-roadmap.md) | 현재 구현·평가 Gap | Current gap register | 개선 구현은 Future |

## 4. 공식 과제 대응 Audit

[공식 프로젝트 가이드](https://www.notion.so/3d0e94b44d07808ab550f092b6b49500)를 직접 확인했습니다. 필수 문서와 실제 소프트웨어·데이터 제출은 별도입니다.

| 가이드 항목 | 현재 대응 | 판정·남은 범위 |
| --- | --- | --- |
| 수집 데이터 및 전처리 문서 | Data Preprocessing + Golden inventory·재현 절차 | 문서 작성 완료. 원본 바이너리는 외부 snapshot; 공식 repo 데이터 동기화 후속 |
| 시스템 아키텍처 | System / Functional Flow / ERD | 문서·Mermaid 완료. 최종 배포 이미지 Phase 2 |
| 개발 소프트웨어: RAG LLM·벡터 DB 통합 코드 | AI Pipeline + pinned Code links; OpenAI·FAISS·LangChain Core 사용 확인 | 구현 구조 설명 완료. 공식 repo 코드 동기화·새 환경 실행은 Phase 4 |
| 테스트 계획 및 결과 | Test Scenarios / Plan / Evaluation | 실제 CI와 source-reported 측정·미검증 상태 구분 완료 |
| One-shot/Few-shot·모델 선택 실험 | 현재 prompt 구조·모델 기본값·Retrieval trade-off 기록 | 예시 유무 비교의 독립 실행 근거 미확인. 추가 평가 필요 |
| 권장: 문제 해결·기능/UX 흐름·향후 개선·역할 | 각 상세 문서 연결 | Phase 1 문서 완료 |

따라서 **필수 문서의 탐색·설명 범위는 갖췄지만 공식 제출 전체 완료는 아닙니다.** 코드·데이터·실행 증거는 후속 Phase에서 채워야 합니다.

## 5. Phase 1 최종 감사 기록

- **1-1 구조 감사:** scaffold `5e7c92c`의 18개 Markdown 구조를 유지했습니다. 본 작업 시작점은 1-2 초안이 있는 `9f273a5`였으며 불필요한 문서를 새로 만들지 않았습니다.
- **1-2~1-8 내용 감사:** 모든 문서를 재검토하고 Current/Partial/Superseded/Future, 구현/실행/독립 승인 상태를 분리했습니다.
- **내부 정합성:** Canonical 8유형, Domain 3상태+provenance, UNKNOWN != ASKABLE, Profile 자동 저장 금지, LLM/Rule/RAG 경계, G2 Draft, Evaluation Partial을 통일했습니다.
- **오래된 문구 정정:** 개발 화면 계약의 Rule v0.2·Copilot Proposed, 과거 ERD 009 범위, real adapter Future 표현을 현재 코드로 대체했습니다. 같은 차수 화면 중복 방지를 Analysis API 전체 멱등성으로 확대하지 않았습니다.
- **ERD 검산:** 그림의 32개 관계를 SQLAlchemy의 실제 FK 컬럼과 대조했습니다. Evidence→Document, Requirement→Evidence 등의 논리 참조를 물리 FK로 그리지 않았습니다.
- **링크 감사:** 공식 저장소의 루트 README 및 docs Markdown 상대 경로를 검사해 깨진 링크 **0건**. 고정 SHA의 GitHub 파일·디렉터리 링크도 개발 checkout에서 존재를 확인했습니다. Notion 권한·외부 서비스의 향후 가용성을 보증하는 검사는 아닙니다.
- **과장 감사:** CI 성공에 가려진 lint 20 errors를 기록했습니다. Rule 초안 회귀·E3 구조 지표를 서비스 정확도로 합치지 않았습니다. G2 승인·Human/배포 성공·독립 Extraction F1을 만들어 넣지 않았습니다.
- **수정 범위:** `docs/` Markdown만 변경했습니다. 제품 코드·루트 README·`main`은 수정하지 않았습니다. 실제 UI 이미지 생성·제품 테스트 재실행·운영 DB 변경은 하지 않았습니다.

원시 CI Run ID와 checksum·측정 분모는 각각 Test Plan / Golden / Evaluation에 유지하여 Index와 장문 중복하지 않습니다.

## 6. 루트 README 후속 변경 목록

이번에는 수정하지 않았으며 Phase 5에서 다음을 반영합니다.

1. 구현 baseline을 최종 Evidence Lock SHA로 갱신하고 이 문서 Index·필수 산출물 링크 추가.
2. Rule v0.3, 비변경 요건의 이전 판정 부재 시 재판정 예외, stale guard 설명 반영.
3. 최신 CI의 범위·실행 SHA·lint 실패, Rule 9/14 유지 결과와 평가 한계를 상세 문서로 연결.
4. `/evaluation`의 제안서 업로드·위치 후보·로컬 확인 범위를 명시하고 최종 점수화와 구분.
5. 승인된 실제 UI/Architecture 이미지만 추가하고 G2/Human/Deployment 완료 상태를 실제 증거에 맞춰 갱신.
6. 공식 코드·데이터 동기화 이후에만 공식 repo 자체 실행법으로 전환.

다음 단계는 **Phase 2 — 제품 증거 / 시각자료**입니다. 캡처 계획은 [Screenshots](05_ui_ux/screenshots.md), 사람 검수는 [Test Scenarios](08_qa_reports/test-scenarios.md), 후속 우선순위는 [Future Roadmap](09_roadmap/future-roadmap.md)에 있습니다.
