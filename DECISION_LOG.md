# Decision Log

확정된 기술·사업 결정을 기록합니다. 이 문서는 확정된 결정의 Single Source of Truth(SSOT)입니다.

## 형식

| 날짜 | 결정 ID | 제목 | 결정 내용 | 근거 | 영향 범위 | 결정자 |
|---|---|---|---|---|---|---|

## Phase 0 결정

| 날짜 | 결정 ID | 제목 | 결정 내용 | 근거 | 영향 범위 | 결정자 |
|---|---|---|---|---|---|---|
| 2026-07-16 | DEC-001 | 프로젝트 저장 경로 | `C:\Projects\b2b-tool-platform` 사용 | 장기 프로젝트에 scratch 경로를 사용하지 않음 | 전체 | 운영자 |
| 2026-07-16 | DEC-002 | 기술 스택 | Next.js App Router + TypeScript strict + Tailwind CSS v4 + Supabase | 프론트엔드와 서버 통합 관리, 안정적 생태계 | 전체 | 운영자 |
| 2026-07-16 | DEC-003 | 사업 발전단계/개발 단계 용어 | Business Evolution Stage 1~6 / Development Phase 0~13 | 사업 모델 확장과 소프트웨어 구축 단계를 명확히 분리 | 전체 문서 | 운영자 |
| 2026-07-16 | DEC-004 | Stage 1~2 판매 주체 | 운영사가 유일한 판매 주체이자 고객응대 주체 | 초기 오픈마켓 복잡성 회피, 거래처 통합 경험 | 주문, 출고, 세금계산서 | 운영자 |
| 2026-07-16 | DEC-005 | 상태 모델 분리 | Order Request / Sales Order / Shipment 3개 독립 State Machine | 주문·출고·결제·세금계산서 상태 혼합 방지 | 주문·출고 설계 | 운영자 |
| 2026-07-16 | DEC-006 | 수량 모델 | accepted_quantity 기반 (requested, accepted, rejected, shipped, cancelled, open, backordered) | confirmed_quantity의 의미 모호성 해소 | 주문 품목 | 운영자 |
| 2026-07-16 | DEC-007 | 엑셀 Import 정책 | STRICT_ATOMIC — 전체 성공 또는 전체 롤백 | 반쪽 가격표, 데이터 불일치 방지 | 데이터 등록 | 운영자 |
| 2026-07-16 | DEC-008 | SKU 식별자 | UUID PK + internal_sku_code (UNIQUE, NOT NULL, 불변) | 시스템 식별과 업무 식별 분리 | 상품, 가격, 주문 | 운영자 |
| 2026-07-16 | DEC-009 | 바코드 구조 | sku_barcodes 별도 테이블, 포장단위별 복수 바코드 지원 | 바코드를 PK로 사용하지 않음 | 상품, 검색 | 운영자 |
| 2026-07-16 | DEC-010 | 판매단위 UOM | sku_uoms 별도 테이블, EA/PACK/BOX 등 복수 주문단위 | 공구 유통에서 낱개/팩/박스 동시 사용 | 상품, 가격, 주문 | 운영자 |
| 2026-07-16 | DEC-011 | 수량구간 가격 | price_book_items에 min_quantity/max_quantity 지원 | B2B 유통에서 수량별 단가 차이 필수 | 가격, 주문 | 운영자 |
| 2026-07-16 | DEC-012 | 조직 다중 역할 | organization_roles 테이블, 한 조직이 buyer+supplier+brand_owner 가능 | 실제 유통 구조 반영 | 조직, 권한 | 운영자 |
| 2026-07-16 | DEC-013 | 조직정보 분리 | organization_business_profiles (사업자정보) + buyer_accounts (거래처 운영정보) 분리 | 역할별 데이터 혼합 방지 | 조직, 거래처 | 운영자 |
| 2026-07-16 | DEC-014 | 구매사 내부 결재 | 기본 비활성화, Feature Flag, Release 1 Optional | 소규모 거래처 강제 방지, 수요 확인 후 구현 | 주문 | 운영자 |
| 2026-07-16 | DEC-015 | 환경 전략 | Local / Preview / Staging / Production 4환경 분리 | 운영 데이터 보호, 안전한 배포 | 전체 | 운영자 |
