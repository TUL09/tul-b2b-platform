# TUL B2B 디자인 시스템 (SSOT)

이 문서는 TUL B2B 공구 유통 플랫폼의 디자인 시스템 단일 진실 공급원(SSOT)입니다. 모든 디자인 시안 및 프론트엔드 개발은 이 문서를 기준으로 진행됩니다.

## 브랜드
- **플랫폼**: TUL
- **운영사**: BaseOn / 베이스온
- **콘셉트**: Industrial Intelligence

---

## 1. 디자인 철학

- **Industrial Intelligence**: B2B 산업재 플랫폼으로서 신뢰감, 전문성, 그리고 직관적이고 지능적인 업무 환경을 제공합니다.
- **B2B 주문 효율 우선 원칙**: 화려한 장식이나 브랜드 홍보보다 상품 검색의 정확성, 빠른 주문 및 재주문 흐름을 최우선으로 설계합니다.
- **모바일 우선**: 390px 기준의 모바일 뷰를 먼저 설계하며, 44px 이상의 충분한 터치 영역과 한눈에 들어오는 가독성을 확보합니다.
- **DEMO 데이터 명시**: 와이어프레임과 목업에 사용된 가격·상품명·수량·주문번호는 모두 DEMO/SAMPLE임을 명시합니다.

---

## 2. 색상 토큰

> 색상만으로 상태를 구분하지 않습니다. 아이콘 + 텍스트 라벨을 함께 사용해야 합니다.

### Brand Colors

| 토큰 | 값 | 용도 |
|---|---|---|
| tul-navy | #0F2044 | 주 브랜드 컬러, 헤더 배경, 핵심 버튼 |
| tul-charcoal | #1E2A38 | 사이드바, 보조 배경 |
| tul-orange-accent | #E8610A | 로고, 아이콘, 선, 포인트 강조 |
| tul-orange-action | #C2410C | 흰색 텍스트가 들어가는 CTA 버튼 (AA 대비 충족) |
| tul-orange-light | #FFF3ED | 오렌지 배경, 강조 영역 배경 |

주의: tul-orange-accent (#E8610A) 위에 흰색 텍스트는 WCAG AA 미충족. 흰 텍스트 CTA에는 tul-orange-action (#C2410C) 사용.

### Text Colors

| 토큰 | 값 | 용도 |
|---|---|---|
| text-primary | #111827 | 기본 텍스트, 제목 |
| text-secondary | #4B5563 | 보조 텍스트, 설명, 라벨 |
| text-muted | #6B7280 | 덜 중요한 보조 정보 |
| text-disabled | #9CA3AF | 비활성화·플레이스홀더·장식 전용. 중요 정보(모델명·규격·납기·상태)에 사용 금지. |
| text-tertiary | #9CA3AF | text-disabled와 동일 |
| text-inverse | #FFFFFF | 어두운 배경 위 텍스트 |
| text-price | #0F2044 | 가격 기본 표기 |
| text-price-highlight | #E8610A | 현재 구간 강조 가격 |

### State Colors

| 토큰 | 배경/아이콘 | 텍스트 전용 | 용도 |
|---|---|---|---|
| state-success | #16A34A | state-success-text: #15803D | 성공, 재고있음 |
| state-warning | #D97706 | state-warning-text: #B45309 | 경고, 입고예정 |
| state-danger | #DC2626 | state-danger-text: #B91C1C | 오류, 품절 |
| state-info | #2563EB | state-info-text: #1D4ED8 | 정보, 안내 |
| state-pending | #7C3AED | state-pending-text: #6D28D9 | 대기 |

### Border Colors

| 토큰 | 값 | 용도 |
|---|---|---|
| border-default | #D1D5DB | 카드, 구분선 |
| border-strong | #9CA3AF | 강조 테두리 |
| border-control | #6B7280 | 인터랙티브 입력 필드 (충분한 대비) |
| border-focus | #E8610A | 포커스 링 |
| border-error | #DC2626 | 오류 입력 필드 |

### WCAG AA 대비 조합표

아래 조합은 AA 충족 기준입니다. 표에 없는 조합은 별도 검증 후 사용하십시오.
"모든 색상이 AA를 충족한다"는 단정을 사용하지 않습니다.

| 전경 | 배경 | 대비비(추정) | 판정 | 용도 |
|---|---|---|---|---|
| #FFFFFF | #0F2044 navy | 16.03:1 | Pass | 헤더 텍스트 |
| #FFFFFF | #C2410C orange-action | 5.18:1 | Pass | CTA 버튼 텍스트 |
| #FFFFFF | #1E2A38 charcoal | 14.55:1 | Pass | 사이드바 텍스트 |
| #111827 | #FFFFFF | 17.74:1 | Pass | 본문 텍스트 |
| #4B5563 | #FFFFFF | 7.56:1 | Pass | 중요 메타데이터·보조 텍스트 |
| #6B7280 | #FFFFFF | 4.83:1 | Pass | 덜 중요한 보조 정보 |
| #9CA3AF | #FFFFFF | 2.54:1 | FAIL AA | 비활성·장식 전용만 허용 |
| #15803D | #F0FDF4 | 4.79:1 | Pass | 성공 텍스트 |
| #B45309 | #FFF7ED | 4.73:1 | Pass | 경고 텍스트 |
| #B91C1C | #FEF2F2 | 5.91:1 | Pass | 오류 텍스트 |
| #FFFFFF | #E8610A orange-accent | 3.42:1 | FAIL AA | 일반 텍스트 조합 금지 |

대비비는 추정값. 구현 전 https://webaim.org/resources/contrastchecker/ 등으로 재검증 필요.

---

## 3. 타이포그래피

- **기본 폰트**: Pretendard
- **Tabular Numbers**: tnum feature 필수
- **최소 폰트**: 모바일 핵심 본문·버튼 13px 미만 금지

### Size Scale

| 토큰 | 크기 | 용도 |
|---|---|---|
| xs | 12px | 단위, 부가 주석 (중요 정보 사용 금지) |
| sm | 13px | 모바일 기본 본문, 모델명, 규격, 납기 (최소 허용) |
| base | 14px | PC 기본 본문 |
| md | 15px | 버튼 텍스트 |
| lg | 16px | 모바일 주요 요소, PC 서브 타이틀 |
| xl | 18px | 섹션 타이틀 |
| 2xl | 20px | 가격 표시 |
| 3xl | 24px | 대제목 |
| 4xl | 30px | 특대 제목 |

주의: 모델명·규격·납기·상태 설명은 최소 13px + text-secondary(#4B5563) 이상. 12px + text-disabled 조합은 가독성 실패.

---

## 4. 반응형 기준

| 구분 | 범위 | 비고 |
|---|---|---|
| Mobile | 0-767px | 기준 390px |
| Tablet | 768-1023px | |
| Desktop | 1024-1279px | 관리자: 중요하지 않은 열 숨김 |
| Wide Desktop | 1280px+ | 디자인 레퍼런스 1440px |

- 구매자 화면 Max content width: 1280px
- 관리자 화면: LNB 200px 제외 나머지 유동 사용, 1440px 기준

### 1024-1279px 관리자 처리

| 항목 | 처리 |
|---|---|
| 중요하지 않은 열 | 숨김 또는 열 설정 |
| 테이블 | 필요 시 테이블 영역만 가로 스크롤 |
| 액션 바 | 하단 고정 |

---

## 5. 스페이싱 (4px 단위)

space-1:4px / space-2:8px / space-3:12px / space-4:16px / space-5:20px / space-6:24px / space-8:32px / space-10:40px / space-12:48px / space-16:64px

---

## 6. Border, Radius, Shadow

Border Radius: radius-sm:4px / radius-md:8px / radius-lg:12px / radius-xl:16px / radius-full:9999px
Shadow: shadow-sm(버튼·입력) / shadow-md(드롭다운) / shadow-lg(모달)

---

## 7. 핵심 컴포넌트

### a) Global Search Bar
- 위치: 모바일 헤더 하단 고정, PC 헤더 중앙
- 최소 높이: 48px
- 실시간 제안 드롭다운 제공

### b) Mobile Bottom Navigation
- 탭: 홈, 카테고리, 빠른주문, 주문내역, 마이
- 활성 탭: 정확히 하나만 활성 표시
- 빠른주문: 중앙 강조 버튼. 현재 활성 탭과 동일하게 보이지 않도록 구분.
- 화면별 활성 탭: WF-01=홈, WF-02=카테고리, WF-03=빠른주문, WF-08=주문내역
- 각 탭 44×44px 이상

### c) Product Compact Card (모바일)
- 권장 높이: 136~152px (기본)
- 오류 메시지 있을 때 자동 확장
- 고정 max-height 금지

### d) Tiered Pricing Table
- 현재 적용 구간: 가장 크게 강조 (색상+라벨)
- 다른 구간: 보조 정보
- 구매자 화면: "거래처 적용가" / "수량별 단가" 표현. A/B등급 가격표명 노출 금지.
- 관리자 화면: price_book_code, tier 구간 표시 허용.

### e) Stock & Lead-Time Badge
- 아이콘 + 텍스트 라벨 병기 (색상만으로 구분 금지)
- 입고예정 임시 표시: "입고예정 · 관리자 확인 후 주문접수" (OD-024 미확정 기간, 모바일/PC 동일)

### f) UOM Selector
- 선택 시 MOQ 및 가격 즉시 갱신

### g) Quantity Stepper
- 최소 터치 높이: 44px
- MOQ 미만: 오류 표시 + "N으로 맞추기" CTA (자동 변경 금지)
- 증가단위 위반: 유효한 값 안내

### h) Price Display (VAT 표시 모드)

| 모드 | 표시 예 |
|---|---|
| tax_exclusive | ₩12,800 (부가세 별도) |
| tax_inclusive | ₩14,080 (부가세 포함) |
| pending_or_quote | 견적문의 |

VAT 정책 미확정 (BUSINESS_RULES.md §4, OD-005~008).
와이어프레임 세액 숫자는 "DEMO 세금표시 예시 (tax_exclusive 예시)" 명시.
"부가세 10%" 단정 사용 금지.

### i) Order Status Display (구매자)
4개 상태 독립 카드. 한국어 라벨만 표시. 내부 코드(revision_pending 등) 직접 노출 금지.

| 상태 유형 | 한국어 라벨 예 |
|---|---|
| order_status | 작성중 / 제출됨 / 검토중 / 수정안 확인 필요 / 확정 / 거절됨 / 취소됨 |
| fulfillment_status | 미시작 / 부분 출고 / 출고됨 / 배달 완료 |
| payment_status | 결제 미발생 / 결제 예정 / 일부 납부 / 납부 완료 / 연체 |
| tax_invoice_status | 미발행 / 발행됨 / 발행 취소 |

### j) Order Line Quantity Display (BUSINESS_RULES.md §6.1 기준)

accepted_quantity는 최종 합의 수량 (백오더 포함).
backordered_quantity는 accepted_quantity 중 미출고 수량.

| 처리 방식 | UI 표시 |
|---|---|
| 전체 수락, 즉시출고 | 확정 N개 |
| 전체 수락 + 일부 백오더 | 확정 N개 · 즉시 출고 X개 · 잔여 수량 후출고 Y개 |
| 일부 수락 + 나머지 거절 | 확정 X개 · 거절 Y개 |

### k) Quick Order Row
- 오류(미발견/MOQ) 존재 시 주문 CTA 비활성: "오류 N건 해결 후 주문 요청 가능"
- MOQ 오류: "N으로 맞추기" CTA 별도 제공 (자동 변경 금지)
- 미발견: "견적 요청" 이동 제공

### l) Revision Comparison
- 일부 품목 변경 시: "변경 품목 소계" 표시 (전체 합계로 표시 금지)
- 전체 합계 표시 시: 모든 품목 포함

### m) Import Error Display
- Blocking Error: 빨간 배경 + `circle-x` 아이콘 + 텍스트 + 행번호 + "수정 필요"
- Warning: 주황 배경 + `triangle-alert` 아이콘 + "진행 가능" 텍스트 + 행번호
- 오류 필드명: brand_code / supplier_code / price_book_code (UUID 금지)
- 버튼: "최종 반영" / "반영 취소"
- Blocking Error 있으면 최종 반영 비활성

### n) Content Renderer
- `CONTENT_SYSTEM.md` §12의 공통 renderer 계약을 사용한다.
- `publicHero`, `publicCompact`, `buyerDiscover`, `adminPreview`는 동일 Content Record와 Claim gate 결과를 공유한다.
- `mobile`, `desktop` variant는 배치만 바꾸며 문구·CTA·status mapping·visibility rule을 복제하거나 재정의하지 않는다.

---

## 8. 아이콘 정책

생산 코드와 와이어프레임 모두 Emoji 아이콘을 사용하지 않는다. 와이어프레임은 Lucide icon name 또는 `[검색]`, `[장바구니]` 같은 중립 placeholder를 사용한다.

| 항목 | 정책 |
|---|---|
| 아이콘 세트 | Lucide Icons (또는 단일 SVG 세트) |
| Stroke width | 1.5px |
| 크기 | 20px(컴팩트) / 24px(기본) |
| 아이콘만 있는 버튼 | aria-label 필수 |

---

## 9. 접근성

- 키보드 탐색: 모든 주요 액션 가능
- Focus Ring: 2px solid #C2410C + 2px offset
- 명도비: 위 §2 대비 조합표 참조. 일반 단정 사용 금지.
- 색상 + 아이콘 + 텍스트 조합 (색각 이상자 배려)
- 주요 CTA: 흰색 텍스트에는 `tul-orange-action` 또는 그보다 높은 대비의 배경만 사용한다.
- 중요 메타데이터: 모델명·규격·납기·주문번호·상태설명은 밝은 배경에서 `text-secondary` 이상을 사용한다. `text-disabled` 사용 금지.
- 활성 Tab: 채움 또는 밑줄, 위치, `aria-current` 중 하나 이상의 비색상 표지를 색상과 함께 제공한다.
- Disabled CTA: 비활성 색상, 인접 상태문구, 실제 `disabled` 또는 `aria-disabled=true`를 함께 제공한다.
- 입력 오류: 오류 텍스트에 고유 ID를 부여하고 입력의 `aria-invalid=true`, `aria-describedby`로 연결한다.
- CTA 문구: "확인", "적용"만 단독 사용하지 않고 "수정안 확인", "최종 반영"처럼 목적이 분명한 동사를 사용한다.
- 44px 터치 타겟 필수 목록:

| 요소 | 최소 |
|---|---|
| 헤더 알림 버튼 | 44×44px |
| 헤더 장바구니 버튼 | 44×44px |
| 헤더 바코드 버튼 | 44×44px |
| 수량 감소 버튼 | 44×44px |
| 수량 입력 필드 | 44px 높이 |
| 수량 증가 버튼 | 44×44px |
| 목록 장바구니 버튼 | 44×44px |
| 신제품 CTA | 44px 높이 |
| 하단 탭 각 항목 | 44×44px |

- user-scalable=no 사용 금지

---

## 10. 모션 토큰

duration-fast:100ms / duration-normal:200ms / duration-slow:300ms / easing-default:cubic-bezier(0.4,0,0.2,1)
과도한 애니메이션 금지. prefers-reduced-motion 준수.

---

## 11. Z-index

base:0 / raised:10 / dropdown:100 / sticky:200 / fixed:300 / modal-backdrop:400 / modal:500 / toast:600
