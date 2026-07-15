# 독립 공구 브랜드 연합형 B2B 유통 웹앱

> **현재 단계**: Development Phase 0 (사업 및 시스템 설계)

## 프로젝트 개요

전국의 공구점, 철물점, 산업재 유통업체, 시공업체가 휴대폰과 PC에서 상품을 검색하고, 자기 거래가격을 확인하고, 반복 주문할 수 있는 B2B 웹앱입니다.

초기에는 운영사의 폐쇄형 B2B몰로 시작하며, 향후 독립 공구 브랜드, 중소 제조사, 수입사, 전문 유통사가 참여하는 다중 공급자 B2B 유통 플랫폼으로 확장합니다.

## 기술 스택

| 영역 | 기술 |
|---|---|
| 프레임워크 | Next.js App Router |
| 언어 | TypeScript (strict mode) |
| 스타일 | Tailwind CSS v4 |
| 데이터베이스 | Supabase PostgreSQL |
| 인증 | Supabase Auth |
| 파일저장 | Supabase Storage |
| 유효성 검사 | Zod |
| 배포 | Vercel |
| DNS | Cloudflare |
| 테스트 | Vitest + Playwright |

## 프로젝트 구조

```
b2b-tool-platform/
├── docs/                    # 프로젝트 문서 (Phase 0에서 작성)
├── src/                     # 애플리케이션 소스 (Phase 1부터)
├── supabase/                # Supabase 설정 및 migration
├── .env.example             # 환경변수 예제
├── .gitignore
└── README.md
```

## 환경 설정

| 환경 | 용도 |
|---|---|
| Local | 로컬 개발 |
| Preview | PR별 미리보기 |
| Staging | 운영 전 최종 검증 |
| Production | 실제 운영 |

## 문서 목록

Phase 0에서 작성하는 프로젝트 문서:

- `PROJECT_OVERVIEW.md` — 프로젝트 비전과 사업 발전단계
- `MARKET_CONTEXT.md` — 시장 배경과 포지셔닝
- `PRODUCT_REQUIREMENTS.md` — 기능 요구사항 (SSOT: MVP 범위)
- `BUSINESS_RULES.md` — 가격·주문·출고 규칙 (SSOT)
- `USER_ROLES.md` — 사용자 역할과 권한
- `USER_FLOWS.md` — 주요 업무 흐름
- `INFORMATION_ARCHITECTURE.md` — 사이트맵과 화면 계층
- `ROADMAP.md` — 개발 로드맵 (SSOT: 개발 순서)
- `DATABASE_SCHEMA.md` — 데이터베이스 구조 (SSOT)
- `SECURITY_RULES.md` — 보안 정책 (SSOT)
- `DATA_IMPORT_SPEC.md` — 엑셀 등록 사양
- `INTERNATIONALIZATION.md` — 다국어 및 해외 확장
- `TEST_PLAN.md` — 테스트 계획
- `DESIGN_SYSTEM.md` — 디자인 시스템 (SSOT)
- `CONTENT_SYSTEM.md` — 콘텐츠 관리 구조
- `AGENTS.md` — AI 에이전트 작업 규칙
- `DECISION_LOG.md` — 확정된 결정 기록 (SSOT)
- `OPEN_DECISIONS.md` — 미결정 사항 (SSOT)
- `CHANGELOG.md` — 변경 이력

## 시작하기

> Phase 0 진행 중 — 아직 애플리케이션 코드가 없습니다.
