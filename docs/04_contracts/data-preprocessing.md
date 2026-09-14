# 데이터 수집 및 전처리

> Status: Current Implementation / live 수집·원본 재파싱 검증은 별도
> Implementation Baseline: `gyuniverse-hq/bid-change-validator` `develop@36f1afba8e1a025006b0bfa1876502e6232b3d28`

## 1. 데이터 출처

나라장터 Open API의 공고 metadata·공고번호·차수·변경이력과 첨부문서를 사용합니다. Company Profile은 사용자 입력이며 Golden 회사 Profile은 합성입니다. 현재 DB 총 건수는 조회하지 않았으므로 기록하지 않습니다. [Golden](../08_qa_reports/golden-set.md)의 snapshot 규모와 live 수집 규모를 구분합니다.

```text
G2B API → Notice / Version / raw payload
→ 첨부 URL → 원본 bytes / hash / storage key
→ Parsing → extracted_text / extracted_blocks / locator
→ Semantic Chunk Selection → LLM 입력
→ Grounding / Canonical Requirement / Evidence
```

## 2. 나라장터 Open API 수집

G2BClient는 SERVICE/GOODS/CONSTRUCTION/FOREIGN/OTHER별 endpoint를 선택합니다. 용역은 getBidPblancListInfoServc, 물품은 getBidPblancListInfoThng입니다. 별도 변경이력 endpoint는 SERVICE/GOODS/CONSTRUCTION에 정의돼 있습니다.

| 구분 | inqryDiv | 입력 |
| --- | --- | --- |
| REGISTERED | 1 | 수집 시작·종료 시각 |
| NOTICE_NUMBER | 2 | bid_notice_no |
| CHANGED | 3 | 변경 조회 시작·종료 시각 |

시간은 KST로 변환합니다. page_size·max_pages·기간 검증을 적용하며 수집 계수와 실패를 기록합니다. 제한 페이지 도달을 전체 이력 완전성으로 간주하지 않습니다.

notice-poller의 Compose 기본값은 300초 간격, 60분 lookback, 5분 overlap, 페이지 100건/최대 10페이지입니다. 신규 발견 공고와 migration으로 등록된 기존 공고는 durable backfill queue에서 공고번호 전체 이력을 재확인합니다. 배치 기본값은 10이며 attempts/next_attempt_at로 재시도합니다.

## 3. Notice / Version과 중복 정합성

- bid_notice_no는 공고 identity입니다. 내부 version_number와 외부 bid_notice_order를 분리합니다.
- payload hash 또는 같은 차수의 기존 Version이 있으면 재사용합니다. DB는 notice+payload_hash, notice+version_number, notice+bid_notice_order unique를 둡니다.
- 전체 이력은 차수 순으로 정렬·재번호화하고 최신 차수만 current로 설정합니다. 과거 차수가 늦게 들어왔다고 최신으로 올리지 않습니다.
- 같은 차수 재수집은 기존 fact/relation을 보완하는 경로이며 원문 payload 전체를 갱신하는 범용 변경감지기가 아닙니다.
- API 이전 공고번호의 재공고 relation과 동일 공고번호의 차수 변경을 분리합니다.

## 4. 첨부 수집과 원본 저장

문서명·URL·순서를 읽어 Version에 NoticeDocument를 만듭니다. Downloader는 timeout과 기본 100 MiB 제한 아래 bytes를 받고 SHA-256을 계산합니다. 같은 수집 내 동일 URL은 다운로드·추출 정보를 재사용하고 같은 hash는 storage key를 공유할 수 있습니다. Version별 문서 metadata는 보존합니다.

LOCAL 기본 경로는 DOCUMENT_STORAGE_PATH, Docker는 /data/notice-documents 및 notice_documents_data 볼륨입니다. S3-compatible adapter는 구현돼 있으나 최종 운영 저장소 채택·동작은 이번 작업에서 검증하지 않았습니다. 다운로드 성공과 Parsing/Analysis 성공은 별개입니다.

## 5. HWP / HWPX / PDF Parsing

확장자, file signature, ZIP 내부 구성으로 분기합니다.

| 형식 | 구현 | locator / 한계 |
| --- | --- | --- |
| PDF | pypdf PdfReader, page.extract_text | 1부터 시작하는 page. OCR 없음; 스캔·표 읽기 순서 미보장 |
| HWP5 | olefile, FileHeader/BodyText, zlib, paragraph text record | section_index/paragraph_index. 암호화 HWP 미지원 |
| HWPX | ZIP Contents/sectionN.xml, XML paragraph/text | section/paragraph 보존. 표 셀 의미 구조 전체 보장 아님 |
| HWPML | XML SECTION/P/CHAR | 줄바꿈·탭과 문단 위치 처리 |
| DOCX | ZIP word/document.xml | 문단 text; Proposal 등 보조 입력 |
| TXT/CSV/MD 또는 text MIME | UTF-8 BOM → CP949 → EUC-KR | 해독 불가 시 실패 |

_finish는 제어문자 제거, CRLF 정리, 연속 공백 정규화, 빈 block 제거, block_index 부여를 수행합니다. 정규화 blocks를 두 줄바꿈으로 연결해 extracted_text를 만듭니다.

## 6. extracted text/block과 해시

| 필드 | 의미 |
| --- | --- |
| file_sha256 / Evidence source_sha256 | 원본 파일 bytes |
| extracted_text_sha256 | 추출 text의 UTF-8 bytes |
| extracted_blocks | text + block_index/page/section_index/paragraph_index/location |
| text_extractor / extracted_at | parser 종류·추출 시각 |
| Dataset blocks_file_sha256 | export된 JSON bytes. 원본 hash와 다름 |

기존 snapshot text와 blocks 재결합 문자열은 다를 수 있습니다. 이를 손상으로 단정하거나 원래 hash를 대체하지 않습니다.

## 7. Semantic Chunk / AI 입력

Backend는 EXTRACTED이면서 blocks가 있는 문서를 QualificationDocumentInput으로 전달합니다. canonical_source_blocks는 locator와 원본·text hash를 보존합니다. 문서별 Semantic Chunk에 분석 전역 ID를 붙이고 참가자격 section 및 하위 항목을 선택합니다. 기본 1,800자 청크·최대 32,000자 본문이므로 절단 진단을 확인해야 합니다.

LLM Structured Extraction → deterministic grounding/정규화 → Canonicalization → Requirement/Evidence 저장 순서입니다. Document RAG index는 별도 현재-Version 검색 경로입니다. [AI Pipeline](../03_ai/llm-rag-pipeline.md)을 참조합니다.

## 8. 실패 처리

| 상황 | 현재 처리 | 남은 확인 |
| --- | --- | --- |
| G2B 개별 항목 오류 | savepoint 격리, failed_item_count와 오류 기록 | API 변경·장기 장애 대응 |
| 이력 누락·backfill 실패 | 영속 queue, 시도 수·다음 시각 | 전체 queue 처리 완료 |
| 다운로드 실패 | FAILED와 오류 | DB key만 있고 원본 없는 기존 자료 복구 |
| 빈 문서·미지원·파싱 오류 | EMPTY / UNSUPPORTED / FAILED 구분 | 형식별 품질·OCR |
| blocks 없음 | Analysis FAILED | 다운로드·추출부터 점검 |
| grounding drop·본문 절단 | diagnostics, dropped requirements, PARTIAL 가능 | 독립 라벨·추출 품질 검수 |

## 9. 재현 방법

아래는 개발 저장소의 고정 SHA에서 실행하는 절차입니다. 공식 저장소에 제품 코드·전체 원본 데이터는 아직 동기화되지 않았습니다. 이번 Phase에서 live 수집·유료 LLM 분석을 실행한 기록은 아닙니다.

```powershell
git clone --branch develop https://github.com/gyuniverse-hq/bid-change-validator.git
cd bid-change-validator
git checkout 36f1afba8e1a025006b0bfa1876502e6232b3d28
Copy-Item .env.example .env
# .env에 격리된 로컬 DB 설정, G2B_SERVICE_KEY, OPENAI_API_KEY 입력
docker compose up -d --build api
docker compose --profile tools run --rm master-data-import
```

Swagger에서 POST /api/v1/notices/sync에 아래 입력을 사용합니다. 인증을 켠 환경은 로그인 세션이 필요합니다.

```json
{
  "business_type": "SERVICE",
  "inquiry_type": "NOTICE_NUMBER",
  "bid_notice_no": "R26BK01634263",
  "page_size": 100,
  "max_pages": 10
}
```

1. collection-runs에서 처리 범위·실패 수를 확인합니다.
2. GET /api/v1/notices에서 UUID를 얻고 GET /api/v1/notices/{notice_id}/versions에서 내부 Version·외부 차수를 확인합니다.
3. 필요하면 POST /api/v1/notices/documents/extract-pending?retry_failed=true를 실행합니다.
4. Version별 document /text, /source에서 상태·locator·원본 존재를 확인합니다.
5. 분석할 Version에 POST .../qualification-analysis를 실행하고 Run ID·contract/status·Evidence·diagnostics를 기록합니다.
6. 취소공고·첨부 없는 차수는 정상 자격분석 입력으로 가정하지 않습니다.

자동 수집은 docker compose --profile collector up -d --build api notice-poller로 별도 실행합니다. 재현에는 공유 DB 대신 격리된 DB를 사용합니다.

## 10. 실사례와 알려진 한계

PR #129의 2026-09-13 보고에서 우치공원 R26BK01634263의 000~005 이력이 확보됐고 004·005 누락이 보완됐습니다. 005는 취소공고·첨부 0건·current로 보고됐습니다. 이번 live DB 검산 결과가 아니라 당시 PR 실데이터 기록입니다. [Troubleshooting](../06_decisions/troubleshooting.md)에 원인·해결을 연결했습니다.

Git의 real dataset에는 text/blocks·metadata가 있고 원본 바이너리는 외부 snapshot에 있습니다. clone만으로 과거 원본을 완전히 재파싱할 수 없습니다. OCR, 첨부 URL 유효성, 실제 저장 파일 존재, RAG index 갱신·영속성, 운영 수집 범위는 추가 검증 대상입니다.

## 11. 고정 근거

- [G2B Client](https://github.com/gyuniverse-hq/bid-change-validator/blob/36f1afba8e1a025006b0bfa1876502e6232b3d28/apps/api/app/services/g2b.py), [Notice 저장](https://github.com/gyuniverse-hq/bid-change-validator/blob/36f1afba8e1a025006b0bfa1876502e6232b3d28/apps/api/app/services/notices.py)
- [원본 저장](https://github.com/gyuniverse-hq/bid-change-validator/blob/36f1afba8e1a025006b0bfa1876502e6232b3d28/apps/api/app/services/document_storage.py), [Parsing](https://github.com/gyuniverse-hq/bid-change-validator/blob/36f1afba8e1a025006b0bfa1876502e6232b3d28/apps/api/app/services/document_extraction.py)
- [Poller](https://github.com/gyuniverse-hq/bid-change-validator/blob/36f1afba8e1a025006b0bfa1876502e6232b3d28/apps/api/app/workers/notice_polling.py), [PR #129](https://github.com/gyuniverse-hq/bid-change-validator/pull/129)
- [Blocks adapter](https://github.com/gyuniverse-hq/bid-change-validator/blob/36f1afba8e1a025006b0bfa1876502e6232b3d28/apps/api/app/ai/qualification/extraction/backend_blocks.py)
