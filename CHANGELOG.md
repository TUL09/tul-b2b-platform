# Changelog

프로젝트 변경 이력을 기록합니다.

형식: `[Phase] YYYY-MM-DD — 변경 내용`

---

## [Phase 0] 2026-08-07 — Checkpoint C1 디자인·콘텐츠 시스템

Checkpoint C1 신규 문서 3개:
- `DESIGN_SYSTEM.md` — Industrial Intelligence 디자인 시스템 SSOT: 색상 토큰(20종), 타이포그래피, 반응형 기준(390/1440px), 스페이싱(4px 단위), 18개 핵심 컴포넌트(검색바/UOM/가격표/수량단계/수정안비교/Import오류 등), 4개 주문상태 독립 표현 원칙, 접근성 규칙
- `CONTENT_SYSTEM.md` — 콘텐츠 유형 8종(신제품브리핑/브랜드스포트라이트/가격경쟁/기획전/연관상품/카탈로그/브랜드이야기/공지), 상태 7단계(draft→archived), 콘텐츠 타깃, 자산 권리 관리, 성과 측정 지표, 금지사항
- `WIREFRAME_INDEX.md` — 8개 핵심 화면 인덱스, 검증 체크리스트

와이어프레임 8개 (`docs/wireframes/`):
- `WF-01-mobile-home.svg` — 모바일 거래처 홈 (390px, 검색우선, 재주문)
- `WF-02-mobile-product-list.svg` — 모바일 상품목록 (인라인장바구니, MOQ오류)
- `WF-03-mobile-quick-order.svg` — 모바일 빠른주문 (다행입력, 이전주문불러오기)
- `WF-04-pc-product-list.svg` — PC 상품목록 (좌측필터, 컴팩트표, 다중선택)
- `WF-05-pc-product-detail.svg` — PC 상품상세 (UOM선택, 수량구간강조, 소계)
- `WF-06-admin-order-review.svg` — 관리자 주문검토 (원본↔수정안, 부분확정)
- `WF-07-admin-import.svg` — 관리자 Import (Blocking Error/Warning, STRICT_ATOMIC)
- `WF-08-buyer-order-status.svg` — 구매자 주문상태 (4개 상태 독립, 타임라인)

운영 문서 갱신:
- `DECISION_LOG.md` — DEC-036~040 추가 (5개)
- `OPEN_DECISIONS.md` — OD-018 해결, OD-023 추가
- `CHANGELOG.md` — 이력 업데이트

브랜치: `docs/checkpoint-c1-design-content`

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
