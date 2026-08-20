# API_SPEC.md — TUL B2B 플랫폼 API 사양 (SSOT)

> **SSOT 참조**
> - 가격·주문·출고 규칙 → BUSINESS_RULES.md
> - 인증·권한·RLS → SECURITY_RULES.md
> - DB 스키마 → DATABASE_SCHEMA.md
> - 기능 범위 → PRODUCT_REQUIREMENTS.md
> - 개발 순서 → ROADMAP.md

이 문서는 각 기능이 어느 실행 경계에서 처리되는지, 누가 호출할 수 있는지,
어떤 입력·출력·오류·감사 규칙을 적용하는지 정의하는 **API 경계 설계 SSOT**입니다.

실제 구현 코드는 포함하지 않습니다.

---

## 1. 실행 경계 분류

| 유형 | 코드 | 설명 | 주요 용도 |
|---|---|---|---|
| Next.js Server Action | **SA** | 로그인 후 폼 제출·화면 액션 | 장바구니, 주소 관리, 관리자 폼 |
| Next.js Route Handler | **RH** | HTTP 엔드포인트 | 검색 자동완성, 파일 다운로드, Webhook |
| Supabase PostgreSQL RPC | **RPC** | DB 함수 직접 호출 | 가격계산, 주문확정, 출고, Import |
| Supabase Edge Function | **EF** | 비동기·외부 연동 | 이메일 발송, 이카운트 연동 |
| External Webhook | **WH** | 외부 서비스 콜백 | 배송추적, 결제알림, ERP응답 |
| Direct Query + RLS | **DQ** | RLS로 보호된 직접 Supabase 쿼리 | 단순 목록 조회 |
| Background / Scheduled Job | **BG** | 예약·반복 작업 | 만료처리, 집계 |
| No API Required | **NA** | 정적 제공 | 공개 콘텐츠, 정적 페이지 |

---

## 2. 기본 배치 원칙

### 2.1 Server Action (SA) 사용

- 로그인 후 폼 제출 (장바구니, 주소, 프로필)
- 서버 측 Zod 유효성 검증이 필요한 관리자 화면 액션
- 파일이 없고 멱등성이 단순한 쓰기 작업

### 2.2 Route Handler (RH) 사용

- 검색 자동완성 등 GET 기반 동적 엔드포인트
- 파일 다운로드 (Import 결과, 카탈로그)
- 외부에서 호출되는 HTTP 엔드포인트
- Webhook 수신 (배송사, 결제사)

### 2.3 PostgreSQL RPC (RPC) 사용

- 트랜잭션과 Row Lock이 필수인 쓰기 작업
- 가격 계산, 주문확정, 출고수량 반영, Import Apply
- 비즈니스 불변식 (BUSINESS_RULES.md 6.2) 강제가 필요한 작업

### 2.4 Edge Function (EF) 사용

- 이메일 발송 (Resend 연동)
- 파일 변환·압축
- 외부 ERP(이카운트) 연동 (Phase 7 이후)
- 재시도·비동기가 필요한 외부 API 호출

### 2.5 Webhook (WH) 사용

- 택배 상태 콜백
- 결제 알림
- 외부 서비스 응답

---

## 3. Database Function 사양

SECURITY_RULES.md 7절과 DATABASE_SCHEMA.md에 정의된 7개 핵심 Function.

모든 Function: REVOKE EXECUTE FROM public 후 필요한 역할에만 GRANT EXECUTE.

---

### 3.1 fn_calculate_price

| 항목 | 내용 |
|---|---|
| purpose | 거래처·SKU UOM·수량에 대한 적용 단가 계산 (4단계 우선순위) |
| execution_boundary | RPC |
| caller | Server Action / Server Component (Server-side only) |
| authentication | authenticated JWT 필수 |
| capability | 승인 거래처(approved buyer) 이상 |
| security_mode | SECURITY INVOKER (호출자 권한으로 실행, RLS 적용) |
| request_input | org_id UUID, sku_uom_id UUID, quantity INTEGER |
| response_output | unit_price, price_source_type, price_source_id, price_source_version, tier_min_quantity, tier_max_quantity, currency_code, tax_inclusion_mode, price_book_code |
| transaction_boundary | READ ONLY |
| row_lock | 없음 |
| OCC_version | 없음 |
| idempotency | 읽기 전용 — 중복 호출 안전 |
| audit | 없음 (읽기) |
| related_test | TC-PRICE-001 ~ TC-PRICE-010 |
| first_needed_phase | Phase 4 |

---

### 3.2 fn_submit_order_request

| 항목 | 내용 |
|---|---|
| purpose | 장바구니 → 주문 요청 제출. 가격 재계산, MOQ/UOM 재검증, 스냅샷 저장 |
| execution_boundary | RPC |
| caller | Server Action (구매자 주문 요청하기 버튼) |
| authentication | authenticated JWT 필수 |
| capability | can_create_order_request |
| security_mode | SECURITY INVOKER (호출자 권한, RLS 적용) |
| request_input | cart_id UUID, idempotency_key UUID, shipping_address_id UUID, buyer_note TEXT |
| response_output | order_request_id, order_request_number, status, items JSONB |
| transaction_boundary | 단일 트랜잭션 (cart 검증, 가격 재계산, order_requests INSERT, items INSERT, cart 비우기) |
| row_lock | carts FOR UPDATE |
| OCC_version | 없음 |
| idempotency | idempotency_key UUID 필수. 동일 키 재호출 시 기존 결과 반환 |
| audit | audit_logs INSERT (action: order_request.submitted) |
| error_codes | CART_EMPTY, MOQ_VIOLATION, UOM_NOT_ORDERABLE, PRICE_NOT_FOUND, DUPLICATE_REQUEST |
| related_test | TC-ORDER-001 ~ TC-ORDER-005 |
| first_needed_phase | Phase 4 |

---

### 3.3 fn_approve_revision

| 항목 | 내용 |
|---|---|
| purpose | 구매자가 운영사 수정안을 승인 또는 거절. 상태 전이 및 만료일 검증 |
| execution_boundary | RPC |
| caller | Server Action (구매자 수정안 확인 CTA) |
| authentication | authenticated JWT 필수 |
| capability | 해당 주문 소속 조직 구매자 |
| security_mode | SECURITY INVOKER (호출자 권한, RLS 적용) |
| request_input | order_revision_id UUID, decision TEXT (approved or rejected), buyer_note TEXT |
| response_output | order_request_id UUID, new_status TEXT |
| transaction_boundary | 단일 트랜잭션 |
| row_lock | order_revisions FOR UPDATE |
| OCC_version | order_revision.version 낙관적 잠금 |
| idempotency | 동일 revision_id + decision → 기존 상태 반환 |
| audit | audit_logs INSERT (action: order_revision.approved or rejected) |
| error_codes | REVISION_EXPIRED, REVISION_ALREADY_PROCESSED, PERMISSION_DENIED, INVALID_TRANSITION |
| related_test | TC-ORDER-020 ~ TC-ORDER-025 |
| first_needed_phase | Phase 4 |

---

### 3.4 fn_create_sales_order

| 항목 | 내용 |
|---|---|
| purpose | accepted 상태 주문요청 → 확정 주문(Sales Order) 생성. 최종 가격 스냅샷 |
| execution_boundary | RPC |
| caller | Server Action (관리자 확정 주문 생성 액션) |
| authentication | authenticated JWT 필수 |
| capability | can_manage_orders |
| security_mode | SECURITY DEFINER (트랜잭션, SET search_path = '', schema-qualified, DEC-035) |
| request_input | order_request_id UUID, idempotency_key UUID, operator_note TEXT |
| response_output | sales_order_id UUID, sales_order_number TEXT, items JSONB, total_amount INTEGER |
| transaction_boundary | 단일 트랜잭션 (order_request 잠금, 상태 전이, sales_orders INSERT, items INSERT with price snapshot) |
| row_lock | order_requests FOR UPDATE |
| OCC_version | order_request.version 낙관적 잠금 |
| idempotency | idempotency_key UUID 필수. 중복 생성 방지 |
| audit | audit_logs INSERT (action: sales_order.created) |
| error_codes | ORDER_REQUEST_NOT_ACCEPTED, PRICE_RECALC_FAILED, DUPLICATE_SALES_ORDER |
| related_test | TC-ORDER-030 ~ TC-ORDER-035 |
| first_needed_phase | Phase 4 |

---

### 3.5 fn_record_shipment

| 항목 | 내용 |
|---|---|
| purpose | 출고 생성 및 수량 불변식 검증. shipped+cancelled <= accepted 강제 |
| execution_boundary | RPC |
| caller | Server Action (관리자 출고 처리 액션) |
| authentication | authenticated JWT 필수 |
| capability | can_manage_shipments |
| security_mode | SECURITY DEFINER (트랜잭션, SET search_path = '', schema-qualified, DEC-035) |
| request_input | sales_order_id UUID, idempotency_key UUID, shipment_items JSONB, carrier_code TEXT, tracking_number TEXT, shipment_type TEXT |
| response_output | shipment_id UUID, shipment_number TEXT, status TEXT |
| transaction_boundary | 단일 트랜잭션 (sales_order_items FOR UPDATE, 수량 검증, shipments INSERT, backordered_quantity 감소, audit INSERT) |
| row_lock | sales_order_items FOR UPDATE |
| OCC_version | 없음 (Row Lock으로 충돌 방지) |
| idempotency | idempotency_key UUID 필수 |
| audit | audit_logs INSERT (action: shipment.created) |
| error_codes | QUANTITY_EXCEEDED, BACKORDERED_VIOLATION, INVALID_SALES_ORDER_STATUS, DUPLICATE_SHIPMENT |
| related_requirement | BUSINESS_RULES.md 6.4 |
| related_test | TC-SHIP-001 ~ TC-SHIP-010 |
| first_needed_phase | Phase 4 |

---

### 3.6 fn_apply_import

| 항목 | 내용 |
|---|---|
| purpose | Dry Run 통과한 Import 배치를 실제 DB에 원자적으로 반영. STRICT_ATOMIC |
| execution_boundary | RPC |
| caller | Server Action (관리자 최종 반영 버튼) |
| authentication | authenticated JWT 필수 |
| capability | can_manage_products |
| security_mode | SECURITY DEFINER (STRICT_ATOMIC, SET search_path = '', schema-qualified, DEC-035) |
| request_input | import_batch_id UUID, idempotency_key UUID |
| response_output | applied_count INTEGER, skipped_count INTEGER, error_count INTEGER, status TEXT |
| transaction_boundary | 단일 원자 트랜잭션. 1건이라도 오류 시 전체 rollback (STRICT_ATOMIC) |
| row_lock | catalog_imports FOR UPDATE, 영향 테이블 잠금 |
| OCC_version | target_data_version 검증 |
| idempotency | idempotency_key UUID 필수. 반영 완료 배치 재호출 시 결과만 반환 |
| audit | audit_logs INSERT (action: catalog_import.applied) |
| error_codes | IMPORT_BLOCKING_ERROR, VERSION_CONFLICT, STRICT_ATOMIC_ROLLBACK, DUPLICATE_APPLY |
| related_test | TC-ADM-001 ~ TC-ADM-010 |
| first_needed_phase | Phase 3 |

---

### 3.7 fn_update_prices

| 항목 | 내용 |
|---|---|
| purpose | 가격표 일괄 업데이트. 수량구간 중복 검증, 기존 확정 주문 불변 보장 |
| execution_boundary | RPC |
| caller | Server Action (관리자 가격표 저장) |
| authentication | authenticated JWT 필수 |
| capability | can_manage_prices |
| security_mode | SECURITY INVOKER (호출자 권한, RLS 적용). price_book_items UPDATE GRANT + RLS can_manage_prices 필요 (SECURITY_RULES §12.3) |
| request_input | price_book_id UUID, items JSONB, effective_from DATE |
| response_output | updated_count INTEGER, conflict_count INTEGER |
| transaction_boundary | 단일 트랜잭션 (구간 중복 검증, price_book_items upsert) |
| row_lock | price_book_items FOR UPDATE (동일 price_book) |
| OCC_version | 없음 |
| idempotency | 동일 데이터 재전송 시 멱등 (upsert) |
| audit | audit_logs INSERT (action: price_book.updated) |
| error_codes | TIER_OVERLAP, INVALID_UOM, PRICE_BOOK_NOT_FOUND |
| related_test | TC-PRICE-020 ~ TC-PRICE-025 |
| first_needed_phase | Phase 4 |

---

## 4. API Catalog

### 4.1 인증·조직 (AUTH)

| api_id | name | type | actor | auth | capability | phase |
|---|---|---|---|---|---|---|
| AUTH-001 | 사업자 회원가입 | SA | Guest | N | — | Phase 2 |
| AUTH-002 | 이메일 로그인 | DQ (Supabase Auth) | Guest | N | — | Phase 2 |
| AUTH-003 | 로그아웃 | SA | Authenticated | Y | — | Phase 2 |
| AUTH-004 | 거래처 승인 | SA | Op Staff | Y | can_manage_customers | Phase 2 |
| AUTH-005 | 거래처 정지 | SA | Op Staff | Y | can_manage_customers | Phase 2 |
| AUTH-006 | 조직 멤버 조회 | DQ | Buyer Admin | Y | — | Phase 2 |
| AUTH-007 | 조직 멤버 초대 | SA | Buyer Admin | Y | — | Phase 2 |
| AUTH-008 | 배송지 추가·수정·삭제 | SA | Buyer | Y | — | Phase 2 |
| AUTH-009 | 비밀번호 재설정 요청 | SA | Guest | N | — | Phase 2 |
| AUTH-010 | 회원가입 승인 이메일 | EF | System | — | — | Phase 2 |

### 4.2 상품 (PRD)

| api_id | name | type | actor | auth | capability | phase |
|---|---|---|---|---|---|---|
| PRD-001 | 상품 검색 | DQ | Buyer | Y | — | Phase 3 |
| PRD-002 | 검색 자동완성 | RH | Buyer | Y | — | Phase 3 |
| PRD-003 | 상품 상세 조회 | DQ | Buyer | Y | — | Phase 3 |
| PRD-004 | 상품 목록 (관리자) | DQ | Op Staff | Y | can_manage_products | Phase 3 |
| PRD-005 | 상품 등록·수정 | SA | Op Staff | Y | can_manage_products | Phase 3 |
| PRD-006 | SKU 등록·수정 | SA | Op Staff | Y | can_manage_products | Phase 3 |
| PRD-007 | UOM 설정 | SA | Op Staff | Y | can_manage_products | Phase 3 |
| PRD-008 | 브랜드 등록·수정 | SA | Op Admin | Y | — | Phase 3 |
| PRD-009 | 카테고리 관리 | SA | Op Admin | Y | — | Phase 3 |
| PRD-010 | 상품 비활성화 | SA | Op Staff | Y | can_manage_products | Phase 3 |

### 4.3 가격 (PRC)

| api_id | name | type | actor | auth | capability | phase |
|---|---|---|---|---|---|---|
| PRC-001 | 거래처 적용가 조회 | RPC | Buyer | Y | approved buyer | Phase 4 |
| PRC-002 | 수량구간 가격 목록 | DQ | Buyer | Y | — | Phase 4 |
| PRC-003 | 가격표 관리 | SA | Op Staff | Y | can_manage_prices | Phase 4 |
| PRC-004 | 가격표 일괄 업데이트 | RPC | Op Staff | Y | can_manage_prices | Phase 4 |
| PRC-005 | 거래처 개별가격 설정 | SA | Op Staff | Y | can_manage_prices | Phase 4 |
| PRC-006 | 공급조건 관리 | SA | Op Staff | Y | can_manage_suppliers | Phase 4 |
| PRC-007 | 공급가 조회 | DQ | Op Staff | Y | can_view_purchase_cost | Phase 4 |

### 4.4 장바구니·주문 (CART / ORD)

| api_id | name | type | actor | auth | capability | phase |
|---|---|---|---|---|---|---|
| CART-001 | 장바구니 조회 | DQ | Buyer | Y | — | Phase 4 |
| CART-002 | 장바구니 품목 추가 | SA | Buyer | Y | — | Phase 4 |
| CART-003 | 장바구니 품목 수량 변경 | SA | Buyer | Y | — | Phase 4 |
| CART-004 | 장바구니 품목 삭제 | SA | Buyer | Y | — | Phase 4 |
| ORD-001 | 주문요청 제출 | RPC | Buyer | Y | can_create_order_request | Phase 4 |
| ORD-002 | 주문요청 목록 조회 (구매자) | DQ | Buyer | Y | — | Phase 4 |
| ORD-003 | 주문요청 상세 조회 | DQ | Buyer/Op | Y | — | Phase 4 |
| ORD-004 | 주문요청 목록 (관리자) | DQ | Op Staff | Y | can_manage_orders | Phase 4 |
| ORD-005 | 수정안 발행 | SA | Op Staff | Y | can_manage_orders | Phase 4 |
| ORD-006 | 수정안 승인·거절 | RPC | Buyer | Y | — | Phase 4 |
| ORD-007 | 확정주문 생성 | RPC | Op Staff | Y | can_manage_orders | Phase 4 |
| ORD-008 | 확정주문 조회 (구매자) | DQ | Buyer | Y | — | Phase 4 |
| ORD-009 | 확정주문 목록 (관리자) | DQ | Op Staff | Y | can_manage_orders | Phase 4 |
| ORD-010 | 주문 취소 요청 | SA | Buyer | Y | — | Phase 4 |
| ORD-011 | 취소 승인·거부 | SA | Op Staff | Y | can_manage_orders | Phase 4 |

### 4.5 출고 (SHIP)

| api_id | name | type | actor | auth | capability | phase |
|---|---|---|---|---|---|---|
| SHIP-001 | 출고 생성 | RPC | Op Staff | Y | can_manage_shipments | Phase 4 |
| SHIP-002 | 부분 출고 | RPC | Op Staff | Y | can_manage_shipments | Phase 4 |
| SHIP-003 | 송장번호 등록 | SA | Op Staff | Y | can_manage_shipments | Phase 4 |
| SHIP-004 | 출고 상태 변경 | SA | Op Staff | Y | can_manage_shipments | Phase 4 |
| SHIP-005 | 출고 목록 조회 | DQ | Op/Buyer | Y | — | Phase 4 |
| SHIP-006 | 배송 Webhook 수신 | RH | 외부 배송사 | Webhook Secret | — | Phase 5 |

### 4.6 RFQ

| api_id | name | type | actor | auth | capability | phase |
|---|---|---|---|---|---|---|
| RFQ-001 | RFQ 생성 | SA | Buyer | Y | — | Phase 5 |
| RFQ-002 | 첨부파일 등록 | RH | Buyer | Y | — | Phase 5 |
| RFQ-003 | RFQ 목록 조회 | DQ | Buyer/Op | Y | — | Phase 5 |
| RFQ-004 | 관리자 응답 | SA | Op Staff | Y | can_manage_orders | Phase 5 |

### 4.7 Import (IMP)

| api_id | name | type | actor | auth | capability | phase |
|---|---|---|---|---|---|---|
| IMP-001 | 파일 업로드 | RH | Op Staff | Y | can_manage_products | Phase 3 |
| IMP-002 | 컬럼 매칭 | SA | Op Staff | Y | can_manage_products | Phase 3 |
| IMP-003 | 검증 (Blocking/Warning) | SA | Op Staff | Y | can_manage_products | Phase 3 |
| IMP-004 | Dry Run | SA | Op Staff | Y | can_manage_products | Phase 3 |
| IMP-005 | Import Apply | RPC | Op Staff | Y | can_manage_products | Phase 3 |
| IMP-006 | 오류 파일 다운로드 | RH | Op Staff | Y | can_manage_products | Phase 3 |
| IMP-007 | Import 이력 조회 | DQ | Op Staff | Y | can_manage_products | Phase 3 |

### 4.8 콘텐츠 (CNT)

| api_id | name | type | actor | auth | capability | phase |
|---|---|---|---|---|---|---|
| CNT-001 | 콘텐츠 목록 조회 (구매자) | DQ | Buyer | Y | — | Phase 5 |
| CNT-002 | 신제품 브리핑 작성 | SA | Op Staff | Y | can_manage_content | Phase 5 |
| CNT-003 | 브랜드 스포트라이트 작성 | SA | Op Staff | Y | can_manage_content | Phase 5 |
| CNT-004 | 프로모션 기획전 작성 | SA | Op Staff | Y | can_manage_content | Phase 5 |
| CNT-005 | 기술자료 업로드 | RH | Op Staff | Y | can_manage_content | Phase 5 |
| CNT-006 | 콘텐츠 승인·발행 | SA | Op Admin | Y | — | Phase 5 |

---

## 5. 오류 카탈로그

### 5.1 공통 오류 응답 구조

모든 오류 응답은 다음 필드를 포함합니다.

- error_code: 시스템 오류 코드 (영문, 대문자)
- user_message: 사용자에게 표시할 한국어 메시지 (민감정보 미포함)
- technical_message: 서버 로그 전용 (클라이언트 미전송)
- field_errors: 필드별 유효성 오류 목록
- request_id: 추적용 UUID
- retryable: 재시도 가능 여부
- timestamp: ISO 8601

내부 DB 오류, 스택 트레이스, SQL 오류는 클라이언트에 전송하지 않습니다.

### 5.2 오류 코드 목록

#### AUTH

| error_code | user_message | retryable |
|---|---|---|
| AUTH_INVALID_CREDENTIALS | 이메일 또는 비밀번호가 올바르지 않습니다 | N |
| AUTH_ACCOUNT_LOCKED | 로그인 시도가 초과되어 계정이 잠겼습니다. 30분 후 시도하세요 | N |
| AUTH_TOKEN_EXPIRED | 세션이 만료되었습니다. 다시 로그인하세요 | N |
| DUPLICATE_EMAIL | 이미 등록된 이메일 주소입니다 | N |
| INVALID_BUSINESS_NUMBER | 사업자등록번호 형식이 올바르지 않습니다 | N |

#### PERMISSION

| error_code | user_message | retryable |
|---|---|---|
| PERMISSION_DENIED | 이 작업을 수행할 권한이 없습니다 | N |
| BUYER_NOT_APPROVED | 승인된 거래처만 이용할 수 있습니다 | N |
| CAPABILITY_REQUIRED | 해당 기능에 대한 권한이 없습니다 | N |
| ORG_ACCESS_DENIED | 다른 조직의 데이터에 접근할 수 없습니다 | N |

#### VALIDATION

| error_code | user_message | retryable |
|---|---|---|
| VALIDATION_ERROR | 입력값을 확인해 주세요 | N |
| REQUIRED_FIELD_MISSING | 필수 항목이 누락되었습니다 | N |

#### PRICE

| error_code | user_message | retryable |
|---|---|---|
| PRICE_NOT_FOUND | 해당 상품의 적용 가격을 찾을 수 없습니다. 견적을 요청해 주세요 | N |
| PRICE_CHANGED | 가격이 변경되었습니다. 화면을 새로고침해 주세요 | N |
| TIER_OVERLAP | 수량 구간이 겹쳐 가격표를 저장할 수 없습니다 | N |

#### UOM

| error_code | user_message | retryable |
|---|---|---|
| UOM_NOT_ORDERABLE | 선택한 판매단위로는 주문할 수 없습니다 | N |
| UOM_NOT_FOUND | 판매단위 정보를 찾을 수 없습니다 | N |

#### MOQ

| error_code | user_message | retryable |
|---|---|---|
| MOQ_VIOLATION | 최소 주문 수량 조건을 충족하지 않습니다 | N |
| ORDER_INCREMENT_VIOLATION | 주문 수량은 증가 단위에 맞게 입력해야 합니다 | N |

#### ORDER_STATE

| error_code | user_message | retryable |
|---|---|---|
| INVALID_TRANSITION | 현재 상태에서 요청한 전환이 불가능합니다 | N |
| CART_EMPTY | 장바구니가 비어 있습니다 | N |
| ORDER_REQUEST_NOT_ACCEPTED | 수락 상태의 주문요청이 아닙니다 | N |

#### REVISION

| error_code | user_message | retryable |
|---|---|---|
| REVISION_EXPIRED | 수정안 응답 기한이 지났습니다. 운영사에 문의하세요 | N |
| REVISION_ALREADY_PROCESSED | 이미 처리된 수정안입니다 | N |

#### INVENTORY / SHIPMENT

| error_code | user_message | retryable |
|---|---|---|
| QUANTITY_EXCEEDED | 출고 수량이 확정 수량을 초과합니다 | N |
| BACKORDERED_VIOLATION | 백오더 수량이 미출고 수량을 초과합니다 | N |
| INVALID_SALES_ORDER_STATUS | 해당 확정 주문 상태에서는 출고를 처리할 수 없습니다 | N |

#### IMPORT

| error_code | user_message | retryable |
|---|---|---|
| IMPORT_BLOCKING_ERROR | Blocking Error가 있어 반영할 수 없습니다 | N |
| STRICT_ATOMIC_ROLLBACK | 오류로 전체가 취소되었습니다. STRICT_ATOMIC 정책 | N |
| VERSION_CONFLICT | 데이터가 변경되었습니다. Dry Run을 다시 실행해 주세요 | Y |

#### FILE

| error_code | user_message | retryable |
|---|---|---|
| FILE_TOO_LARGE | 파일 크기가 허용 범위를 초과합니다 | N |
| FILE_TYPE_NOT_ALLOWED | 허용되지 않는 파일 형식입니다 | N |

#### CONFLICT

| error_code | user_message | retryable |
|---|---|---|
| DUPLICATE_REQUEST | 동일한 요청이 이미 처리 중입니다 | N |
| DUPLICATE_SALES_ORDER | 이미 확정 주문이 생성된 주문요청입니다 | N |
| DUPLICATE_SHIPMENT | 동일한 출고가 이미 처리되었습니다 | N |
| DUPLICATE_APPLY | 이미 반영된 Import 배치입니다 | N |

#### RATE_LIMIT

| error_code | user_message | retryable |
|---|---|---|
| RATE_LIMIT_EXCEEDED | 요청이 너무 많습니다. 잠시 후 다시 시도해 주세요 | Y |

#### INTERNAL

| error_code | user_message | retryable |
|---|---|---|
| INTERNAL_ERROR | 일시적인 오류가 발생했습니다. 잠시 후 다시 시도하거나 운영사에 문의하세요 | Y |

---

## 6. Idempotency

다음 작업에 Idempotency Key 또는 동등한 중복 방지 전략을 적용합니다.

| 작업 | api_id | 전략 |
|---|---|---|
| 주문요청 제출 | ORD-001 | idempotency_key UUID 파라미터 + DB unique constraint |
| 확정주문 생성 | ORD-007 | idempotency_key UUID + order_request 상태 검증 |
| 출고 등록 | SHIP-001 | idempotency_key UUID + Row Lock |
| Import Apply | IMP-005 | idempotency_key UUID + import_batch 상태 |
| 이메일 발송 | AUTH-010 | 발송 이력 기록 후 중복 방지 (저장 구조는 OD-031 참조) |
| Webhook 처리 | SHIP-006 | event_id 중복 확인 (저장 구조는 OD-032 참조) |

Idempotency Key 생성: 클라이언트에서 crypto.randomUUID()로 생성. 서버에서 형식 검증 필수.

---

## 7. API 보안 원칙

SECURITY_RULES.md 1절과 7절 기준.

| 원칙 | 적용 방법 |
|---|---|
| 가격·주문금액 서버 재계산 | fn_calculate_price 호출, 클라이언트 전달 가격 신뢰 금지 |
| 클라이언트 capability 신뢰 금지 | JWT + organization_members.permissions 서버 검증 |
| service_role key 브라우저 노출 금지 | 서버 환경변수 전용 |
| SECURITY DEFINER Function 직접 노출 금지 | Server Action 경유만 허용 |
| 역할별 EXECUTE 권한 | REVOKE EXECUTE FROM public 후 필요 역할에만 GRANT |
| 중요 액션 감사로그 | audit_logs INSERT (actor, target, before/after, request_id) |
| 조직 ID 클라이언트 신뢰 금지 | JWT sub로 organization_members 조회하여 org_id 도출 |
| Storage signed URL 만료 | private: 10분 (비즈니스 문서), 30분 (RFQ 첨부) |
| 민감한 오류정보 비노출 | user_message만 전달, technical_message는 서버 로그 전용 |
| request_id 사용 | 모든 응답에 request_id 포함 |

---

## 8. Phase별 API 활성화 시점

| Phase | 활성화 영역 |
|---|---|
| Phase 1 | 정적 화면 — API 없음 |
| Phase 2 | AUTH-001~010 |
| Phase 3 | PRD-001~010, IMP-001~007 |
| Phase 4 | PRC-001~007, CART-001~004, ORD-001~011, SHIP-001~005 |
| Phase 5 | RFQ-001~004, CNT-001~006, SHIP-006 |
| Phase 6+ | 모니터링 강화, 성능 최적화 |
