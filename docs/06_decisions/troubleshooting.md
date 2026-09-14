# Troubleshooting

> Status: Current Evidence Summary / 실제 사례 6개
> Implementation Baseline: `gyuniverse-hq/bid-change-validator` `develop@36f1afba8e1a025006b0bfa1876502e6232b3d28`

## 1. 선정·해석 기준

코드·테스트·PR·평가 보고서에 근거가 있는 사례만 정리했습니다. 아래 Options는 확인된 해결을 이해하기 위한 기술적 비교이며, 모든 대안이 회의에서 논의됐다는 뜻은 아닙니다. PR의 당시 실데이터 보고와 이번 CI 로그 검증은 구분합니다.

## TS-01 — 변경 조회만으로는 전체 차수가 완전하지 않았다

- **Problem:** 우치공원 R26BK01634263 검토에서 기존 확보 범위 밖의 004/005 차수가 발견됐습니다.
- **Cause:** 변경 기간 조회·기존 snapshot만으로 전체 이력 완전성을 보장할 수 없었습니다. 늦게 들어온 과거 차수의 순서·current 처리, 항목 오류도 수집을 방해할 수 있었습니다.
- **Options:** 발견 때마다 수동 보충 / 공고번호 전체 이력을 영속 queue로 재확인.
- **Chosen solution:** 신규·기존 공고 backfill queue, 차수 정렬, notice+bid_notice_order unique, 항목별 savepoint, failed_item_count 및 재시도 기록. migration 022에 반영했습니다.
- **Verification:** [PR #129](https://github.com/gyuniverse-hq/bid-change-validator/pull/129)의 2026-09-13 실데이터 보고는 000~005 저장, 004 첨부 5개 다운로드, 005 취소공고·첨부 0개·유일 current, 처리 6/신규 2/실패 0입니다. 이번 문서 작업의 live DB 결과는 아닙니다. 최신 소스의 수집·백필 테스트와 통합 CI를 확인했습니다.
- **Remaining / prevention:** 모든 공고의 backfill 완료·기존 storage key의 원본 존재를 보증하지 않습니다. 전체 차수·첨부 coverage를 확인한 뒤 G2 후보를 고정해야 합니다.

## TS-02 — 오래된 판정과 부분 분석을 현재 참가 가능으로 보여줄 위험

- **Problem:** Rule 의미가 바뀌어도 v0.2 판정이 재사용되거나, 화면이 최신 Analysis와 다른 Judgment를 선택할 수 있었습니다. PARTIAL 분석의 일부 충족 결과도 전체 eligible로 오해될 수 있습니다.
- **Cause:** Run 생성시각만으로 최신성을 판단하면 Analysis·Version·Company·Rule의 호환성을 놓칩니다. 부분 분석은 누락 요건의 부재를 증명하지 않습니다.
- **Options:** 화면 문구만 변경 / 공유 상태 선택·Rule·API guard에서 호환성을 확인.
- **Chosen solution:** Rule v0.3, Frontend summary의 analysis/notice/company/rule 선택, Copilot 현재 Rule 재조회, 재검증의 Rule·Profile·기준일 guard. 전체 상태 도출은 PARTIAL의 eligible 승격을 막습니다.
- **Verification:** [PR #135](https://github.com/gyuniverse-hq/bid-change-validator/pull/135), test_product_baseline_regression.py, test_copilot_product_tools.py, case-workspace.test.cjs. [CI](../08_qa_reports/test-plan-and-results.md)에서 동일 tree의 Backend 742개와 build 성공을 확인했습니다.
- **Remaining / prevention:** 과거 판정은 보존하되 현재 결과로 재사용하지 않습니다. 변경된 Profile·Rule·기준일은 기준 차수부터 전체 재판정합니다. Human 화면 검산은 별도입니다.

## TS-03 — 같은 차수를 baseline/current로 두어 두 번 실행

- **Problem:** baseline=current인 Case에서 기준 분석과 현재 분석이 중복 실행되고 의미 없는 변경 재검증이 가능해 보였습니다.
- **Cause:** 잘못된 Case 범위와 Golden 시드가 동일 차수를 두 경로에 전달했습니다.
- **Options:** 중복 응답을 화면에서 숨김 / Case 범위·시드·실행 분기를 함께 수정.
- **Chosen solution:** 신규 Case는 baseline이 current보다 이전인지 검사, 기존 동일 차수 Case는 분석·판정 한 번만 실행하고 변경 재검증 비활성화. 시드는 유효한 이전 baseline만 보존합니다.
- **Verification:** [PR #137](https://github.com/gyuniverse-hq/bid-change-validator/pull/137), Case/seed 회귀와 현재 case-workspace.ts를 확인했습니다. J14의 실제 v1→v2 baseline은 보존하도록 설계됐습니다.
- **Remaining / prevention:** Analysis POST 전체의 멱등성·분산 동시 실행 잠금이 구현됐다는 뜻은 아닙니다. PR의 후속 공용 DB 정리를 이번 작업에서 실행·검증하지 않았습니다.

## TS-04 — 원문 요건이 판정기에 도달하지 못했다

- **Problem:** Rule fixture는 잘 동작해도 우치공원 실제 추출에서 기대 요건 3개가 판정기에 0개 도달했습니다.
- **Cause:** “3. 입찰참가자격” 아래 가/나/다를 동급 제목으로 오인한 선별, 법령 문자열 safety 오인, 호환 문장부호에 의한 grounding miss가 있었습니다.
- **Options:** Rule 기대값을 낮춤 / 선별·비교·safety의 원인을 수정하고 추출 도달률을 별도 측정.
- **Chosen solution:** 제목 위계·문서 경계를 보존하고 비교 문자열만 정규화합니다. raw는 보존하며 복합 대안·부정 조건은 계속 안전하게 보류합니다. 법령 인용을 원문 위치로 쓰지 않는 locator 검증도 유지합니다.
- **Verification:** [PR #128](https://github.com/gyuniverse-hq/bid-change-validator/pull/128) 보고에서 해당 공고 0/3→2/3, Rule 104/138→110/138입니다. 두 지표의 분모·의미가 다릅니다. 최신 SHA 안전성 suite에서 관련 extraction·clause safety 테스트를 확인했습니다.
- **Remaining / prevention:** 20공고 live 도달 18/59는 PR 보고 수준이고 독립 F1이 아닙니다. 마지막 복합요건을 단순 업종으로 오확정하지 않습니다. 독립 라벨과 반복 실험이 필요합니다.

## TS-05 — UNKNOWN을 모두 질문으로 바꾸면 위험하다

- **Problem:** 낮은 confidence·정보 부족만 보고 질문하면 복합 법적/절차 조건을 사용자 단일 답변으로 과도하게 확정할 수 있습니다.
- **Cause:** 모델 추출 불확실성, 회사 정보 부재, 사용자 답변 가능성을 같은 상태로 취급한 초기 설계였습니다.
- **Options:** 모든 UNKNOWN 질문 / Askability를 판정과 별도 분류.
- **Chosen solution:** UNKNOWN != ASKABLE. 지원 유형·연산자·단일 사실·clause safety guard를 적용하고 apply_to_profile=true를 거부합니다.
- **Verification:** [Notion Reconciliation](https://www.notion.so/3d7e94b44d07810aa113ca36d6f4cc34), [askability.py](https://github.com/gyuniverse-hq/bid-change-validator/blob/36f1afba8e1a025006b0bfa1876502e6232b3d28/apps/api/app/qualification/rules/askability.py), test_askability.py, unsafe direct-answer API 회귀. 최신 CI 안전성 70개 실행 범위에 포함됩니다.
- **Remaining / prevention:** 실제 질문이 사용자가 오해 없이 답할 수 있는지는 Human 검수 대상입니다. 회사 Profile 승격은 별도 기능·검증으로 남깁니다.

## TS-06 — Rerank Recall 개선보다 latency·호출 비용 증가가 컸다

- **Problem:** Dense retrieval의 strict Evidence Recall@4가 29.17%였고 기대 근거를 놓쳤습니다.
- **Cause:** 코드·고유명사·구체 조건은 Dense만으로 충분하지 않았고, Citation 존재만으로 질문에 직접 답하는 근거를 보장하지 못했습니다.
- **Options:** Dense / Hybrid / Hybrid+LLM rerank.
- **Chosen solution:** Hybrid 기본 경로를 채택하고 LLM rerank는 미채택. Prompt는 근거 없음 보류, 부분 근거의 과잉 일반화 방지, 불완전 식별자 임의 복원 금지로 보완했습니다.
- **Verification:** [E3 평가](../08_qa_reports/llm-rag-evaluation.md)의 동일 DRAFT fixture에서 Hybrid 50.00%, rerank 55.56%. p50은 170.58ms→2849.73ms, 추가 모델 호출 59회입니다. source-reported 실험이며 이번 재실행은 아닙니다.
- **Remaining / prevention:** selective rerank는 향후 후보입니다. Citation Version integrity 100%를 의미 정답률로 표현하지 않으며 사용자 과업·의미 검수는 별도 수행합니다.

## 연결

[수집·전처리](../04_contracts/data-preprocessing.md) · [AI Pipeline](../03_ai/llm-rag-pipeline.md) · [Test 결과](../08_qa_reports/test-plan-and-results.md) · [Future](../09_roadmap/future-roadmap.md)
