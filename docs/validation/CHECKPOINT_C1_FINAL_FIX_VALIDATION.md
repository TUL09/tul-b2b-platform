# Checkpoint C1 Final Fix Validation

- 검증일: 2026-08-19
- 접근성 미세 보정 재검증일: 2026-08-20
- Branch: `docs/checkpoint-c1-final-fixes`
- Base: `f4950833d34e3fd25f95a4e9deba412da7965882`
- 범위: 문서·SVG 정적 검증. 애플리케이션 코드, DB Migration, Hosted DB 변경 없음.

## Automated Results

| 검증 항목 | 결과 | 증거 |
|---|---|---|
| SVG XML parse | PASS | WF-01~08, 8/8 |
| Markdown local links | PASS | 깨진 로컬 링크 0 |
| `file:///` 절대 링크 | PASS | 0 |
| 저장소·Worktree 절대경로 링크 | PASS | 0 |
| 중복 Requirement 정의 ID | PASS | DEC, OD, TC, C1-AUD 정의 중복 0 |
| 중복 Demo key 정의 | PASS | 0 |
| DEMO 표시 | PASS | SVG 8/8 |
| Wireframe Emoji | PASS | 지정 Emoji/Pictograph 0 |
| 구매자 `B등급 가격표` | PASS | WF-01~05·08에서 0 |
| 구매자 `적용 가격표` | PASS | WF-01~05·08에서 0 |
| UUID `brand_id` 입력 요구 | PASS | SVG 0 |
| Apply 전 `전체 롤백` | PASS | WF-07 0, `전체 미반영` 사용 |
| 고정 VAT 표현 | PASS | 구매자 SVG의 `VAT 별도/포함`, 고정 절사·반올림 0 |
| WF-07 Disabled Action 의미 | PASS | `role="button"` 2, `aria-disabled="true"` 2, `aria-label` 2, 상태문구 `aria-describedby` 2 |
| 전체 `aria-disabled="true"` | PASS | WF-02 1, WF-03 1, WF-07 2, 합계 4 |
| `git diff --check` | PASS | whitespace error 0 |

중복 ID는 정의 SSOT별로 검사했다. 다른 문서에서 ID를 참조하는 것은 중복 정의로 계산하지 않았다. Emoji 검사는 `docs/wireframes/*.svg`의 OS·Font 의존 pictograph를 대상으로 한다.

## Cross-Screen Contracts

### N-150

| 항목 | Canonical DEMO 값 | 결과 |
|---|---|---|
| base UOM | EA | PASS |
| MOQ / increment | 1 / 1 | PASS |
| tier 1 | 1~9 / ₩12,800 | PASS |
| tier 2 | 10~49 / ₩11,500 | PASS |
| tier 3 | 50+ / ₩10,200 | PASS |
| WF-02·04 수량 10 | tier 2 / ₩11,500 | PASS |
| WF-03·06·08 수량 20 | tier 2 / ₩11,500 | PASS |

### Order 0847

| 품목 | 수량 × 단가 | 소계 |
|---|---|---:|
| N-150 | 20 EA × ₩11,500 | ₩230,000 |
| LB-10 | 10 PACK × ₩9,000 | ₩90,000 |
| DS-PH2 | 50 EA × ₩5,800 | ₩290,000 |
| 주문 전체 수정 후 소계 | 세 품목 합계 | ₩610,000 |

- 변경 품목 수정 전 소계: ₩530,000
- 변경 품목 수정 후 소계: ₩520,000
- 주문 전체 수정 전 소계: ₩620,000
- 수정안 차액: -₩10,000
- WF-01·06·08의 품목 수와 금액 의미 일치: PASS

### DS-PH2

`accepted 50 = immediate 30 + remaining later shipment 20`으로 표현한다. `backordered 20`은 accepted 50 안에 포함되며 추가 수량이 아니다. `BUSINESS_RULES.md` §6 불변식과 WF-06·08 표현 일치: PASS.

### VAT and OD-024

- VAT: `DEMO VAT 예시 · 정책 확정 전`, 최종 세액은 확정 정책에 따라 계산. 고정 세율·절사·반올림을 만들지 않음.
- OD-024: WF-02·04 모두 `입고예정 · 관리자 확인 후 주문접수` 사용, 2/2 PASS.

## Content Governance

| 계약 | 결과 |
|---|---|
| Claim 최소 필드 12개 | PASS |
| verification status 5개 | PASS |
| Publication Gate 6조건 | PASS |
| 검증되지 않거나 만료된 Claim의 Public·Buyer 차단 | PASS |
| 동일 Content Record·Claim gate·CTA·status mapping·visibility rule | PASS |
| Surface·viewport variant가 공개 규칙을 덮어쓰지 않음 | PASS |
| DB 논리 구조·TC-CNT-001~006·Phase 5 연결 | PASS |
| C1-AUD-001~008 Traceability | 8/8 PASS |

## Accessibility Evidence

WCAG 2.x 상대휘도 공식으로 실제 토큰 조합을 계산했다.

| 전경 / 배경 | 대비 | 판정 | 실제 용도 |
|---|---:|---|---|
| `#FFFFFF` / `#C2410C` | 5.18:1 | AA PASS | 주요 주황 CTA |
| `#FFFFFF` / `#0F2044` | 16.03:1 | AA PASS | navy CTA·Header |
| `#FFFFFF` / `#B45309` | 5.02:1 | AA PASS | 경고 Action |
| `#FFFFFF` / `#DC2626` | 4.83:1 | AA PASS | Danger Action |
| `#4B5563` / `#FFFFFF` | 7.56:1 | AA PASS | 중요 Metadata |
| `#4B5563` / `#F4F5F7` | 6.93:1 | AA PASS | 중요 Metadata |
| `#D1D5DB` / `#1E2A38` | 9.87:1 | AA PASS | Dark surface 보조 텍스트 |

- 흰색 텍스트가 있는 `#E8610A`, `#D97706` 배경: 0
- 밝은 Surface의 중요 Metadata에 `#9CA3AF`: 0 (`fill="#9CA3AF"` 자체 0)
- 활성 Tab: 채움 영역 + 굵기 + `aria-current="page"` 사용
- WF-07 Dry Run: `role="button"`, `aria-disabled="true"`, `aria-label="Dry Run, 현재 비활성화됨"` 적용. `aria-describedby`로 "오류 해결 후 활성화" 상태문구와 연결
- WF-07 최종 반영: `role="button"`, `aria-disabled="true"`, `aria-label="최종 반영, 현재 비활성화됨"` 적용. `aria-describedby`로 "Dry Run 후 활성화" 상태문구와 연결
- 두 WF-07 비활성 Action에는 `tabindex="0"` 및 실행 Event가 없으며 활성 Tab 순서에 추가되지 않음
- 구현 Handoff: native HTML `button`과 `disabled`를 우선하고, `aria-disabled` 사용 시 Pointer·Keyboard Event와 Server Command에서도 실행을 차단
- 입력 오류: 인접 오류 문구 + `aria-invalid` + `aria-describedby` 연결
- CTA: 목적이 드러나는 동사형 문구 사용

## Manual Review

- 8개 SVG의 viewBox·기준 크기 유지
- Chrome headless에서 각 SVG를 기준 크기(390px 또는 1440px)로 8/8 렌더링하고 핵심 문구 잘림·겹침 없음 확인
- 공개 Claim 중 근거 없는 제조경력·인증 문구 제거
- 관리자 전용 가격등급은 WF-06에만 유지하고 구매자 화면에는 미노출
- C2 API·배포 문서 또는 기존 C2 작업트리 변경 없음
