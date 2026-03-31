---
name: work-start
description: "plan.md를 기반으로 phase와 handoff 파일을 생성하고 실제 개발 업무를 시작하는 스킬. '업무 시작', 'phase 생성', 'work start', '작업 시작', '개발 시작', 'phase 만들어' 등을 요청하거나, plan이 준비된 상태에서 본격적으로 개발을 진행하고 싶을 때 이 스킬을 사용한다. 다음 phase로 넘어가고 싶을 때도 트리거된다."
---

# work-start: 업무 시작 및 Phase 진행

plan.md의 Steps를 기반으로 phase_{n}.md를 생성하고, 이전 phase가 완료되면 handoff_{n}.md를 작성한다.

## 워크플로우

### 1. 대상 프로젝트 확인

`projects/` 디렉토리에서 대상 프로젝트를 특정한다.
- 사용자가 프로젝트 이름을 지정하면 해당 프로젝트를 사용한다
- 프로젝트가 하나뿐이면 자동 선택한다
- 여러 개인데 지정이 없으면 목록을 보여주고 선택하게 한다

### 2. 현재 진행 상태 파악

프로젝트 디렉토리의 기존 파일들을 읽어 현재 상태를 파악한다:
- `plan.md` 읽기 (필수 — 없으면 work-plan 스킬을 먼저 실행하라고 안내)
- 기존 `phase_{n}.md` 파일들이 있는지 확인
- 기존 `handoff_{n}.md` 파일들이 있는지 확인
- 마지막 handoff의 Next 항목을 확인하여 다음 phase 번호를 결정한다

### 3. Phase 파일 생성

`template/phase_{n}.md` 구조를 기반으로 다음 phase를 생성한다:

- **Objective**: plan.md의 해당 step에서 도출한 한 줄 목표
- **Context**: plan.md 경로, 선행 조건(이전 phase 완료 여부), PM 문서 경로
- **Tasks**: 이 phase에서 수행할 구체적인 작업 목록 (체크리스트)
- **Constraints**: 이 phase에서 하지 말아야 할 것, 범위 밖 작업
- **Acceptance Criteria**: 완료 판단 기준 (체크리스트)

Tasks 작성 기준:
- 하나의 task는 하나의 명확한 작업이어야 한다
- 코드 작성, 테스트 작성, 설정 변경 등 구체적으로 기술한다
- `context/conventions.md`의 컨벤션을 반영한다 (DDD 아키텍처, given/when/then 테스트 등)

### 4. 이전 Phase 완료 시 Handoff 작성

이전 phase가 완료된 상태에서 다음 phase로 넘어가는 경우, `template/handoff_{n}.md` 구조를 기반으로 handoff를 작성한다:

- **Finished**: 완료한 작업 목록
- **Blocked**: 미완료 작업과 사유
- **Changed**: 기존 계획 대비 변경된 내용
- **Next**: 다음 phase 번호

이전 phase의 Tasks와 Acceptance Criteria를 검토하여 Finished/Blocked를 판단한다.

### 5. 실제 코드 작업 안내

phase 파일이 생성되면, 사용자에게 다음을 안내한다:
- 생성된 phase의 Tasks 요약
- 대상 코드 프로젝트 경로 (`context/project_structure.md` 참고)
- sub-agent 활용 안내: code-writer, test-code-writer, code-verifier를 필요에 따라 활용할 수 있음

## 첫 번째 Phase vs 후속 Phase

**첫 시작 (phase_1 생성):**
- plan.md의 첫 번째 step을 기반으로 phase_1.md를 생성한다
- handoff는 아직 없으므로 생성하지 않는다

**후속 Phase (phase_2 이후):**
- 이전 phase의 완료 여부를 확인한다
- handoff_{n-1}.md를 먼저 작성한 후, 다음 phase_{n}.md를 생성한다
- 이전 phase가 완료되지 않았으면 완료를 먼저 하라고 안내한다

## 주의사항

- plan.md 없이 phase를 생성하지 않는다
- phase 번호는 1부터 순차적으로 증가한다
- phase를 건너뛰지 않는다 (phase_1 → phase_2 → phase_3 순서)
- 이전 phase가 완료되지 않은 상태에서 다음 phase를 강제로 생성하지 않는다 (사용자가 명시적으로 요청한 경우 제외)
- Tasks를 임의로 추가하거나 제거하지 않는다. plan.md에 없는 작업은 사용자 확인 후 추가한다
