# Changelog

프로젝트 변경 이력을 기록합니다.

형식: `[Phase] YYYY-MM-DD — 변경 내용`

---

## [Phase 0] 2026-07-30 — Checkpoint B 최종 종결 패치

접근·정합성 수정 (Checkpoint B Final Closure Patch):
- `DATABASE_SCHEMA.md` — Schema Inventory에 read_path/write_path 컬럼 추가, 상태 이력 3개 테이블 authenticated_direct로 변경, api_exposure 합계 10+25+7+2=44 확정, fn_record_shipment 트랜잭션 문서화, CHECK 보강
- `SECURITY_RULES.md` — GRANT 매트릭스 write_path 기반 차등 적용, 상태 이력 접근 변경, SECURITY DEFINER search_path='' 강화, fn_update_prices GRANT/RLS 일관성
- `BUSINESS_RULES.md` — shipped_quantity/open_quantity SoT 명시, 출고 트랜잭션 규칙(6.4), 백오더 정책, CHECK 제약조건 추가
- `TEST_PLAN.md` — 출고 동시성·백오더 테스트 5개 추가 (160 TC, 116 Critical)
- `DECISION_LOG.md` — DEC-032~035 추가 (4개)
- 용어 드리프트 수정: company_price_overrides→organization_price_overrides (BUSINESS_RULES, PRD), NUMERIC(15,0)→NUMERIC(19,4) (INTERNATIONALIZATION)
- `CHANGELOG.md` — 이력 업데이트

---

## [Phase 0] 2026-07-24 — Checkpoint B 보안 강화

보안 감사 (Checkpoint B Security Correction):
- `DATABASE_SCHEMA.md` — Numeric 정밀도 통합 (NUMERIC(19,4)/NUMERIC(18,6)/NUMERIC(20,8)/NUMERIC(9,6)), Schema Inventory 45항목 추가, shipped_quantity 계산 전환, 수량 불변식 CHECK 제약조건 추가, sku_search_index SECURITY INVOKER View 확정
- `SECURITY_RULES.md` — RLS 기본원칙 13개항 교체, 자식 테이블 9개 자체 RLS 정책 27개 추가, 공개 상품 테이블 10개 RLS 정책, GRANT/REVOKE 매트릭스, DB Function 보안 목록 7개
- `TEST_PLAN.md` — 보안 강화 테스트 15개 + 수량 정합성 5개 추가 (155 TC, 111 Critical)
- `DECISION_LOG.md` — DEC-025~031 추가 (7개)
- `CHANGELOG.md` — 이력 업데이트

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
