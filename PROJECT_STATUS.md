# PROJECT_STATUS.md

> 최종 업데이트: 2026-08-19

## Current Baseline

| 항목 | 상태 |
|---|---|
| Development Phase | Phase 0 설계 문서 보정 |
| main 기준 | `f4950833d34e3fd25f95a4e9deba412da7965882` |
| Checkpoint C1 | main 포함 완료, 최종 정합성 Fix Pass 진행 |
| C1 Fix Branch | `docs/checkpoint-c1-final-fixes` |
| C2 | 별도 `docs/checkpoint-c2-api-deployment` 작업트리에서 진행 중이며 이 브랜치 범위 밖 |
| 애플리케이션 코드 | 없음 |
| DB Migration / Hosted DB 변경 | 없음 |

## This Branch

- **CONFIRMED**: C1 와이어프레임의 DEMO 가격·주문·수량 의미를 `BUSINESS_RULES.md`와 일치시킨다.
- **CONFIRMED**: Claim Publication Gate, 공통 Content Renderer 계약, 접근성·아이콘 규칙을 문서화한다.
- **CONFIRMED**: C2 문서, API, 배포 설계와 기존 C2 작업트리는 변경하지 않는다.
- **OPEN**: VAT 계산 기준·반올림·표시 방식은 OD-005~008에 따라 미확정 상태를 유지한다.
- **OPEN**: OD-024 입고예정 주문 정책은 확정 전 canonical 임시 문구만 사용한다.

## Next Gate

C1 Final Fix 브랜치의 최종 읽기 전용 감사를 수행한 뒤 사용자 승인에 따라 main 병합 여부를 결정한다. 이 문서는 C2 착수 또는 병합을 승인하지 않는다.
