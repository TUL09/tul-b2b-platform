# Decision Log

확정된 기술·사업 결정을 기록합니다. 이 문서는 확정된 결정의 Single Source of Truth(SSOT)입니다.

## 형식

| 날짜 | 결정 ID | 제목 | 결정 내용 | 근거 | 영향 범위 | 결정자 |
|---|---|---|---|---|---|---|

---

## Phase 0 결정

| 날짜 | ID | 제목 | 결정 내용 | 근거 | 영향 범위 |
|---|---|---|---|---|---|
| 2026-07-16 | DEC-001 | 프로젝트 저장 경로 | `C:\Projects\b2b-tool-platform` | scratch 경로에 장기 프로젝트 미생성 | 전체 |
| 2026-07-16 | DEC-002 | 기술 스택 | Next.js App Router + TypeScript strict + Tailwind CSS v4 + Supabase | 통합 관리, 안정적 생태계 | 전체 |
| 2026-07-16 | DEC-003 | 단계 용어 구분 | Business Evolution Stage 1~6 / Development Phase 0~13 | 사업·개발 단계 분리 | 전체 문서 |
| 2026-07-16 | DEC-004 | Stage 1~2 판매 주체 | 운영사가 유일한 판매·고객응대 주체 | 초기 오픈마켓 복잡성 회피, 통합 경험 | 주문, 출고, 정산 |
| 2026-07-16 | DEC-005 | 3개 독립 State Machine | Order Request / Sales Order / Shipment 분리 | 주문·출고·결제·세금계산서 혼합 방지 | 주문·출고 설계 |
| 2026-07-16 | DEC-006 | 수량 모델 | accepted_quantity 기반 7종 수량 필드 | confirmed_quantity 모호성 해소 | 주문 품목 |
| 2026-07-16 | DEC-007 | Import 정책 | STRICT_ATOMIC — 전체 성공 또는 전체 롤백 | 반쪽 가격표·데이터 불일치 방지 | 데이터 등록 |
| 2026-07-16 | DEC-008 | SKU 식별자 | UUID PK + internal_sku_code (UNIQUE, NOT NULL) | 시스템 식별과 업무 식별 분리 | 상품, 가격, 주문 |
| 2026-07-16 | DEC-009 | 바코드 구조 | sku_barcodes 별도 테이블, 포장단위별 복수 바코드 | 바코드를 PK로 미사용 | 상품, 검색 |
| 2026-07-16 | DEC-010 | 판매단위 UOM | sku_uoms 별도 테이블, EA/PACK/SET/BOX 등 복수 단위 | 공구 유통에서 다단위 사용 | 상품, 가격, 주문 |
| 2026-07-16 | DEC-011 | 수량구간 가격 | price_book_items에 min_quantity/max_quantity | B2B 수량별 단가 차이 필수 | 가격, 주문 |
| 2026-07-16 | DEC-012 | 조직 다중 역할 | organization_roles 테이블, buyer+supplier+brand_owner 가능 | 실제 유통 구조 반영 | 조직, 권한 |
| 2026-07-16 | DEC-013 | 조직정보 분리 | organization_business_profiles + buyer_accounts 분리 | 역할별 데이터 혼합 방지 | 조직, 거래처 |
| 2026-07-16 | DEC-014 | 구매사 내부 결재 | 기본 비활성화, Feature Flag, Release 1 Optional | 소규모 거래처 강제 방지, 수요 확인 후 | 주문 |
| 2026-07-16 | DEC-015 | 환경 전략 | Local/Preview/Staging/Production 4환경 분리 | 운영 데이터 보호, 안전 배포 | 전체 |
| 2026-07-16 | DEC-016 | 상태 분리 원칙 | order/fulfillment/payment/tax_invoice 상태 별도 관리 | "거래완료" 모호성 제거 | 주문, 출고, 정산 |
| 2026-07-16 | DEC-017 | 상품 데이터 확대 순서 | 20→50→200→시험→1,000 단계적 확대 | 50에서 1,000으로 직행 금지 | 운영 |
| 2026-07-16 | DEC-018 | RFQ 분류 | Customer Pilot Must-Have | 1,000 SKU로 미등록 상품 요청 필수 | 주문 |
| 2026-07-23 | DEC-019 | 금액 데이터형 | NUMERIC + currency_code — KRW는 NUMERIC(15,0), 외화 NUMERIC(15,4), 환율 NUMERIC(12,6) | 운영자 가독성, 외화 확장성, FLOAT 오차 방지 | 전체 금액 필드 |
| 2026-07-23 | DEC-020 | open_quantity | 저장하지 않고 계산: accepted - shipped - cancelled | 동기화 부담 제거, 불변식 단순화 | 주문 품목 |
| 2026-07-23 | DEC-021 | 상태 이력 구조 | 3개 독립 이력 테이블 + sales_order는 status_type 컬럼으로 4종 구분 | FK 무결성 보장, RLS 단순화 | 주문, 출고 |
| 2026-07-23 | DEC-022 | 낙관적 동시성 제어 | version 컬럼 (INTEGER, DEFAULT 1) — UPDATE 시 version 검증 | 동시 수정 충돌 방지 | 주요 업무 테이블 |
| 2026-07-23 | DEC-023 | Import State Machine | 12단계 상태: uploaded→validating→validated→dry_run→approval→applying→applied | 전체 흐름 표준화 | Import |
| 2026-07-23 | DEC-024 | Storage 버킷 전략 | 3 버킷: public-product-assets, private-business-documents, private-rfq-attachments | 공개/비공개 분리, signed URL 정책 차등 | 파일 보안 |
| 2026-07-24 | DEC-025 | RLS 기본원칙 강화 | Supabase Data API에 노출되는 모든 테이블은 RLS를 활성화한다. 공개 읽기 테이블도 RLS 필수. 자식 테이블은 자체 RLS 정책을 가진다. 부모 RLS·FK·CASCADE로 간접 보호한다는 가정을 폐기한다 | Supabase PostgREST는 RLS 미활성 테이블에 무제한 접근 허용 | 전체 44 테이블 + 1 View |
| 2026-07-24 | DEC-026 | 자식 테이블 자체 RLS | cart_items, order_request_items, order_revisions, order_revision_items, sales_order_items, shipment_items, order_request_status_history, sales_order_status_history, shipment_status_history — 9개 자식 테이블에 EXISTS 서브쿼리 기반 자체 RLS 적용 | FK·CASCADE는 조회 격리를 제공하지 않음 | 보안, 테스트 |
| 2026-07-24 | DEC-027 | Numeric 정밀도 통합 | 단가/금액: NUMERIC(19,4), 수량: NUMERIC(18,6), 환율: NUMERIC(20,8), 세율: NUMERIC(9,6). 기존 NUMERIC(15,0)/(15,4)/(12,4)/(10,4)/(5,4)/(12,6) 폐기 | 다중통화 확장, USD·CNY 소수 지원, 정밀 변환 | 전체 금액·수량 필드 |
| 2026-07-24 | DEC-028 | shipped_quantity Source of Truth | sales_order_items에 저장하지 않음. shipment_items의 유효 출고수량 합계로 계산. open_quantity도 계산으로 도출 | 중복 저장 시 동기화 부담, 1,000 SKU에서 JOIN 비용 무시 가능 | 주문 품목, 출고 |
| 2026-07-24 | DEC-029 | sku_search_index 유형 | SECURITY INVOKER View로 확정. MV 전환은 검색 성능이 목표 초과 시 검토 (OD-022) | 초기 1,000 SKU에서 일반 View 충분, SECURITY INVOKER로 기반 테이블 RLS 적용 | 검색, 보안 |
| 2026-07-24 | DEC-030 | GRANT/REVOKE 정책 | anon/authenticated의 DELETE 명시적 REVOKE. server_only 테이블은 anon/authenticated REVOKE ALL. Function은 REVOKE EXECUTE ON ALL 후 개별 GRANT | RLS만으로 불충분한 권한 제어 보완 | 전체 테이블·함수 |
| 2026-07-24 | DEC-031 | tax_rate 소수 비율 | 세율은 소수 비율로 정의: 0.100000 = 10%. NUMERIC(9,6). KRW 확정 금액은 Application에서 소수부 0 검증 | 국제 표준, 계산 편의, 정밀도 확보 | 세금 정책, 주문 금액 |
| 2026-07-30 | DEC-032 | 상태 이력 테이블 접근 | 3개 상태 이력 테이블(order_request/sales_order/shipment_status_history)을 server_only → authenticated_direct로 변경. authenticated SELECT + RLS(자사+운영사). INSERT는 definer_rpc(상태 전이 함수)만 허용. UPDATE/DELETE 금지 | 구매자 주문상태 화면에 이력 표시 필요. server_only면 별도 RPC 필요하여 불필요한 복잡성 | 보안, 주문, GRANT |
| 2026-07-30 | DEC-033 | Write Path 분류 | 모든 Release 1 테이블에 write_path 지정: direct_authenticated_rls, invoker_rpc, definer_rpc, server_route_service_role, no_write. GRANT는 write_path에 따라 차등 적용 | GRANT/RLS 모순 해소. 쓰기 경로 불명확 시 권한 과잉 또는 미달 발생 | Schema Inventory, GRANT 매트릭스 |
| 2026-07-30 | DEC-034 | 백오더 출고 정책 | 출고 대상이 backorder 물량이면 같은 트랜잭션에서 backordered_quantity 감소. 출고 후 backordered > open_quantity 허용 안 함. fn_record_shipment에서 FOR UPDATE + 불변식 검증 | backordered 저장값과 shipped 계산값 간 일관성 유지 | 수량 모델, 출고 |
| 2026-07-30 | DEC-035 | SECURITY DEFINER search_path | 모든 DEFINER 함수는 SET search_path = '' 사용. 모든 테이블·뷰·함수 참조는 schema-qualified (public.table_name). public 스키마 CREATE 권한을 PUBLIC/anon/authenticated에서 제거 | search_path injection 방지 | 전체 DEFINER 함수 |
