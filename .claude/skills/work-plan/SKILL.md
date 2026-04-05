---
name: work-plan
description: "PM의 requirements, PRD, tech_spec을 참고하여 개발 계획(plan.md)을 생성하는 스킬. '계획 세워', 'plan 만들어', '업무 계획', 'work plan', '개발 계획 작성', '플랜 생성' 등을 요청하거나, PM 기획서를 기반으로 개발 업무를 시작하고 싶을 때 반드시 이 스킬을 사용한다. 새 프로젝트 개발을 시작하거나 PM 문서를 개발 계획으로 변환하고 싶을 때도 트리거된다."
---

# work-plan: 개발 계획 생성

PM vault의 기획서(requirements, PRD, tech_spec)를 읽고, develop vault에 프로젝트 디렉토리와 plan.md를 생성한다.

## 워크플로우

### 1. PM 프로젝트 특정

PM 프로젝트 경로를 확인한다.
- 사용자가 프로젝트 이름을 지정하면 `../pm/projects/` 하위에서 해당 프로젝트를 찾는다
- 지정하지 않은 경우 `../pm/projects/`의 목록을 보여주고 선택하게 한다

### 2. PM 문서 읽기

해당 PM 프로젝트에서 다음 파일들을 읽는다:
- `requirements.md` — 요구사항 (필수)
- `PRD.md` — 기획서 (없을 수 있음)
- `tech_spec.md` — 테크스펙 (없을 수 있음)

requirements.md가 비어있거나 존재하지 않으면, PM 프로젝트에서 먼저 요구사항을 작성해달라고 안내하고 중단한다.

### 3. 대상 코드 프로젝트 확인

`context/conventions/project_structure.md`를 읽어서 실제 코드를 작성할 프로젝트를 특정한다.
- tech_spec에 대상 프로젝트가 명시되어 있으면 해당 프로젝트를 사용한다
- 명시되어 있지 않으면 사용자에게 확인한다
- 여러 프로젝트에 걸친 작업인 경우 모두 기록한다

### 4. develop 프로젝트 디렉토리 생성

`projects/{yyyyMMdd}_{project_name}/` 형식으로 디렉토리를 생성한다.
- `yyyyMMdd`는 오늘 날짜 기준
- `project_name`은 PM 프로젝트와 동일한 이름을 사용한다

### 5. plan.md 생성 (plan-writer Agent)

Agent 도구로 `plan-writer` sub-agent를 호출한다.

프롬프트에 다음을 포함한다:
- PM 프로젝트 경로 (requirements, PRD, tech_spec 파일 위치)
- develop 프로젝트 디렉토리 경로
- 템플릿 경로: `template/plan.md`
- 대상 코드 프로젝트 (이미 특정된 경우)

plan-writer가 plan.md를 생성하고 결과를 보고한다.

### 6. 사용자 리뷰

생성된 plan.md를 요약하여 보여준다:
- 목표
- phase 목록과 각 phase의 핵심 내용
- 대상 코드 프로젝트

수정이 필요한 부분이 있는지 확인한다.

## 주의사항

- PM의 requirements가 없으면 plan을 생성하지 않는다
- tech_spec이 없는 경우 requirements와 PRD만으로 plan을 작성할 수 있지만, phase 분할이 거칠 수 있음을 사용자에게 안내한다
- plan.md의 Steps에 나열된 phase 수와 실제 생성할 phase 파일 수가 일치해야 한다 (phase 파일 생성은 work-start 스킬에서 수행)
- `context/rules/coding_rule.md`의 기술 스택 프로필 정보를 plan의 `## Profile` 필드에 반영한다
