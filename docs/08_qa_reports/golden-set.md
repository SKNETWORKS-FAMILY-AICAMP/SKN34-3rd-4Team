# Golden Set / Ground Truth 상태

> Status: Current Dataset Inventory / DRAFT 라벨 승인 Pending
> Implementation Baseline: `gyuniverse-hq/bid-change-validator` `develop@36f1afba8e1a025006b0bfa1876502e6232b3d28`

## 1. Dataset의 목적을 분리한다

Golden이라는 이름만으로 실제 공고·독립 정답·제품 전체 검증을 뜻하지 않습니다.

| 등급 / Dataset | 실제·합성 구분 | 측정 가능한 범위 | 현재 승인·검증 상태 |
| --- | --- | --- | --- |
| G0 · mvp-e2e-v0.1 | 합성 공고/회사/Canonical 요건 | Judgment→Ask-back→Requirement Diff→부분 재검증 API/DB | 자동 회귀. 원본 Parsing/LLM/화면 Human 검증 아님 |
| Rule fixture · qualification-v0.2 | 실제 고유 공고 20건 기반 발췌 + 합성 회사 Profile 32세트 | Canonical 입력에 대한 결정론적 Rule 회귀 | DRAFT_NOT_APPROVED. 독립 Ground Truth 아님 |
| Harness · qualification-quality-v0.1 | 손으로 작성한 blocks·라벨·주입 slots | selection 및 합성 extraction 배선·scoring | 실공고 benchmark나 모델 점수 아님 |
| G1 · qualification-real-v0.1 | 실제 공고 고정 snapshot, 5 Case / 23 문서 metadata | source identity·hash·locator·selection reality check | 원본 확보와 독립 semantic 라벨 승인은 별개 |
| G2 · real dataset 안의 G2 Case | 실제 공고 R26BK01686455 v1/000→v2/001 | 의미 있는 변경 후보·before/after 원문 관찰 | candidate_removed / DRAFT / not_ground_truth |
| Copilot E3 fixture | DOCUMENT_QA 60문항과 기대 발췌 | 현재-Version Retrieval/Citation 구조 비교 | 독립 검수자 승인 전 DRAFT |

G1은 실제 공고 reality check, G2는 의미 있는 변경공고 검증 목표입니다. G2 후보 확보와 G2 Ground Truth 확정을 구분합니다.

## 2. Rule Fixture v0.2

고정 fixture의 실제 JSON을 읽어 현재 cases 32개, 고유 공고 20개, 고유 합성 company_id 32개를 확인했습니다. previous_cases 8개를 함께 실행하므로 총 판정 입력은 40개, 요건 행은 138개입니다. changes 8개가 있어도 이를 승인된 실제 변경공고 8건이라고 해석하지 않습니다.

```text
review_status: DRAFT_NOT_APPROVED
fixture SHA256:
ae6ce8cd89c415fef9902e5a8bc983b4d322beca3ff6a724203edf9c63c76e7f
현재 rows 104 + 이전차수 rows 34 = 총 138
```

fixture_bundle.json, baseline_993e5cf.json과 checksums.sha256를 함께 고정합니다. summary.json은 초기/후속 Rule 회귀 history와 gate를 담습니다. 기대값과 Rule이 같은 팀의 초안에 의존하므로 서비스 전체 정확도로 사용하지 않습니다.

최신 확인 수치는 [Evaluation](llm-rag-evaluation.md)에 있습니다. Rule runner는 회사 실제 정보 검증이나 원문에서 요건이 제대로 추출되는지 측정하지 않습니다.

## 3. G0 대표 Story

합성 서울 소기업은 개발인력 5명과 최근 공공기관 실적 5억원을 보유합니다. v1은 실적 4억원과 등록 보유를 요구합니다. 등록 정보 부족으로 UNKNOWN이 발생하고 보유 답변 후 USER_ANSWER 근거로 재판정합니다. v2에서 실적 기준만 6억원이 되면 실적 요건만 재검증해 UNSATISFIED가 됩니다.

test_mvp_golden_e2e.py는 Canonical 요건을 DB에 직접 저장해 이 흐름을 검증합니다. 실제 첨부 Parsing·모델 Extraction·Evidence Viewer를 거치지 않으며, 합성 성공을 실공고 Human 성공으로 바꾸지 않습니다.

## 4. G1 실제 snapshot과 provenance

| Case | 공고번호 | 분류 / identity | 문서 metadata 수 |
| --- | --- | --- | ---: |
| G2 | R26BK01686455 | PRODUCT_GOLDEN / VERIFIED | 8 |
| C04 | R26BK01687395 | PRODUCT_GOLDEN / VERIFIED | 6 |
| C01 | R26BK01705963 | SOURCE_GOLDEN / PARTIAL | 3 |
| C02 | R26BK01715895 | SOURCE_GOLDEN / PARTIAL | 3 |
| C03 | R26BK01716363 | SOURCE_GOLDEN / PARTIAL | 3 |

VERIFIED는 확보 당시 DB snapshot의 identity 확인입니다. 실시간 DB·의미 라벨 승인이 아닙니다. SOURCE_GOLDEN의 DB UUID/version_number는 null이며 로컬 document_key를 DB ID로 만들지 않습니다.

manifest → cases/<case>/case.json → text/blocks 파일로 연결합니다. 동일 내용의 text/blocks는 중복 제거하되 문서 metadata row는 보존합니다. 원본 바이너리 23개는 외부 exports/golden-candidates-2026-09-10 snapshot에 있으며 개발 Git clone만으로 원본을 다시 파싱할 수 없습니다. 공식 저장소 데이터·코드 동기화는 별도 후속입니다.

원본 binary hash, 기존 extracted text hash, blocks JSON 파일 hash는 서로 다릅니다. snapshot 문서 중 20개는 blocks newline join과 원래 text hash가 다르다고 source README에 기록돼 있습니다. 원래 hash를 재구성 hash로 대체하지 않습니다.

Source README의 “다음 Adapter PR”은 과거 계획입니다. 현재 real_adapter.py가 구현돼 있으며 C01/G2 등 선택 case를 selection-only로 처리하고 model_quality_claim=false, Ground Truth 없음 상태를 유지합니다. Adapter 존재도 라벨 승인이나 최종 F1 측정이 아닙니다.

## 5. G2 후보와 사람이 확정할 것

R26BK01686455 v1/000 PDF 6페이지, block_index 5의 참가자격 2)에는 강원도 본점 제한 문언이 있습니다. v2/001의 4개 첨부에서 공백 정규화한 같은 문장이 발견되지 않았다는 관찰입니다.

```text
observed_change: candidate_removed
review_status: DRAFT
not_ground_truth: true
```

의미상 삭제, 다른 위치/표현으로 이동, 별도 문서 참조, 효력 문제는 사람이 확인해야 합니다. 현재 observation은 expected_change_type 승인 정답이 아닙니다. G2 완료에는 전체 차수·첨부 coverage, before/after Requirement/Evidence 라벨, 영향 요건·판정 기대값, 독립 검수자·승인 시각, 제품 실제 Run 및 Human E2E가 필요합니다.

우치공원 004/005 누락·취소공고 사례는 데이터 완전성 사례이며 이 G2 지역문구 후보와 다른 공고입니다.

## 6. 재현·검증

개발 저장소 고정 SHA, Python 3.12 및 requirements-dev 설치 후:

```powershell
python scripts/run_golden_regression.py
python -m apps.api.app.scripts.validate_real_golden_dataset --dataset samples/golden/qualification-real-v0.1
python -m apps.api.app.ai.quality_eval.real_adapter --dataset samples/golden/qualification-real-v0.1 --cases C01 G2
python -m apps.api.app.scripts.quality_eval_report --dataset samples/golden/qualification-quality-v0.1
```

real validator는 checksum·path containment·UUID/null·문서-Version 연결·hash·G2 coverage를 검사합니다. 원본 비교는 별도 외부 snapshot을 확보하고 --snapshot 옵션을 사용해야 합니다. 합성 quality runner 기본값은 selection-only이며 --synthetic-extraction은 고정 slots 주입입니다. 실제 LLM 평가가 아닙니다.

## 7. 고정 근거

- [Rule fixture](https://github.com/gyuniverse-hq/bid-change-validator/tree/36f1afba8e1a025006b0bfa1876502e6232b3d28/samples/golden/qualification-v0.2)
- [G0](https://github.com/gyuniverse-hq/bid-change-validator/tree/36f1afba8e1a025006b0bfa1876502e6232b3d28/samples/golden/mvp-e2e-v0.1)
- [Real snapshot](https://github.com/gyuniverse-hq/bid-change-validator/tree/36f1afba8e1a025006b0bfa1876502e6232b3d28/samples/golden/qualification-real-v0.1), [G2 observation](https://github.com/gyuniverse-hq/bid-change-validator/blob/36f1afba8e1a025006b0bfa1876502e6232b3d28/samples/golden/qualification-real-v0.1/cases/G2/observations/source-change.json)
- [Real adapter](https://github.com/gyuniverse-hq/bid-change-validator/blob/36f1afba8e1a025006b0bfa1876502e6232b3d28/apps/api/app/ai/quality_eval/real_adapter.py), [Synthetic harness](https://github.com/gyuniverse-hq/bid-change-validator/tree/36f1afba8e1a025006b0bfa1876502e6232b3d28/samples/golden/qualification-quality-v0.1)
