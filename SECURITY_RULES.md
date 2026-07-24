# SECURITY_RULES.md

이 문서는 **인증, 권한, RLS, 파일 접근, 감사정책의 Single Source of Truth(SSOT)**입니다.

> **SSOT 참조**
> - 역할·권한 → [USER_ROLES.md](file:///C:/Projects/b2b-tool-platform/USER_ROLES.md)
> - DB 스키마 → [DATABASE_SCHEMA.md](file:///C:/Projects/b2b-tool-platform/DATABASE_SCHEMA.md)
> - 가격·주문 규칙 → [BUSINESS_RULES.md](file:///C:/Projects/b2b-tool-platform/BUSINESS_RULES.md)

---

## 1. 기본 원칙

| # | 원칙 | 설명 |
|---|---|---|
| 1 | **모든 노출 테이블 RLS 활성화** | Supabase Data API에 노출되는 모든 테이블은 RLS를 활성화한다 |
| 2 | **공개 읽기도 RLS 필수** | 공개 읽기가 필요한 테이블도 RLS를 활성화하고 공개 SELECT 정책을 명시적으로 생성한다 |
| 3 | **자식 테이블 자체 RLS** | 각 자식 테이블은 자기 테이블의 RLS 정책을 가진다. 부모 RLS나 FK가 자식 테이블의 직접 조회를 보호하지 않는다 |
| 4 | **DB RLS ≠ Storage 정책** | DB 테이블 RLS와 Storage 정책을 별도로 설계한다. Storage를 DB 보안의 대체수단으로 사용하지 않는다 |
| 5 | **비노출 스키마 격리** | RLS를 사용하지 않는 테이블은 반드시 비노출 스키마(private)에 두고 anon/authenticated 권한을 제거한다 |
| 6 | **Deny by default** | 명시적으로 허용하지 않은 접근은 모두 거부 |
| 7 | **조직 간 데이터 격리** | A 거래처는 B 거래처 데이터를 절대 볼 수 없음 |
| 8 | **최소권한** | 필요한 최소한의 권한만 부여 |
| 9 | **서버 재검증** | 클라이언트 전달 가격·권한·수량을 신뢰하지 않음 |
| 10 | **service_role key 보호** | 브라우저에 노출 금지, 서버 환경변수로만 관리 |
| 11 | **MFA** | 운영사 관리자 MFA 필수 |
| 12 | **민감 데이터 최소수집** | 사업에 필요한 최소한의 개인정보만 수집 |
| 13 | **감사로그 append-only** | 기존 로그를 수정·삭제 불가 |

> [!IMPORTANT]
> 다음 가정은 폐기되었다:
> - ~~자식 테이블은 부모 RLS로 간접 보호된다~~
> - ~~ON DELETE CASCADE가 접근 격리를 제공한다~~
> - ~~공개 읽기 테이블은 RLS가 필요 없다~~
> - ~~Storage 정책이 DB 테이블을 보호한다~~

---

## 2. 인증과 MFA

### 2.1 인증 방식

- Supabase Auth 기반 이메일+비밀번호
- 세션 관리: Supabase JWT
- 비밀번호 정책: 최소 8자, 영문+숫자 조합

### 2.2 MFA 정책

| 역할 | MFA 요구 | 시점 |
|---|---|---|
| Operator Admin | **필수** | Phase 2 |
| Operator Staff (can_manage_prices, can_manage_orders) | **필수** | Phase 2 |
| Operator Staff (기타) | 권장 | Phase 4 |
| Buyer Admin | 선택 | Phase 4 |
| Approved Buyer | 선택 | — |

### 2.3 계정 보안

| 항목 | 정책 |
|---|---|
| 로그인 실패 잠금 | 10회 연속 실패 시 30분 잠금 |
| 초대 만료 | 7일 후 초대 링크 만료 |
| 비활성 세션 | 24시간 미사용 시 재인증 요구 |
| 퇴사/탈퇴 | organization_members.status → 'removed', 모든 활성 세션 무효화 |
| 조직 멤버 정지 | organization_members.status → 'suspended', 로그인 차단 |

---

## 3. 접근 매트릭스

### 3.1 주요 테이블별 접근 권한

범례: **R** = SELECT, **C** = INSERT, **U** = UPDATE, **D** = DELETE, **—** = 접근 불가, **자사** = 자기 조직 데이터만

#### 조직·사용자

| 테이블 | Guest | Pending Buyer | Approved Buyer | Buyer Admin | Op Staff | Op Admin |
|---|---|---|---|---|---|---|
| profiles | — | R(자기) | R(자기) U(자기) | R(자기) U(자기) | R U | R U C |
| organizations | — | R(자사) | R(자사) | R(자사) U(자사) | R | R C U |
| organization_members | — | — | R(자사) | R(자사) C U | R | R C U D |
| buyer_accounts | — | R(자사) | R(자사) | R(자사) | R | R C U |
| addresses | — | — | R(자사) C U D | R(자사) C U D | R | R C U |

#### 상품·가격

| 테이블 | Guest | Pending | Buyer | Buyer Admin | Op Staff | Op Admin |
|---|---|---|---|---|---|---|
| brands | R | R | R | R | R | R C U |
| products | R(공개) | R(공개) | R | R | R | R C U |
| product_skus | R(공개) | R(공개) | R | R | R | R C U |
| sku_uoms | — | — | R | R | R | R C U |
| price_books | — | — | — | — | R* | R C U |
| price_book_items | — | — | — | — | R* | R C U |
| org_price_books | — | — | R(자사) | R(자사) | R* | R C U |
| org_price_overrides | — | — | R(자사) | R(자사) | R* | R C U |
| supplier_offers | — | — | — | — | R** | R C U |

*R*: can_manage_prices 필요
**R**: can_view_purchase_cost 필요

#### 주문·출고

| 테이블 | Buyer | Buyer Admin | Op Staff | Op Admin |
|---|---|---|---|---|
| carts | R(자사) C U D | R(자사) C U D | — | R |
| order_requests | R(자사) C | R(자사) C | R* U* | R C U |
| order_request_items | R(자사) | R(자사) | R* U* | R C U |
| order_revisions | R(자사) | R(자사) | R* C* | R C |
| sales_orders | R(자사) | R(자사) | R* U* | R C U |
| shipments | R(자사) | R(자사) | R* C* U* | R C U |
| audit_logs | — | — | — | R |

*: 해당 capability 필요 (can_manage_orders, can_manage_shipments)

### 3.2 Capability 매트릭스

| Capability | Op Staff | Op Admin | 설명 |
|---|---|---|---|
| can_manage_orders | 배정 시 | ✅ | 주문 검토·확정·수정안 |
| can_manage_prices | 배정 시 | ✅ | 가격표·개별가격 관리 |
| can_manage_customers | 배정 시 | ✅ | 거래처 승인·등급 |
| can_manage_content | 배정 시 | ✅ | 프로모션·배너 |
| can_manage_shipments | 배정 시 | ✅ | 출고·송장 |
| can_manage_products | 배정 시 | ✅ | 상품·SKU·UOM |
| can_manage_suppliers | 배정 시 | ✅ | 공급조건 |
| can_view_purchase_cost | 배정 시 | ✅ | 매입가 조회 |

구매사 내부 승인 Capability (기본 비활성화):

| Capability | Buyer | Order Requester | Buyer Approver | Buyer Admin |
|---|---|---|---|---|
| can_create_order_request | ✅ | ✅ | ✅ | ✅ |
| can_submit_order_request | ✅ | ❌ | ✅ | ✅ |
| can_approve_buyer_order | — | ❌ | ✅ | ✅ |
| can_manage_buyer_approval_policy | — | ❌ | ❌ | ✅ |

---

## 4. RLS 정책 카탈로그

### 4.1 RLS 정책 이름 규칙

```
{table}_{action}_{actor}_{scope}
```

예시:
- `order_requests_select_buyer_own_org`
- `price_book_items_select_operator_staff`
- `supplier_offers_select_operator_purchase_cost`

### 4.2 핵심 테이블 RLS 정책

#### profiles

| 정책 | 액션 | 조건 |
|---|---|---|
| `profiles_select_own` | SELECT | auth.uid() = id |
| `profiles_select_org_members` | SELECT | 같은 조직 소속 |
| `profiles_update_own` | UPDATE | auth.uid() = id |
| `profiles_select_operator` | SELECT | 운영사 역할 보유 |

#### organizations / organization_business_profiles

| 정책 | 액션 | 조건 |
|---|---|---|
| `orgs_select_own` | SELECT | 소속 조직 |
| `orgs_select_operator` | SELECT | 운영사 역할 |
| `orgs_update_own_admin` | UPDATE | 자사 + admin/owner 역할 |

#### buyer_accounts

| 정책 | 액션 | 조건 |
|---|---|---|
| `buyer_accounts_select_own` | SELECT | 자사 organization_id |
| `buyer_accounts_select_operator` | SELECT | 운영사 역할 |
| `buyer_accounts_update_operator` | UPDATE | 운영사 + can_manage_customers |

#### price_book_items

| 정책 | 액션 | 조건 |
|---|---|---|
| `pbi_select_operator` | SELECT | 운영사 + can_manage_prices |
| `pbi_insert_operator` | INSERT | 운영사 + can_manage_prices |
| `pbi_update_operator` | UPDATE | 운영사 + can_manage_prices |

> 구매자는 price_book_items를 직접 조회하지 않음. 가격 조회는 **서버 함수**를 통해 결과만 반환.

#### organization_price_overrides

| 정책 | 액션 | 조건 |
|---|---|---|
| `opo_select_own_org` | SELECT | 자사 organization_id |
| `opo_select_operator` | SELECT | 운영사 + can_manage_prices |
| `opo_insert_operator` | INSERT | 운영사 + can_manage_prices |
| `opo_update_operator` | UPDATE | 운영사 + can_manage_prices |

#### supplier_offers

| 정책 | 액션 | 조건 |
|---|---|---|
| `so_select_operator_cost` | SELECT | 운영사 + can_view_purchase_cost |
| `so_select_supplier_own` | SELECT | 자사 supplier_organization_id (Stage 3) |
| `so_insert_operator` | INSERT | 운영사 + can_manage_suppliers |

#### order_requests

| 정책 | 액션 | 조건 |
|---|---|---|
| `or_select_buyer_own` | SELECT | 자사 organization_id |
| `or_select_buyer_own_visible` | SELECT | buyer_approval_status != 'pending' OR requested_by = auth.uid() |
| `or_select_operator` | SELECT | 운영사, buyer_approval_status NOT IN ('pending','rejected') |
| `or_insert_buyer` | INSERT | 자사 + approved buyer |
| `or_update_buyer_draft` | UPDATE | 자사 + status='draft' |
| `or_update_operator` | UPDATE | 운영사 + can_manage_orders |

#### sales_orders

| 정책 | 액션 | 조건 |
|---|---|---|
| `so_select_buyer_own` | SELECT | 자사 organization_id |
| `so_select_operator` | SELECT | 운영사 |
| `so_update_operator` | UPDATE | 운영사 + can_manage_orders |

#### audit_logs

| 정책 | 액션 | 조건 |
|---|---|---|
| `audit_select_admin` | SELECT | Operator Admin |
| `audit_insert_service` | INSERT | service_role only |
| audit_update | **없음** | 수정 금지 |
| audit_delete | **없음** | 삭제 금지 |

### 4.3 자식 테이블 RLS 정책

모든 자식 테이블은 부모의 organization_id를 EXISTS 서브쿼리로 확인하여 조직 격리를 보장한다.

#### cart_items

| 정책 | 액션 | 조건 |
|---|---|---|
| `cart_items_select_own_org` | SELECT | EXISTS (SELECT 1 FROM carts WHERE carts.id = cart_items.cart_id AND carts.organization_id IN (사용자 소속 조직)) |
| `cart_items_insert_own_org` | INSERT | EXISTS (SELECT 1 FROM carts WHERE carts.id = cart_items.cart_id AND carts.organization_id IN (사용자 소속 조직)) |
| `cart_items_update_own_org` | UPDATE | 동일 조건 |
| `cart_items_delete_own_org` | DELETE | 동일 조건 (장바구니 항목 삭제 허용) |

#### order_request_items

| 정책 | 액션 | 조건 |
|---|---|---|
| `ori_select_own_org` | SELECT | EXISTS (SELECT 1 FROM order_requests WHERE order_requests.id = order_request_items.order_request_id AND order_requests.organization_id IN (사용자 소속 조직)) |
| `ori_select_operator` | SELECT | 운영사 역할 + can_manage_orders |
| `ori_insert_buyer_draft` | INSERT | 자사 + order_request status='draft' |

#### order_revisions

| 정책 | 액션 | 조건 |
|---|---|---|
| `orev_select_own_org` | SELECT | EXISTS (SELECT 1 FROM order_requests WHERE order_requests.id = order_revisions.order_request_id AND order_requests.organization_id IN (사용자 소속 조직)) |
| `orev_select_operator` | SELECT | 운영사 역할 |
| `orev_insert_operator` | INSERT | 운영사 + can_manage_orders |

#### order_revision_items

| 정책 | 액션 | 조건 |
|---|---|---|
| `orevi_select_own_org` | SELECT | EXISTS via order_revisions → order_requests.organization_id |
| `orevi_select_operator` | SELECT | 운영사 역할 |
| `orevi_insert_operator` | INSERT | 운영사 + can_manage_orders |

#### sales_order_items

| 정책 | 액션 | 조건 |
|---|---|---|
| `soi_select_own_org` | SELECT | EXISTS (SELECT 1 FROM sales_orders WHERE sales_orders.id = sales_order_items.sales_order_id AND sales_orders.organization_id IN (사용자 소속 조직)) |
| `soi_select_operator` | SELECT | 운영사 역할 |

#### shipment_items

| 정책 | 액션 | 조건 |
|---|---|---|
| `shi_select_own_org` | SELECT | EXISTS (SELECT 1 FROM shipments JOIN sales_orders ON shipments.sales_order_id = sales_orders.id WHERE shipments.id = shipment_items.shipment_id AND sales_orders.organization_id IN (사용자 소속 조직)) |
| `shi_select_operator` | SELECT | 운영사 역할 |
| `shi_insert_operator` | INSERT | 운영사 + can_manage_shipments |

#### order_request_status_history

| 정책 | 액션 | 조건 |
|---|---|---|
| `orsh_select_own_org` | SELECT | EXISTS via order_requests.organization_id |
| `orsh_select_operator` | SELECT | 운영사 역할 |
| `orsh_insert_service` | INSERT | service_role only |
| UPDATE/DELETE | **없음** | append-only |

#### sales_order_status_history

| 정책 | 액션 | 조건 |
|---|---|---|
| `sosh_select_own_org` | SELECT | EXISTS via sales_orders.organization_id |
| `sosh_select_operator` | SELECT | 운영사 역할 |
| `sosh_insert_service` | INSERT | service_role only |
| UPDATE/DELETE | **없음** | append-only |

#### shipment_status_history

| 정책 | 액션 | 조건 |
|---|---|---|
| `shsh_select_own_org` | SELECT | EXISTS via shipments → sales_orders.organization_id |
| `shsh_select_operator` | SELECT | 운영사 역할 |
| `shsh_insert_service` | INSERT | service_role only |
| UPDATE/DELETE | **없음** | append-only |

### 4.4 공개 상품 테이블 RLS 정책

다음 테이블은 RLS를 활성화하고 공개 SELECT 정책을 생성한다.
내부 전용 데이터(비공개 상품, 관리자 메모, 비활성 항목)는 공개 SELECT에서 제외한다.

| 테이블 | 공개 SELECT 조건 | INSERT | UPDATE | DELETE |
|---|---|---|---|---|
| brands | status = 'active' | Op + can_manage_products | Op + can_manage_products | ❌ (status 비활성화) |
| categories | status = 'active' | Op + can_manage_products | Op + can_manage_products | ❌ |
| products | status = 'active' | Op + can_manage_products | Op + can_manage_products | ❌ |
| product_skus | status = 'active' AND product.status = 'active' | Op + can_manage_products | Op + can_manage_products | ❌ |
| product_images | product.status = 'active' AND permission_status = 'approved' | Op + can_manage_products | Op + can_manage_products | ❌ |
| product_documents | product.status = 'active' AND permission_status = 'approved' | Op + can_manage_products | Op + can_manage_products | ❌ |
| sku_barcodes | sku.status = 'active' | Op + can_manage_products | Op + can_manage_products | ❌ |
| product_search_terms | sku.status = 'active' | Op + can_manage_products | Op + can_manage_products | ❌ |
| search_synonyms | status = 'active' | Op + can_manage_products | Op + can_manage_products | ❌ |
| sku_uoms | sku.status = 'active' AND active = true | Op + can_manage_products | Op + can_manage_products | ❌ |

> [!NOTE]
> 비공개 상품 자료, 관리자 메모, draft 상태 상품은 공개 SELECT에 포함하지 않는다.
> Storage 정책은 DB RLS와 별도로 설계한다.

---

## 5. 특별 보안 시나리오

| # | 시나리오 | 보장 방법 |
|---|---|---|
| SEC-01 | 구매사 내부 승인 중 draft → 운영사 비공개 | RLS: `or_select_operator`에서 buyer_approval_status 필터 |
| SEC-02 | A 거래처 → B 거래처 주문 조회 불가 | RLS: organization_id 격리 |
| SEC-03 | A 거래처 → B 거래처 가격 조회 불가 | RLS: org_price_overrides/org_price_books organization_id 격리 |
| SEC-04 | 공급업체 A → B 공급가 조회 불가 | RLS: supplier_offers supplier_organization_id 격리 |
| SEC-05 | 승인대기 회원 → 가격 조회 불가 | 서버 함수에서 buyer_accounts.approval_status 확인 |
| SEC-06 | 권한 없는 직원 → 매입가 조회 불가 | RLS: can_view_purchase_cost 검증 |
| SEC-07 | 거래처 개별가격 → 해당 거래처+운영사만 접근 | RLS + 서버 검증 |
| SEC-08 | 주문 확정가격 → 클라이언트 조작 불가 | 서버 재계산, 스냅샷 불변, RLS로 직접 UPDATE 차단 |

---

## 6. 서버 측 권한검증

RLS만으로 모든 업무규칙을 대신하지 않음. 다음은 **서버 함수/Edge Function**에서 재검증:

| 작업 | 검증 내용 |
|---|---|
| 거래처 승인 | approval_status 전이 유효성, 관리자 권한 |
| 가격 계산 | 4단계 우선순위, 수량구간, UOM |
| UOM 변환 | conversion_to_base 정확성 |
| MOQ 검증 | minimum_order_quantity, order_increment |
| 수량구간 가격 | min_quantity/max_quantity 경계 |
| 주문 확정 | 가격 재계산, 수량 불변식, 스냅샷 생성 |
| 수정안 승인 | expires_at 확인, 상태 전이 |
| 출고 수량 | shipped + cancelled ≤ accepted |
| Import 승인 | 재검증 (target_data_version 확인) |
| 매입가 접근 | can_view_purchase_cost |
| 관리자 capability | organization_members.permissions 확인 |

---

## 7. View와 Database Function 보안

### 7.1 설계 원칙

| 원칙 | 설명 |
|---|---|
| **SECURITY INVOKER** | View와 Function은 기본적으로 호출자 권한으로 실행 |
| **SECURITY DEFINER 제한** | 가격 계산 등 특정 서비스 함수에만 사용, 반드시 고정 search_path |
| **고정 search_path** | `SET search_path = public, pg_temp` |
| **공개 실행권한 제거** | `REVOKE EXECUTE ON ALL FUNCTIONS FROM public` |
| **함수별 권한** | 필요한 역할에만 GRANT EXECUTE |
| **RLS 우회 불가** | View가 RLS를 우회하지 않도록 SECURITY INVOKER 사용 |

### 7.2 SECURITY DEFINER 사용 조건

SECURITY DEFINER를 사용하려면:
1. 내부 서비스 로직에만 사용 (클라이언트 직접 호출 금지)
2. 입력 파라미터 유효성 검증 필수
3. 고정 search_path 필수
4. 감사로그 기록
5. 코드 리뷰 필수

---

## 8. Storage 보안

### 8.1 버킷 전략

#### `public-product-assets`

| 항목 | 정책 |
|---|---|
| 용도 | 공개 승인된 상품 이미지, 브랜드 로고 |
| 업로드 | 운영사 + can_manage_products |
| 조회 | 공개 (CDN 캐시 가능) |
| signed URL | 불필요 (공개) |
| 허용 확장자 | .jpg, .jpeg, .png, .webp, .svg |
| MIME 검증 | ✅ |
| 최대 크기 | 5MB |
| 악성 파일 | 이미지 재처리(리사이징)로 스크립트 제거 |
| 삭제 정책 | 상품 비활성화 시 보존, 수동 정리 |

#### `private-business-documents`

| 항목 | 정책 |
|---|---|
| 용도 | 사업자등록증, 계약서, 개인정보 문서 |
| 업로드 | 해당 조직 멤버 또는 운영사 |
| 조회 | 해당 조직 멤버 + 운영사 관리자 |
| signed URL | ✅ 필수 |
| URL 만료 | 10분 |
| 허용 확장자 | .pdf, .jpg, .jpeg, .png |
| MIME 검증 | ✅ |
| 최대 크기 | 10MB |
| 다운로드 감사로그 | ✅ |
| 보관기간 | 거래 종료 후 5년 또는 법적 의무 기간 |
| 삭제 | 보관기간 만료 후 수동 삭제 |

#### `private-rfq-attachments`

| 항목 | 정책 |
|---|---|
| 용도 | RFQ 사진, 도면, 기술 문서 |
| 업로드 | 요청 거래처 + 운영사 |
| 조회 | 요청 거래처 + 운영사 |
| signed URL | ✅ 필수 |
| URL 만료 | 30분 |
| 허용 확장자 | .jpg, .jpeg, .png, .pdf, .dwg, .dxf |
| 최대 크기 | 20MB |
| 악성 파일 | 확장자+MIME 검증, 업로드 후 안티바이러스(향후) |
| 삭제 정책 | RFQ 종료 후 1년 보존 |

---

## 9. 감사로그

### 9.1 감사 대상

| 대상 | 기록 시점 |
|---|---|
| 회원 승인/거절 | buyer_accounts.approval_status 변경 |
| 조직 역할 변경 | organization_roles 변경 |
| capability 변경 | organization_members.permissions 변경 |
| 가격표 변경 | price_book_items 생성/수정 |
| 개별가격 변경 | organization_price_overrides 변경 |
| 공급가 변경 | supplier_offers.purchase_price 변경 |
| 주문 상태 변경 | order_request_status, order_status 변경 |
| 주문 확정 | Sales Order 생성 |
| 취소 승인/거부 | cancel_requested 처리 |
| 출고 생성/변경 | shipments 생성, 상태 변경 |
| Import 승인/반영 | catalog_imports 승인, 반영 |
| 민감 문서 다운로드 | private-business-documents 접근 |

### 9.2 로그 필드

DATABASE_SCHEMA.md `audit_logs` 테이블 참조.

| 필드 | 설명 |
|---|---|
| actor_user_id | 행위자 |
| actor_organization_id | 행위자 소속 조직 |
| action | 행위 유형 |
| target_type | 대상 테이블명 |
| target_id | 대상 레코드 ID |
| before_data | 변경 전 (민감 필드 마스킹) |
| after_data | 변경 후 |
| reason | 변경 사유 |
| request_id | 요청 추적 ID |
| ip_address | 보안 감사용 |

### 9.3 민감 데이터 로그 원칙

- 비밀번호, 토큰은 로그에 기록하지 않음
- 사업자등록번호, 대표자명은 마스킹 또는 참조만 기록
- before_data/after_data에 전체 레코드를 복사하지 않음 — 변경된 필드만 기록
- 개인정보 보관기간 준수

---

## 10. 개인정보 보호

### 10.1 최소수집 원칙

| 데이터 | 수집 근거 | 보관 기간 |
|---|---|---|
| 이메일 | 인증, 알림 | 계정 활성 기간 |
| 이름, 연락처 | 주문·배송 | 마지막 거래 후 5년 |
| 사업자등록번호 | 거래처 확인 | 거래 종료 후 5년 |
| 사업자등록증 사본 | 거래처 승인 | 승인 후 5년 |
| 주소 | 배송 | 계정 활성 기간 |
| 검색어 | 서비스 개선 | 90일 후 익명화 |

### 10.2 데이터 삭제

- 탈퇴 시: profiles.status → 'deactivated', 개인정보 마스킹 (완전 삭제는 법적 의무 보관 종료 후)
- 주문·출고 기록은 법적 보관 의무에 따라 보존
- 감사로그는 삭제하지 않음

---

## 11. GRANT/REVOKE 매트릭스

### 11.1 테이블 권한

| 테이블 분류 | anon | authenticated | service_role | 비고 |
|---|---|---|---|---|
| **public_read** (brands, categories 등 10개) | SELECT | SELECT | ALL | RLS로 active 필터 |
| **authenticated_direct** (profiles, orders 등 20개) | ❌ REVOKE ALL | SELECT, INSERT, UPDATE | ALL | RLS로 조직 격리 |
| **operator_direct** (price_books, imports 등 8개) | ❌ REVOKE ALL | SELECT (RLS 필터) | ALL | 운영사 RLS 정책 |
| **server_only** (audit_logs, status_history 등 5개) | ❌ REVOKE ALL | ❌ REVOKE ALL | ALL | 서버 함수만 접근 |
| **view_only** (sku_search_index) | SELECT | SELECT | ALL | SECURITY INVOKER |

### 11.2 명시적 DELETE 권한

| 테이블 | anon DELETE | authenticated DELETE | 비고 |
|---|---|---|---|
| 모든 테이블 (기본) | ❌ REVOKE | ❌ REVOKE | soft delete 정책 |
| cart_items | ❌ | ✅ (자사 장바구니만, RLS) | 장바구니 항목 삭제 허용 |
| addresses | ❌ | ✅ (자사만, RLS) | 배송지 삭제 허용 |

### 11.3 append-only 테이블

| 테이블 | SELECT | INSERT | UPDATE | DELETE |
|---|---|---|---|---|
| audit_logs | Op Admin (RLS) | service_role only | ❌ 모두 금지 | ❌ 모두 금지 |
| order_request_status_history | 자사 + 운영사 (RLS) | service_role only | ❌ 모두 금지 | ❌ 모두 금지 |
| sales_order_status_history | 자사 + 운영사 (RLS) | service_role only | ❌ 모두 금지 | ❌ 모두 금지 |
| shipment_status_history | 자사 + 운영사 (RLS) | service_role only | ❌ 모두 금지 | ❌ 모두 금지 |

### 11.4 Function 실행 권한

```sql
-- 기본: 모든 함수의 public 실행권한 제거
REVOKE EXECUTE ON ALL FUNCTIONS IN SCHEMA public FROM public;
REVOKE EXECUTE ON ALL FUNCTIONS IN SCHEMA public FROM anon;
REVOKE EXECUTE ON ALL FUNCTIONS IN SCHEMA public FROM authenticated;

-- 각 함수별로 필요한 역할에만 GRANT
GRANT EXECUTE ON FUNCTION fn_name TO authenticated;
```

---

## 12. Database Function 보안 목록 (예상)

아직 함수를 구현하지 않지만, 고위험 기능의 예상 Function/RPC 목록:

| # | Function 이름 (예상) | 기능 | security_mode | 호출 가능 역할 | search_path | RLS 우회 | EXECUTE 권한 | 관련 테스트 |
|---|---|---|---|---|---|---|---|---|
| F-01 | `fn_calculate_price` | 거래처별 가격 계산 (4단계 우선순위) | SECURITY INVOKER | authenticated | `SET search_path = public, pg_temp` | ❌ | authenticated | TC-PRC-001~034, TC-RLS-012 |
| F-02 | `fn_submit_order_request` | 주문 요청 제출 (가격 재계산, 불변식 검증) | SECURITY INVOKER | authenticated | 고정 | ❌ | authenticated | TC-ORD-001, TC-RLS-013 |
| F-03 | `fn_approve_revision` | 수정안 승인 (만료 확인, 상태 전이) | SECURITY INVOKER | authenticated | 고정 | ❌ | authenticated | TC-ORD-010~013 |
| F-04 | `fn_create_sales_order` | 확정 주문 생성 (스냅샷, 이중 확정 차단) | SECURITY DEFINER | service_role (Server Action에서 호출) | `SET search_path = public, pg_temp` | ✅ (트랜잭션 내) | service_role | TC-ORD-023, TC-ORD-050~053, TC-RLS-013 |
| F-05 | `fn_record_shipment` | 출고 수량 반영 (shipped ≤ accepted 검증) | SECURITY DEFINER | service_role | 고정 | ✅ | service_role | TC-ORD-040~043, TC-SHP-003 |
| F-06 | `fn_apply_import` | Import Apply (STRICT_ATOMIC 트랜잭션) | SECURITY DEFINER | service_role | 고정 | ✅ | service_role | TC-IMP-001~012 |
| F-07 | `fn_update_prices` | 관리자 가격 변경 (감사로그 기록) | SECURITY INVOKER | authenticated (Op + can_manage_prices) | 고정 | ❌ | authenticated | TC-PRC-030~034, TC-ADM-003 |

### SECURITY DEFINER 사용 조건

| 조건 | 설명 |
|---|---|
| 내부 서비스 로직만 | 클라이언트 직접 호출 금지 (Server Action/Edge Function 경유) |
| 입력 검증 필수 | 모든 파라미터 유효성 검증 |
| 고정 search_path | `SET search_path = public, pg_temp` |
| 감사로그 기록 | audit_logs에 변경 기록 |
| 코드 리뷰 필수 | PR 리뷰 없이 배포 금지 |

---

## 관련 문서

| 문서 | 참조 내용 |
|---|---|
| DATABASE_SCHEMA.md | 테이블 구조, 삭제정책 |
| USER_ROLES.md | 역할 정의, Capability |
| BUSINESS_RULES.md | 가격·주문·출고 규칙 |
| DATA_IMPORT_SPEC.md | Import 보안 |
| OPEN_DECISIONS.md | OD-005~007 VAT 정책 미결정 |
