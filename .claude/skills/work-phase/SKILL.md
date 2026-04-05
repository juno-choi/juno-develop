---
name: work-phase
description: "phase_{n}.md의 Tasks를 sub-agent 파이프라인(code-writer → build-check → code-verifier → Decision Gate → test-code-writer → perf/security-reviewer → code-simplifier → review)으로 자동 실행하는 스킬. 다음 상황에서 반드시 이 스킬을 사용한다: '코드 작업 진행', 'phase 실행', 'work phase', '작업 진행해줘', '코드 작업 시작', 'phase 진행', '다음 phase', '다음 진행', '개발 시작', '코드 짜줘', '구현해줘'. work-start로 phase가 생성된 후 코드 작업을 진행할 때 반드시 이 스킬을 사용한다. 사용자가 phase 파일이 있는 상태에서 개발을 진행하거나 다음 단계로 넘어가고 싶다고 하면 항상 이 스킬을 트리거한다."
---

# work-phase: Phase 자동 실행

phase_{n}.md를 읽고 pre-flight → code-writer → build → code-verifier → **Decision Gate** → test-writer → reviewers(병렬) → simplifier → review 파이프라인으로 코드 작업을 자동 실행한다.
code-verifier 통과 후 **반드시 사용자에게 코드 방향성 판단을 요청**(Decision Gate)하고, APPROVE 후에만 후속 단계를 진행한다.

## Sub-Agent 모델 지정

| Agent | Model | 이유 |
|---|---|---|
| code-writer | opus | 코드 작성 품질 |
| test-code-writer | opus | 테스트 품질 |
| code-simplifier | opus | 리팩토링 품질 |
| code-verifier | sonnet | 검증은 sonnet으로 충분 |
| performance-reviewer | sonnet | 리포트 작업은 sonnet으로 충분 |
| security-reviewer | sonnet | 리포트 작업은 sonnet으로 충분 |

사용자가 별도 모델을 지정하면 그것을 우선한다.

## Git Commit Convention

형식: `{type}: {project-name} [phase-{n}] {설명}` — `{project-name}`은 plan.md의 Target Project명.

| type | 시점 |
|---|---|
| feat | code-writer 코드 작성 + 빌드 성공 후 |
| test | test-code-writer 테스트 작성 후 |
| refactor | code-simplifier 리팩토링 + 빌드 성공 후 |
| fix | 재작업 수정 + 빌드 성공 후 |

커밋 대상: 대상 프로젝트 내 변경 파일만. `git add`는 파일을 명시적으로 지정 (`git add -A` 금지).

## 재작업 횟수 관리

| 카운터 | 대상 | 최대 | 초과 시 |
|---|---|---|---|
| 빌드 카운터 | 컴파일/테스트 실패 | 3회 | 사용자 보고 |
| 검증 카운터 | verifier/reviewer/review FAIL | 3회 | 사용자 보고 |
| 테스트 카운터 | test-writer 자체 문제 | 2회 | 사용자 보고 |
| decision 카운터 | Decision Gate MODIFY/REJECT | 무제한 | 사용자가 APPROVE할 때까지 |

빌드 실패는 사소한 컴파일 에러가 대부분이므로 검증 카운터와 분리한다. Decision Gate의 MODIFY/REJECT는 사용자 의도적 판단이므로 검증 카운터에 포함하지 않는다.

### 단계적 전략 에스컬레이션 (검증 카운터와 연동)

| 재시도 | 전략 |
|---|---|
| 1회차 | 실패 사유 + 수정 방향 (`phase_fail_{n}.md` 전달) |
| 2회차 | 1회차 + 참고 프로젝트 유사 코드 전달 + "다른 접근법을 사용해라" |
| 3회차 | 2회차 + "문제 파일을 처음부터 새로 작성해라. 기존 접근법을 버려라" |
| 4회차 | 사용자에게 보고, 진행 방향 확인 |

빌드 카운터 실패는 에스컬레이션 없이 단순 수정 요청으로 처리한다.

## 파이프라인 상태 관리 (Pipeline State)

진행 상태를 `phase_{n}.md`의 `## Pipeline State` 섹션에 기록한다. 중단 시 이어서 실행할 수 있도록 한다.

- 각 단계 완료 시 `- [ ]` → `- [x]`로 체크 + 커밋 해시/결과 기록
- `/work-phase` 실행 시 Pipeline State가 있으면: 마지막 체크 단계 확인 → 커밋 해시 검증 → 다음 미완료 단계부터 재개
- Pipeline State가 없으면 새로 생성하고 처음부터 실행

```markdown
## Pipeline State
- [x] pre-flight (commit: abc1234)
- [x] code-writer (commit: def5678)
- [x] build-check: PASS
- [x] code-verifier: PASS
- [x] decision-gate: APPROVE
- [ ] test-code-writer
- [ ] reviewers
- [ ] simplifier
- [ ] review
- counters: { build: 0, verification: 0, decision: { approve: 1, modify: 0, reject: 0 } }
- fail_log: none
```

## 실패 기록 (Fail Log)

실패 이력을 `phase_fail_{n}.md`에 누적 기록. code-writer 재작업 요청 시 **반드시 함께 전달**하여 동일 실수 반복을 방지한다. verifier/reviewer/review FAIL 및 빌드 실패 시 기록한다.

재작업 시 프롬프트 작성 상세 가이드: `references/rework-prompt-guide.md` 참조.

## 워크플로우

### 1. 현재 Phase 특정

- `projects/` 하위에서 대상 프로젝트 특정 (하나면 자동 선택, 여럿이면 사용자 확인)
- `plan.md` 읽어 phase 목록 파악 → 기존 `handoff_{n}.md`로 마지막 완료 phase 확인 → 다음 `phase_{n}.md` 결정
- phase 파일 없으면 자동 생성: plan.md에서 Objective/Tasks/Constraints 도출, `template/phase_{n}.md` 구조 사용, 이전 handoff의 Changed/Blocked 반영
- Pipeline State 존재 시 재개(Resume) 모드 진입

### 2. Phase 문서 분석

`phase_{n}.md`에서 Objective, Context(프로젝트 경로), Tasks, Constraints, Acceptance Criteria, Pipeline State를 추출. plan.md의 Target Project에서 실제 코드 프로젝트 경로 확인.

**Profile 확인**: plan.md의 `Profile` 필드에서 기술 스택 프로필명을 읽는다 (예: `java-spring`). 이후 모든 빌드 명령어와 agent 호출 시 `context/conventions/{profile}/` 하위 파일을 참조한다.

### 3. Pre-flight Check

Bash tool로 즉시 검증 (Agent 호출 없음): `cd {프로젝트경로} && git status --short && {profile의 Full build 명령어}`
- 빌드 명령어는 `context/conventions/{profile}/code-writer.md`의 `## Build Commands` → **Full build** 항목을 사용한다.

| 결과 | 처리 |
|---|---|
| 빌드+테스트 green | Pipeline State 기록 후 4단계로 |
| 경로/gradlew 없음 | 사용자 보고, 중단 |
| uncommitted 변경 | 사용자에게 확인 |
| 빌드/테스트 red | "이미 red 상태" 보고, 중단 |

### 4. code-writer Agent 실행 (opus)

프롬프트에 포함: phase Objective/Tasks 전체, 프로젝트 경로, Constraints, 참고 소스 경로, **프로필 경로 (`context/conventions/{profile}/code-writer.md`)**, `phase_fail_{n}.md` 경로 (있으면). 정보 부족 시 참고 프로젝트 파일 목록을 미리 조사하여 포함. Pipeline State 기록.

### 5. 빌드 검증 + Git Commit

**5-a. 컴파일**: profile의 **Compile only** 명령어 실행 — 실패 시 에러 로그로 code-writer 수정 요청 (빌드 카운터)
**5-b. 테스트**: profile의 **Incremental build** 명령어 실행 — 성공 시 `feat` 커밋. 실패 시 "기존 동작 보존하면서 수정" 요청 (빌드 카운터), 성공 시 `fix` 커밋

중간 단계는 incremental build 사용. **Full build**는 pre-flight(3단계)과 최종 review(11단계)에서만. 모든 빌드 명령어는 `context/conventions/{profile}/code-writer.md`의 `## Build Commands` 섹션을 참조한다.

### 6. code-verifier Agent 실행 (sonnet) — diff 기반

프롬프트에 포함: AC 체크리스트, 프로젝트 경로, `git diff HEAD~1` (변경 파일만 검증 대상), "직접 수정하지 말고 보고만 할 것"

- **PASS**: 7단계(Decision Gate)로
- **FAIL**: `phase_fail_{n}.md` 기록 → code-writer 수정 요청 (검증 카운터 증가, 에스컬레이션) → 5단계부터 재실행

### 7. Decision Gate (사용자 판단)

code-verifier PASS 후, **반드시 사용자에게 코드 방향성 판단을 요청**한다. 매 phase 항상 실행.

사용자에게 보여주는 정보:
- **변경 요약**: 파일 목록 + code-writer 작업 요약 2-3줄
- **AC 평가**: 항목별 PASS/FAIL 테이블
- **핵심 코드 변경**: 설계 판단이 드러나는 부분만 발췌 (전체 diff 아님, 핵심 2-3개 파일)
- **선택지**: APPROVE / MODIFY (피드백 입력) / REJECT (새 지시 입력)

**분기:**

| 선택 | 동작 | 커밋 타입 | 카운터 |
|---|---|---|---|
| APPROVE | 8단계(test-writer)로 진행 | — | decision.approve++ |
| MODIFY | 피드백으로 code-writer 부분 수정 → 5→6→7 재실행 | fix | decision.modify++ |
| REJECT | `git revert HEAD --no-edit` → code-writer 재작성 → 5→6→7 재실행 | feat | decision.reject++ |

MODIFY/REJECT는 **검증 카운터에 미포함** (사용자 의도적 판단). **횟수 제한 없음** — APPROVE까지 반복.

재진입 시 추가 표시: 이전 선택, 사용자 피드백 원문, 이전 대비 변경 요약.

### 8. test-code-writer Agent 실행 + Commit (opus)

Decision Gate APPROVE 후 실행. 프롬프트: 대상 파일 목록, **프로필 경로 (`context/conventions/{profile}/test-code-writer.md`)**, AC 중 테스트 검증 가능 항목.

빌드 확인 (profile의 **Incremental build** 명령어): 성공 시 `test` 커밋. 실패 시 원인 분류 — 상세 기준은 `references/recovery-matrix.md` 참조.

**건너뛰는 경우**: phase Tasks에 "테스트 제외" 명시 또는 인프라/설정 변경만 포함된 phase.

### 9. reviewers 병렬 실행 (sonnet)

performance-reviewer + security-reviewer를 **단일 메시지에서 2개 Agent 호출**로 병렬 실행. 둘 다 read-only.

프롬프트 공통: 프로젝트 경로, 변경 파일 목록, git diff, "코드 수정 말고 리포트만"

**결과 처리:**
- 둘 다 PASS → 10단계
- CONDITIONAL PASS → handoff Changed에 기록 후 10단계
- FAIL (CRITICAL 1+개 또는 HIGH 2+개) → `phase_fail_{n}.md` 기록 → code-writer 수정 (검증 카운터 1 증가) → 5→6→7→9 재실행 (simplifier 건너뜀)
- **reviewer 간 상충** → 사용자 인터뷰. 상세: `references/rework-prompt-guide.md` 참조

### 10. code-simplifier Agent 실행 + Commit (opus)

모든 검증 PASS 후 1회만 실행. 기능 변경 없이 가독성/일관성/유지보수성만 개선. 기존 테스트 통과 유지.

빌드 확인: 성공 시 `refactor` 커밋. 실패 시 **1회 수정 기회** → 재실패 시 `git checkout -- .`으로 전체 롤백. 재작업 루프에서는 항상 건너뛴다.

### 11. Review + review_{n}.md 작성

code-verifier를 **최종 확인 모드**로 호출. 6단계에서 이미 AC 검증했으므로 여기서는:
- simplifier 기능 보존 (있는 경우): `git diff {simplifier전commit}..HEAD`
- 테스트 커버리지/패턴 준수/누락 케이스
- 범위 위반 (Constraints 밖 작업)
- 최종 `./gradlew clean build`

review_{n}.md를 `template/review_{n}.md` 구조로 작성: Summary, Document Review, Code Review, Test Review, Verdict (PASS/PASS WITH NOTES/FAIL), Action Items.

**Verdict 분기:**
- PASS / PASS WITH NOTES → 12단계
- FAIL → 재작업 루프: 검증 카운터 확인 → `phase_fail_{n}.md` 기록 → code-writer 수정 (에스컬레이션) → 5→6→7 재실행. 상세 복귀 경로: `references/recovery-matrix.md` 참조

### 12. Phase 완료 처리

Review PASS 확정 후에만 문서 작성 (FAIL 가능성 시점에 미리 작성하지 않음):

1. **phase_{n}.md**: Tasks/AC 체크 업데이트, Pipeline State 최종 상태
2. **handoff_{n}.md** (`template/` 구조): Finished/Blocked/Changed/Next
3. **review_{n}.md**: Document Review에 Handoff 검토 추가 (AC vs Finished vs 실제 코드)
4. **plan.md**: 완료 phase 체크, Done When 갱신
5. **log.md** (`template/` 구조): 완료 작업/변경 이력/미해결 사항 누적 기록. 없으면 신규 생성. `## 회고`는 work-outputs에서 작성

### 13. 결과 보고

사용자에게 간결 보고: 빌드 결과, 테스트 요약, AC PASS/FAIL 테이블, Review Verdict, 재작업 횟수 (빌드/검증 각각), Agent 호출 요약 테이블 (decision-gate 포함), Git 커밋 이력, 업데이트 파일 목록.

### 14. 다음 Phase 파일 생성

Review PASS 시 자동 생성 (사용자 확인 불필요): plan.md에서 다음 phase 도출 → `phase_{n+1}.md` 생성 (이전 handoff Changed/Blocked + review Action Items를 Context 반영) → "/work-phase를 실행하세요" 안내.

검증 카운터 3회 초과: FAIL 보고 → 사용자가 재작업 또는 스킵(Blocked 기록) 선택.

### 15. 마지막 Phase 완료 시

11~12단계 동일 수행 → "모든 phase 완료. `/work-outputs`로 마무리하세요." 안내.

## 연속 실행 모드

"전체 진행" 요청 시 남은 phase를 순차 실행. 검증 카운터 3회 초과 시 중단. **연속 모드에서도 Decision Gate는 항상 실행.**

## 사용자 피드백 반영

중간 수정 요청 시: code-writer 재작업 → 빌드 검증(5) → verifier 재검증(6) → Decision Gate(7) → handoff Changed 기록 → review 갱신.

## 산출물

**파일**: phase_{n}.md, phase_fail_{n}.md (재작업 시), handoff_{n}.md, review_{n}.md, plan.md, log.md
**커밋**: feat / test / refactor / fix

## 핵심 규칙

- **코드 수정은 오직 code-writer만** — verifier/reviewer는 보고만 (verifier는 Edit/Write 권한 없음)
- **code-verifier에게 항상 git diff 전달** — 변경 파일만 검증 대상
- **각 단계별 git commit으로 checkpoint 관리**
- **재작업 시 `phase_fail_{n}.md` 기록 + code-writer에 전달** 필수
- **simplifier는 모든 검증 통과 후 1회만** — 재작업 루프에서 건너뜀
- **문서 생성은 Review PASS 후에만**
- **Decision Gate는 매 phase 항상 실행** — APPROVE까지 반복, 횟수 무제한
- **REJECT 시 `git revert` 사용** (reset --hard 아님)
- **`clean build`는 pre-flight + 최종 review에서만**
- 재작업 복귀 최적화 상세: `references/recovery-matrix.md`
- 재작업 프롬프트 가이드 상세: `references/rework-prompt-guide.md`
