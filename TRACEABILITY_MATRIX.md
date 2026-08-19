# TRACEABILITY_MATRIX.md

이 문서는 Checkpoint C1 Final Fix의 감사 항목을 기존 SSOT, 화면, 구현 Handoff 및 검증 증거에 연결한다. 비즈니스 규칙을 재정의하지 않는다.

| Audit ID | Canonical SSOT | C1 산출물 | Test / Handoff | 검증 증거 |
|---|---|---|---|---|
| C1-AUD-001 | [BUSINESS_RULES.md](BUSINESS_RULES.md) §§2~3 | [WIREFRAME_INDEX.md](WIREFRAME_INDEX.md) N-150 Master, WF-02·03·04·06·08 | TC-PRC-020~028, TC-UOM-004~005 | [C1 Final Fix Validation](docs/validation/CHECKPOINT_C1_FINAL_FIX_VALIDATION.md) |
| C1-AUD-002 | [BUSINESS_RULES.md](BUSINESS_RULES.md) §§3.8, 8 | Order 0847 Master, WF-01·06·08 | TC-ORD-050~053 | C1 Final Fix Validation |
| C1-AUD-003 | [BUSINESS_RULES.md](BUSINESS_RULES.md) §6 | DS-PH2 Master, WF-06·08 | TC-ORD-030~034, TC-QTY-001~010 | C1 Final Fix Validation |
| C1-AUD-004 | [CONTENT_SYSTEM.md](CONTENT_SYSTEM.md) §§11~13 | WF-01·05, [DATABASE_SCHEMA.md](DATABASE_SCHEMA.md) §4.1 | TC-CNT-001~006, [ROADMAP.md](ROADMAP.md) Phase 5 | C1 Final Fix Validation |
| C1-AUD-005 | [DESIGN_SYSTEM.md](DESIGN_SYSTEM.md) §§2, 8~9 | WF-01~08 | 구현 접근성 검사 + 브라우저/E2E Handoff | C1 Final Fix Validation |
| C1-AUD-006 | [DESIGN_SYSTEM.md](DESIGN_SYSTEM.md) §8 | WF-01~08 | 단일 SVG icon set Handoff | C1 Final Fix Validation |
| C1-AUD-007 | [BUSINESS_RULES.md](BUSINESS_RULES.md) §12, [DATA_IMPORT_SPEC.md](DATA_IMPORT_SPEC.md) §§1, 6 | WF-07 | TC-IMP-002, TC-IMP-010 | C1 Final Fix Validation |
| C1-AUD-008 | 저장소 상대 링크 원칙 | 8개 기존 Markdown 문서 | Clone·Worktree 링크 검사 | C1 Final Fix Validation |
| OD-024 | [OPEN_DECISIONS.md](OPEN_DECISIONS.md), [WIREFRAME_INDEX.md](WIREFRAME_INDEX.md) | WF-02·04 canonical 임시 문구 | 정책 확정 시 회귀검증 | C1 Final Fix Validation |
| OD-005~008 | [BUSINESS_RULES.md](BUSINESS_RULES.md) §4, [OPEN_DECISIONS.md](OPEN_DECISIONS.md) | WF-03·05·06·08 미확정 경계 | VAT 결정 후 별도 테스트 확정 | C1 Final Fix Validation |

## Content Implementation Handoff

`CONTENT_SYSTEM.md`의 Claim Record와 Publication Gate가 기준이다. Phase 5에서 `DATABASE_SCHEMA.md` §4.1의 논리 구조를 물리 모델로 승인하고, `TEST_PLAN.md`의 TC-CNT-001~006을 구현한다. 관리자 Preview와 Public·Buyer renderer는 동일 Content Record, Claim gate, CTA, status mapping, visibility rule을 사용한다.
