# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

이 레포지토리는 **코드 프로젝트가 아닌 업무 관리 워크스페이스(Obsidian vault)**이다.
PM 기획서를 기반으로 개발 계획을 수립하고, phase 단위로 업무를 추적하며, 실제 코드 작성은 별도 프로젝트 경로에서 수행한다.

## Workflow (Skills)

업무는 다음 순서로 진행된다. 각 skill은 `/skill명`으로 호출한다.

1. **work-plan** (`/work-plan`) — PM 기획서(`../pm/projects/{project명}/` 하위 requirements, PRD, tech_spec)를 참고하여 `plan.md` 생성. 내부적으로 `plan-writer` agent를 호출한다.
2. **work-phase** (`/work-phase`) — plan.md를 기반으로 phase 파일을 자동 생성하고, 아래 파이프라인으로 코드 작업 실행:
   - pre-flight → code-writer → build-check+commit → code-verifier(diff 기반) → **Decision Gate(사용자 판단)** → test-code-writer+commit → perf/security-reviewer(병렬) → code-simplifier+commit → review
   - 단계별 git checkpoint 관리, 재작업 카운터(빌드/검증/테스트/decision) 분리, FAIL 유형별 최소 복귀 지점 적용
   - review PASS 확정 후 handoff/log 작성 및 다음 phase 자동 전환
3. **work-review** (`/work-review`) — 현재까지 진행된 업무를 검증하고 변경사항 확인. 내부적으로 `work-reviewer` agent를 호출한다.
4. **work-outputs** (`/work-outputs`) — 업무 종료 시 `log.md`를 생성하고 `outputs/`로 아카이브. 내부적으로 `log-writer` agent를 호출한다.

## Directory Structure

- `context/` — 사용자 정보(`about-me.md`), 규칙(`rules/`), 컨벤션(`conventions/` — 프로젝트 구조, 기술 스택 프로필)
- `template/` — plan, phase, phase_fail, handoff, review, log 템플릿 파일. 프로젝트 파일 생성 시 반드시 이 템플릿을 따른다
- `projects/` — 실제 업무 진행 디렉토리. 경로 형식: `{시작일 yyyyMMdd}_{project name}/`
- `outputs/` — 종료된 업무 아카이브

## Rules

- 프로젝트 파일 생성 시 `template/` 폴더의 템플릿을 사용한다
- 프로젝트 경로는 `context/conventions/project_structure.md`를 통해 접근한다
- PM 기획서 경로: `../pm/projects/{project명}/` 하위 requirements, PRD, tech_spec

## Sub-Agents

### 코드 작업 파이프라인 (work-phase에서 사용)

| Agent | Model | 역할 | 코드 수정 권한 |
|---|---|---|---|
| **code-writer** | opus | 대상 프로젝트의 CLAUDE.md/REVIEW.md를 참고하여 코드 작성 | O (유일) |
| **test-code-writer** | opus | 테스트 코드 작성 (given/when/then 패턴) | O (테스트만) |
| **code-verifier** | sonnet | 작성된 코드 검증 — 보고만 수행, 직접 수정하지 않음 | X |
| **performance-reviewer** | sonnet | JPA/Hibernate N+1, 인덱스, 트랜잭션 등 성능 이슈 검토 (읽기 전용 리포트) | X |
| **security-reviewer** | sonnet | OWASP Top 10, 인증/인가, 암호화 등 보안 취약점 검토 (읽기 전용 리포트) | X |

> **code-simplifier**는 work-phase 파이프라인의 최후단에서 1회 실행되는 리팩토링 단계이나, 별도 agent 파일은 아직 미생성 상태이다. code-writer가 이 역할을 겸한다.

### 워크플로우 지원

| Agent | Model | 역할 |
|---|---|---|
| **plan-writer** | sonnet | PM 문서를 읽고 plan.md 생성 (work-plan skill에서 호출) |
| **work-reviewer** | sonnet | plan/phase/handoff 문서를 분석하여 진행 현황 리포트 (work-review skill에서 호출) |
| **log-writer** | sonnet | 프로젝트 문서를 종합하여 log.md 생성 및 아카이브 (work-outputs skill에서 호출) |

## Coding Conventions (실제 코드 프로젝트에 적용)

기술 스택별 코딩 규칙은 `context/conventions/{profile}/` 하위 파일에서 관리한다.
plan.md의 `## Profile` 필드에서 사용할 프로필을 지정한다.

**사용 가능한 프로필:**
- `java-spring` — Java/Kotlin + Spring Boot + DDD + EDA (기본값)
- `typescript-react` — TypeScript + React + Next.js (예시 프로필)
- 새 프로필 추가: `context/conventions/{이름}/` 디렉토리에 `code-writer.md`, `test-code-writer.md` 생성

## Target Projects

실제 코드 프로젝트는 `/Users/junho.choi/Desktop/project/company/` 하위에 위치한다:
- `custody-core` — 핵심 비즈니스 로직 (custody, staking, user, admin)
- `admin-gateway` / `user-gateway` — URL 라우팅 및 JWT 검증
- `admin-service` / `user-service` — admin/user 기능 관리
- `onchain-service` — 가상자산 기능
- `custody-aml` — AML 기능
- `notification` — 알림/슬랙 전송
- `custody-client` — FE 코드
- `terraform` — 인프라 관리
