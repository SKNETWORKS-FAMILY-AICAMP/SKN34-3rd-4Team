# 공식 제출 문서 안내

> Status: Draft / Final Submission Preparation
> Implementation Source: `gyuniverse-hq/bid-change-validator` `develop`

## 목적

공식 제출 문서의 구성과 작성 상태를 안내하는 Index입니다. 각 문서는 실제 구현과 Current 문서를 검증한 뒤 단계별로 완성합니다.

## Source of Truth

- 실제 동작과 상태: 개발 저장소의 Code / Test
- 현재 구조와 계약: 개발 저장소 `docs/README.md` 및 `docs/01_product`~`docs/09_roadmap`
- 작업 이력: 실제 GitHub Issue / Pull Request / Review / Test / Merge 기록

## 문서 목록

| 영역 | 문서 | 목적 | 현재 상태 |
| --- | --- | --- | --- |
| Product | [requirements](01_product/requirements.md) | 최종 요구사항 | Draft |
| Architecture | [system architecture](02_architecture/system-architecture.md) | 전체 시스템 구조 | Draft |
| Architecture | [traceability](02_architecture/feature-traceability.md) | 요구사항 ↔ 구현 ↔ Test | Draft |
| Architecture | [functional flow](02_architecture/functional-flow.md) | 사용자 기능 흐름 | Draft |
| AI | [LLM/RAG pipeline](03_ai/llm-rag-pipeline.md) | AI / Rule / RAG 책임 경계 | Draft |
| Data | [ERD](04_contracts/db-erd-current.md) | 실제 데이터 Lineage | Draft |
| Data | [preprocessing](04_contracts/data-preprocessing.md) | 수집·전처리 | Draft |
| UX | [UX Flow](05_ui_ux/ux-flow.md) | 화면·사용자 흐름 | Draft |
| UX | [screenshots](05_ui_ux/screenshots.md) | 최종 화면 증거 관리 | Draft |
| QA | [test scenarios](08_qa_reports/test-scenarios.md) | 사용자 / Test Case | Draft |
| QA | [test plan/results](08_qa_reports/test-plan-and-results.md) | 테스트 계획·결과 | Draft |
| QA | [LLM/RAG evaluation](08_qa_reports/llm-rag-evaluation.md) | 정량 평가 | Draft |
| QA | [golden set](08_qa_reports/golden-set.md) | 평가 데이터 정의 | Draft |
| Decisions | [troubleshooting](06_decisions/troubleshooting.md) | 문제 해결 사례 | Draft |
| Collaboration | [collaboration](07_handoff/collaboration.md) | 실제 협업 방식 | Draft |
| Roadmap | [WBS](09_roadmap/wbs.md) | 실제 프로젝트 진행 | Draft |
| Roadmap | [future](09_roadmap/future-roadmap.md) | 현재 한계 → 향후 개선 | Draft |

## 작성 예정 내용

- Phase 1 순서에 따라 각 문서의 실제 근거와 내용을 채웁니다.
- 완료 근거가 확인된 문서만 상태를 갱신합니다.

## 검증 원칙

확인하지 않은 구현이나 수치는 완료로 표현하지 않는다.
