# Review_{number}: Phase {n} 리뷰

## Summary
- Phase: phase_{n} — {phase objective}
- 리뷰 일시: yyyy-MM-dd
- 리뷰 범위: {문서 리뷰 / 코드 리뷰 / 문서+코드}

## Document Review (문서 리뷰)

### Plan 정합성
- plan Done When 달성 현황: {달성}/{전체}
- plan Steps 진행 현황: phase_{n} {완료/진행중}

### Phase 정합성
- Tasks: {완료}/{전체}
  - [x] {완료된 task}
  - [ ] {미완료 task}
- Acceptance Criteria: {달성}/{전체}
  - [x] {PASS된 AC}
  - [ ] {FAIL된 AC}

### Handoff 검토
- Finished: {항목 수}건 기록됨
- Blocked: {항목 수}건 — {해소 여부}
- Changed: {항목 수}건 — {plan 영향도 평가}

## Code Review (코드 리뷰)

### 검증 대상
- {변경된 파일 목록과 변경 요약}

### 통과 항목
- {검증 통과한 항목을 구체적으로}

### 개선 권장
- {당장 문제는 아니지만 개선하면 좋을 점, 없으면 "없음"}

### 수정 필요
- {반드시 고쳐야 할 문제, 없으면 "없음"}

### 빌드/테스트 결과
- 빌드: {성공/실패}
- 테스트: {성공/실패/스킵}

## Verdict (판정)
- 종합 판정: {PASS / PASS WITH NOTES / FAIL}
- 다음 Phase 진행 가능 여부: {가능 / 불가 — 사유}

## Action Items
- {리뷰에서 도출된 후속 조치 목록}
- {다음 phase에 반영해야 할 사항}
