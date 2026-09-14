# LLM / RAG Pipeline

> Status: Draft / Final Submission Preparation
> Implementation Source: `gyuniverse-hq/bid-change-validator` `develop`

## 목적

Requirement Extraction, Document RAG, Rule Engine, AI Copilot의 책임과 데이터 경계를 설명합니다.

## Source of Truth

- `docs/03_ai/README.md`
- `docs/03_ai/retrieval-current-state.md`
- `docs/03_ai/core-copilot-contract.md`, `apps/api/app/ai/**`, Copilot 구현·Test

## 작성 예정 내용

- 문서 입력부터 Requirement / Evidence 생성까지의 Pipeline
- LLM, Retrieval, Rule Judgment의 책임 분리
- Copilot의 조회·도구 호출·Grounding 범위

## 검증 원칙

확인하지 않은 구현이나 수치는 완료로 표현하지 않는다.
