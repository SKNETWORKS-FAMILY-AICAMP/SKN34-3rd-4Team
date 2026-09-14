# LLM / RAG Evaluation

> Status: Current Evaluation Evidence / 독립 정답·사용자 평가 일부 Pending
> Implementation Baseline: `gyuniverse-hq/bid-change-validator` `develop@36f1afba8e1a025006b0bfa1876502e6232b3d28`

## 1. 평가 원칙

Extraction, Rule, Retrieval, Citation, Copilot, Changed Notice를 하나의 “AI 정확도”로 합치지 않습니다. Dataset 성격·분모·코드/실행 출처·승인 수준을 함께 읽습니다. [Golden 정의](golden-set.md), [CI 실행 기록](test-plan-and-results.md)에 구체적인 범위를 분리했습니다.

| 대상 | 확인된 평가 방식 | 아직 확정하지 않은 것 |
| --- | --- | --- |
| A. Requirement Extraction | selection harness, 실제 snapshot adapter, PR의 live 도달률 보고 | 독립 holdout Extraction precision/recall/F1 |
| B. Rule | 고정 Canonical fixture 40입력/138행 회귀 | 실서비스 전체 정확도 |
| C. Retrieval / RAG | 동일 DRAFT fixture의 strict Evidence Recall@4 | 의미상 retrieval 정답률·모든 문서 일반화 |
| D. Citation / Evidence | source 참조·Version·기대 발췌 일치 | 문장별 의미적 entailment·법적 판단 |
| E. Copilot | routing/tool/action·안전 경계 회귀 | 실제 사용자 과업 성공률 |
| F. Changed Notice | 합성 G0, 실제 G2 후보 | 실제 G2 승인 Ground Truth·Human E2E |

## 2. A — Requirement Extraction

[PR #128](https://github.com/gyuniverse-hq/bid-change-validator/pull/128)은 우치공원 R26BK01634263에서 골든 요건 3개가 판정기까지 도달하지 않는 문제를 보고했습니다. 제목 하위 가/나/다 선별, 법령 문자열 safety 오인, 비교용 문장부호 정규화를 보완했습니다.

| 측정·보고 | Before | After | 증거 수준 |
| --- | ---: | ---: | --- |
| 우치공원 3개 요건의 판정기 도달 | 0/3 | 2/3 | PR #128 당시 실행 보고 |
| 같은 PR의 20공고 live 도달 | 비교값 미확인 | 18/59 (30.5%) | PR 보고; 원시 live artifact는 현재 tracked 파일 목록에서 확인하지 못함 |
| 복합 수집·운반/장비 조건 | 미분류 포함 | 단순 요건으로 오확정하지 않고 검토 대상 유지 | 코드·PR 설명 |

이 수치는 **독립 Dataset의 Extraction F1이 아닙니다.** scripts/run_extraction_recall.py는 기대 요건과 제품 추출을 REACHED/DROPPED/UNMAPPED/MISSED로 대조합니다. --source db는 기존 분석 Run을 읽고 --source live는 모델을 호출합니다. 실행별 변동·원문·모델·Run ID를 고정해야 합니다.

qualification-quality-v0.1은 합성 harness이며, real_adapter.py는 실제 snapshot selection-only입니다. 일부 초안 라벨이나 runner 존재로 최종 Extraction 정확도를 생성하지 않았습니다. PR live 숫자는 현재 SHA 재실행 결과로 승격하지 않습니다.

## 3. B — Deterministic Rule Regression

Dataset: qualification-v0.2, 고유 실제 공고 20건 기반 + 합성 Profile 32세트 + 이전차수 입력 8개. 총 40입력/138요건, **DRAFT_NOT_APPROVED**입니다. Canonical 요건을 직접 Rule에 주입합니다.

| 시점 / 코드 | 초안 기대값 일치 | Safe abstention | Wrong determinate | Overall 일치 |
| --- | ---: | ---: | ---: | ---: |
| 2026-09-12 / 993e5cf | 104/138 | 34/138 | 0 | 36/40 |
| 2026-09-13 / 31ec928 | 110/138 | 28/138 | 0 | 37/40 |
| 2026-09-14 12:02:15 UTC / 36f1afb | 110/138 | 28/138 | 0 | 37/40 |

초기→9/13은 일치 +6행, 안전 보류 -6행, Overall +1입력입니다. 9/14 최신 SHA CI에서는 해당 수치가 유지됐습니다. 잘못된 확정 0은 이 초안 fixture 안의 관측이며 실제 업무에서 오판이 없다는 보장이 아닙니다.

Safe abstention은 초안 기대값과 다르더라도 UNKNOWN으로 확정을 보류한 범주이며, 기대값 일치와 합쳐 “100% 정확”으로 계산하지 않습니다. Gate는 wrong_determinate=0, match>=110입니다. 분모 138은 요건 행, 40은 판정 입력, 20은 고유 공고 수입니다.

근거: [summary history](https://github.com/gyuniverse-hq/bid-change-validator/blob/36f1afba8e1a025006b0bfa1876502e6232b3d28/samples/golden/qualification-v0.2/summary.json), [최신 CI](https://github.com/gyuniverse-hq/bid-change-validator/actions/runs/34841183959).

## 4. C — Retrieval / Document RAG

[E3 보고서](https://github.com/gyuniverse-hq/bid-change-validator/blob/36f1afba8e1a025006b0bfa1876502e6232b3d28/docs/08_qa_reports/ai-copilot-v2/e3-rag-evaluation.md)의 source-reported 결과입니다. 문서 최종 관련 commit은 11c882ca51cfb53af10ff6f5f828f459c6113516 (2026-09-13 02:30 KST)이며 이번 작업에서 유료 평가를 재실행하지 않았습니다.

- DOCUMENT_QA 60문항 중 현재-Version으로 평가 가능한 59문항.
- D04-3 한 문항은 현재/이전 Version 근거를 동시에 요구해 cross-version scope gap으로 분리.
- 기대 evidence 72개, k=4, fetch_k=12, 동일 cases/evidence hash.
- cases SHA256: b0aa4e891da01004d32473f41b6e5a26386feab1a1f71b559969f48f89dae585
- evidence SHA256: 94952b895818d576b7b45b640132d38a752e66114943ee341e3b50b77b48e6f4
- 두 hash는 이번 작업에서 현재 fixture bytes와 일치함을 확인했습니다. 라벨은 DRAFT입니다.

| 방식 | strict evidence hit | Recall@4 | Case any-hit | Case all-hit | latency p50 / p95 |
| --- | ---: | ---: | ---: | ---: | --- |
| Dense | 21/72 | 29.17% | 32.20% | 27.12% | 144.50 / 210.98 ms |
| Hybrid · 채택 | 36/72 | 50.00% | 52.54% | 49.15% | 170.58 / 230.54 ms |
| Hybrid+LLM rerank · 미채택 | 40/72 | 55.56% | 57.63% | 57.63% | 2849.73 / 4556.06 ms |

Evidence Recall@4는 상위 4개 결과에서 strict matcher가 찾은 기대 발췌 수/72입니다. Any-hit/All-hit은 각각 하나 이상/모든 기대 발췌가 맞은 평가 문항 수/59입니다.

Hybrid는 Dense보다 Recall +20.83%p, p50 약 +26ms여서 기본 제품 경로로 채택했습니다. rerank는 Hybrid보다 Recall +5.56%p이지만 59개 추가 모델 호출과 약 2.85초 p50으로 증가해 미채택했습니다. 실제 화폐 비용은 측정 근거가 없어 기재하지 않습니다.

초기 provenance 정정 전 Hybrid 75%는 다른 evidence fixture의 historical 결과입니다. 현재 50%와 직접 증감 비교하지 않습니다. 표현 차이·오래된 target·동일 의미의 다른 근거 때문에 strict miss가 의미상 실패와 일치하지 않을 수 있습니다.

## 5. D — Citation / Evidence

같은 E3의 Grounded Answer v4 구조 지표입니다.

| 지표 | 결과 | 해석 |
| --- | ---: | --- |
| generation success | 59/59 | 생성 실행 성공, 정답률 아님 |
| citation present | 57/59 (96.61%) | 2개 zero-citation 보류. 무조건 인용 생성하지 않음 |
| citation Version integrity | 59/59 (100%) | 버전 혼입 검사 기준. 모든 문장의 의미적 지지 아님 |
| strict expected evidence cited | 33/72 (45.83%) | 기대 발췌의 문자열 기준 citation recall |
| case any/all expected cited | 47.46% / 44.07% | 평가 문항 기준 |
| eligibility-language flags | 6 | 검토 후보이며 자동 위반 확정 아님 |
| total latency p50/p95 | 2925.77 / 5449.73 ms | 해당 실행 환경의 결과 |

Prompt v1에서는 citation이 있어도 질문을 직접 지지하지 않는 source를 인용했습니다. v2는 근거 없으면 보류, v3는 일부 근거로 전체 조건 충족 단정 금지, v4는 원문의 불완전 식별자를 임의 복원하지 않는 방향으로 보완됐습니다. Citation presence 100% 자체를 최적화 목표로 삼지 않았습니다.

독립 검수자가 답변 문장별 source 지지·과잉 결론·보류 적절성을 평가하는 단계는 Pending입니다.

## 6. E — Copilot

routing/product tools/action/동의·Case 격리·stale 문맥·중복 확인 안전성을 자동 검증합니다. [CI 기록](test-plan-and-results.md)의 design 100 passed / 5 deselected, mock UI 및 contract 성공은 합성 계약 검사입니다. 전체 사용자 Task 성공률로 환산하지 않습니다.

Stage 11 계획의 핵심은 설명 없이 과업을 수행하는 사용자의 완료율·소요시간·막힘·잘못된 신뢰를 측정하는 것입니다. 최신 사용자가 수행한 확정 scorecard는 이번 검토에서 확인하지 못했습니다.

## 7. F — Changed Notice / Revalidation

G0는 최소 실적 4억→6억 변경에서 영향 요건만 재판정하는 회귀 근거입니다. G2는 R26BK01686455의 지역문구 candidate_removed 관찰이며 DRAFT/not_ground_truth입니다. 의미상 요건 삭제·변경 영향·before/after Judgment 라벨 승인·Human Click 성공률은 미측정입니다.

Rule v0.3으로 stale 판정 재사용을 막은 PR #135는 안전성 보완이며 G2 Ground Truth 완료를 뜻하지 않습니다.

## 8. 다음 평가의 완료 조건

| 대상 | 다음 단계 / 완료 근거 |
| --- | --- |
| Extraction | 독립 source/Canonical 라벨, 고정 holdout, 실제 모델별 반복 실행·precision/recall/F1·drop 사유 |
| Rule | 기대값 독립 승인 후 DRAFT와 approved 결과를 별도 버전으로 비교 |
| Retrieval | 동일 fixture 유지, hard-query 분석, 변경된 target 검수 후 새 버전 발행 |
| Citation | 의미 지지·보류 적절성의 독립 평가, 구조 지표와 분리 |
| Copilot | 실제 사용자 Task scorecard, 실패 발화·소요시간·안전성 기록 |
| G2 | 전체 첨부 검수, 독립 변화 라벨, 실제 Run lineage와 Human E2E |

실행 방법과 Dataset은 [Golden](golden-set.md), [Test plan](test-plan-and-results.md), 개발 저장소의 [평가 scripts](https://github.com/gyuniverse-hq/bid-change-validator/tree/36f1afba8e1a025006b0bfa1876502e6232b3d28/apps/api/app/scripts)에서 찾을 수 있습니다.
