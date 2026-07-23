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

