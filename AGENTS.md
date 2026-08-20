# AI Agents 작업 규칙

이 프로젝트에서 AI 에이전트가 따라야 할 작업 규칙입니다.

## 현재 작업 제약

- **현재 단계**: Development Phase 0 완료 (사업 및 시스템 설계)
- **허용**: 프로젝트 문서 작성, 설계, 분석
- **금지**: 애플리케이션 코드 작성, DB migration, Supabase 연결, Vercel 배포
- **Phase 1 시작 조건**: Phase 0 최종감사(Checkpoint D) 완료 및 승인 후

## 핵심 규칙

1. 한 번에 전체 쇼핑몰을 만들지 않는다
2. 작업 전에 계획을 제시한다
3. 한 단계씩 구현하고 검증한다
4. 중요한 사업규칙을 임의로 바꾸지 않는다
5. 데이터베이스 구조 변경 전에 영향을 설명한다
6. 파괴적 명령을 자동 실행하지 않는다
7. 실제 운영 데이터처럼 보이는 가짜 수치를 만들지 않는다
8. 테스트 데이터는 DEMO 또는 SAMPLE로 표시한다
9. API 키와 비밀번호를 코드에 넣지 않는다
10. 기능 완료 후 CHANGELOG.md를 업데이트한다
11. 확정 결정은 DECISION_LOG.md에 기록한다
12. 미확정 사항은 OPEN_DECISIONS.md에 기록한다
13. SSOT 원칙을 준수한다 — 기준 문서 외 중복 금지
14. 확정되지 않은 사업규칙을 임의로 확정하지 않는다
15. MVP 범위를 임의로 확대하지 않는다

## SSOT 지정

| 주제 | 기준 문서 |
|---|---|
| 가격, VAT, MOQ, 주문, 출고 | BUSINESS_RULES.md |
| 테이블, 컬럼, 관계 | DATABASE_SCHEMA.md |
| 인증, 권한, RLS | SECURITY_RULES.md |
| 디자인 토큰 | DESIGN_SYSTEM.md |
| MVP 기능 범위 | PRODUCT_REQUIREMENTS.md |
| 개발 순서 | ROADMAP.md |
| API 실행 경계 | API_SPEC.md |
| 배포·환경·CI·Migration | DEPLOYMENT_GUIDE.md |
| 확정 결정 | DECISION_LOG.md |
| 미확정 사항 | OPEN_DECISIONS.md |

## 구현 단계 AI 작업 규칙 (Phase 1 이후 적용)

아래 규칙은 Phase 1 구현 시작 시점부터 적용합니다.

### 작업 시작 전

- 작업 전에 관련 SSOT 문서를 반드시 읽는다.
- main 브랜치에 직접 작업하지 않는다. feature/* 또는 fix/* 브랜치를 생성한다.
- 한 번에 하나의 작은 기능 단위로 작업한다.
- 구현 전에 변경할 파일과 접근 방법을 계획으로 제출한다.
- 변경 대상 파일 목록을 사전에 보고한다.

### 데이터베이스

- DB 변경은 반드시 supabase/migrations/ 디렉터리의 Migration 파일로 작성한다.
- 임의로 supabase db reset을 실행하지 않는다.
- 기존 Migration 파일을 수정하지 않는다. Forward Fix 원칙.
- Destructive Migration(테이블·컬럼 삭제, 타입 변경)은 사람의 승인을 받는다.

### 보안

- SECURITY_RULES.md의 보안 정책을 우회하지 않는다.
- RLS 정책을 비활성화하거나 약화하지 않는다.
- service_role key를 코드에 하드코딩하지 않는다.
- NEXT_PUBLIC_ 변수에 서버 전용 Secret을 설정하지 않는다.

### 가격·수량 로직

- 가격 계산을 클라이언트 전용으로 구현하지 않는다. 반드시 서버(fn_calculate_price)를 사용한다.
- 수량 불변식(BUSINESS_RULES.md 6.2)을 클라이언트에서만 검증하지 않는다.
- 클라이언트가 전달한 가격을 서버에서 신뢰하지 않는다.

### 테스트와 완료 선언

- 테스트가 통과하기 전에 기능 완료를 선언하지 않는다.
- 구현 후 모바일(390px)과 PC(1440px) 화면을 모두 확인한다.
- 접근성 기준(DESIGN_SYSTEM.md)을 위반하지 않는다.

### 정보 보호

- 민감정보(비밀번호, 토큰, 사업자등록번호)를 로그에 기록하지 않는다.
- Production 데이터를 개발 환경에 복사하지 않는다.
- 감사로그에 비밀번호나 전체 레코드를 기록하지 않는다.

### 이력 관리

- 기능 구현 완료 후 CHANGELOG.md를 갱신한다.
- 의미 있는 커밋 메시지를 사용한다. (type: 내용 형식)
- Phase 범위를 자동으로 확장하지 않는다. 추가 기능은 승인을 받는다.
