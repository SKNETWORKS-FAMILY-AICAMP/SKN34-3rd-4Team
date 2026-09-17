# BidCheck 프로젝트 문서

> SK Networks Family AI Camp 34기 · 3차 프로젝트 · 4팀  
> 프로젝트: **나라장터 변경공고 대응형 입찰 제출 검증기 · BidCheck**  
> 문서 목적: 구현 저장소의 세부 기술 기록을 평가·발표 관점에서 재구성한 제출용 문서입니다.

## 문서 읽는 순서

| 순서 | 문서 | 무엇을 설명하나요? |
| --- | --- | --- |
| 1 | [01-프로젝트-개요.md](./01-프로젝트-개요.md) | 어떤 문제를 해결하고 어떤 사용자 흐름을 만들었는지 |
| 2 | [02-시스템-아키텍처.md](./02-시스템-아키텍처.md) | Frontend·Backend·DB·AI Core·Rule·Copilot이 어떻게 연결되는지 |
| 3 | [데이터-수집-전처리.md](./데이터-수집-전처리.md) | 나라장터 데이터 수집·원문 추출·전처리·DB 운영 |
| 4 | [03-골든셋-검증데이터.md](./03-골든셋-검증데이터.md) | Golden Set과 합성 회사 Profile을 왜/어떻게 구성했는지 |
| 5 | [04-AI-Core-Copilot.md](./04-AI-Core-Copilot.md) | Requirement Extraction과 AI Copilot의 역할 분리 및 현재 구조 |
| 6 | [05-테스트-평가.md](./05-테스트-평가.md) | Rule·Routing·Retrieval·Grounding·Guided Job을 어떻게 평가했는지 |
| 7 | [06-협업.md](./06-협업.md) | Discord·Notion·GitHub·GitHub Projects·MCP를 어떻게 사용했는지 |
| 8 | [07-트러블슈팅.md](./07-트러블슈팅.md) | 대표 문제를 어떻게 측정하고 개선했는지 |

## 구현 Source of Truth

제출 저장소는 평가자가 프로젝트를 빠르게 이해할 수 있도록 핵심 내용을 정리합니다. 실제 코드·API·세부 기술 문서의 Source of Truth는 구현 저장소입니다.

- 구현 저장소: `gyuniverse-hq/bid-change-validator`
- 통합 브랜치: `develop`
- Product Baseline, AI Core, AI Copilot, QA 원본 문서는 구현 저장소 `docs/`에 보존합니다.
- 제출 README와 이 폴더는 **구현 원본을 복제하는 것이 아니라 평가용으로 요약·연결**하는 역할을 합니다.

## 핵심 설계 원칙

```text
나라장터 데이터
→ 문서 수집·버전 관리
→ AI Core: Requirement / Evidence 구조화
→ deterministic Rule: 참가자격 판정
→ Ask-back / Evidence / Changed Notice Revalidation
→ Product API
→ Frontend + AI Copilot
```

- **LLM이 최종 참가 가능/불가를 직접 결정하지 않습니다.**
- `UNKNOWN`과 `ASKABLE`을 구분합니다.
- 근거와 판정은 Notice Version / Analysis Run / Judgment Run 계보를 유지합니다.
- Copilot은 별도 판정기를 만들지 않고 기존 Product 결과와 현재 공고문 근거를 조회·설명합니다.
- 자동평가 수치는 평가 범위가 다르므로 하나의 “AI 정확도”로 합치지 않습니다.

## 기존 assets

기존 `docs/assets/`의 화면 캡처·회귀 그래프·RAG 그래프·협업 이미지는 그대로 재사용합니다.

- `collaboration-workflow.png`
- `requirements-light.png` / `requirements-dark.png`
- `regression-light.png` / `regression-dark.png`
- `rag-light.png` / `rag-dark.png`
- `s0-company.png`, `s1-notices.png`, `s2-qualification-v1.png`, `s3-evidence.png` 등

README는 모든 기술 내용을 반복하지 않고 이 문서들을 요약·링크하는 역할로 최종 정리합니다.
