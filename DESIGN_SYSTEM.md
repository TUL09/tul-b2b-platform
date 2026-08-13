# TUL B2B 디자인 시스템 (SSOT)

이 문서는 TUL B2B 공구 유통 플랫폼의 디자인 시스템 단일 진실 공급원(SSOT)입니다. 모든 디자인 시안 및 프론트엔드 개발은 이 문서를 기준으로 진행됩니다.

## 브랜드
- **플랫폼**: TUL
- **운영사**: BaseOn / 베이스온
- **콘셉트**: Industrial Intelligence
- **색상**: Deep Navy (#0F2044), Charcoal (#1E2A38), Orange (#E8610A), White (#FFFFFF), Soft Gray (#F4F5F7), Border (#D1D5DB)

---

## 1. 디자인 철학
- **Industrial Intelligence**: B2B 산업재 플랫폼으로서 신뢰감, 전문성, 그리고 직관적이고 지능적인 업무 환경을 제공합니다.
- **B2B 주문 효율 우선 원칙**: 화려한 장식이나 브랜드 홍보보다 상품 검색의 정확성, 빠른 주문 및 재주문 흐름을 최우선으로 설계합니다.
- **모바일 우선**: 390px 기준의 모바일 뷰를 먼저 설계하며, 44px 이상의 충분한 터치 영역과 한눈에 들어오는 가독성을 확보합니다.

---

## 2. 색상 토큰

모든 색상은 WCAG 2.1 AA 기준 명도비를 충족해야 합니다.

### Brand Colors
- `tul-navy`: `#0F2044` (주 브랜드 컬러, 헤더 배경, 핵심 브랜드 요소)
- `tul-charcoal`: `#1E2A38` (사이드바, 보조 배경, 강한 텍스트)
- `tul-orange`: `#E8610A` (CTA 버튼, 강조, 신제품 뱃지)
- `tul-orange-light`: `#FFF3ED` (오렌지 배경, 강조 영역 배경)

### Surface Colors
- `surface-primary`: `#FFFFFF` (기본 배경, 카드 배경)
- `surface-secondary`: `#F4F5F7` (전체 화면 배경, 구분 영역)
- `surface-tertiary`: `#EAECEF` (비활성화 배경, 3차 표면)

### Text Colors
- `text-primary`: `#111827` (기본 텍스트, 제목, 중요 정보)
- `text-secondary`: `#4B5563` (보조 텍스트, 설명, 라벨)
- `text-tertiary`: `#9CA3AF` (비활성화 텍스트, 플레이스홀더, 규격 정보)
- `text-inverse`: `#FFFFFF` (어두운 배경 위의 텍스트)
- `text-price`: `#0F2044` (가격 기본 표기)
- `text-price-highlight`: `#E8610A` (할인/강조 가격 표기)

### Border Colors
- `border-default`: `#D1D5DB` (기본 테두리, 구분선)
- `border-strong`: `#9CA3AF` (강조된 테두리, 인풋 포커스 아웃라인)

### State Colors
사용자에게 상태를 명확히 인지시키기 위한 목적. (단, 색상만으로 상태를 구분하지 않고 아이콘/텍스트 병기 필수)
- `state-success`: `#16A34A` (성공, 재고 있음, 완료)
- `state-warning`: `#D97706` (경고, 입고 예정, 보류)
- `state-danger`: `#DC2626` (오류, 품절, 실패, 삭제)
- `state-info`: `#2563EB` (정보, 주문 가능, 안내)
- `state-pending`: `#7C3AED` (대기중, 진행중)

---

## 3. 타이포그래피

- **기본 폰트**: Pretendard (한국어 및 라틴 UI)
- **Tabular Numbers**: `tnum` feature 활성화 필수 (가격, 수량 등 숫자가 나열되는 곳)

### Size Scale
모바일 가독성을 고려하여 최소 크기를 13px로 제한합니다. (단위 등 예외 11/12px)
- `xs`: 12px (규격, 부가정보)
- `sm`: 13px (모바일 기본 본문, 모델명)
- `base`: 14px (PC 기본 본문)
- `md`: 15px (버튼 텍스트)
- `lg`: 16px (모바일 주요 요소, PC 서브 타이틀)
- `xl`: 18px (섹션 타이틀)
- `2xl`: 20px (가격 표시, 주요 타이틀)
- `3xl`: 24px (대제목)
- `4xl`: 30px (특대제목)

### Line Height
- `tight`: 1.2 (제목용)
- `snug`: 1.375 (리스트, 버튼용)
- `normal`: 1.5 (기본 본문용)
- `relaxed`: 1.625 (긴 안내문용)

### Font Weight
- `regular`: 400
- `medium`: 500
- `semibold`: 600
- `bold`: 700

### 주요 사용 케이스
- **가격 표시**: `2xl`(20px) `bold` + `tnum`
- **모델명**: `sm`(13px) `semibold` + monospace 경향 폰트
- **규격**: `xs`(12px) `regular` + `text-tertiary`
- **단위 (UOM)**: 11px `regular` uppercase

---

## 4. 반응형 기준

- **Mobile**: 0-767px (디자인 기준 390px)
- **Tablet**: 768-1023px
- **Desktop**: 1024px+ (디자인 기준 1440px)
- **Max content width**: 1280px (데스크톱에서 콘텐츠가 과도하게 퍼지는 것을 방지)

### 내비게이션 구조
- **Mobile**: 화면 하단 고정 5탭 (홈, 카테고리, 빠른주문, 주문내역, 마이)
- **Desktop (구매자)**: 상단 GNB 네비게이션
- **Desktop (관리자)**: 좌측 LNB 사이드바

---

## 5. 스페이싱 (4px 단위 시스템)

일관된 여백을 위해 4의 배수를 사용합니다.
- `space-1`: 4px
- `space-2`: 8px
- `space-3`: 12px
- `space-4`: 16px
- `space-5`: 20px
- `space-6`: 24px
- `space-8`: 32px
- `space-10`: 40px
- `space-12`: 48px
- `space-16`: 64px

---

## 6. Border, Radius, Shadow

### Border Radius
- `radius-sm`: 4px (체크박스, 작은 뱃지)
- `radius-md`: 8px (버튼, 입력 폼, 모바일 카드)
- `radius-lg`: 12px (PC 카드, 다이얼로그)
- `radius-xl`: 16px (모달 창, 바텀 시트)
- `radius-full`: 9999px (원형 아바타, 둥근 뱃지)

### Border
- `border-default`: `1px solid #D1D5DB`

### Shadow
- `shadow-sm`: 미세한 깊이 (버튼, 입력창)
- `shadow-md`: 일반적인 공중에 뜬 요소 (드롭다운, 떠있는 카드)
- `shadow-lg`: 최상단 레이어 (모달, 바텀 시트)

---

## 7. 핵심 컴포넌트 상세 설명

### a) Global Search Bar
- **용도**: 상품명, 모델명, 코드 기반 빠른 검색
- **구조**: 검색 아이콘 + 입력창 + 삭제 버튼 + (음성/바코드 향후 확장)
- **위치**: 모바일(헤더 하단 고정), PC(헤더 중앙)
- **특징**: "상품명, 모델명, 상품코드 검색" 플레이스홀더, 실시간 제안 드롭다운 제공, 최소 44px 터치 높이.

### b) Mobile Bottom Navigation
- **용도**: 모바일 뷰의 주요 메뉴 이동
- **구조**: 아이콘 + 텍스트 라벨 (홈, 카테고리, 빠른주문, 주문내역, 마이)
- **특징**: '빠른주문' 탭은 `tul-navy` 컬러로 강조. Safe area 여백 확보, 배지(Badge) 알림 기능 포함.

### c) Product Compact Card (모바일 상품목록)
- **용도**: 모바일에서 좁은 화면에 많은 상품을 효과적으로 노출
- **구조**: 브랜드, 상품명, 모델명, 규격, UOM/재고 뱃지, 가격, 수량 Stepper, 장바구니 버튼.
- **특징**: 높이 최대 120px 유지. 가로 스크롤 없음. Stepper는 MOQ 기반 작동.

### d) Product Table Row (PC 상품목록)
- **용도**: PC 환경의 리스트형 상품 표시
- **구조**: 체크박스, 브랜드, 상품/모델명, 규격, UOM 선택, 단가, 수량 Stepper, 납기, 장바구니.

### e) Tiered Pricing Table
- **용도**: 수량에 따른 단가 할인 구간 표기
- **구조**: 테이블 형태 (수량구간 | 단가 | 할인율)
- **특징**: 현재 장바구니/입력 수량에 해당하는 구간 하이라이트. 모바일에서는 스크롤 없이 작은 폰트로 컴팩트하게 노출.

### f) Stock & Lead-Time Badge
- **용도**: 재고 상태 및 납기 안내
- **상태별 색상**:
  - `state-success` (녹색): 재고 있음
  - `state-info` (파랑): 주문 가능
  - `state-warning` (주황): 입고 예정
  - `tul-charcoal` (어두운 회색): 직배송
  - `state-danger` (빨강): 품절/판매중지
  - `text-secondary` (회색): 견적문의
- **접근성**: 색상에 의존하지 않고 명확한 텍스트 라벨 및 아이콘 필수.

### g) UOM Selector
- **용도**: 박스, EA, 롤 등 포장 단위 선택
- **구조**: 세그먼트 컨트롤 형태. UOM 코드 + 단위 설명
- **특징**: 선택 시 해당 단위에 맞는 MOQ 및 가격 즉시 갱신.

### h) Quantity Stepper
- **용도**: 수량 입력
- **구조**: 감소(-) 버튼 + 입력 필드 + 증가(+) 버튼
- **특징**: 최소 44px 터치 높이, MOQ 미만 입력 시 경고, 증가 단위(예: 박스당 10개) 위반 시 경고 또는 자동 반올림.

### i) Price Display
- **용도**: 단가 및 총합계 표기
- **특징**: 거래처 적용가를 크게 표시, VAT 별도 명시 필수. 수량 × 단가 = 소계 공식 직관적 노출. 가격이 없는 경우 "견적문의" 노출. **(매입가는 구매자 화면에 절대 노출 금지)**

### j) Order Status Timeline
- **용도**: 주문 진행 상태 시각화
- **구조**: 주문요청 → 검토중 → 수정안확인 → 확정 → 출고준비 → 배송중 → 완료
- **특징**: 현재 단계 강조, 각 단계별 타임스탬프 표기. 4개 상태(주문, 이행, 결제, 세금계산서)는 하나의 뱃지에 합치지 않고 독립적으로 표기.

### k) Admin Action Bar
- **용도**: 관리자/승인자의 처리 액션 영역
- **구조**: 승인/거절/수정안발행/부분확정 버튼
- **특징**: 거절 같은 위험 액션은 `destructive` 스타일 적용 및 확인 모달 띄움.

### l) Import Error Display
- **용도**: 대량 주문 엑셀 업로드 시 에러 피드백
- **Blocking Error**: 붉은 배경, ❌ 아이콘, 행 번호, 에러 사유
- **Warning**: 주황 배경, ⚠️ 아이콘, 행 번호, 경고 사유
- **특징**: 상단에 전체 요약(N건 에러, M건 경고) 표기. 색상 외 아이콘/텍스트 필수.

### m) Revision Comparison
- **용도**: 주문 수정안 원본과 비교
- **구조**: 좌우(또는 상하) 나란히 배치된 전/후 데이터
- **특징**: 변경된 필드(수량, 가격 등)에 노란색 배경(`tul-orange-light`) 강조 및 사유 표기.

### n) Modal
- **구조**: 반투명 오버레이 + 다이얼로그 (제목, 내용, CTA)
- **특징**: 모바일에서는 바텀 시트 형태로 동작. Focus Trap, ESC 키 닫기 지원. Z-index 500.

### o) Toast
- **용도**: 일회성 피드백 알림
- **특징**: 화면 상/하단 고정, 자동 소멸(3-5초), 스크린리더 인식(`aria-live="polite"`).

### p) Empty State
- **용도**: 데이터가 없는 상태 안내 (검색 결과 없음, 장바구니 빔 등)
- **구조**: 아이콘/일러스트 + 제목 + 설명 텍스트 + CTA 버튼.

### q) Loading Skeleton
- **용도**: 콘텐츠 로딩 중 레이아웃 유지
- **특징**: 실제 컴포넌트 형태와 일치하는 회색 박스, Shimmer 애니메이션 효과.

### r) MOQ Error
- **용도**: 최소 주문 수량 미달 안내
- **구조**: 붉은 테두리 인라인 폼 에러 + 오류 메시지 텍스트 + 유효 수량 제안.

---

## 8. 상태 표현 (분리 원칙)

B2B 주문의 복잡성을 관리하기 위해, 주문 상태를 하나의 뱃지에 단순 병합하지 않고 반드시 4가지 차원의 독립된 상태로 분리하여 표시합니다.

1. **주문 상태 (Order Status)**: `draft`, `submitted`, `under_review`, `revision_pending`, `accepted`, `converted`, `cancelled`
2. **이행 상태 (Fulfillment Status)**: `not_started`, `partial`, `shipped`, `delivered`
3. **결제 상태 (Payment Status)**: `not_due`, `due`, `partially_paid`, `paid`, `overdue`
4. **세금계산서 상태 (Tax Invoice Status)**: `not_issued`, `issued`, `cancelled`

**원칙**: 각 상태는 색상 뱃지 + 아이콘 + 명확한 텍스트 라벨을 조합하여 표기.

---

## 9. 가격 표시 규칙

B2B 비즈니스 특성상 가격 노출은 매우 민감한 정보입니다.
- **비로그인 방문자**: 가격 미표시, "로그인 후 확인" 유도.
- **가입/승인 대기자**: 가격 미표시.
- **정상 승인 회원**: 해당 회원사(거래처)의 할인율/단가표가 적용된 가격 표시.
- **단가 미정 상품**: "견적문의" 상태로 표시.
- **매입가(원가) 보안**: 구매자 단(Front-end)으로 데이터 자체를 내려보내지 않으며 화면에 절대 표시 금지.

---

## 10. 접근성 (Accessibility)

- **키보드 탐색**: 마우스 없이 모든 주요 액션 가능해야 함.
- **Focus Ring**: 포커스 시 `2px solid #E8610A(tul-orange)` 외곽선과 2px offset 제공.
- **명도비**: WCAG 2.1 AA 기준 명도비(텍스트 4.5:1, UI 요소 3:1) 준수.
- **정보 전달**: 색상 외에도 아이콘, 텍스트 형태, 패턴 등으로 정보를 전달하여 색각 이상자 배려.
- **터치 타겟**: 모바일 환경에서 모든 클릭 요소는 최소 **44px × 44px** 공간 확보.
- **ARIA 속성**: 스크린리더를 위한 `aria-label`, `role` 속성 정의 및 에러 메시지는 `aria-describedby`로 폼 컨트롤과 연결.
- **모바일 줌**: `<meta name="viewport">` 내 `user-scalable=no` 사용을 엄격히 금지하여 사용자 확대 권리 보장.

---

## 11. 모션 토큰 (Motion & Animation)

- `duration-fast`: 100ms (버튼 호버, 체크박스 트랜지션)
- `duration-normal`: 200ms (모달 열기, 드롭다운 펼치기)
- `duration-slow`: 300ms (페이지 전환, 바텀 시트 슬라이드)
- `easing-default`: `cubic-bezier(0.4, 0, 0.2, 1)` (자연스러운 가감속)
- **규칙**: 업무 효율을 떨어뜨리는 불필요하고 과도한 애니메이션은 금지합니다.
- **접근성**: OS 단의 `prefers-reduced-motion: reduce` 설정을 존중하여 모션을 최소화해야 합니다.

---

## 12. Z-index 체계

Z-index 경합을 방지하기 위해 정해진 단계만 사용합니다.
- `base`: 0 (일반 문서 흐름)
- `raised`: 10 (호버된 카드, 플로팅 버튼 등 작은 레이어)
- `dropdown`: 100 (검색 자동완성, 셀렉트 박스 드롭다운)
- `sticky`: 200 (고정 헤더, 모바일 하단 네비게이션)
- `fixed`: 300 (떠있는 배너, FAB)
- `modal-backdrop`: 400 (어두운 배경 오버레이)
- `modal`: 500 (모달 창, 바텀 시트)
- `toast`: 600 (토스트 알림, 가장 최상단)
