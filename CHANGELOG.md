# Changelog

프로젝트 변경 이력을 기록합니다.

형식: `[Phase] YYYY-MM-DD — 변경 내용`

---

## [Phase 0] 2026-07-23 — Checkpoint B 기술 설계

Git 정리:
- `master` → `main` 브랜치 변경
- `phase0-checkpoint-a-approved` 태그 생성
- `docs/checkpoint-b` 작업 브랜치 생성

Checkpoint A 감사:
- PRODUCT_REQUIREMENTS.md State Machine 상태값을 BUSINESS_RULES.md SSOT와 일치 수정

Checkpoint B 문서 5개 작성:
- `DATABASE_SCHEMA.md` — 42 Required + 10 Optional + 15 Future = 67 테이블, 8 Migration Group, 8 ERD
- `SECURITY_RULES.md` — RLS 정책, 접근 매트릭스, 8 보안 시나리오, Storage 3 버킷
- `DATA_IMPORT_SPEC.md` — 3 Import 템플릿, Import State Machine, 검증 규칙
- `INTERNATIONALIZATION.md` — MVP 기본값, 번역 구조, 통화, UOM 표시
- `TEST_PLAN.md` — 111 테스트 케이스 (74 Critical), 8 영역, 성능 기준

운영 문서 갱신:
- `DECISION_LOG.md` — DEC-019~024 추가 (6개)
- `OPEN_DECISIONS.md` — OD-020~022 추가 (3개)
- `CHANGELOG.md` — 이력 업데이트

---

## [Phase 0] 2026-07-16 — Checkpoint A 문서 작성

Checkpoint A 문서 8개 작성 완료:
- `PROJECT_OVERVIEW.md` — 사업 배경, 비전, Stage 1~6, 핵심 원칙
- `MARKET_CONTEXT.md` — 시장 구조, 공급·구매 측 문제, 포지셔닝
- `PRODUCT_REQUIREMENTS.md` — 기능 분류 (43 Must-Have, 5 Pilot, 9 Optional, 11 Future, 8 Excluded)
- `BUSINESS_RULES.md` — 가격, UOM, 수량구간, 주문 State Machine, 수량 모델, Import 정책
- `USER_ROLES.md` — 10개 역할, 8개 Permission, 4개 내부 승인 Capability
- `USER_FLOWS.md` — 8개 업무 흐름 Mermaid 다이어그램
- `INFORMATION_ARCHITECTURE.md` — 4개 영역 사이트맵, 페이지 목록, 내비게이션
- `ROADMAP.md` — Phase 0~13, 상품 확대 순서, Operational Readiness Track

운영 문서 업데이트:
- `DECISION_LOG.md` — Phase 0 확정 결정 18개 기록
- `OPEN_DECISIONS.md` — 미결정 항목 19개 기록
- `CHANGELOG.md` — 이력 업데이트

---

## [Phase 0] 2026-07-16 — 프로젝트 초기화

- 프로젝트 디렉토리 생성: `C:\Projects\b2b-tool-platform`
- Git 저장소 초기화
- `.gitignore` 생성
- `.env.example` 생성
- Scaffold 문서 생성: README.md, DECISION_LOG.md, OPEN_DECISIONS.md, CHANGELOG.md, AGENTS.md
