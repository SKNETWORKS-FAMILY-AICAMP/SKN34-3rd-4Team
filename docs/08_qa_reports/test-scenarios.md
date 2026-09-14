# 테스트 시나리오

> Status: Current Test Specification / Human Click E2E Validation Pending
> Implementation Baseline: `gyuniverse-hq/bid-change-validator` `develop@36f1afba8e1a025006b0bfa1876502e6232b3d28`

## 1. 목적과 기록 규칙

시나리오는 사용자가 수행할 작업과 기대 결과를 정의합니다. **기대 결과는 실제 성공 기록이 아닙니다.** 자동 테스트·합성 Golden·실공고 사람 검수·배포 Smoke를 분리합니다. 실행 근거는 [테스트 계획 및 결과](test-plan-and-results.md), 요구사항 연결은 [Traceability](../02_architecture/feature-traceability.md)에 있습니다.

공통 실행 기록에는 실행자·시각·code SHA·환경, company/case/notice/version/analysis/judgment ID, Rule·기준일, 실제 결과·실패·화면 증거를 남깁니다. 합성 데이터는 실공고 검수로 표시하지 않습니다.

## 2. Scenario A — 기본 참가자격 검토

| 항목 | 정의 |
| --- | --- |
| Req | CUR-NOT-01/02, CUR-COM-01/02, CUR-ANL-01/02, CUR-JDG-01/02, CUR-EVD-01 |
| Preconditions | 로그인·회사 Profile·수집된 공고/Version·원문 저장·EXTRACTED blocks. 실제 분석 시 OpenAI 설정 필요 |
| Input | 선택한 공고·현재 Version, 회사 Profile/completeness, 판정 기준일 |
| Steps | ① /notices 검색 ② Case 선택/생성 ③ /qualification Analysis ④ Judgment ⑤ 요건별 상태와 provenance 확인 ⑥ /evidence에서 해당 문서·위치·인용 확인 |
| Expected | 같은 Version/Analysis/Company/Rule의 결과만 표시. SATISFIED/UNSATISFIED/UNKNOWN 구분. Evidence가 선택 요건 원문으로 연결 |
| Negative | 원문 없음→분석 실패 안내; PARTIAL→전체 eligible 금지; stale 판정→최신 결과 재조회/재판정; 미분석 공고→확정 판정 표시 금지 |
| Actual validation level | API·분석·판정 회귀 파일 및 PR CI 확인. 실제 원문→모델→화면→locator 전체 흐름의 Human 성공 기록은 미확인 |
| Automated / Human | test_notices.py, test_qualification_analysis_service.py, test_analysis_pipeline.py, test_ai_integration.py, test_product_baseline_regression.py / Human Pending |
| Current status | 구현 Current, 실데이터 전체 Human E2E Validation Pending |

## 3. Scenario B — UNKNOWN → Ask-back → Targeted Re-judgment

| 항목 | 정의 |
| --- | --- |
| Req | CUR-ASK-01/02, CUR-NFR-04/05 |
| Preconditions | 최신 호환 Judgment의 UNKNOWN 중 askable=true인 단일 사실 존재 |
| Input | 합성 G0에서는 정보통신공사업 등록 여부. certifications completeness=false, 사용자가 보유 사실과 증빙 보유 여부 답변 |
| Steps | ① /ask-back 질문 확인 ② 답변 입력 ③ 제출/확인 ④ 새 Judgment Run 조회 ⑤ 대상 요건 및 나머지 요건·Profile 비교 |
| Expected | 답변 대상만 재판정, USER_ANSWER provenance. 회사 Profile은 자동 변경 안 됨. G0의 보유 답변은 해당 요건 SATISFIED |
| Negative | 비질문 대상 UNKNOWN에는 강제 질문 없음. apply_to_profile=true 거부. source가 바뀌면 stale 답변 거부·최신 상태 안내 |
| Actual validation level | 합성 API/DB Golden에서 답변→새 Run→provenance 회귀 확인. 실공고에서 안전한 질문인지·사용자가 오해 없이 답했는지는 별도 |
| Automated / Human | test_askability.py, test_mvp_golden_e2e.py, test_copilot_action_transactions.py / Human Pending |
| Current status | 구현·합성 자동 회귀 근거 있음, 실공고 safe-answer Human E2E Pending |

## 4. Scenario C — Changed Notice → affected-only Revalidation

| 항목 | 정의 |
| --- | --- |
| Req | CUR-CHG-01/02, CUR-NFR-03/05 |
| Preconditions | baseline v1과 current v2가 다름. 두 Analysis와 최신 baseline Judgment, 동일 Profile·Rule·기준일 |
| Input | G0: 회사 실적 5억원, v1 최소 4억원→v2 최소 6억원. 나머지 요건은 동일 |
| Steps | ① v1 판정·Ask-back 결과 확보 ② v2 선택/분석 ③ /changes에서 Requirement Diff ④ 재검증 ⑤ revalidated_keys·before/after 결과·원문 비교 |
| Expected | 실적 요건만 MODIFIED 및 재판정→UNSATISFIED, 전체 ineligible. 비변경 요건은 이전 판정·USER_ANSWER provenance 승계. 현재 Evidence key 연결 |
| Negative | Profile/Rule/기준일 변경→전체 재판정 요구. baseline=current 신규 Case 거부. REMOVED는 현재 판정 제외. 이전 판정 없는 현재 요건은 재판정 |
| Actual validation level | G0의 API/DB 합성 회귀 확인. 테스트는 Canonical Requirement를 직접 저장하므로 Parsing/LLM/Evidence Viewer 검증이 아님 |
| Automated / Human | test_requirement_diff.py, test_mvp_golden_e2e.py, test_product_baseline_regression.py / G2 Human Pending |
| Current status | 코드·G0 자동 회귀 근거 있음. G2 R26BK01686455는 candidate_removed / DRAFT / not_ground_truth |

G2 사람 검수에서는 v1의 지역 문구가 v2 다른 위치·표현으로 남아 있는지 먼저 확인한 뒤 독립 before/after 라벨을 작성해야 합니다. 현재 원문 문구 부재 관찰을 법적·의미적 삭제 확정으로 쓰지 않습니다.

## 5. Scenario D — 현재 Case 중심 AI Copilot

| Task | Preconditions / Input | Steps / Expected | Actual / Status |
| --- | --- | --- | --- |
| 판정·사유 조회 | 최신 호환 Case·Run, “왜 미달이야?” | 저장 결과 재조회→요건·사유·근거 제공. 새 자격판정 생성 안 함 | Product Tool/API 회귀; 사용자 과업 평가 Pending |
| Evidence 참조 | 현재 응답의 유효한 요건 참조, “두 번째 조건 근거” | 제한된 참조 해결→Backend Evidence 조회. 모호하면 확인 요청 | 참조·case 격리 회귀; 실제 원문 이동 Human Pending |
| 확인 필요·Profile | 현재 Case, 질문 가능 요건·회사 정보 질문 | 필요한 단일 사실과 정보 부족/직접 검토를 구분 | askability·tool 회귀 |
| 변경 조회·재검증 | baseline/current 및 source Run | 변경 조회→재검증 Proposal→사용자 확인→최신성 검사→실행 | 합성 Action 회귀; G2 Human Pending |
| Document QA | 현재 Version 공개 문서, 별도 외부처리 동의 | Hybrid Retrieval→SOURCE 답변. 0-hit/유효 citation 없음→보류; 타 Version 거부 | E3 DRAFT 평가 보고서; 의미 정답·사용자 Task Pending |

자동 파일: test_copilot_flow.py, test_copilot_product_tools.py, test_copilot_grounded_document_qa.py, test_copilot_action_transactions.py, test_copilot_semantic_optin.py. 브라우저 mock 검증은 실제 모델·실공고 Human 평가가 아닙니다.

## 6. 재현·사람 검수 완료 조건

G0 입력은 [mvp-e2e-v0.1](https://github.com/gyuniverse-hq/bid-change-validator/tree/36f1afba8e1a025006b0bfa1876502e6232b3d28/samples/golden/mvp-e2e-v0.1), 실제 자동 흐름은 [test_mvp_golden_e2e.py](https://github.com/gyuniverse-hq/bid-change-validator/blob/36f1afba8e1a025006b0bfa1876502e6232b3d28/apps/api/tests/test_mvp_golden_e2e.py)를 기준으로 합니다. 파일 이름의 E2E를 브라우저 Human E2E로 해석하지 않습니다.

사람은 A→B→C와 D의 조회·확인형 Action을 실제 클릭하고, 각 단계의 기대/실제 결과와 실패 증거를 기록합니다. 모든 단계에 같은 Case·Version·Run lineage가 유지돼야 합니다. 캡처 계획은 [screenshots](../05_ui_ux/screenshots.md)에 있습니다. 현재 문서에는 사람 실행자·완료 시각을 임의 생성하지 않았습니다.
