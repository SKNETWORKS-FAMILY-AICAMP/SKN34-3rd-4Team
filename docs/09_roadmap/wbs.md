# WBS — 계획 대비 실제 진행

> Status: Current Retrospective / 향후 Gate는 계획
> Implementation Baseline: `gyuniverse-hq/bid-change-validator` `develop@36f1afba8e1a025006b0bfa1876502e6232b3d28`

## 1. 목적

초기 일정표를 완료 사실로 복제하지 않고 Git commit·PR 병합·현재 문서 상태로 실제 진행을 정리합니다. 날짜는 merge 이력의 날짜이며 작업 시작·투입 시간·개인 기여율을 역산하지 않습니다. 9/15 이후 일정은 계획입니다.

## 2. 실제 진행 흐름

```text
Topic / Planning → Initial Parallel Development
→ MVP Integration Baseline → 공통 기준선 확보
→ Frontend·Backend·DB·AI 병렬 고도화
→ Hardening → Golden / Evaluation → Finalization
```

| 단계 / 기간 | Plan | Actual / 산출 근거 | 변경 이유·남은 일 |
| --- | --- | --- | --- |
| 주제·기획 / 9/2~9/4 초기 기록 | 주제·MVP·역할·Figma·프로토타입 검토 | 나라장터 변경공고 대응 방향과 초기 역할 확정 기록 | 모든 초기 기능을 현재 MVP로 승격하지 않음 |
| 초기 파트 개발 / 9/7 이전~9/7 | Front/Back/DB/LLM 각 영역 구현 | PR #44 LLM/RAG integration 등 초기 병합 | 독립 구현만으로 전체 흐름·계약 정합성을 보증하기 어려움 |
| Integration Baseline / 9/8 | 공통 사용자 흐름 기준 확보 | PR #75, f5fce26, integration/mvp-baseline→develop | 최종 기능 완성 대신 공통 Contract·E2E 기반을 먼저 확보 |
| 역할·계약 정리 / 9/10 | 담당별 병렬 고도화 | PR #84 문서 체계, #99 AI baseline, Notion ownership/마감 gate | Core/Copilot 경계와 Handoff 구체화 |
| Front↔Back↔DB↔AI 통합 / 9/11~9/12 | 공통 Runtime·P0 vertical slice | #101 migration, #105 Backend, #106 AI Core, #107 Proposal, #117 Copilot | Proposal·Evaluation의 구현 범위를 구분; route 존재로 완성 판정 금지 |
| Hardening / 9/12~9/14 | 안전성·표시·수집 안정화 | #112/#114 dropped requirements, #119 실패 분석 문구, #129 이력 backfill, #132 변경 표시, #134 계정/Case 격리, #137 같은 차수 이중 실행, #135 Rule v0.3 | 실데이터·통합에서 드러난 오류를 공유 경계에서 수정 |
| Golden / Evaluation / 9/12~9/14 | 실공고·변경공고 평가 잠금 | Rule 초기→9/13 개선, 9/14 CI 유지; E3 비교 문서, G1 snapshot/G2 후보 | 독립 라벨·G2 Human·Task·최종 F1은 미완료 |
| Finalization / 9/14 현재 | 제출 산출물 구성·검산 | 공식 docs Phase 1-1 감사, 1-2~1-8 문서 완성·감사 | UI 이미지·Evidence Lock·코드 동기화·발표는 다음 Phase |

## 3. Baseline 전략이 바꾼 일

공통 DB/API·Requirement/Evidence 계약이 없으면 Frontend는 mock에, AI는 개별 입력에, Copilot은 별도 결과에 의존하게 됩니다. PR #75는 이를 연결하는 기준선을 만들고 이후 각 담당자가 같은 흐름 위에서 개선하도록 했습니다.

Baseline 병합은 개별 기능의 Human 검증 완료가 아닙니다. 후속 PR에서 계약·schema·stale guard를 보완했고, 실공고와 자동 fixture의 차이를 확인하며 평가를 분리했습니다. 특히 Rule 회귀가 좋아도 Extraction 도달률과 G2 Ground Truth는 별도 문제가 남았습니다.

## 4. 내부 Gate와 현재 상태

[Notion Master Plan](https://www.notion.so/3d7e94b44d078133878dc29571061798)의 일정은 다음과 같습니다. 지난 마감 시각만으로 Gate 통과를 표시하지 않습니다.

| Gate | 계획 시점 | 현재 문서에서 확인 가능한 상태 |
| --- | --- | --- |
| Scope/Contract Freeze | 9/10 18:00 | 기준·계약 문서 존재; 이후 실제 수정 이력 있음 |
| Shared Runtime | 9/11 18:00 | 공통 실행·DB 운영 기록; 이번 live 환경 Smoke 미실행 |
| Part P0 / Cross-part Integration | 9/12~9/13 | 병합·자동 CI 근거; Human 완료와 구분 |
| Golden E2E / Eval Lock | 9/14 18:00 | Rule/E3 근거 확보, 전체 Human·G2·독립 라벨은 Pending |
| Product Freeze | 9/15 18:00 | 계획. P0 blocker·최종 설정·실행법·데모 고정 확인 필요 |
| Docs / Presentation / Rehearsal | 9/16 | 계획. 신규 기능보다 문서·리허설 우선 |
| 발표 | 9/17 | 계획 |

## 5. 담당·의존성

DB/Data·Backend의 원문/Version/API가 AI 평가의 입력을 만듭니다. AI의 Canonical Requirement/Evidence가 Judgment·Evidence 화면·변경 재검증·Copilot 설명의 기반입니다. Frontend/Copilot 통합 후 실제 사용자가 같은 Case를 완료하는지 검증해야 합니다.

주 담당과 Handoff는 [협업 문서](../07_handoff/collaboration.md), 현재 한계의 다음 작업은 [Future Roadmap](future-roadmap.md)에 있습니다. 실제 Task 상태는 GitHub Issue를 우선하며 WBS를 실시간 상태 원장으로 중복 운영하지 않습니다.

## 6. 근거

- [초기 가이드 / Historical](https://www.notion.so/3c9e94b44d0781bf95daec60218b0788)
- [PR #75](https://github.com/gyuniverse-hq/bid-change-validator/pull/75), [PR #84](https://github.com/gyuniverse-hq/bid-change-validator/pull/84), [PR #117](https://github.com/gyuniverse-hq/bid-change-validator/pull/117)
- [PR #128](https://github.com/gyuniverse-hq/bid-change-validator/pull/128), [PR #129](https://github.com/gyuniverse-hq/bid-change-validator/pull/129), [PR #135](https://github.com/gyuniverse-hq/bid-change-validator/pull/135), [PR #137](https://github.com/gyuniverse-hq/bid-change-validator/pull/137)
- [고정 개발 history](https://github.com/gyuniverse-hq/bid-change-validator/commits/36f1afba8e1a025006b0bfa1876502e6232b3d28/), [최종 산출물 계획](https://www.notion.so/3c9e94b44d078171bb8dc9b3c1bedd35)
