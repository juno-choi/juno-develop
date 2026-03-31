---
name: work-outputs
description: "완료된 업무를 log로 정리하고 outputs로 아카이브하는 스킬. '업무 종료', '업무 완료', 'work done', 'outputs', '아카이브', '프로젝트 마무리', '로그 정리', 'work outputs' 등을 요청하거나, 모든 phase가 완료되어 프로젝트를 마무리하고 싶을 때 이 스킬을 사용한다. 개발 프로젝트 종료 및 정리 요청에도 트리거된다."
---

# work-outputs: 업무 종료 및 아카이브

모든 phase가 완료된 프로젝트를 log.md로 정리하고, `outputs/`로 이동한다.

## 워크플로우

### 1. 대상 프로젝트 확인

`projects/` 디렉토리에서 아카이브할 프로젝트를 특정한다.
- 사용자가 프로젝트 이름을 지정하면 해당 프로젝트를 사용한다
- 프로젝트가 하나뿐이면 자동 선택하되 사용자에게 확인한다
- 여러 개인데 지정이 없으면 목록을 보여주고 선택하게 한다

### 2. 완료 상태 확인

프로젝트의 모든 문서를 읽어 완료 여부를 판단한다:
- `plan.md`의 Done When이 모두 체크되었는지
- 모든 phase의 Acceptance Criteria가 달성되었는지
- 미해결 Blocked 항목이 없는지

완료되지 않은 항목이 있으면 사용자에게 알리고 진행 여부를 확인한다.

### 3. log.md 생성

`template/log.md`를 기반으로, 프로젝트의 전체 이력을 정리한다:

```markdown
# Log: {project_name}

## 기간
- 시작: {yyyyMMdd}
- 종료: {오늘 날짜}

## 목표
- {plan.md의 Goal}

## 완료된 작업
### Phase 1: {objective}
- {주요 완료 작업들}

### Phase 2: {objective}
- {주요 완료 작업들}

...

## 변경 이력
- {handoff들의 Changed 항목 종합}

## 미해결 사항
- {남은 Blocked 항목 또는 "없음"}

## 회고
- {잘된 점}
- {개선할 점}
- {다음에 참고할 사항}
```

회고 섹션은 handoff들의 Changed/Blocked 패턴을 분석하여 초안을 작성하되, 사용자가 직접 보완하도록 안내한다.

### 4. 프로젝트 이동

사용자 확인 후 프로젝트 디렉토리를 `outputs/`로 이동한다.

```bash
mv projects/{yyyyMMdd}_{project_name} outputs/
```

`outputs/`에 동일 이름이 이미 존재하면 사용자에게 알리고 진행 여부를 확인한다.

### 5. 완료 보고

이동 결과를 사용자에게 보여준다:

```
프로젝트 아카이브 완료:
projects/{yyyyMMdd}_{project_name}/  →  outputs/{yyyyMMdd}_{project_name}/

포함 파일:
  ├── plan.md
  ├── phase_1.md ~ phase_{n}.md
  ├── handoff_1.md ~ handoff_{n}.md
  └── log.md (신규 생성)
```

## 주의사항

- 이동 전 반드시 사용자에게 확인한다
- log.md의 회고 섹션을 임의로 완성하지 않는다. 초안을 제시하고 사용자가 보완하게 한다
- 완료되지 않은 프로젝트도 사용자가 명시적으로 요청하면 아카이브할 수 있다. 이 경우 미해결 사항 섹션에 명확히 기록한다
