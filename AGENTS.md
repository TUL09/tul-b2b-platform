# AI Agents 작업 규칙

이 프로젝트에서 AI 에이전트가 따라야 할 작업 규칙입니다.

## 현재 작업 제약

- **현재 단계**: Development Phase 0 (사업 및 시스템 설계)
- **허용**: 프로젝트 문서 작성, 설계, 분석
- **금지**: 애플리케이션 코드 작성, DB migration, Supabase 연결, Vercel 배포

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
| 확정 결정 | DECISION_LOG.md |
| 미확정 사항 | OPEN_DECISIONS.md |

## Checkpoint C에서 보강 예정

- AI 에이전트 활용 방안 상세
- 문서 생성 자동화 규칙
- 코드 리뷰 가이드라인
