# TUL B2B 공구 유통 플랫폼 콘텐츠 시스템 (SSOT)

## 1. 콘텐츠 철학
- **구매 판단 정보 우선**: 콘텐츠는 단순한 광고가 아니라, 고객의 구매 결정을 돕는 핵심 정보로 기능해야 합니다.
- **검색·주문 흐름 방해 금지**: 사용자의 주 목적인 상품 검색과 재주문 프로세스를 최우선으로 보장합니다.
- **독립 브랜드 노출**: 브랜드 고유의 가치와 특장점이 훼손되지 않도록 독립적으로 소개합니다.
- **과도한 프로모션 배너 금지**: 화면을 가리거나 시선을 빼앗는 대형 배너, 자극적인 프로모션 문구를 지양합니다.

## 2. 콘텐츠 유형 정의

### a) 신제품 브리핑 (new_product_briefing)
- **무엇이 새로워졌는가**: 핵심 개선 사항 요약
- **기존 제품과 무엇이 다른가**: 차이점 비교
- **어떤 작업에 적합한가**: 추천 적용 분야
- **핵심 특징 3개**: 가장 중요한 기술적 특장점
- **규격**: 상세 스펙
- **가격**: 거래처 적용가격 (승인 거래처만 노출)
- **UOM (Unit of Measure)**: 판매 단위
- **MOQ (Minimum Order Quantity)**: 최소 주문 수량
- **납기**: 예상 배송 소요 시간
- **기술자료 링크**: 매뉴얼, 도면 등 연결
- **바로 주문 CTA**: 즉시 구매 가능 버튼
- **견적 요청 CTA**: 대량 구매 또는 별도 협의용 버튼

### b) 브랜드 스포트라이트 (brand_spotlight)
- **브랜드명**: 공식 브랜드 명칭
- **브랜드 이야기**: 배경, 역사 등 스토리텔링
- **제조사 또는 수입사**: 공식 유통 경로 정보
- **전문 분야**: 주력 공구 카테고리
- **기술적 강점 3가지**: 타 브랜드 대비 우위 요소
- **대표 상품 목록**: 베스트셀러 SKU 노출
- **신제품**: 최근 출시 SKU 안내
- **카탈로그 다운로드**: 최신 브랜드 카탈로그 제공
- **A/S 정책**: 보증 및 수리 절차 안내
- **기술문의 연결**: 전문 상담 창구

### c) 이달의 가격 경쟁 상품 (competitive_pricing)
- **SKU 목록 + 거래처 적용가격**: 로그인한 승인 거래처 기준으로 정확한 단가 제공
- **유효기간 명시**: 해당 가격이 유지되는 명확한 기간 안내
- **할인율 강조 금지**: '-50%' 같은 자극적 표현 대신 실제 최종 가격 중심의 정보 전달

### d) 작업 목적별 기획전 (work_purpose_collection)
- **작업 종류**: 절단, 체결, 측정, 연마 등 목적 기반 분류
- **관련 상품 목록**: 해당 작업에 필요한 공구/소모품 종합
- **추천 이유**: 큐레이션 배경 및 상품 간 시너지 효과

### e) 연관상품 / 대체상품 / 소모품 / 호환상품
- **관계 유형**: alternative(대체), accessory(액세서리), consumable(소모품), compatible(호환), frequently_bundled(자주 묶어 사는 상품)
- **표시 위치**: 상품 상세 페이지 하단 영역에 노출

### f) 카탈로그 / 기술자료 / 인증서
- **파일 유형**: PDF, 이미지 등
- **저작권 확인 필요**: 라이선스 확보 여부 명시
- **다운로드 권한**: 전체 공개, 승인 거래처 한정 등 권한 설정

### g) 브랜드 이야기 (brand_story)
- **독립 브랜드 소개**: 브랜드 아이덴티티와 철학 전달 (Stage 5 미니몰과는 분리된 콘텐츠)

### h) 공지사항 (announcement)
- **운영 공지**: 서비스 정책 변경 등
- **시스템 점검**: 플랫폼 점검 일정 안내
- **배송 지연**: 명절, 기상 악화 등으로 인한 배송 이슈

## 3. 콘텐츠 상태
- `draft`: 작성 중 상태
- `review_pending`: 관리자 검토 대기
- `approved`: 승인 완료 (게시 대기)
- `scheduled`: 특정 일시에 게시 예약됨
- `published`: 현재 라이브(게시 중) 상태
- `expired`: 게시 기간 종료
- `archived`: 영구 보관용 처리

**상태 전이 규칙**:
`draft` → `review_pending` → `approved` → (`scheduled`) → `published` → `expired` → `archived`

## 4. 콘텐츠 타깃
- `all`: 전체 승인 거래처 대상
- `price_book_id`: 특정 가격등급을 부여받은 거래처 대상
- `org_type`: 공구점, 시공업체 등 특정 업종 대상
- `org_ids`: 화이트리스트 방식의 특정 거래처 목록
- `with_purchase_history`: 이전 구매 이력이 있는 활성 거래처 대상

## 5. 콘텐츠 스케줄링
- **게시 시작일 / 종료일**: 분 단위까지 정밀한 노출 기간 설정
- **우선순위**: 숫자가 높을수록 상단 또는 우선 노출
- **모바일 이미지 / PC 이미지 별도**: 해상도 및 비율(PC 1440px, 모바일 390px)에 최적화된 에셋 분리 등록
- **CTA 버튼 텍스트 + 링크**: 행동 유도를 위한 명확한 문구와 도착 URL 설정

## 6. 콘텐츠와 상품 연결
- **연결 상품 목록**: 관련 SKU IDs
- **연결 브랜드**: 관련 브랜드 ID
- **연결 카테고리**: 카테고리 트리 맵핑
- **연결 기획전**: 기획전 ID

## 7. 자산 권리 (Asset Rights)
- **owner_organization_id**: 자산의 실제 소유 조직
- **source_type**: 직접 제작 / 브랜드 제공 / 외부 구매
- **permission_status**: `pending`(대기), `confirmed`(확인됨), `expired`(만료됨)
- **allowed_channels**: `web`, `mobile`, `export` (외부 카탈로그 추출 등)
- **valid_from / valid_to**: 자산 사용 허가 기간
- **content_owner**: 콘텐츠 담당자 (작성·관리 책임자, 조직 내부 사용)
- **review_due_at**: 콘텐츠 재검토 예정일 (사실 정보가 유효한지 주기적으로 검토)

**※ 권한 정책**: 권한이 명확히 확인(`confirmed`)되지 않은 로고, 이미지, 카탈로그, 기술자료는 절대 공개하지 않습니다.

## 8. 콘텐츠 배치 위치
- **모바일 홈**: 검색창 아래 배치하되, 중요도 순서상 5~7위로 설정하여 빠른 검색/재주문 흐름을 방해하지 않음
- **PC 홈**: 우측 사이드바 또는 하단 영역에 배치하여 메인 워크스페이스 확보
- **상품 상세 하단**: 연관/대체/소모품/호환상품 자연스러운 연결
- **카테고리 상단**: 작업 목적별 기획전 노출
- **브랜드 페이지**: 브랜드 스포트라이트 및 스토리 영역

## 9. 성과 측정
- **노출 (impression)**: 뷰 카운트
- **클릭 (click)**: 상호작용
- **상품 상세 이동 (product_view)**: 상세 페이지 진입
- **장바구니 추가 (add_to_cart)**: 장바구니 전환
- **RFQ 제출 (rfq_submitted)**: 견적 요청 발생
- **주문 전환 (order_converted)**: 실제 결제/주문 완료
- **신규 SKU 구매 (new_sku_purchase)**: 해당 거래처의 첫 구매 SKU
- **기존에 구매하지 않던 브랜드 구매 (new_brand_purchase)**: 교차 판매 성과 지표

**※ 데이터 정책**: 과도한 개인(담당자) 단위의 행동 추적은 금지하며, 조직 단위의 집계(Aggregated) 지표만 활용합니다.

## 10. 관리자 콘텐츠 워크플로
`콘텐츠 작성(Draft)` → `검토(Review)` → `승인(Approve)` → `스케줄링(Schedule)` → `발행(Publish)` → `만료(Expire)` → `아카이브(Archive)`

## 11. 사실 주장 단위 계약 (Claim Verification)

콘텐츠 본문과 사실 주장을 분리한다. 제조경력, 인증, 성능, 비교우위처럼 외부 검증이 필요한 문장마다 독립 Claim Record를 두며, 콘텐츠 전체가 승인되었더라도 개별 Claim의 게시 조건을 다시 평가한다.

| 필드 | 필수 | 설명 |
|---|---|---|
| `claim_id` | 필수 | Claim의 불변 식별자 |
| `claim_text` | 필수 | 공개 후보 문장. 출처 문구를 임의로 확대 해석하지 않음 |
| `claim_type` | 필수 | `certification`, `performance`, `history`, `comparison`, `distribution`, `other` |
| `source_reference` | 필수 | 인증서, 공식 자료 또는 검증 결과의 추적 가능한 참조 |
| `verification_status` | 필수 | Claim 검증 상태 |
| `verified_by` | 검증 시 필수 | 검증 담당자 또는 기관 |
| `verified_at` | 검증 시 필수 | 검증 완료 시각 |
| `content_owner` | 필수 | 작성·갱신·철회 책임자 |
| `review_due_at` | 공개 시 필수 | 재검토 기한. 이 시각이 지나면 자동 공개 차단 |
| `allowed_surfaces` | 필수 | Claim을 허용한 Surface 목록 |
| `publication_status` | 필수 | `draft`, `review_pending`, `approved`, `published`, `withdrawn` |
| `rejection_or_expiry_reason` | 조건부 | 거절·만료·철회 사유 |

### 11.1 Verification Status

| 상태 | 의미 | Public·Buyer 노출 |
|---|---|---|
| `unverified` | 아직 근거를 검토하지 않음 | 금지 |
| `review_required` | 근거 보강 또는 재검토 필요 | 금지 |
| `verified` | 담당자가 근거와 문구를 검증함 | 게시 게이트 충족 시 허용 |
| `rejected` | 근거 부족 또는 표현 부적합 | 금지 |
| `expired` | 재검토 기한 경과 또는 근거 만료 | 금지 |

### 11.2 Publication Gate

Claim은 아래 조건을 **모두** 만족할 때만 Public·Buyer Surface에 노출한다.

1. `verification_status = verified`
2. `review_due_at`이 현재 시각보다 뒤에 있음
3. 현재 Surface가 `allowed_surfaces`에 포함됨
4. `source_reference`가 존재함
5. `content_owner`가 존재함
6. `publication_status`가 `approved` 또는 `published`임

하나라도 충족하지 않으면 Public·Buyer renderer는 해당 Claim을 생략한다. 관리자 Preview에서는 검증되지 않은 Claim을 볼 수 있지만 상태·차단 사유를 함께 표시하며, Preview 결과를 공개 화면으로 복사하지 않는다.

**근거 없이 사용 금지 표현**:
- 업계 1위, 국내 1위, 시장 점유율 1위
- 최저가, 최고 품질
- 공식 유통, 독점 공급
- 인증 획득 (인증서 번호·발급 기관 없이)
- 국내 최초
- 제조경력 ○○년 (검증된 출처 없이)

근거가 있는 경우에도 Publication Gate를 모두 통과한 뒤에만 공개합니다.

---

## 12. 공통 Content Renderer 계약

관리자 Preview와 Public·Buyer 화면은 별도 마크업으로 콘텐츠를 복제하지 않는다. 하나의 공통 renderer가 아래 입력과 판정 결과를 사용한다.

| 계약 항목 | 공통 규칙 |
|---|---|
| Content Record | 동일한 콘텐츠 레코드와 버전을 사용 |
| Claim 검증결과 | §11 Publication Gate의 동일한 판정 함수를 사용 |
| CTA 계약 | 동일한 label, target, permission, enabled 조건을 사용 |
| Status mapping | 내부 상태를 Surface별 승인된 사용자 문구로 변환 |
| Visibility rule | 인증, 조직, 권리, 기간, Claim gate를 같은 순서로 평가 |
| Variant | 데이터·검증을 바꾸지 않고 레이아웃만 변경 |

Surface variant는 `publicHero`, `publicCompact`, `buyerDiscover`, `adminPreview`를 사용하고, viewport variant는 `mobile`, `desktop`을 사용한다. Variant가 Claim 문구, 검증 상태, CTA 목적지 또는 공개 여부를 덮어쓸 수 없다. `adminPreview`만 차단된 Claim과 사유를 관리용으로 표시할 수 있다.

구현 시 `DATABASE_SCHEMA.md`의 Content 논리 구조, `TEST_PLAN.md`의 `TC-CNT-*`, `TRACEABILITY_MATRIX.md`, `ROADMAP.md` Phase 5를 함께 적용한다.

---

## 13. 렌더링 시 결합 원칙 (Rendering-time Composition)

프로모션 콘텐츠에 가격과 재고 숫자를 직접 복사 저장하지 않습니다.

| 시스템 | 책임 |
|---|---|
| **콘텐츠 시스템** | 노출할 SKU 목록과 문구만 저장 |
| **가격 시스템** | 로그인 거래처의 현재 적용 가격 실시간 조회 |
| **공급조건 시스템** | 현재 재고 상태와 납기 실시간 조회 |

렌더링 시점에 세 시스템의 데이터를 결합하여 표시합니다. 이를 통해:
- 가격 변경이 즉시 반영됩니다 (콘텐츠 수동 수정 불필요)
- 재고 소진 시 자동으로 주문 불가 상태가 표시됩니다
- 거래처별 개별 단가가 정확하게 적용됩니다

---

## 14. 금지사항 (Don'ts)
- **과도한 할인율 표현 금지**: "-50%! 지금 바로!" 등 신뢰를 떨어뜨리는 소매형 광고 문구 사용 절대 불가.
- **가격 미확인 노출 금지**: 로그인하지 않았거나 승인되지 않은 거래처에게 단가 노출 금지.
- **권한 미확인 에셋 사용 금지**: 저작권 및 사용 권한이 불분명한 이미지, 로고, 자료 업로드 금지.
- **대형 배너 금지**: 전체 화면을 차지하거나 사용자의 통제를 벗어나는 자동 슬라이드 배너 사용 금지.
- **근거 없는 주장 금지**: §11 Publication Gate를 통과하지 않은 사실 주장 게시 금지.
- **와이어프레임 DEMO 데이터**: 모든 와이어프레임/목업의 가격·수량·주문번호·브랜드 정보는 DEMO/SAMPLE임을 명시.
