# Feature / Implementation / Test Traceability

> Status: Current Submission Traceability
> Implementation Baseline: `gyuniverse-hq/bid-change-validator` `develop@36f1afba8e1a025006b0bfa1876502e6232b3d28`

## 1. 문서 목적

[요구사항 명세](../01_product/requirements.md)의 Current Requirement를 실제 화면, API, 저장 상태, AI/Rule, 자동 테스트와 연결합니다. 자동 테스트, Golden Regression, Human Click E2E, Deployment Smoke는 서로 다른 검증 단계로 기록합니다.

## 2. Source of Truth

1. 개발 저장소 `develop`의 실제 Code / Test
2. `docs/02_architecture/feature-traceability.md`
3. `docs/08_qa_reports/requirement-test-traceability.md`
4. `samples/golden/**`, CI Workflow와 실제 검증 결과

## 3. 검증 단계

```text
Requirement / Product Invariant
→ Unit / Domain Test
→ Service / API Test
→ Golden Regression
→ Human Click E2E
→ Deployment Smoke
```

`Current`는 구현 근거가 확인됐다는 뜻입니다. 자동 테스트가 존재해도 실제 G1/G2 Ground Truth, 브라우저 사용자 흐름 또는 운영 배포가 완료됐다는 뜻은 아닙니다.

## 4. 기능 Traceability

| Req ID | 요구사항 | 상태 | Screen / Route | API / Service | DB / State | AI / Rule | Automated Test | Golden / Human E2E | Remaining Gap |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| CUR-NOT-01 | 공고 검색·선택 | Current | `/notices` | `GET /api/v1/notices`, `routers/notices.py` | `bid_notices`, `bid_notice_versions` | 없음 | `test_notices.py` | Human Click 대기 | 검색·빈 상태·오류의 최종 Human E2E |
| CUR-NOT-02 | 공고 Version·첨부문서 보존 | Current | `/notices` | Version·Document text/source/preview API, Notice Poller | `bid_notice_versions`, `notice_documents`, `notice_collection_runs` | 문서 Parsing은 Backend Service | `test_notices.py`, `test_notice_polling.py`, `test_document_extraction.py` | G1 Dataset 존재, Human Click 대기 | 문서 유형별 실데이터 품질·운영 수집 검증 |
| CUR-MAT-01 | 회사 기준 공고 매칭 | Current | `/notices` | `GET /api/v1/companies/{company_id}/notice-matches`, `qualification/matching.py` | Company + 최신 호환 Analysis/Judgment | cached deterministic matching | `test_qualification_judgment.py`, Product Regression | Golden 회귀 일부 | 추천 근거 표시 Human E2E |
| CUR-COM-01 | 회사 Profile·실적·인증 관리 | Current | `/company` | `/api/v1/companies/**`, `routers/companies.py` | `companies`, Company Industry / Performance / Certification | Judgment 입력 | `test_companies.py` | Human Click 대기 | 전체 CRUD·검증 메시지 E2E |
| CUR-COM-02 | 현재 Profile과 판정 Snapshot 분리 | Current | `/company`, `/qualification` | Judgment Service, Profile Completeness API | `qualification_judgment_runs.profile_snapshot` | deterministic Judgment | `test_qualification_judgment.py`, `case-workspace.test.cjs` | G0 회귀 | Profile 변경 후 전체 재판정 사용자 흐름 |
| CUR-ANL-01 | Requirement·Evidence 구조화 | Current | `/qualification` | `POST .../qualification-analysis`, `qualification/analysis.py` | `qualification_analysis_runs`, `qualification_requirements`, `qualification_evidence` | Extraction → Grounding → Canonicalization | `test_analysis_pipeline.py`, `test_requirement_extraction.py`, `test_ai_integration.py` | G1 Snapshot 존재 | 실공고 승인 라벨과 field-level 품질 확정 |
| CUR-ANL-02 | 분석 상태·같은 차수의 이중 실행 방지 | Current | `/qualification` | Analysis API + `case-workspace.ts`, Case 생성 validation | Analysis 3상태, baseline/current Version | Grounding Diagnostic | `test_qualification_analysis_service.py`, `test_product_baseline_regression.py`, `test_seed_golden_v02_accounts.py` | PR #137 자동 회귀 | 화면의 baseline=current 처리이며 Analysis API 전반의 멱등성·동시 요청 잠금은 미보장 |
| CUR-JDG-01 | 결정론적 참가자격 판정 | Current | `/qualification` | `POST /api/v1/preflight-cases/{case_id}/qualification-judgments` | `qualification_judgment_runs`, `qualification_judgments` | `qualification/rules/judgment.py` | `test_qualification_judgment.py` | G0 Golden 회귀 | Fixture 독립 승인과 실공고 Coverage 확대 |
| CUR-JDG-02 | Judgment·Overall·Basis 상태 분리 | Current | `/qualification` | Judgment Run list/detail | 3상태, Overall 3상태, `basis_type`, `reason_code` | Overall derivation Rule | `test_qualification_judgment.py`, `test_product_baseline_regression.py` | G0 회귀 | 전체 UI 상태 표현 Human 검산 |
| CUR-ASK-01 | 안전한 Askability 분류 | Current | `/ask-back` | `GET .../qualification-questions`, `qualification/ask_back.py` | Judgment `UNKNOWN`, Profile Completeness | `rules/askability.py`, `rules/clause_safety.py` | `test_askability.py`, Judgment Regression | G0 회귀 | 실제 공고 질문 가능성 Human 검수 |
| CUR-ASK-02 | 답변 대상만 재판정 | Current | `/ask-back` | `POST .../qualification-answers` | `qualification_answers`, 새 Judgment Run, `USER_ANSWER` | targeted re-judgment | `test_mvp_golden_e2e.py`, Copilot Flow Test | 합성 Golden 경로 | 브라우저 E2E와 실제 사용자 답변 검증 |
| CUR-EVD-01 | 판정 원문 근거 추적 | Current | `/evidence`, `/qualification` | Analysis detail + Document text/source/preview API | `requirement_key`, `evidence_key`, `document_id`, `location` | Evidence Grounding / Locator | `test_analysis_pipeline.py`, `test_ai_integration.py`, Copilot Evidence Test | G1 Snapshot 존재 | Evidence 의미 정확도·원문 이동 Human 검증 |
| CUR-CHG-01 | Requirement 의미 Diff | Current | `/changes` | Notice Version 조회, `rules/requirement_diff.py` | Baseline / Current Analysis Run | `UNCHANGED / MODIFIED / ADDED / REMOVED` | `test_requirement_diff.py` | 합성 G0, G2 후보 존재 | G2 `DRAFT / not_ground_truth` Human Validation |
| CUR-CHG-02 | affected-only Revalidation | Current | `/changes` | `POST .../qualification-revalidation`, `qualification/revalidation.py` | `qualification_revalidation_runs`, Source / Result Judgment Run | 변경 요건 재판정, 비변경 판정 승계 | `test_mvp_golden_e2e.py`, Copilot Changed Notice Test | 합성 Golden 통과 | 실제 G2 before/after Judgment Ground Truth |
| CUR-COP-01 | 현재 Case 결과 조회·설명 | Current | 공통 Copilot UI | `POST /api/v1/copilot/chat`, `copilot/product_tools.py` | 현재 Case / Analysis / Judgment / Evidence 조회 | Intent, Product Tool, Grounded Presentation; 별도 판정 없음 | `test_copilot_flow.py`, `test_copilot_product_tools.py`, Document QA Test | Scenario Contract 평가 | 사용자 Task·응답 품질 평가 |
| CUR-COP-02 | Action 제안·명시적 확인 | Current | 공통 Copilot UI | `POST /api/v1/copilot/actions/confirm`, `copilot/actions.py` | Qualification Answer 또는 Revalidation Run | Freshness Guard, explicit confirmation | `test_copilot_action_transactions.py`, `test_copilot_flow.py` | 합성 Action 시나리오 | 실제 사용자 확인 흐름 Human E2E |
| CUR-EVL-01 | 공고 요구 항목과 제안서 위치 후보 대조 | Partial | `/evaluation` | Preflight Case Document upload/text/source API; 전용 Evaluation API 없음 | `proposal_documents`, 사용자 확인은 화면 로컬 상태 | keyword 기반 위치 후보; 점수 판정 없음 | 전용 화면 Test 미확인 | Human Click 대기 | 전용 Product Contract·자동 Test·지속 저장 범위 미확정 |
| CUR-RSK-01 | 계약 위험조항 9종 분류 결과 저장 | Partial | 전용 Product 화면 확인 필요 | Contract Clause Review create/list/detail API | Contract Clause Review Run / Finding | `ai/clause_review/**`, Canonical 9종 | `test_contract_clause_reviews.py`, `test_clause_risk_categories.py` | Backend 자동 검증 | 전용 화면과 전체 사용자 E2E |

## 5. 비기능 Traceability

| Req ID | Product Invariant | 구현 근거 | Automated Test | 상위 검증 | Remaining Gap |
| --- | --- | --- | --- | --- | --- |
| CUR-NFR-01 | LLM이 최종 판정을 직접 만들지 않음 | `qualification/rules/judgment.py`, Judgment Run의 `rule_version` | `test_qualification_judgment.py` | G0 Rule Regression | 승인된 실제 Requirement / Profile Coverage |
| CUR-NFR-02 | 판정에서 원문까지 추적 가능 | `ai/contracts.py`, `analysis_models.py`, Document Source API | Analysis / Evidence / Copilot Product Tool Test | G1 Snapshot | 의미 단위 Citation Human 검수 |
| CUR-NFR-03 | Version / Run Lineage 보존 | Notice·Analysis·Judgment·Revalidation Model | `test_notices.py`, `test_mvp_golden_e2e.py` | 합성 Golden | 실제 변경공고 전체 Lineage 검증 |
| CUR-NFR-04 | 불명확 조건은 안전하게 보류 | Askability / Clause Safety / Analysis Diagnostic | `test_askability.py`, `test_clause_safety_decorations.py` | G0 Regression | 실공고 abstention 독립 검수 |
| CUR-NFR-05 | Stale Context 쓰기 차단 | Ask-back, Revalidation, Copilot Confirm Freshness Guard | Copilot Flow / Action Transaction Test, Workspace Test | 합성 시나리오 | 동시 탭·운영 환경 E2E |
| CUR-NFR-06 | 사용자·회사·Case 접근 경계 | Protected API Router, Authorization Guard | `test_auth.py`, `test_copilot_flow.py` | 자동 검증 | 배포 환경 인증·권한 Smoke Test |

## 6. 핵심 Lineage

```text
BidNotice
└─ BidNoticeVersion
   ├─ NoticeDocument
   └─ QualificationAnalysisRun
      ├─ QualificationRequirementRecord
      └─ QualificationEvidenceRecord

Company + PreflightCase + AnalysisRun
└─ QualificationJudgmentRun (profile_snapshot, rule_version)
   ├─ QualificationJudgmentRecord
   ├─ QualificationAnswer → result JudgmentRun
   └─ QualificationRevalidationRun → result JudgmentRun
```

02~06 화면은 같은 `preflight_case_id`를 공유합니다. Frontend `apps/web/lib/case-workspace.ts`는 현재 Version과 호환되는 Analysis / Judgment를 선택하며, 기준 판정을 현재 결과로 대신 사용하지 않습니다.

## 7. 현재 검증 대기 항목

- 실제 공고 Requirement / Evidence G1 승인 라벨과 품질 확정
- G2 변경공고의 before/after Requirement·Judgment Human Ground Truth
- 공고 찾기 → 판정 → Ask-back → Evidence → 변경공고의 전체 Human Click E2E
- `/evaluation` 전용 Product Contract, 자동 테스트와 상태 persistence 범위
- 계약 위험조항의 전용 화면과 사용자 E2E
- Copilot 사용자 Task 평가
- Production Deployment Smoke와 운영 인증·권한 검증

## 8. 검증 원칙

확인하지 않은 구현이나 수치는 완료로 표현하지 않는다. Test 파일의 존재는 해당 테스트가 현재 CI에서 통과했거나 Human E2E가 완료됐다는 주장으로 확장하지 않는다.

## 9. 근거 탐색과 실행 범위

표의 `routers/`, `qualification/`, `copilot/`, 모델 파일은 [Backend app](https://github.com/gyuniverse-hq/bid-change-validator/tree/36f1afba8e1a025006b0bfa1876502e6232b3d28/apps/api/app) 기준입니다. `test_*.py`는 [Backend tests](https://github.com/gyuniverse-hq/bid-change-validator/tree/36f1afba8e1a025006b0bfa1876502e6232b3d28/apps/api/tests), `.test.cjs`는 [Web tests](https://github.com/gyuniverse-hq/bid-change-validator/tree/36f1afba8e1a025006b0bfa1876502e6232b3d28/apps/web/tests) 기준입니다. Product Regression은 `test_product_baseline_regression.py`, Copilot Flow는 `test_copilot_flow.py`, Document QA는 `test_copilot_grounded_document_qa.py`를 뜻합니다.

최신 SHA에서 독립 확인한 CI 실행은 Golden 안전성 70개와 Rule Fixture gate입니다. 전체 Backend 742개와 Frontend build는 PR #135의 통합 CI 기록으로 출처를 분리합니다. 정확한 Run ID·경고·lint 실패는 [테스트 결과](../08_qa_reports/test-plan-and-results.md)에 기록했습니다. 자동 검증 열은 파일 매핑이며 각 기능의 Human 완료율이 아닙니다.

`CUR-NFR-05`의 현재 Rule은 v0.3입니다. `CUR-NFR-06`은 인증 guard의 구현 근거이며 `AUTH_REQUIRED=false` 개발 기본값에서 운영 보안을 보장한다는 뜻이 아닙니다. 위 Lineage는 처리 관계이며 물리 FK는 [ERD](../04_contracts/db-erd-current.md)에서 구분합니다.
