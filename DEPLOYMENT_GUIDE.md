# DEPLOYMENT_GUIDE.md — TUL B2B 플랫폼 배포 및 운영 가이드 (SSOT)

> **SSOT 참조**
> - API 실행 경계 → API_SPEC.md
> - 개발 순서 → ROADMAP.md
> - 인증·권한·RLS → SECURITY_RULES.md
> - 미확정 사항 → OPEN_DECISIONS.md

이 문서는 환경 구성, 배포 흐름, 브랜치 전략, CI, migration,
Secret 관리, 롤백, 백업·복구, 모니터링의 SSOT입니다.

실제 운영 환경 자격증명은 이 문서에 기록하지 않습니다.

---

## 1. 환경 분리

### 1.1 환경 개요

| 환경 | 목적 | Git 이벤트 | Vercel | Supabase 프로젝트 |
|---|---|---|---|---|
| Local | 개발자 로컬 개발 | — | — | 별도 Local Supabase (CLI) |
| Preview | PR별 화면·기능 검토 | PR 오픈·업데이트 | 자동 Preview | 별도 Preview Supabase 또는 로컬 |
| Staging | 운영 전 최종 검증 | main 병합 | Staging 환경 | Staging 전용 Supabase |
| Production | 실제 거래처 서비스 | 수동 승인 | Production 환경 | Production 전용 Supabase |

### 1.2 환경별 상세 정의

#### Local

| 항목 | 내용 |
|---|---|
| 목적 | 기능 개발, 단위 테스트, DB 함수 개발 |
| Supabase | Supabase CLI로 로컬 Docker 실행 |
| 데이터 | 개발용 DEMO/SAMPLE 데이터만 사용. Production 데이터 복사 금지 |
| 환경변수 | .env.local (Git 제외) |
| Migration | supabase db reset으로 초기화 가능 |
| 접근 | 개발자 본인 PC만 |
| 외부 서비스 | 이메일: 개발용 메일 트랩(Inbucket) |

#### Preview

| 항목 | 내용 |
|---|---|
| 목적 | PR 단위 UI 검토, 기능 흐름 확인 |
| 이벤트 | GitHub PR 오픈 시 Vercel 자동 배포 |
| Supabase | 별도 Preview 프로젝트 또는 Staging 공유 (OPEN_DECISIONS 참조) |
| 데이터 | DEMO/SAMPLE 데이터. Production DB 접근 절대 금지 |
| Migration | Preview 자동 migration 금지. 수동 또는 Staging에서 검증 후만 적용 |
| 접근 | 팀 내부 검토용. URL 공개 불가 |
| 승인자 | PR 리뷰어 |

#### Staging

| 항목 | 내용 |
|---|---|
| 목적 | 운영 전 최종 통합 테스트, UAT |
| 이벤트 | main 브랜치 병합 시 자동 또는 수동 배포 |
| Supabase | Staging 전용 프로젝트 (Production과 완전 분리) |
| 데이터 | 실제 데이터 구조와 동일한 DEMO 데이터 세트. Production 데이터 복사 금지 |
| Migration | Staging에서 먼저 검증 후 Production 적용 |
| 접근 | 운영사 내부 + 신뢰하는 파일럿 거래처 |
| 승인자 | 운영사 책임자 |

#### Production

| 항목 | 내용 |
|---|---|
| 목적 | 실제 거래처 서비스 |
| 이벤트 | 수동 승인 후 Vercel Production 배포 |
| Supabase | Production 전용 프로젝트 |
| 데이터 | 실제 거래처·주문·가격 데이터 |
| Migration | Staging 검증 완료 후 사람이 직접 승인 |
| 접근 | 승인된 거래처 + 운영사 |
| 승인자 | 운영사 책임자 (사람) |

### 1.3 핵심 환경 규칙

- Production 데이터를 개발·Preview·Staging 환경에 복사하지 않습니다.
- Preview에서 Production DB에 접근하지 않습니다.
- Preview 자동 migration은 금지입니다.
- 환경별 Supabase 프로젝트를 완전히 분리합니다.
- 환경별 비밀키를 분리합니다. 환경 간 키를 공유하지 않습니다.
- Production 배포는 반드시 사람의 승인을 받습니다.

---

## 2. Secret 관리

### 2.1 Secret 분류

| Secret | 브라우저 노출 | 서버 전용 | 비고 |
|---|---|---|---|
| NEXT_PUBLIC_SUPABASE_URL | 가능 (공개) | — | anon 클라이언트 초기화 |
| NEXT_PUBLIC_SUPABASE_ANON_KEY | 가능 (공개) | — | RLS로 보호, 제한된 권한 |
| SUPABASE_SERVICE_ROLE_KEY | **절대 금지** | 서버 전용 | DB 함수 호출, RLS 우회 가능 |
| DATABASE_URL | **절대 금지** | 서버 전용 | Migration 전용 |
| RESEND_API_KEY | **절대 금지** | 서버 전용 | 이메일 발송 |
| VERCEL_TOKEN | **절대 금지** | CI/CD 전용 | 배포 |
| CLOUDFLARE_TOKEN | **절대 금지** | 서버/CI 전용 | DNS |
| ECOUNT_CREDENTIALS | **절대 금지** | 서버 전용 | Phase 7 이후 |
| 배송사 API KEY | **절대 금지** | 서버 전용 | Phase 5 이후 |
| WEBHOOK_SECRET | **절대 금지** | 서버 전용 | Webhook 서명 검증 |

### 2.2 Secret 관리 원칙

- .env.example에는 키 이름만 기록합니다. 실제 값은 절대 기록하지 않습니다.
- 비밀키를 Git에 커밋하지 않습니다.
- 환경별 Secret은 Vercel Environment Variables 또는 Supabase Vault에 저장합니다.
- service_role key를 NEXT_PUBLIC_ prefix를 가진 변수에 설정하지 않습니다.
- Secret 노출이 의심되면 즉시 교체합니다.

---

## 3. Git 브랜치 전략

### 3.1 브랜치 종류

| 브랜치 | 목적 | 병합 대상 | 규칙 |
|---|---|---|---|
| main | 항상 승인된 상태. Staging/Production 기준 | — | 직접 push 금지. PR 필수 |
| feature/* | 기능 개발 | main | 작업 단위로 생성. 완료 후 PR |
| fix/* | 버그 수정 | main | 버그 단위로 생성 |
| docs/* | 문서 작업 | main | 문서 전용 |
| release/* | 릴리즈 준비 | main | Production 배포 직전 |
| hotfix/* | 긴급 Production 수정 | main + release/* | 최소 범위. 즉시 병합 |

### 3.2 브랜치 규칙

- main은 항상 테스트를 통과한 승인 상태를 유지합니다.
- 기능별·버그별 브랜치를 생성합니다. 여러 기능을 한 브랜치에 넣지 않습니다.
- Pull Request를 통해서만 main에 병합합니다.
- 병합 전 필수 테스트 통과를 확인합니다.
- 코드 리뷰를 거쳐 승인 후 병합합니다.
- Phase 0 완료 후 승인 태그(phase0-checkpoint-*-approved)를 부착합니다.
- --no-ff 병합으로 이력을 명확히 합니다.
- main 브랜치 보호 규칙: force push 금지, 직접 push 금지.
- GitHub Private 저장소를 유지합니다.

---

## 4. CI 검증 단계

### 4.1 CI 파이프라인 단계

CI는 구현 시점에 GitHub Actions 또는 동등한 CI 서비스로 구성합니다.

| 단계 | 내용 | 실패 시 |
|---|---|---|
| 1. install | 의존성 설치 | 중단 |
| 2. lint | ESLint, Prettier 검사 | 중단 |
| 3. typecheck | TypeScript strict mode | 중단 |
| 4. unit tests | Vitest 단위 테스트 | 중단 |
| 5. database tests | DB 함수 정합성 테스트 | 중단 |
| 6. RLS security tests | RLS 정책 검증 | 중단 |
| 7. build | Next.js 프로덕션 빌드 | 중단 |
| 8. E2E smoke tests | Playwright 주요 흐름 | 중단 |
| 9. dependency audit | npm audit (High/Critical 차단) | 중단 |
| 10. secret scan | 환경변수·하드코드 Secret 탐지 | 중단 |

### 4.2 Phase별 CI 활성화 시점

| CI 단계 | Phase 1 | Phase 2 | Phase 3 | Phase 4+ |
|---|---|---|---|---|
| install, lint, typecheck | 활성 | 활성 | 활성 | 활성 |
| unit tests | 부분 | 활성 | 활성 | 활성 |
| database tests | 미활성 | 활성 | 활성 | 활성 |
| RLS security tests | 미활성 | 활성 | 활성 | 활성 |
| build | 활성 | 활성 | 활성 | 활성 |
| E2E smoke tests | 미활성 | 부분 | 활성 | 활성 |
| dependency audit | 활성 | 활성 | 활성 | 활성 |
| secret scan | 활성 | 활성 | 활성 | 활성 |

---

## 5. Database Migration

### 5.1 Migration 원칙

- Migration 파일은 supabase/migrations/ 디렉터리가 SSOT입니다.
- Migration 파일 이름: YYYYMMDDHHMMSS_descriptive_name.sql
- Migration은 순차 적용합니다. 순서를 임의로 변경하지 않습니다.
- 한 번에 전체 44개 테이블을 생성하지 않습니다. Phase별로 필요한 테이블만 추가합니다.
- 적용된 Migration을 수정하지 않습니다. Forward Fix 원칙을 따릅니다.

### 5.2 Migration Group 순서 (Phase별 권장)

| Group | 이름 | 테이블 수 | 대상 Phase | 비고 |
|---|---|---|---|---|
| MG-01 | Identity & Organizations | 7 | Phase 2 | profiles, organizations, organization_roles, organization_business_profiles, buyer_accounts, organization_members, addresses |
| MG-02 | Catalog Core | 7 | Phase 3 | brands, categories, products, product_skus, product_images, product_documents, search_synonyms |
| MG-03 | UOM & Search | 4 + 1 View | Phase 3~5 | sku_uoms, sku_barcodes, product_search_terms, search_events, sku_search_index(View) |
| MG-04 | Pricing | 5 | Phase 3~4 | price_books, price_book_items, organization_price_books, organization_price_overrides, supplier_offers |
| MG-05 | Cart & Order Request | 6 | Phase 4 | carts, cart_items, order_requests, order_request_items, order_revisions, order_revision_items |
| MG-06 | Sales Order & Shipment | 6 | Phase 4 | sales_orders, sales_order_items, shipments, shipment_items, order_request_status_history, sales_order_status_history |
| MG-07 | Import & Audit | 4 | Phase 2~5 | catalog_imports, catalog_import_rows, catalog_import_errors, audit_logs |
| MG-08 | RFQ & Pilot | 5 | Phase 4~5 | rfqs, rfq_items, rfq_attachments, shipment_status_history, tax_policies |

> DB Functions와 RLS 정책은 별도 Group이 아닌, 각 테이블 Group의 Migration 파일 내에서 해당 테이블과 함께 생성합니다.
> Migration Group ID는 DATABASE_SCHEMA.md의 MG 번호를 SSOT로 따릅니다.

### 5.3 Migration 적용 흐름

`
Local 개발 및 검증
→ Staging 적용 (수동 또는 CI)
→ UAT 확인
→ Production 승인 (사람이 직접)
→ Production 적용
→ 결과 확인 및 모니터링
`

### 5.4 Migration 실패 대응

- Migration 실패 시 즉시 중단하고 후속 단계를 진행하지 않습니다.
- 가역적 변경(컬럼 추가 등): Rollback Migration 작성 가능.
- 비가역적 변경(컬럼 삭제, 타입 변경): 별도 승인, 데이터 백업 필수, Forward Fix 원칙.
- Destructive Migration (테이블·컬럼 삭제): 운영사 책임자 서면 승인 필요.

### 5.5 Schema Drift 검증

- Staging 적용 후 supabase db diff로 Local과 Staging 스키마를 비교합니다.
- Production 적용 전 Staging과 Production 스키마를 비교합니다.
- 불일치 발견 시 원인을 파악하고 해결 후 진행합니다.

---

## 6. 배포 흐름

### 6.1 표준 배포 흐름

`
feature 브랜치 개발
→ CI 통과 (lint, typecheck, test, build)
→ Pull Request 생성
→ Preview 배포 자동 생성
→ 코드 리뷰 + 기능 검토
→ main 병합
→ Staging 자동 배포
→ Staging Migration 적용
→ UAT (사용자 수락 테스트)
→ Release 승인 결정
→ Production 배포 승인 (사람)
→ Production Migration 적용
→ Production 배포
→ 배포 후 검증 (핵심 기능 스모크 테스트)
→ 이상 없으면 완료
→ Release Tag 생성
`

### 6.2 단계별 통과조건

| 단계 | 통과조건 |
|---|---|
| CI | 모든 테스트 통과, 빌드 성공 |
| Preview | UI 흐름 확인, 주요 오류 없음 |
| Staging | 인수 시나리오 통과, 성능 기준 충족 |
| Production 승인 | 운영사 책임자 명시적 승인 |
| 배포 후 검증 | 핵심 페이지 로드, 로그인, 주문요청 흐름 정상 |

### 6.3 중단 조건

- CI 실패 → 배포 중단. 원인 수정 후 재시도.
- Staging Migration 실패 → Production 배포 금지.
- Staging 테스트 실패 → Production 배포 금지.
- Production 배포 후 오류율 급증 → 즉시 이전 버전으로 롤백.

---

## 7. Rollback과 장애 대응

### 7.1 애플리케이션 롤백

- Vercel에서 이전 성공한 배포로 즉시 롤백합니다.
- 롤백은 DB Migration과 독립적입니다. 애플리케이션 코드만 롤백됩니다.
- Migration 롤백이 필요한 경우 별도 Rollback Migration을 작성하여 적용합니다.

### 7.2 장애 유형별 대응

| 유형 | 대응 |
|---|---|
| 주문 실패 | 오류 로그와 request_id 확인. 해당 주문요청 상태 확인. 수동 처리 또는 재시도 안내 |
| 가격 계산 오류 | fn_calculate_price 로그 확인. 가격표 데이터 확인. 가격 표시 일시 중단 |
| 이메일 발송 실패 | Edge Function 로그 확인. Resend 대시보드 확인. 수동 발송 또는 재발송 |
| 외부 API 실패 | 오류 로그 확인. 재시도 정책 확인. 외부 서비스 상태 페이지 확인 |
| Storage 장애 | Supabase Storage 상태 확인. 임시 대체 경로 마련 |
| 보안 사고 의심 | 해당 계정 즉시 정지. 세션 무효화. audit_logs 즉시 확인. 필요 시 서비스 중단 |
| DB Migration 실패 | 즉시 중단. 오류 SQL 확인. Rollback Migration 작성 |

### 7.3 Read-Only 비상모드

- 주문 접수나 데이터 변경이 위험한 상황에서 서비스를 Read-Only로 전환합니다.
- 구현 방법: system_settings에 maintenance_mode 플래그를 설정합니다.
- 구매자 화면에 공지를 표시하고 주문요청 제출을 일시 차단합니다.

### 7.4 장애 공지

- 장애 발생 시 관리자 화면에 배너로 공지합니다.
- 장애 종료 후 거래처에 이메일로 안내합니다.
- 장애 이력을 CHANGELOG.md에 기록합니다.

---

## 8. 백업 및 복구

### 8.1 DB 백업

- Supabase는 기본 일별 자동 백업을 제공합니다.
- PITR(Point-in-time Recovery) 가능 여부는 Supabase 플랜에 따라 다릅니다. (OPEN_DECISIONS 참조)
- Production DB 백업은 운영 시작 전 백업 정책을 확인하고 설정합니다.

### 8.2 Storage 백업

- 공개 상품 이미지: Supabase Storage + CDN 캐시.
- 비공개 비즈니스 문서: Supabase Storage 버킷. 정기 외부 백업 권장.
- 원본 파일은 삭제하지 않고 보존합니다.

### 8.3 비상 내보내기

- 주문·가격·거래처 데이터의 CSV/JSON 비상 내보내기 기능을 Phase 5에서 구현합니다.
- 비상 내보내기는 Op Admin만 실행 가능합니다.
- 다운로드 감사로그를 남깁니다.

### 8.4 복구 책임자와 목표

- 복구 책임자: 운영사 책임자.
- RTO(목표 복구 시간)와 RPO(허용 데이터 손실)는 미확정입니다. (OPEN_DECISIONS 참조)

---

## 9. 모니터링

### 9.1 모니터링 대상

모니터링 도구는 구현 시점에 확정합니다. 현재 유료 서비스를 확정하지 않습니다.

| 항목 | 중요도 | 설명 |
|---|---|---|
| 애플리케이션 오류 (5xx) | 높음 | 즉시 알림 |
| API 오류율 | 높음 | 오류율 급증 시 알림 |
| 주문요청 제출 실패 | 높음 | fn_submit_order_request 오류 |
| 가격 계산 실패 | 높음 | fn_calculate_price 오류 |
| 출고 정합성 실패 | 높음 | QUANTITY_EXCEEDED, BACKORDERED_VIOLATION |
| Import 실패 | 중간 | STRICT_ATOMIC_ROLLBACK |
| 이메일 발송 실패 | 중간 | Resend API 오류 |
| Webhook 처리 실패 | 중간 | 배송사 Webhook 오류 |
| 비정상 로그인 시도 | 높음 | AUTH_ACCOUNT_LOCKED 급증 |
| 관리자 중요 작업 | 높음 | 가격표 변경, 거래처 승인·정지, Import Apply |
| API 응답시간 p95 | 중간 | p95 > 3초 시 점검 |
| 검색결과 없음 비율 | 낮음 | 상품 DB 점검 신호 |

### 9.2 감사로그 활용

- 모든 중요 작업에 audit_logs가 기록됩니다 (SECURITY_RULES.md §9).
- request_id로 특정 요청의 전체 흐름을 추적합니다.
- Op Admin만 audit_logs를 조회합니다.
