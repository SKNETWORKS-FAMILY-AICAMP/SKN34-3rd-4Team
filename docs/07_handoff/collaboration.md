# 역할 분담과 협업 방식

> Status: Current Evidence Summary
> Implementation Baseline: `gyuniverse-hq/bid-change-validator` `develop@36f1afba8e1a025006b0bfa1876502e6232b3d28`

## 1. 실제 개발 흐름

```text
GitHub Issue → Branch → Commit → Pull Request
→ Review / Test → Merge → Integration 확인
```

기능 코드는 개발 저장소 develop 기준으로 통합했고, 공식 제출 문서는 별도 저장소 docs 브랜치에서 정리합니다. Issue를 연결하는 것, PR을 병합하는 것, 독립 검증을 끝내는 것은 다른 상태입니다.

## 2. 역할과 Handoff

| 팀원 | 중심 역할 | 다른 파트에 넘기는 결과 |
| --- | --- | --- |
| 황수빈 | Frontend / UI·UX | Figma 기준 화면·입력·loading/error/empty·Case 상태·원문 이동 |
| 전진환 | Backend / API·Deployment | Product API, 입력/권한/오류 경계, Run 저장, 실행 구성 |
| 정예린 | DB / Data | Schema·migration·공고 수집·Version/원본 정합성 |
| 김재현 | LLM/RAG Core / Evaluation | 구조화 Requirement/Evidence·Grounding·품질평가·위험조항 Core |
| 이홍규 | AI Copilot / Integration·협업 운영 | Product tool·확인형 Action, Core/Backend/Frontend 계약, 문서·통합 기준 |

이는 실제 ownership 문서의 주 담당 기준이며 개인별 모든 commit을 독점 작성했다는 뜻이 아닙니다. 김재현의 AI Core와 이홍규의 Copilot은 Requirement/Evidence/Judgment/Askability/Change contract에서 만납니다. Copilot이 Core 판정을 다시 생성하지 않습니다.

## 3. 실제 대표 사례

| 사례 | Issue / Branch / Commit / PR | Review·Test·Merge·통합 근거 |
| --- | --- | --- |
| 통합 기준선 | integration/mvp-baseline → PR #75 → f5fce26 | 2026-09-08 develop 병합. 개별 기능 Done과 전체 통합 기준선 확보를 분리 |
| 추출 품질 | Issue #29 → feat/extraction-recall-harness → PR #128 → 31ec928 | 2026-09-13 병합. 요건 선별·안전 보류·회귀 개선. Issue #29는 확인 시 OPEN이며 품질 전체 완료로 자동 종료되지 않음 |
| stale Rule 차단 | fix/qualification-rule-version-v03 → PR #135 → 36f1afb | Backend뿐 아니라 Frontend 타입/선택·Copilot read 경로까지 수정. CI 확인 후 2026-09-14 병합 |

PR #135에는 테스트·검토 과정과 타입 오류 보완 기록이 있으나 조회한 formal review 제출 목록은 비어 있었습니다. 모든 PR이 필수 reviewer 승인을 거쳤다고 쓰지 않습니다. 과거 ownership 문서의 required approval=0 기록을 현재 GitHub ruleset 실조회 결과로 간주하지도 않습니다.

## 4. 통합 방식

처음부터 파트별 최종 구현을 독립 완성하는 대신 **MVP Integration Baseline → 공통 기준선 확보 → 담당별 병렬 고도화**를 택했습니다. 공통 DB/API와 Requirement/Evidence 계약을 먼저 연결해 Frontend·AI 평가·Copilot이 같은 상태를 보게 했습니다.

변경 PR에서는 API·DB·AI·화면 중 영향을 받는 부분, migration/env 변화, 실행 방법, 테스트, 남은 한계를 공유합니다. 최신 Rule 변경이 화면·Copilot의 stale source 선택까지 영향을 준 PR #135는 이 필요성을 보여줍니다.

## 5. 도구별 실제 활용 수준

| 도구 | 실제 역할 / 표현 경계 |
| --- | --- |
| GitHub Issue / Branch / PR / Actions | 작업 연결, 코드 통합·검증 이력의 중심 |
| GitHub Projects | 후반 일부 Issue·상태 연결의 보조 수단. 전체 기간의 완전한 일정 원장으로 주장하지 않음 |
| Discord | 작업 현황·blocker·파트 간 요청 공유. 이번 작업에서 전체 채팅을 감사한 것은 아님 |
| Figma | UI/UX source와 화면 기준 공유. 실제 화면과 완전 일치·Human E2E 완료는 별도 |
| Notion | 기획·설계·Evaluation·Decision 배경. Current overlay와 Historical 본문 구분 |
| Jira | 초기 학습·도입 계획 기록. 실제 프로젝트 운영 도구로 사용했다고 주장하지 않음 |

## 6. 근거

- [Current ownership](https://github.com/gyuniverse-hq/bid-change-validator/blob/36f1afba8e1a025006b0bfa1876502e6232b3d28/docs/07_handoff/current-ownership.md), [Notion 역할 분담](https://www.notion.so/3c9e94b44d07810faeecdaad8087b62a)
- [Issue #29](https://github.com/gyuniverse-hq/bid-change-validator/issues/29), [PR #75](https://github.com/gyuniverse-hq/bid-change-validator/pull/75), [PR #128](https://github.com/gyuniverse-hq/bid-change-validator/pull/128), [PR #135](https://github.com/gyuniverse-hq/bid-change-validator/pull/135)
- [Notion 최종 산출물 Control Center](https://www.notion.so/3c9e94b44d078171bb8dc9b3c1bedd35), [검증 로그](../08_qa_reports/test-plan-and-results.md), [실제 WBS](../09_roadmap/wbs.md)
