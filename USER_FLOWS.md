# User Flows

주요 업무 흐름을 Mermaid 다이어그램으로 정의합니다.

> **SSOT 참조**
> - 가격·주문·출고 상세 규칙 → [BUSINESS_RULES.md](file:///C:/Projects/b2b-tool-platform/BUSINESS_RULES.md)
> - 역할·권한 → [USER_ROLES.md](file:///C:/Projects/b2b-tool-platform/USER_ROLES.md)
> - DB 스키마 → DATABASE_SCHEMA.md
> - 미확정 사항 → [OPEN_DECISIONS.md](file:///C:/Projects/b2b-tool-platform/OPEN_DECISIONS.md)

---

## 1. 일반 구매자의 주문 요청 흐름

구매사 내부 승인이 **비활성화**된 일반 거래처의 기본 주문 흐름입니다.

```mermaid
flowchart TD
    A[로그인] --> B[상품 검색 / 카테고리 탐색]
    B --> C[상품 상세 조회]
    C --> D{UOM 선택}
    D --> E[수량 입력]
    E --> F{MOQ·증가단위 검증}
    F -- 부적합 --> E
    F -- 적합 --> G[장바구니 추가]
    G --> H{추가 상품?}
    H -- 예 --> B
    H -- 아니오 --> I[장바구니 확인]
    I --> J[배송지·결제조건 확인]
    J --> K["주문 요청 제출 (submitted)"]
    K --> L["서버: 가격 재계산 + 수량 검증"]
    L --> M["주문 요청 접수 완료"]

    style K fill:#4A90D9,color:#fff
    style L fill:#E8A838,color:#fff
```

**핵심 규칙:**
- 가격은 로그인 후에만 표시
- UOM 선택 시 해당 UOM의 MOQ와 증가단위가 자동 적용
- 수량구간 가격이 있으면 수량 변경 시 단가가 실시간 갱신
- 주문 제출 시 서버에서 가격을 재계산 (브라우저 전달값 미신뢰)
- 가격 규칙 상세 → BUSINESS_RULES.md 섹션 3 참조

---

## 2. 구매사 내부 승인이 있는 주문 흐름

구매사 내부 승인이 **활성화**된 거래처의 흐름입니다. `buyer_approval_status`와 `order_request_status`는 별도 필드입니다.

> 이 기능은 Release 1 Optional이며, Feature Flag로 활성화합니다.

```mermaid
flowchart TD
    subgraph buyer_org["구매사 내부 (운영사에 비공개)"]
        A[요청자: 주문 작성] --> B["draft 상태"]
        B --> C["내부 승인 요청<br/>buyer_approval_status: pending"]
        C --> D{내부 승인자 검토}
        D -- 승인 --> E["buyer_approval_status: approved"]
        D -- 거절 --> F["buyer_approval_status: rejected<br/>운영사에 제출되지 않음"]
        D -- 기한 만료 --> G["buyer_approval_status: expired"]
    end

    subgraph operator["운영사 영역"]
        E --> H["운영사에 submitted<br/>order_request_status: submitted"]
        H --> I["관리자 접수<br/>(under_review)"]
    end

    style buyer_org fill:#F0F4FF,stroke:#4A90D9
    style operator fill:#FFF4E6,stroke:#E8A838
```

**핵심 규칙:**
- 내부 승인 중인 `draft` 주문은 **운영사에서 볼 수 없음**
- 한 사용자가 요청자와 승인자 역할을 동시에 가질 수 있음 (소규모 거래처)
- 내부 승인 거절 시 운영사에 제출되지 않음
- 기본값: `buyer_approval_status = not_required`
- 상세 Capability → USER_ROLES.md 섹션 4 참조

---

## 3. 운영사의 주문 검토와 수정안 흐름

운영사 관리자가 주문 요청을 검토하고, 필요 시 수정안을 발행하는 흐름입니다.

```mermaid
flowchart TD
    A["주문 요청 접수<br/>under_review"] --> B{재고·가격·납기 확인}

    B -- 변경 없음 --> C["accepted"]
    C --> D["확정 주문 생성<br/>converted_to_sales_order"]

    B -- 변경 필요 --> E["수정안 작성"]
    E --> F["품목별 변경 내용 입력"]
    F --> G["응답기한 설정<br/>expires_at"]
    G --> H["수정안 발행<br/>revision_pending"]
    H --> I["구매자에게 알림"]

    I --> J{구매자 응답}
    J -- 승인 --> C
    J -- 재수정 요청 --> K["under_review로 복귀"]
    K --> B
    J -- 거절 --> L["rejected"]
    J -- 기한 만료 --> M["expired"]

    style D fill:#27AE60,color:#fff
    style L fill:#E74C3C,color:#fff
    style M fill:#95A5A6,color:#fff
```

**수정안(Revision) 내용:**

| 항목 | 테이블 | 필드 |
|---|---|---|
| 수정안 메타 | `order_revisions` | revision_number, status, reason, expires_at |
| 품목별 변경 | `order_revision_items` | proposed_quantity, proposed_unit_price, proposed_lead_time, change_reason |

- 수정안 상태: `pending` → `approved` / `rejected` / `expired` / `superseded`
- 상세 규칙 → BUSINESS_RULES.md 섹션 5.4 참조

---

## 4. 확정 주문과 부분출고 흐름

주문 요청이 확정되어 Sales Order가 생성된 후, 출고까지의 흐름입니다.

```mermaid
flowchart TD
    A["accepted → converted_to_sales_order"] --> B["Sales Order 생성<br/>스냅샷 보존"]
    B --> C["confirmed 상태"]
    C --> D["처리 시작<br/>processing"]
    D --> E{출고 방식 결정}

    E --> F1["전체 출고"]
    E --> F2["부분 출고"]
    E --> F3["공급업체 직배송"]

    F1 --> G["Shipment 생성<br/>preparing"]
    F2 --> G
    F3 --> G

    G --> H["송장번호 등록<br/>dispatched"]
    H --> I["배달 완료<br/>delivered"]

    F2 --> J{나머지 품목?}
    J -- 추가 출고 --> G
    J -- 입고 대기 --> K["backordered"]
    K --> G

    I --> L{전체 출고 완료?}
    L -- 예 --> M["fulfillment_complete"]
    M --> N["closed"]
    L -- 아니오 --> D

    style B fill:#4A90D9,color:#fff
    style N fill:#27AE60,color:#fff
```

**수량 필드 변화:**

| 단계 | requested | accepted | rejected | shipped | cancelled | open | backordered |
|---|---|---|---|---|---|---|---|
| 주문 확정 | 100 | 80 | 20 | 0 | 0 | 80 | 0 |
| 1차 출고 (50) | 100 | 80 | 20 | 50 | 0 | 30 | 0 |
| 일부 입고대기 | 100 | 80 | 20 | 50 | 0 | 30 | 10 |
| 2차 출고 (30) | 100 | 80 | 20 | 80 | 0 | 0 | 0 |

불변식: `accepted = shipped + cancelled + open`, `backordered ≤ open`

상세 수량 모델 → BUSINESS_RULES.md 섹션 6 참조

---

## 5. 확정 전 취소 흐름

구매자가 주문 확정 전에 직접 취소하는 흐름입니다. **관리자 승인이 불필요**합니다.

```mermaid
flowchart TD
    A["주문 요청 상태:<br/>draft / submitted / under_review / revision_pending"]
    A --> B{구매자가 취소 요청}
    B --> C["order_request_status → cancelled"]
    C --> D["취소 완료"]

    style C fill:#E74C3C,color:#fff
    style D fill:#95A5A6,color:#fff
```

**규칙:**
- `draft`, `submitted`, `under_review`, `revision_pending` 상태에서 취소 가능
- 즉시 취소 — 관리자 승인 불필요
- 취소 사유는 기록
- 확정 주문(`confirmed`) 이후에는 이 흐름이 아닌 **확정 후 취소 요청 흐름**을 따름

---

## 6. 확정 후 취소 요청 흐름

확정 주문(`confirmed`) 이후 구매자가 취소를 요청하는 흐름입니다. **관리자 승인이 필요**합니다.

```mermaid
flowchart TD
    A["Sales Order 상태:<br/>confirmed / processing"]
    A --> B[구매자: 취소 요청]
    B --> C["cancel_requested"]
    C --> D{관리자 검토}

    D -- 승인 --> E{출고 시작 여부 확인}
    E -- 출고 전 --> F["cancelled<br/>cancelled_quantity 반영"]
    E -- 출고 시작됨 --> G["반품·교환·클레임<br/>절차로 전환"]

    D -- 거부 --> H["cancel_rejected"]
    H --> I["processing으로 복귀"]

    style F fill:#E74C3C,color:#fff
    style G fill:#F39C12,color:#fff
    style H fill:#95A5A6,color:#fff
```

**규칙:**
- 확정 주문 이후에는 단순 취소 불가 → 관리자 승인 필요
- 출고가 시작된 이후에는 일반 취소 아님 → **반품·교환·클레임 절차로 전환**
- 반품·교환·클레임의 상세 구현은 MVP 범위 밖 (원칙만 확정)
- 상세 → BUSINESS_RULES.md 섹션 5.5, 섹션 7 참조

---

## 7. 미등록 상품 RFQ 흐름

초기 상품 수가 약 1,000 SKU이므로, 등록되지 않은 상품을 거래처가 견적 요청하는 흐름입니다.

> Customer Pilot Must-Have

```mermaid
flowchart TD
    A[구매자: 원하는 상품을 찾지 못함] --> B[RFQ 작성]
    B --> C["입력 항목:<br/>상품명, 브랜드, 모델명<br/>규격, 수량, 원하는 납기<br/>사진·문서, 요청사항"]
    C --> D["RFQ 제출"]
    D --> E["운영사 관리자 확인"]

    E --> F{공급 가능?}
    F -- 가능 --> G["공급업체에 확인"]
    G --> H["견적 응답 작성"]
    H --> I["구매자에게 전달"]
    I --> J{구매자 확인}
    J -- 주문 전환 --> K["일반 주문 흐름으로 이동"]
    J -- 거절/보류 --> L["RFQ 종료"]

    F -- 불가 --> M["불가 사유 안내"]
    M --> L

    style D fill:#4A90D9,color:#fff
    style K fill:#27AE60,color:#fff
```

**규칙:**
- 초기에는 **운영사 관리자만** RFQ에 응답
- 공급업체가 직접 RFQ에 답하는 기능은 Future Extension
- RFQ에서 반복 요청되는 상품은 정식 등록 후보로 관리

---

## 8. 거래처 승인 흐름

신규 구매 거래처가 가입하고 운영사가 승인하는 흐름입니다.

```mermaid
flowchart TD
    A[방문자: 사업자 회원가입] --> B["입력 항목:<br/>사업자등록번호, 회사명<br/>대표자, 업종, 담당자"]
    B --> C["회원가입 완료<br/>approval_status: pending"]
    C --> D["로그인 가능<br/>가격 조회·주문 불가"]

    D --> E["운영사 관리자에게 알림"]
    E --> F{관리자 검토}

    F -- 승인 --> G["approval_status: approved"]
    G --> H["가격등급 배정<br/>default_price_book 설정"]
    H --> I["거래 시작 가능"]

    F -- 추가 정보 요청 --> J["보완 요청"]
    J --> K[거래처: 정보 보완]
    K --> F

    F -- 거절 --> L["approval_status: rejected<br/>사유 안내"]

    style C fill:#F39C12,color:#fff
    style I fill:#27AE60,color:#fff
    style L fill:#E74C3C,color:#fff
```

**규칙:**
- 승인 전: 로그인 가능, 일부 상품정보 조회 가능, **가격 조회·주문 불가**
- 승인 시: 가격등급(price_book) 배정
- 가격등급이 없으면 기본 B2B 가격표 적용
- 거래처 개별가격은 별도 설정
- 조직 정보: `organization_business_profiles`(사업자정보) + `buyer_accounts`(운영정보) 분리

---

## 흐름 간 관계 요약

```mermaid
flowchart LR
    subgraph before["주문 요청 단계"]
        A1[거래처 승인] --> A2[상품 검색·장바구니]
        A2 --> A3["내부 승인<br/>(선택)"]
        A3 --> A4[주문 요청 제출]
    end

    subgraph review["검토 단계"]
        A4 --> B1[운영사 검토]
        B1 --> B2["수정안 발행<br/>(필요 시)"]
        B2 --> B3[구매자 재승인]
    end

    subgraph confirm["확정·출고 단계"]
        B1 --> C1[주문 확정]
        B3 --> C1
        C1 --> C2[출고]
        C2 --> C3[배달 완료]
    end

    style before fill:#F0F4FF,stroke:#4A90D9
    style review fill:#FFF4E6,stroke:#E8A838
    style confirm fill:#F0FFF4,stroke:#27AE60
```

---

## 관련 문서

| 문서 | 참조 내용 |
|---|---|
| BUSINESS_RULES.md | State Machine 상세, 수량 모델, 가격 규칙, 출고 규칙 |
| USER_ROLES.md | 역할별 capability, 내부 승인 Capability |
| PRODUCT_REQUIREMENTS.md | 기능 분류 (Must-Have / Optional / Future) |
| INFORMATION_ARCHITECTURE.md | 화면 구조와 페이지 매핑 |
| DATABASE_SCHEMA.md | 테이블 구조 (Checkpoint B) |
