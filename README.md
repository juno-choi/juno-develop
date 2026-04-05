# bdacs-develop

Claude Code 기반 업무 관리 워크스페이스 (Obsidian vault).
PM 기획서 → 개발 계획 → phase 단위 코드 작업 → 아카이브까지 자동화된 워크플로우를 제공한다.

## Structure

```
develop/
├── CLAUDE.md                # Claude Code 프로젝트 가이드
├── context/                 # 사용자 정보, 규칙, 프로젝트 구조, 기술 스택 프로필
│   ├── about-me.md
│   ├── rules/
│   │   └── coding_rule.md   # 코딩 규칙 (프로필 가이드)
│   └── conventions/         # 컨벤션 (프로젝트 구조 + 기술 스택 프로필)
│       ├── project_structure.md
│       ├── java-spring/
│       │   ├── code-writer.md
│       │   └── test-code-writer.md
│       └── typescript-react/
│           ├── code-writer.md
│           └── test-code-writer.md
├── template/                # 문서 템플릿 (plan, phase, phase_fail, handoff, review, log)
├── projects/                # 진행 중인 업무 ({yyyyMMdd}_{project_name}/)
│   └── {project}/
│       ├── plan.md
│       ├── phase_{n}.md
│       ├── phase_fail_{n}.md  (재작업 시)
│       ├── handoff_{n}.md
│       ├── review_{n}.md
│       └── log.md
├── outputs/                 # 종료된 업무 아카이브
└── .claude/
    ├── agents/              # Sub-agent 정의
    └── skills/              # Skill 정의
```

## Workflow

```
/work-plan → /work-phase → /work-review → /work-outputs
```

| Skill            | 설명                                                    |
| ---------------- | ----------------------------------------------------- |
| **work-plan**    | PM 기획서(requirements, PRD, tech_spec)를 참고하여 plan.md 생성 |
| **work-phase**   | plan.md 기반 phase 자동 생성 + 코드 작업 파이프라인 실행 (아래 참조)       |
| **work-review**  | 현재 진행 상황 검증 및 변경사항 확인                                 |
| **work-outputs** | 업무 종료 — log.md 생성 후 outputs/로 아카이브                    |

### work-phase 파이프라인

```
pre-flight → code-writer → build-check+commit
  → code-verifier(diff) → Decision Gate(사용자 판단)
  → test-code-writer+commit → perf/security-reviewer(병렬)
  → code-simplifier+commit → review
```

- 단계별 git checkpoint 관리
- 재작업 카운터 분리: 빌드 / 검증 / 테스트 / decision
- FAIL 유형별 최소 복귀 지점 적용 (recovery-matrix.md)
- Decision Gate: 매 phase 필수, APPROVE까지 반복

## Sub-Agents

### 코드 작업 파이프라인

| Agent | Model | 역할 | 수정 권한 |
|---|---|---|---|
| **code-writer** | opus | CLAUDE.md/REVIEW.md 기반 코드 작성 | O (유일) |
| **test-code-writer** | opus | given/when/then 패턴 테스트 작성 | O (테스트) |
| **code-verifier** | sonnet | 코드 검증 (보고만, Edit/Write 없음) | X |
| **performance-reviewer** | sonnet | 성능 이슈 검토 (N+1, 인덱스, 트랜잭션 등) | X |
| **security-reviewer** | sonnet | 보안 취약점 검토 (OWASP Top 10 등) | X |

### 워크플로우 지원

| Agent | Model | 역할 |
|---|---|---|
| **plan-writer** | sonnet | PM 문서 → plan.md 생성 |
| **work-reviewer** | sonnet | plan/phase/handoff 분석 → 진행 현황 리포트 |
| **log-writer** | sonnet | 프로젝트 문서 종합 → log.md 생성 및 아카이브 |

## Plugins

| Plugin | 용도 |
|---|---|
| **code-review** | PR 코드 리뷰 |
| **skill-creator** | 스킬 생성/수정/평가 |
