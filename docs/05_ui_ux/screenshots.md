# 화면 증거 · Phase 2 캡처 계획

> Status: Current capture plan / 실제 Screenshot Validation Pending
> Implementation Baseline: `gyuniverse-hq/bid-change-validator` `develop@36f1afba8e1a025006b0bfa1876502e6232b3d28`

## 1. 현재 확보 상태

이번 Phase 1에서는 UI를 실행·촬영하지 않았습니다. 아래 파일명은 **저장 예정 이름**이며 이미지 링크가 아닙니다. 최종 Architecture image도 Phase 2 대상이고, 현재 구조의 검토 가능한 원본은 [Mermaid 아키텍처](../02_architecture/system-architecture.md)입니다. 생성형 가짜 UI와 존재하지 않는 이미지 링크는 사용하지 않습니다.

## 2. 캡처 목록

| 예정 파일 (`docs/images/` 아래) | 화면·내용 | 필요한 데이터·검수 조건 | 현재 상태 |
| --- | --- | --- | --- |
| `01-notice-search.png` | `/notices` 검색·선택·차수 | 공개 공고번호와 차수, 선택 회사 확인. 검색 결과를 자격 판정으로 오인하지 않게 설명 | Pending |
| `02-company-profile.png` | `/company` Profile·completeness | 공개 가능한 합성 회사 사용, 합성 표기, 민감정보 제외 | Pending |
| `03-qualification.png` | `/qualification` 조건·회사 값·판정·Evidence | Case/Analysis/Judgment/Rule 일치. 3상태와 provenance를 캡션으로 설명 | Pending |
| `04-analysis-partial.png` | 일부 분석·진단·확인 필요 | 재현 가능한 PARTIAL Case. 일부 분석을 참가 가능으로 표현하지 않음 | Pending |
| `05-ask-back.png` | ASKABLE 질문·답변 후 결과 | 동일 Case의 전후 Run 기록, USER_ANSWER 표시, Profile 자동 변경 없음 확인 | Pending |
| `06-evidence.png` | 조건 인용과 원문 위치 | 실제 PDF 또는 HWP/HWPX 문서·locator 대조, 가짜 page 없음 | Pending |
| `07-changes.png` | baseline/current Diff와 영향 조건 | G0 합성 Demo 또는 승인된 G2를 명시. 조건 내용과 변화 종류 표시 | Pending |
| `08-revalidation.png` | 영향 조건 재검증·전후 판정 | source/result Run, 승계/재판정 key 확인. 캡처만으로 G2 Ground Truth 승인하지 않음 | Pending |
| `09-copilot.png` | 현재 Case 질의·근거·확인 Action | 실제 지원 Task 하나를 끝까지 수행, 문서 QA 동의·확인 단계 보존 | Pending |
| `10-proposal-reference.png` | `/evaluation` 제안서 참고 | 필요 시 보조 캡처. Partial과 로컬 확인 상태를 표기, 점수화 완료로 설명하지 않음 | Pending |
| `architecture.png` | 최종 컴포넌트 연결 이미지 | Mermaid와 실제 최종 배포 구성이 일치하는지 검수 후 export | Pending |

## 3. 캡처마다 기록할 증거

1. 캡처 시각(KST), 담당자, 실행 commit, Web/API 환경 식별자와 브라우저·viewport.
2. 공고번호·차수, Case ID, Analysis/Judgment Run ID, Rule version, 기준일. 내부 식별자가 공개 부적절하면 별도 검수 기록에 보관.
3. 데이터 성격(G0 synthetic / 실제 공고 / G2 draft 또는 human-approved), 회사 Profile이 합성인지 여부.
4. 수행한 사용자 단계, 기대 결과, 관찰 결과, PASS/FAIL 및 제한. [시나리오 A~D](../08_qa_reports/test-scenarios.md)와 연결.
5. 이미지 파일, 기능 설명, 코드 baseline과 다른 변경 사항. 이미지 수정이 필요하면 민감정보 가림 등 편집 내역을 표시.

로그인 세션·키·개인정보가 보이지 않도록 공개용 데이터로 촬영합니다. Figma는 UI/UX 설계 근거이고 실제 실행 Screenshot의 대체 증거가 아닙니다.

## 4. 완료 판정

실제 파일 존재 → 이미지와 캡션 검수 → 상대 링크 추가 → Human 시나리오 결과 연결 순으로 완료합니다. 현재는 링크할 이미지가 없습니다. Screenshot은 관찰한 화면의 증거이며 전체 API·Rule 품질, 운영 배포 성공, G2 의미적 변경 정답의 자동 증명이 아닙니다.
