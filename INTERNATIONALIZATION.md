# INTERNATIONALIZATION.md

이 문서는 **국제화(i18n) 구조의 SSOT**입니다. 미래 확장에 대비하되 국내 MVP를 복잡하게 만들지 않습니다.

> **SSOT 참조**
> - DB 스키마 → [DATABASE_SCHEMA.md](DATABASE_SCHEMA.md)
> - 가격·통화 → [BUSINESS_RULES.md](BUSINESS_RULES.md)
> - 해외 확장 → [ROADMAP.md](ROADMAP.md) Phase 12

---

## 1. MVP 기본값

| 항목 | 값 | 비고 |
|---|---|---|
| default_locale | `ko-KR` | 한국어 |
| default_currency | `KRW` | 대한민국 원 |
| display_timezone | `Asia/Seoul` | UI 표시 |
| storage_timezone | `UTC` | DB 저장 |
| default_country | `KR` | |
| 가격 표시 | BUSINESS_RULES.md 참조 | VAT 정책 미결정 (OD-005~007) |

MVP에서는 **한국어 단일 언어**로 운영합니다. 다국어 기능은 Phase 12에서 구현합니다.

---

## 2. 번역 가능한 데이터 vs 언어중립 데이터

### 2.1 번역 대상 (Phase 12에서 다국어 지원)

| 데이터 | 현재 저장 위치 | 향후 번역 테이블 |
|---|---|---|
| 상품명 | products.name_ko | product_translations |
| 상품 설명 | products.description | product_translations |
| 브랜드 소개 | brands.description | brand_translations (Future) |
| 카테고리명 | categories.name_ko | category_translations |
| 검색 키워드 | product_search_terms | locale별 검색어 |
| UI 텍스트 | 코드 내 i18n 파일 | 다국어 리소스 번들 |
| 기술 콘텐츠 | product_documents | locale별 문서 |

### 2.2 언어중립 데이터 (번역 불필요)

| 데이터 | 설명 |
|---|---|
| internal_sku_code | SKU 코드 — 언어와 무관 |
| model_name | 모델명 — 원본 그대로 사용 |
| barcode | 바코드 — 국제 표준 |
| 수량·수치 | 숫자 — 포맷팅만 locale별 |
| uom_code | 단위 코드 (EA, PACK) — 표시명만 번역 |
| HS Code | 국제 관세 코드 |
| origin_country_code | ISO 국가코드 |
| 중량·치수 | 수치 (g, mm) |
| 인증서 식별자 | KS, ISO 등 |
| currency_code | ISO 4217 통화코드 |

---

## 3. 번역 구조

### 3.1 product_translations (Future Extension)

Phase 12에서 구현 예정. DATABASE_SCHEMA.md Future Extension 테이블 참조.

| 컬럼 | 타입 | 설명 |
|---|---|---|
| id | UUID PK | |
| product_id | UUID FK → products | |
| sku_id | UUID FK → product_skus | SKU 특정 번역 시 |
| locale | TEXT NOT NULL | ko-KR, en-US, zh-CN |
| translated_name | TEXT | 번역된 상품명 |
| translated_description | TEXT | 번역된 설명 |
| translated_search_terms | TEXT[] | 번역된 검색어 |
| translation_status | TEXT | draft, reviewed, published |
| translated_by | UUID FK → profiles | 번역자 |
| reviewed_by | UUID FK → profiles | 검수자 |
| reviewed_at | timestamptz | 검수일시 |
| created_at | timestamptz | |
| updated_at | timestamptz | |

UNIQUE(product_id, sku_id, locale)

### 3.2 번역 품질 정책

| 원칙 | 설명 |
|---|---|
| **자동번역 비공개** | 자동번역(기계번역) 결과를 검수 없이 공식 상품정보로 공개하지 않음 |
| **검수 필수** | translation_status가 `reviewed` 또는 `published`인 번역만 노출 |
| **원본 fallback** | 번역이 없는 항목은 기본 언어(ko-KR) 원본 표시 |
| **모델명 보존** | 모델명은 번역하지 않고 원본 그대로 표시 |

---

## 4. 통화

### 4.1 통화 원칙

| 원칙 | 설명 |
|---|---|
| ISO 4217 | 통화코드는 국제 표준 사용 (KRW, USD, CNY) |
| 금액+통화 필수 | 금액을 저장할 때 통화코드를 반드시 함께 저장 |
| 데이터형 | NUMERIC — FLOAT 사용 금지 (DATABASE_SCHEMA.md 참조) |
| 환율 기록 | 외화 거래 시 환율 적용 시점을 명시적으로 기록 |
| 스냅샷 | 주문 확정 시 환율을 스냅샷으로 보존 |

### 4.2 통화별 정밀도

| 통화 | 금액 정밀도 | 표시 형식 |
|---|---|---|
| KRW | NUMERIC(19,4) | ₩1,234,567 (Application에서 소수부 0 검증) |
| USD | NUMERIC(19,4) | $1,234.5678 (단가 4자리, 합계 2자리) |
| CNY | NUMERIC(19,4) | ¥1,234.5678 |

> [!NOTE]
> 모든 통화는 동일한 `NUMERIC(19,4)` 데이터형을 사용합니다.
> KRW는 Application 레이어에서 소수부가 0인지 검증합니다.
> 정밀도 상세는 `DATABASE_SCHEMA.md` Section 1.4를 참조하세요.

### 4.3 MVP 제외 항목

국내 MVP에서는 다음을 구현하지 않음:

| 제외 항목 | Phase |
|---|---|
| 자동 환율 조회 | Phase 12 |
| 외화 결제 | Phase 12 |
| 다중 통화 가격표 | Phase 12 |
| 환율 변동 알림 | Phase 12 |

---

## 5. 단위와 국가

### 5.1 UOM 표시

- 내부 UOM 코드 (`uom_code`)는 언어중립적 유지: EA, PACK, SET, BOX 등
- 화면 표시명은 locale별 제공 가능:

| uom_code | ko-KR | en-US | zh-CN |
|---|---|---|---|
| EA | 낱개 | Each | 个 |
| PACK | 팩 | Pack | 包 |
| SET | 세트 | Set | 套 |
| BOX | 박스 | Box | 箱 |

MVP에서는 한국어 표시명만 사용. 다국어 표시명은 Phase 12에서 UOM 표시 리소스로 관리.

### 5.2 국가·원산지

| 항목 | 표준 | 예시 |
|---|---|---|
| 국가코드 | ISO 3166-1 alpha-2 | KR, JP, US, CN |
| 원산지 | product_skus.origin_country_code | KR |

### 5.3 수출 관련 데이터 (Phase 12 대비)

국내 MVP 화면에 모두 노출하지 않지만, **데이터 구조만 준비**:

| 필드 | 테이블 | 현재 상태 |
|---|---|---|
| hs_code | product_skus | ✅ 컬럼 존재 (선택) |
| weight_g | product_skus | ✅ 컬럼 존재 (선택) |
| gross_weight_g | product_skus | ✅ 컬럼 존재 (선택) |
| dimensions | product_skus | ✅ JSONB (선택) |
| origin_country_code | product_skus | ✅ 컬럼 존재 (선택) |
| export MOQ | supplier_offers | 향후 별도 필드 |
| 리드타임 | supplier_offers | ✅ lead_time_min/max_days |

---

## 6. 숫자·날짜 포맷

### 6.1 숫자 포맷

| locale | 천단위 구분 | 소수점 | 예시 |
|---|---|---|---|
| ko-KR | 쉼표 | 점 | 1,234,567 |
| en-US | 쉼표 | 점 | 1,234,567.89 |
| zh-CN | 쉼표 | 점 | 1,234,567.89 |

### 6.2 날짜·시각 포맷

| locale | 날짜 | 시각 | 예시 |
|---|---|---|---|
| ko-KR | YYYY.MM.DD 또는 YYYY년 MM월 DD일 | HH:mm | 2026.07.23 09:30 |
| en-US | MMM DD, YYYY | h:mm A | Jul 23, 2026 9:30 AM |

DB 저장: 항상 `timestamptz` (UTC). UI에서 사용자 locale에 맞춰 변환.

---

## 7. UI 텍스트 국제화 (Phase 12)

### 7.1 MVP 구조

MVP에서는 한국어 하드코딩. 그러나 다음 원칙을 준수:

| 원칙 | 설명 |
|---|---|
| UI 텍스트 분리 | 컴포넌트 코드에 직접 한국어를 넣되, 향후 i18n 추출이 용이한 구조 |
| 메시지 키 패턴 | `{page}.{section}.{element}` 형태 권장 |
| 날짜·숫자 포맷 | 라이브러리(Intl API 등) 사용 |

### 7.2 Phase 12 목표

| 지원 언어 | locale |
|---|---|
| 한국어 (기본) | ko-KR |
| 영어 | en-US |
| 중국어 간체 | zh-CN |

---

## 8. 해외 RFQ 구조 (Phase 12)

해외 바이어의 견적 요청을 위한 추가 데이터:

| 항목 | 설명 |
|---|---|
| Proforma Invoice | 수출 견적서 — RFQ 응답 시 생성 |
| HS Code | 상품별 관세 코드 |
| Incoterms | FOB, CIF, EXW 등 무역조건 |
| 수출 MOQ | 국내 MOQ와 다를 수 있음 |
| 포장 사양 | 수출용 포장 크기, 중량 |
| 통화 | USD, CNY 등 |
| 환율 스냅샷 | 견적 시점 환율 |

이 구조는 Phase 12에서 `export_rfqs` 테이블과 함께 설계. DATABASE_SCHEMA.md Future Extension 참조.

---

## 관련 문서

| 문서 | 참조 내용 |
|---|---|
| DATABASE_SCHEMA.md | product_translations, exchange_rates (Future) |
| BUSINESS_RULES.md | 금액 데이터형, 통화 정책 |
| ROADMAP.md | Phase 12 해외 RFQ와 다국어 |
| PROJECT_OVERVIEW.md | 해외 확장 방향 |
| OPEN_DECISIONS.md | OD-005~007 VAT 정책 |
