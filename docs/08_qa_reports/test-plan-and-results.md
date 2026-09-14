# 테스트 계획 및 결과

> Status: Current Evidence Register / Human·Deployment Validation Pending
> Implementation Baseline: `gyuniverse-hq/bid-change-validator` `develop@36f1afba8e1a025006b0bfa1876502e6232b3d28`

## 1. 목적과 PASS 기준

테스트 파일 존재, 실행 성공, 독립 실공고 검증을 분리합니다. 이번 작업에서는 제품 코드·DB·LLM을 실행하거나 수정하지 않고 **GitHub Actions의 실제 로그와 고정 소스**를 검증했습니다. 문서 자체의 링크·정합성 검사는 [docs Index Audit](../README.md)에 기록합니다.

- PASS: 특정 Run·SHA·실행 범위에서 성공 로그 확인.
- Source-reported: 당시 PR/평가 문서가 보고한 결과. 이번 작업에서 원시 실행 전체를 독립 재현한 것은 아님.
- Implemented test / latest execution not independently verified: 테스트만 확인한 범위.
- Validation Pending: Human/G2/배포 등 완료 기록이 없는 범위.

## 2. 계층별 계획

| 계층 | 대상 / 통과 조건 | 실행·데이터 경계 |
| --- | --- | --- |
| Unit / Domain | 정규화, grounding, Rule, Askability, Requirement Diff의 기대값·안전 보류 | 고정 입력·mock. 실제 LLM 정확도와 구분 |
| Service / API | Case·Run·Version 소속, 입력 오류, Profile snapshot, stale 거부, 답변/재검증 저장 | 격리 PostgreSQL·TestClient |
| Integration | migration→Backend→Frontend 타입/build·Copilot 계약 연결 | CI 임시 환경, mock browser 포함 |
| Golden Regression | fixture hash 일치, wrong determinate=0, match>=110 | Canonical 직접 주입; Extraction 제외 |
| LLM/RAG Evaluation | 추출 도달·Retrieval·Citation·과업별 지표 | DRAFT/독립 라벨 여부와 모델 실행을 별도 기록 |
| Human Click E2E | A~D 사용자 흐름·원문 이동·safe answer·before/after 확인 | 실제 사용자·실공고·실행 ID·캡처 필요 |
| Deployment Smoke | 최종 URL·health·로그인·DB/storage·대표 흐름 | 최종 배포 환경 확정 후 실행 |

## 3. 실제 확인한 CI 결과 — 2026-09-14

| 실행 | 코드 / 시간(UTC) | 확인 결과 | 해석 |
| --- | --- | --- | --- |
| [Golden 34841183959](https://github.com/gyuniverse-hq/bid-change-validator/actions/runs/34841183959) | develop 36f1afb…, 12:02:14~15 | **PASS**: 안전성 70 passed; fixture checksum 확인; 40 cases/138 rows, match 110, safe abstention 28, wrong determinate 0, overall 37; fatal_errors 없음 | 최신 SHA 직접 push 실행. Rule 회귀이며 모델·Human 검증 아님 |
| [MVP CI 34839177518](https://github.com/gyuniverse-hq/bid-change-validator/actions/runs/34839177518) | PR #135, 11:38:40~56 | **PASS**: migration 022까지 upgrade; Backend 742 passed / 1 warning; Frontend production build 성공 | PR merge checkout 실행. 아래 tree 동일성 확인 |
| 같은 MVP CI의 lint step | 같은 PR checkout, 11:38:15 | **FAIL: 20 errors, 0 warnings**. continue-on-error=true | workflow success를 lint 성공으로 바꾸면 안 됨 |
| [Copilot CI 34839177497](https://github.com/gyuniverse-hq/bid-change-validator/actions/runs/34839177497) | PR #135, 11:38~11:40 | workflow success; Backend 742 passed, design 100 passed / 5 deselected, web mock/contract 및 display-state 검사 성공 로그 | 자동 통합 검사. 실제 사람·모델 Task 성공률 아님 |
| [Golden 34841150267](https://github.com/gyuniverse-hq/bid-change-validator/actions/runs/34841150267) | 같은 develop SHA | cancelled | 실패/통과 표본에 더하지 않음. 이후 성공 Run을 기준으로 사용 |

Backend warning 1건은 Starlette/AnyIO BlockingPortal deprecation입니다. Frontend lint 오류 20건은 별도의 미해결 품질 부채입니다. 서로 다른 suite의 테스트 수는 중복될 수 있으므로 합산해 “총 N개 통과”라고 쓰지 않습니다.

### PR CI와 현재 구현의 연결

MVP CI metadata의 head는 9f4afcc972e35b5a5a39a1fa0860b0085c861c81이며 실제 checkout 로그는 PR merge commit **afc67f142a0f4e68ddcebaf71fc3165eb9ed55a6**입니다.

GitHub Git commit API로 확인한 tree SHA:

```text
PR merge afc67f142a0f4e68ddcebaf71fc3165eb9ed55a6
develop  36f1afba8e1a025006b0bfa1876502e6232b3d28
공통 tree 2cc7162e4354b25dffe32d888f63d41c2d67909a
```

두 commit의 파일 tree는 동일합니다. 따라서 현재 소스 내용에 연결되는 자동 테스트 근거이지만, 최신 develop SHA에서 전체 suite를 다시 실행했다고 쓰지 않습니다. [PR merge commit](https://github.com/gyuniverse-hq/bid-change-validator/commit/afc67f142a0f4e68ddcebaf71fc3165eb9ed55a6)과 [구현 commit](https://github.com/gyuniverse-hq/bid-change-validator/commit/36f1afba8e1a025006b0bfa1876502e6232b3d28)을 구분합니다.

## 4. 기능별 자동 범위와 제외 범위

| 범위 | 대표 파일 | 이번 확인 수준 |
| --- | --- | --- |
| 수집·Version·Parsing | test_notices.py, test_notice_polling.py, test_document_extraction.py | 전체 Backend CI에 포함. 실 API/문서 전수 품질 검증 아님 |
| Extraction/Grounding | test_requirement_extraction.py, test_analysis_pipeline.py, test_ai_integration.py | suite/안전성 회귀. mock extractor 결과로 실공고 F1 추정 금지 |
| Rule/Askability | test_qualification_judgment.py, test_askability.py, test_region_hierarchy.py | 최신 SHA 안전성 70개 실행 범위에 포함 |
| G0 답변·재검증 | test_mvp_golden_e2e.py, test_requirement_diff.py | 합성 Canonical 입력과 API/DB lineage. 실제 Parsing·Evidence Viewer 포함 아님 |
| stale Rule·Case | test_product_baseline_regression.py, test_copilot_product_tools.py, case-workspace.test.cjs | PR #135/137 코드 및 CI 근거 |
| Copilot | test_copilot_flow.py, test_copilot_action_transactions.py, test_copilot_grounded_document_qa.py | API/mock/안전 경계. 실제 사용자 과업 평가와 구분 |

목록은 전체 테스트 이름을 복제한 것이 아닙니다. 모든 테스트 파일은 [Backend tests](https://github.com/gyuniverse-hq/bid-change-validator/tree/36f1afba8e1a025006b0bfa1876502e6232b3d28/apps/api/tests), [Web tests](https://github.com/gyuniverse-hq/bid-change-validator/tree/36f1afba8e1a025006b0bfa1876502e6232b3d28/apps/web/tests)에 있습니다. 개별 `/evaluation` 화면의 전용 제품 테스트·전체 Human 결과는 독립 확인되지 않았습니다.

## 5. 재현 명령

개발 저장소의 위 SHA에서 Python 3.12, Node 22, pnpm 및 격리 PostgreSQL을 사용합니다. [수집·실행 준비](../04_contracts/data-preprocessing.md) 후 Backend CI의 동등한 준비·명령은 다음과 같습니다.

```powershell
python -m pip install -r apps/api/requirements-dev.txt
# DATABASE_URL을 새 테스트 DB로 설정한 뒤 실행
python -m alembic -c apps/api/alembic.ini upgrade head
python -m pytest -q apps/api/tests
python scripts/run_golden_regression.py
cd apps/web
pnpm install --frozen-lockfile
pnpm lint
pnpm build
```

테스트는 DB seed와 cleanup을 수행하므로 운영·공유 DB에 실행하지 않습니다. CI 실제 migration 명령·작업 디렉터리는 [MVP workflow](https://github.com/gyuniverse-hq/bid-change-validator/blob/36f1afba8e1a025006b0bfa1876502e6232b3d28/.github/workflows/mvp-integration-baseline.yml)를 기준으로 합니다. Golden은 DB/API key 없이 실행 가능하며 [Golden workflow](https://github.com/gyuniverse-hq/bid-change-validator/blob/36f1afba8e1a025006b0bfa1876502e6232b3d28/.github/workflows/golden-regression.yml)에 gate가 있습니다. required status check 설정 여부는 workflow 존재만으로 확인할 수 없습니다.

## 6. 미검증·사람 작업

| 항목 | 현재 상태 | 완료 증거 |
| --- | --- | --- |
| G1 Requirement/Evidence 독립 라벨 | Pending | 검수자·버전·승인 라벨·holdout |
| G2 의미 변경 및 결과 | DRAFT / not_ground_truth | before/after Requirement·Judgment 독립 승인 |
| Human Click E2E | Pending | [시나리오 A~D](test-scenarios.md) 실제 결과·실행 ID·캡처 |
| Copilot 사용자 Task | Pending | 과업 완료율·소요시간·오해/실패 기록 |
| Deployment Smoke | Pending | 최종 URL·설정·health·인증·DB/storage 접근 결과 |
| Frontend lint | 오류 20개 기록 | 후속 코드 수정 후 lint 재실행 |
| 모델 성능 최종 평가 | 일부 source-reported | [Evaluation](llm-rag-evaluation.md)의 영역별 gate 충족 |

이 문서의 PASS는 명시된 실행에만 적용됩니다. 공식 저장소 코드 동기화, 운영 배포, 실공고 Ground Truth 완료 주장은 포함하지 않습니다.
