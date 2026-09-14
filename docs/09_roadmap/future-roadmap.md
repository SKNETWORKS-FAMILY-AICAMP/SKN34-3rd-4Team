# 향후 개선 계획

> Status: Current Gap Register / 실행 계획은 Future
> Implementation Baseline: `gyuniverse-hq/bid-change-validator` `develop@36f1afba8e1a025006b0bfa1876502e6232b3d28`

## 1. 우선순위

변경공고 대응의 근거를 강화하는 순서로 개선합니다. 구현·검증 상태를 혼합하지 않으며, 아래 작업을 이번 Phase에서 구현하거나 완료했다고 주장하지 않습니다.

## 2. Current Limitation → 영향 → 다음 단계

| 우선 | 현재 한계 | 왜 중요한가 | 구체적인 다음 단계 / 완료 증거 | 주 담당 영역 |
| --- | --- | --- | --- | --- |
| P0 검증 | G2 candidate_removed, DRAFT/not_ground_truth | 핵심 차별점의 실공고 증명이 미완성 | 전체 차수·첨부 확보→독립 before/after 요건·Evidence·Judgment 라벨→실제 재검증 Run과 Human E2E | Data + AI + Integration |
| P0 검증 | Extraction 독립 F1 미확정 | Rule이 좋아도 요건이 누락되면 판정 입력 자체가 부족 | 독립 source/Canonical 라벨·holdout 고정→반복 모델 실행→precision/recall/F1·drop/미분류 사유 기록 | AI Core |
| P0 검증 | 전체 Human Click E2E 미확인 | API/자동 테스트가 실제 사용자 완료를 대신하지 못함 | [시나리오 A~D](../08_qa_reports/test-scenarios.md) 실제 클릭, 실행자·환경·ID·캡처·실패 기록 | Frontend + Integration |
| P0 제출 | 공식 repo 코드·데이터 미동기화 | 문서만으로 제품 실행·원본 재파싱 불가 | 개발 code/migration/fixtures/실행법의 배포 범위 확인 후 공식 repo 동기화, 새 환경 실행 검증 | Backend + Data |
| P0 배포 | 최종 URL·운영 인증·storage 미확정 | 개발 기본 설정을 운영 완료로 오인할 수 있음 | 격리 배포·AUTH_REQUIRED 등 설정·migration/health·로그인/소유권·원본 접근·backup 복원 Smoke | Backend + Data |
| P1 | Rule fixture DRAFT_NOT_APPROVED | 기대값이 독립 정답이 아님 | 독립 검수 후 approved dataset 새 버전, 기존 초안 history 보존 | AI + 검수자 |
| P1 | 합성 회사 Profile | 실제 기업 데이터 품질·증빙과 차이 | 실제 기업 정보 연동 범위·동의·갱신 주기와 증빙 검증 계약 설계, 익명화 pilot 평가 | Backend + Data |
| P1 | Copilot 사용자 Task 미평가 | routing·citation 검사만으로 사용자 오해를 알 수 없음 | Stage 11 과업 평가, 완료율·시간·안전한 보류·오해 기록 | Copilot + Frontend |
| P1 | strict Citation/Recall은 의미 정답과 다름 | 같은 의미 다른 표현·오래된 target이 점수를 왜곡 | source-target 재검수·새 fixture 버전, 답변 문장별 의미 지지 평가 | AI / Copilot |
| P1 | Parsing OCR 미지원·원본 누락 가능 | 일부 문서가 EMPTY/FAILED 또는 재현 불가 | 원본 inventory·hash 대조·누락 복구, 문서 유형별 품질 측정 후 OCR 도입 판단 | Data + Backend |
| P1 | RAG index 갱신·영속성 미확정 | 같은 Version 원문 변경·재배포 후 index 재사용 문제 | hash/model 기반 invalidation 규칙과 저장소 확정, 재배포·문서 변경 회귀 | Backend + Copilot |
| P1 | 같은 차수 화면 중복 방지만 확인 | 병렬 Analysis POST는 별도 중복 실행 가능 | 실제 중복 요청 로그·부하 확인 후 API 멱등 키/잠금 범위 결정, 동시 요청 검사 | Backend |
| P1 | Frontend lint 오류 20건 | build 성공과 정적 품질 검증이 분리됨 | 코드 담당이 오류 수정→lint 재실행→continue-on-error gate 정책 재검토 | Frontend |
| P1 | One-shot/Few-shot 비교 근거 미확인 | 공식 가이드의 prompt 실험 대응이 부족 | 현 prompt 고정→동일 라벨셋에서 예시 유무 비교→수치·token/latency·채택 이유 기록 | AI |
| P2 | polling 이후 알림·자동 재검증 미완성 | 사용자가 변경을 다시 확인해야 함 | 변경 event와 사용자 동의·최신성 정책 정의 후 알림→확인형 재검증 확장 | Backend + Frontend |
| P2 | /evaluation 전용 추출·점수 예측 미지원 | Route를 완성 제품으로 오인할 수 있음 | 현재 제안서 위치 후보·직접 확인 UX 검증 후 전용 criteria contract 범위 결정. 점수 예측은 현 MVP 제외 유지 | Frontend + AI |
| P2 | 계약 위험조항 Core/API와 전용 화면 간 gap | 저장·분류가 전체 사용자 기능 완성을 뜻하지 않음 | 독립 라벨·사용자 화면·Evidence와 E2E 범위를 먼저 합의 | AI + Frontend |

## 3. 재검토 조건

Rerank는 Hybrid 대비 의미 품질 개선이 사용자 과업에서 확인되고 추가 latency·비용을 감당할 때 선택 적용을 검토합니다. 모든 질의에 자동 적용하는 계획으로 확정하지 않습니다.

Profile 자동 승격, cross-version Document QA, 범용 Agent·장기 기억은 현 MVP 밖입니다. 별도 사용자 필요·안전 계약·검증 근거가 생기면 범위를 결정합니다.

## 4. 후속 Finalization

Phase 2는 [실제 UI 캡처](../05_ui_ux/screenshots.md)와 최종 Architecture 이미지입니다. 이후 Evidence Lock, 공식 코드 동기화, 루트 README 연결, 발표자료·Demo 순서로 진행합니다. G2/Human/배포 미검증을 문서 완료와 별개로 계속 표시합니다.

근거: [Requirements](../01_product/requirements.md), [Data](../04_contracts/data-preprocessing.md), [Test 결과](../08_qa_reports/test-plan-and-results.md), [Evaluation](../08_qa_reports/llm-rag-evaluation.md), [WBS](wbs.md).
