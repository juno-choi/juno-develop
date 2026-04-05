# Phase Fail Log: Phase_{number}

## 실패 이력

### Attempt {m} — {yyyy-MM-dd}

**발생 단계**: {code-verifier / performance-reviewer / security-reviewer / review / build}
**카운터**: {빌드: n/3, 검증: n/3}
**에스컬레이션 단계**: {1회차: 기본 수정 / 2회차: 참고 코드 보강 / 3회차: 재작성 지시}

#### 실패 항목

| # | 파일:라인 | 실패 사유 (원문) | 심각도 | 수정 방향 |
|---|---|---|---|---|
| 1 | {파일 경로}:{라인 범위} | {reviewer/verifier 리포트 원문 인용} | {CRITICAL/HIGH/MEDIUM} | {제안된 수정 방향} |
| 2 | ... | ... | ... | ... |

#### code-writer 수정 내용 요약
- {수정 후 어떤 변경을 했는지 간략 요약}
- {변경된 파일 목록}

#### 수정 결과
- {PASS / FAIL — FAIL이면 다음 Attempt에서 계속}

---

### Attempt {m+1} — {yyyy-MM-dd}
...
