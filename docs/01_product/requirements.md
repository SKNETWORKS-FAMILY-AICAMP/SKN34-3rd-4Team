# 요구사항 명세

> Status: Current Submission Specification
> Implementation Baseline: `gyuniverse-hq/bid-change-validator` `develop@cafd5dba82a54e467eb59e9578995d3362c5ca9b`

## 1. 문서 목적

최종 제출 시점의 제품 범위와 완료 조건을 실제 구현 기준으로 정의합니다. 초기 기획의 의도와 ID는 추적에 활용하되, 현재 동작은 `develop`의 Code / Test와 Current 문서를 우선합니다.

상세 구현·검증 연결은 [Feature / Implementation / Test Traceability](../02_architecture/feature-traceability.md)에서 확인합니다.

### Source of Truth

1. 개발 저장소 `develop`의 실제 Code / Test
2. `docs/01_product/README.md`, `docs/02_architecture/feature-traceability.md`, `docs/08_qa_reports/requirement-test-traceability.md`
3. Current Reconciliation 문서
4. 초기 요구사항·PRD는 ID와 의도 확인용 Historical Source

## 2. 상태 및 ID 정의

| 상태 | 의미 |
| --- | --- |
| Current | 코드와 자동 테스트 또는 명시된 구현 근거가 현재 `develop`에서 확인됨 |
| Partial | 일부 경로만 구현됐거나 제품 전체 연결·검증이 남아 있음 |
| Pending | 범위 또는 완료 근거가 아직 확정되지 않음 |
| Superseded | 초기 요구가 현재 정책으로 대체됨 |
| Out of Scope / Future | 현재 MVP에 포함하지 않고 향후 검토함 |

현재 구현에서 새로 정리한 요구사항은 `CUR-<영역>-<번호>`를 사용합니다. 검증된 의미를 알 수 없는 과거 ID에 새 기능을 억지로 연결하지 않으며, 확인된 과거 ID는 6장에서 별도로 보존합니다.

## 3. 제품 범위

### In Scope

- 나라장터 공고 검색·수집, 공고 Version과 첨부문서 관리
- 회사 Profile과 수행실적·인증·업종 정보 관리
- 공고문 기반 Qualification Requirement / Evidence 구조화
- Canonical 8유형과 결정론적 참가자격 판정
- 안전한 Askability 분류, Ask-back 답변 기반 대상 요건 재판정
- Evidence 원문 추적
- Requirement Diff와 affected-only Revalidation
- 현재 Case의 저장된 Product 결과를 조회·설명하는 AI Copilot

### Partial / Validation Pending

- `/evaluation`: 제안서 업로드·원문·관련 위치 후보·사용자 확인은 구현됐으나 평가기준 전용 추출과 점수 산정은 지원하지 않음
- 계약 위험조항: AI Core 분류, 저장 API와 자동 테스트는 존재하나 전용 사용자 화면·전체 E2E 연결은 확인 필요
- 변경공고: Diff와 Revalidation 코드는 구현됐으나 실제 G2 Ground Truth Human Validation은 진행 중
- Requirement Extraction / Evidence: 실제 공고 G1 정답 라벨과 최종 품질 수치 미확정
- AI Copilot: API·도구·확인형 Action은 구현됐으나 사용자 Task 평가는 대기
- 주요 화면의 전체 Human Click E2E와 외부 Deployment Smoke는 대기

### Out of Scope / Future

- 입찰 평가 점수 예측 또는 낙찰 가능성 예측
- Ask-back 답변의 Company Profile 자동 승격
- LLM 또는 Copilot이 만드는 별도 참가 가능/불가 판정
- 일반 목적 Agent, 무제한 장기 기억, 여러 공고 Version을 동시에 검색하는 Document QA
- 완료 근거가 없는 Production 배포 주장

## 4. 비기능 요구사항

| ID | 요구사항 | Acceptance Criteria | 상태 | 근거 |
| --- | --- | --- | --- | --- |
| CUR-NFR-01 | 최종 참가자격 판정은 재현 가능한 Rule이 담당한다. | 동일한 Canonical Requirement, Company `profile_snapshot`, 기준일, Rule Version 입력은 동일한 Judgment를 만든다. | Current | `qualification/rules/judgment.py`, `test_qualification_judgment.py` |
| CUR-NFR-02 | 판정과 원문 근거를 추적할 수 있어야 한다. | `requirement_key → evidence_key → document_id → location` 연결이 저장·응답에서 유지된다. | Current | `analysis_models.py`, `ai/contracts.py`, `/evidence` |
| CUR-NFR-03 | 공고·분석·판정·재검증 이력을 덮어쓰지 않는다. | Notice Version과 Analysis / Judgment / Revalidation Run ID로 기준·현재 결과를 구분할 수 있다. | Current | `models.py`, `analysis_models.py`, `judgment_models.py`, `revalidation_models.py` |
| CUR-NFR-04 | 불명확한 조건을 확정 판정이나 무분별한 질문으로 바꾸지 않는다. | 복합·예외·근거 부족 조건은 `UNKNOWN` 또는 직접 확인으로 남고 Askability Guard를 통과한 단일 사실만 질문한다. | Current | `rules/askability.py`, `rules/clause_safety.py`, `test_askability.py` |
| CUR-NFR-05 | 오래되거나 변경된 문맥으로 쓰기 작업을 수행하지 않는다. | Analysis, Judgment, Company Snapshot, Rule Version 또는 기준일이 달라지면 Ask-back/Revalidation을 거부하거나 전체 재판정을 요구한다. | Current | `qualification/ask_back.py`, `qualification/revalidation.py`, Copilot Action Test |
| CUR-NFR-06 | 사용자·회사·Case 접근 경계를 확인한다. | 보호 API가 인증 문맥을 사용하고 다른 회사 또는 Case 접근을 거부하는 자동 테스트가 존재한다. | Current | `auth.py`, `main.py`, `test_auth.py`, `test_copilot_flow.py` |

## 5. 기능 요구사항

### 5.1 공고 찾기

| ID | 요구사항 | Acceptance Criteria | 상태 | 관련 Route |
| --- | --- | --- | --- | --- |
| CUR-NOT-01 | 사용자는 수집된 나라장터 공고를 검색하고 검토 대상을 선택할 수 있다. | `GET /api/v1/notices` 결과를 `/notices`에서 조회하며 미분석 상태를 확정 판정처럼 표시하지 않는다. | Current | `/notices` |
| CUR-NOT-02 | 동일 공고의 차수와 첨부문서를 Version 단위로 보존한다. | 공고별 Version 목록과 Version에 귀속된 문서·추출 상태를 조회할 수 있고 동일 payload는 중복 Version을 만들지 않는다. | Current | `/notices` |
| CUR-MAT-01 | 회사 기준으로 분석 완료 공고의 자격 매칭 결과를 조회할 수 있다. | `GET /api/v1/companies/{company_id}/notice-matches`가 저장된 분석·판정 근거 범위에서 결과를 반환한다. | Current | `/notices` |

### 5.2 회사 Profile

| ID | 요구사항 | Acceptance Criteria | 상태 | 관련 Route |
| --- | --- | --- | --- | --- |
| CUR-COM-01 | 회사 기본정보·업종·수행실적·인증을 관리할 수 있다. | Company CRUD와 Performance / Certification API가 입력 검증과 함께 동작한다. | Current | `/company` |
| CUR-COM-02 | 현재 Profile과 과거 판정 입력을 구분한다. | Judgment Run에 `profile_snapshot`을 저장하고 현재 Profile 변경이 과거 결과를 소급 변경하지 않는다. | Current | `/company`, `/qualification` |

### 5.3 참가자격 분석 / 판정

| ID | 요구사항 | Acceptance Criteria | 상태 | 관련 Route |
| --- | --- | --- | --- | --- |
| CUR-ANL-01 | 추출된 공고문에서 Requirement와 Evidence를 구조화한다. | Analysis Run이 Canonical 8유형의 Requirement와 Evidence locator를 저장하고 조회 API로 반환한다. | Current | `/qualification` |
| CUR-ANL-02 | 분석 성공·부분 성공·실패를 구분하고 동일 차수의 중복 실행을 제어한다. | `SUCCEEDED / PARTIAL / FAILED`가 보존되고 최신 호환 Analysis를 사용하며 동일 Version의 중복 실행을 방지한다. | Current | `/qualification` |
| CUR-JDG-01 | 회사 Profile을 Canonical Requirement와 결정론적으로 비교한다. | LLM 출력이 아닌 Rule Engine이 Requirement별 판정을 생성하고 Rule Version을 기록한다. | Current | `/qualification` |
| CUR-JDG-02 | 개별 판정, 전체 판정, 판정 근거 축을 분리한다. | 개별 상태는 `SATISFIED / UNSATISFIED / UNKNOWN`, 전체 상태는 `eligible / ineligible / insufficient_data`, 근거는 `basis_type`으로 표현한다. | Current | `/qualification` |

Canonical Requirement 8유형은 `PERFORMANCE_AMOUNT`, `PERFORMANCE_COUNT`, `INDUSTRY`, `REGION`, `STAFF`, `REGISTRATION_CERTIFICATION`, `EXPERIENCE_FIELD`, `COMPANY_SIZE`입니다.

### 5.4 Ask-back

| ID | 요구사항 | Acceptance Criteria | 상태 | 관련 Route |
| --- | --- | --- | --- | --- |
| CUR-ASK-01 | `UNKNOWN` 중 안전하게 답변 가능한 항목만 질문한다. | `classify_askability`가 단일 사실·지원 유형·연산자·구조화 값·복합조건 Guard를 확인하고 `askable`과 reason code를 반환한다. | Current | `/ask-back` |
| CUR-ASK-02 | 사용자 답변으로 해당 Requirement만 다시 판정한다. | 답변 결과는 새 Judgment Run에 `basis_type=USER_ANSWER`로 기록하며 `apply_to_profile=true`는 거부한다. | Current | `/ask-back` |

### 5.5 Evidence

| ID | 요구사항 | Acceptance Criteria | 상태 | 관련 Route |
| --- | --- | --- | --- | --- |
| CUR-EVD-01 | 사용자는 판정의 공고 원문 근거를 확인할 수 있다. | Requirement의 Evidence Key에서 문서와 block/page/section 등 실제 locator 및 인용문으로 이동할 수 있다. | Current | `/evidence`, `/qualification` |

### 5.6 변경공고 / Revalidation

| ID | 요구사항 | Acceptance Criteria | 상태 | 관련 Route |
| --- | --- | --- | --- | --- |
| CUR-CHG-01 | 이전·현재 Requirement를 의미 단위로 비교한다. | 각 Requirement를 `UNCHANGED / MODIFIED / ADDED / REMOVED`로 분류하고 변경 전·후 구조를 보존한다. | Current | `/changes` |
| CUR-CHG-02 | 영향을 받은 Requirement만 다시 판정한다. | 변경되지 않은 판정은 승계하고 `MODIFIED / ADDED`를 재판정한다. Profile Snapshot, Rule Version 또는 기준일 변경 시 전체 재판정을 요구한다. | Current | `/changes` |

### 5.7 AI Copilot

| ID | 요구사항 | Acceptance Criteria | 상태 | 관련 Route |
| --- | --- | --- | --- | --- |
| CUR-COP-01 | 현재 Case의 저장된 판정·근거·확인 필요 항목·Profile·변경 내역을 자연어로 조회한다. | Copilot Product Tool은 저장된 최신 호환 상태만 읽고 별도 판정·추출·쓰기 작업을 수행하지 않는다. | Current | 공통 Copilot UI |
| CUR-COP-02 | 상태 변경 Action은 제안과 명시적 확인을 분리한다. | 답변 반영·재검증은 Proposal을 먼저 반환하고 `/api/v1/copilot/actions/confirm`에서 최신 문맥을 재검증한 뒤 실행한다. | Current | 공통 Copilot UI |

### 5.8 평가 대응 / 계약 위험조항

| ID | 요구사항 | Acceptance Criteria | 상태 | 관련 Route |
| --- | --- | --- | --- | --- |
| CUR-EVL-01 | 공고 요구 항목과 업로드한 제안서의 관련 위치 후보를 나란히 확인한다. | 제안서 원문·후보 위치·사용자 확인 상태를 제공하되 평가 점수나 충족 여부를 자동 확정하지 않는다. | Partial | `/evaluation` |
| CUR-RSK-01 | 계약 위험조항 분류 결과와 근거를 Version 단위로 저장·조회한다. | AI Core의 9종 Canonical Category 결과를 Backend가 재분류하지 않고 저장하며 Version 불일치를 거부한다. | Partial | 전용 Product 화면 확인 필요 |

## 6. 변경된 초기 요구사항

| 기존 ID | 초기 요구 | 현재 정책 | 상태 | 변경 이유 |
| --- | --- | --- | --- | --- |
| NFR-4 | `confidence low`이면 Ask-back 질문을 생성 | `UNKNOWN != ASKABLE`; Grounding, Diagnostic, Profile completeness, Askability를 분리 | Superseded | 낮은 confidence만으로 복합·법적 조건을 사용자 단일 답변으로 축약할 수 없음 |
| NFR-5 | 사용자 답변 기반 결과를 별도 색상·판정 상태로 표현 | Domain Judgment 3상태 유지, `basis_type=USER_ANSWER`로 provenance 표현 | Superseded / Redefined | 판정 결과와 데이터 출처를 독립된 축으로 관리하기 위함 |
| FR-P-7 / FR-A-6 | Ask-back 답변을 Company Profile에 저장·재사용 | 현재 Case의 `USER_ANSWER` 근거로 사용, Profile 자동 승격 없음 | Superseded for MVP | 일회 답변이 장기 Profile 사실로 오염되는 것을 방지 |
| FR-J-2 | 7유형 Requirement와 4색 판정 | Canonical 8유형, 개별 3상태, Overall Status와 provenance 분리 | Superseded | 현재 Contract와 결정론적 판정 모델에 맞게 축을 분리 |
| Evaluation 관련 | 평가 Route에서 평가기준 추출·점수화까지 수행 | 제안서 위치 후보와 사용자 직접 확인 중심, 점수 예측 제외 | Redefined / Partial | Route 존재와 전용 Product Pipeline 완성을 구분 |
| 위험조항 관련 | 과거 PoC 범위를 현재 제품 기능으로 간주 | 9종 Core 분류와 저장 API까지만 Current 근거로 사용 | Partial | 전용 화면과 전체 사용자 E2E 근거가 아직 부족함 |

## 7. 최종 MVP 경계

최종 MVP의 중심은 **공고 Version별 Requirement / Evidence를 구조화하고 회사 Snapshot으로 판정한 뒤, 변경공고에서 영향받은 Requirement만 재검증하는 것**입니다.

자동 테스트는 구현 회귀 근거이며 Human Click E2E, 실제 공고 G1/G2 Ground Truth, 사용자 Task 평가, Deployment Smoke 완료를 대신하지 않습니다. 확인하지 않은 구현이나 수치는 완료로 표현하지 않습니다.
