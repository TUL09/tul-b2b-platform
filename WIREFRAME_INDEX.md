# WIREFRAME_INDEX.md

> **SSOT**: 이 문서는 TUL B2B 플랫폼 **Checkpoint C1 와이어프레임 인덱스**입니다.
> - 디자인 토큰 → `DESIGN_SYSTEM.md`
> - 콘텐츠 구조 → `CONTENT_SYSTEM.md`
> - 기능 범위 → `PRODUCT_REQUIREMENTS.md`

---

## 브랜치

`docs/checkpoint-c1-design-content`

## 디렉토리

```
docs/wireframes/
├── WF-01-mobile-home.svg
├── WF-02-mobile-product-list.svg
├── WF-03-mobile-quick-order.svg
├── WF-04-pc-product-list.svg
├── WF-05-pc-product-detail.svg
├── WF-06-admin-order-review.svg
├── WF-07-admin-import.svg
└── WF-08-buyer-order-status.svg
```

---

## 와이어프레임 목록

| ID | 화면 | 플랫폼 | 기준 너비 | 파일 | 주요 검증 기준 |
|---|---|---|---|---|---|
| WF-01 | 모바일 거래처 홈 | 구매자 | 390px | [WF-01-mobile-home.svg](docs/wireframes/WF-01-mobile-home.svg) | 검색 최상단, 대형배너 없음, 3탭이내 재주문 |
| WF-02 | 모바일 상품목록 | 구매자 | 390px | [WF-02-mobile-product-list.svg](docs/wireframes/WF-02-mobile-product-list.svg) | 인라인 장바구니, 가로스크롤 없음, MOQ 오류 인라인 |
| WF-03 | 모바일 빠른 주문 | 구매자 | 390px | [WF-03-mobile-quick-order.svg](docs/wireframes/WF-03-mobile-quick-order.svg) | 다행 주문, 코드 직접입력, 이전주문 불러오기 |
| WF-04 | PC 상품목록 | 구매자 | 1440px | [WF-04-pc-product-list.svg](docs/wireframes/WF-04-pc-product-list.svg) | 좌측필터, 컴팩트표, 다중선택 일괄추가 |
| WF-05 | PC 상품상세 | 구매자 | 1440px | [WF-05-pc-product-detail.svg](docs/wireframes/WF-05-pc-product-detail.svg) | UOM선택, 수량구간 현재구간 강조, 매입가 미표시 |
| WF-06 | 관리자 주문검토 | 운영사 | 1440px | [WF-06-admin-order-review.svg](docs/wireframes/WF-06-admin-order-review.svg) | 원본↔수정안 비교, 부분확정/백오더, 가격근거 |
| WF-07 | 관리자 상품 Import | 운영사 | 1440px | [WF-07-admin-import.svg](docs/wireframes/WF-07-admin-import.svg) | Blocking Error/Warning 구분, Dry Run, STRICT_ATOMIC |
| WF-08 | 구매자 주문상태 조회 | 구매자 | 390px (모바일) | [WF-08-buyer-order-status.svg](docs/wireframes/WF-08-buyer-order-status.svg) | 4개 상태 독립표시, 타임라인 이력, 수정안 CTA |

---

## WF-01: 모바일 거래처 홈 (390px)

**목적**: 로그인한 거래처 사용자의 첫 화면. 검색과 재주문이 광고보다 항상 우선.

**콘텐츠 우선순위**:
1. 🔍 통합 검색창 (최상단 고정)
2. 📦 최근 주문 상품 + 빠른 재주문 버튼
3. ⚡ 빠른 재주문 리스트 (이전 주문 수량 기억)
4. 📊 진행 중인 주문 상태 요약
5. 🆕 신제품 브리핑
6. 💰 이달의 가격 경쟁 상품
7. 🏷️ 브랜드 스포트라이트

**설계 결정**:
- 대형 자동 슬라이드 배너 없음
- 진행 중인 수정안은 주황색 강조 + 즉시 확인 CTA
- 하단 탭바: 홈/카테고리/빠른주문(강조)/주문내역/마이

---

## WF-02: 모바일 상품목록 (390px)

**목적**: 검색결과 또는 카테고리 상품목록. 상세페이지 없이 장바구니 추가.

**핵심 요소**:
- 브랜드 배지, 모델명, 규격, UOM 배지, 재고 배지
- 거래처 적용가 + MOQ 표시
- 인라인 수량 stepper (−/입력/+)
- 인라인 장바구니 버튼
- MOQ 오류: 빨간 테두리 + 인라인 경고 메시지
- 견적문의 상태: 가격 대신 "견적문의" + RFQ 버튼
- 가로 스크롤 없음

---

## WF-03: 모바일 빠른 주문 (390px)

**목적**: 상품코드/모델명 직접 입력으로 빠른 다행 주문.

**핵심 기능**:
- 이전 주문 불러오기
- 코드 입력 시 자동완성 (상품명 미리보기)
- UOM 셀렉터
- 수량 입력
- 오류 행: 빨간 테두리 (상품 없음, MOQ 위반)
- 행 추가 / 엑셀 붙여넣기
- 주문 요약 (소계, VAT, 합계)
- 주문 요청하기 CTA

---

## WF-04: PC 상품목록 (1440px)

**목적**: 데스크톱에서 고밀도 상품 탐색과 다중 장바구니 추가.

**레이아웃**:
- 좌측: 필터 패널 (브랜드, 규격, 재고, UOM)
- 우측: 컴팩트 상품 테이블
- 상단: 검색 + 정렬 + 열 설정

**컬럼**: 브랜드 · 상품명/모델명 · 규격 · UOM선택 · 단가(수량구간) · 납기 · 재고 · 수량stepper · 장바구니

**다중선택**: 체크박스로 여러 상품 선택 후 일괄 추가

---

## WF-05: PC 상품상세 (1440px)

**목적**: 상품 정보의 완전한 표시와 주문 의사결정.

**레이아웃 (2컬럼)**:
- 좌측: 상품이미지 + 썸네일 + 문서 다운로드
- 우측: 브랜드/상품명/모델명 → UOM선택 → 수량구간 가격표(현재구간 강조) → MOQ 안내 → 수량stepper → 소계 → CTA

**가격표 규칙**:
- 현재 입력 수량에 해당하는 구간을 주황색으로 강조
- 매입가 절대 미표시
- 견적문의 가능 상품은 가격 대신 RFQ 버튼

---

## WF-06: 관리자 주문검토 (1440px)

**목적**: 운영사 관리자의 주문 검토, 수정안 발행, 부분확정.

**3패널 구조**:
1. 상단: 거래처 정보 + 주문번호 + 상태
2. 중앙: 품목 테이블 (요청 vs 적용가, 부분수락/백오더 편집)
3. 하단: 원본↔수정안 비교 + 재승인 플래그 + 변경사유

**핵심 설계**:
- 가격 근거를 "B등급 가격표 · 1~9EA" 형태로 표시
- 부분확정: accepted/backordered를 행별로 개별 편집
- 수정안 발행 전 원본↔수정안 나란히 비교
- Dangerous action (거절)은 빨간 아웃라인

---

## WF-07: 관리자 상품 Import (1440px)

**목적**: 엑셀 SKU Master Import의 단계별 흐름 관리.

**8단계 Progress**:
업로드 → 컬럼매칭 → 검증 → 오류검토 → Dry Run → 승인 → 적용 → 결과

**오류 구분**:
- ⛔ Blocking Error: 빨간 배경 + ⛔ 아이콘 + 색상 + "수정 필요" 레이블 (색상만으로 구분하지 않음)
- ⚠ Warning: 주황 배경 + ⚠ 아이콘 + "진행 가능" 레이블

**STRICT_ATOMIC 정책**: Blocking Error 존재 시 Dry Run/Commit 버튼 비활성화

---

## WF-08: 구매자 주문상태 조회 (모바일 중심)

**목적**: 구매자가 모바일에서 주문 전체 상태를 한눈에 파악.

**4개 독립 상태 카드**:
| 카드 | 필드 |
|---|---|
| 주문 상태 | order_status |
| 이행 상태 | fulfillment_status |
| 결제 상태 | payment_status |
| 세금계산서 | tax_invoice_status |

→ 4개를 하나의 배지에 합치지 않음

**타임라인**: 완료(✓)/진행중(●)/대기(숫자) 3단계 구분

**수정안 CTA**: revision_pending 상태에서 주황색 강조 + 즉시 응답 버튼

---

## 검증 체크리스트

| 검증 항목 | WF-01 | WF-02 | WF-03 | WF-04 | WF-05 | WF-06 | WF-07 | WF-08 |
|---|---|---|---|---|---|---|---|---|
| TUL 브랜드 일관 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| 검색이 홍보보다 우선 | ✅ | ✅ | ✅ | ✅ | - | - | - | - |
| 3회 이내 재주문 가능 | ✅ | - | ✅ | - | - | - | - | ✅ |
| 상세 없이 장바구니 추가 | - | ✅ | ✅ | ✅ | - | - | - | - |
| 가로 스크롤 없음 | ✅ | ✅ | ✅ | N/A | N/A | N/A | N/A | ✅ |
| UOM/수량구간 이해 가능 | - | ✅ | ✅ | ✅ | ✅ | ✅ | - | - |
| MOQ 오류 명확 | - | ✅ | ✅ | ✅ | ✅ | - | - | - |
| 신제품/브랜드가 주문 방해 안함 | ✅ | - | - | - | - | - | - | - |
| 수정안 비교 가능 | - | - | - | - | - | ✅ | - | ✅ |
| Import 오류/경고 구분 | - | - | - | - | - | - | ✅ | - |
| 4개 상태 분리 | - | - | - | - | - | ✅ | - | ✅ |
| 기준 너비 준수 | 390 ✅ | 390 ✅ | 390 ✅ | 1440 ✅ | 1440 ✅ | 1440 ✅ | 1440 ✅ | 390 ✅ |

---

*최종 업데이트: 2026-08-07 · Checkpoint C1*
