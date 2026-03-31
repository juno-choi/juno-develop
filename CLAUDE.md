# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

이 레포지토리는 **코드 프로젝트가 아닌 업무 관리 워크스페이스(Obsidian vault)**이다.
PM 기획서를 기반으로 개발 계획을 수립하고, phase 단위로 업무를 추적하며, 실제 코드 작성은 별도 프로젝트 경로에서 수행한다.

## Workflow (Skills)

업무는 다음 순서로 진행된다:

1. **work_plan** — PM 기획서(`../pm/projects/{project명}/` 하위 requirements, PRD, tech_spec)를 참고하여 `plan.md` 생성
2. **work_start** — plan에 따라 `phase_{n}.md`와 `handoff_{n}.md`를 생성하며 업무 진행
3. **work_review** — 현재까지 진행된 업무를 검증하고 변경사항 확인
4. **done** — 업무 종료 시 `log.md`에 내용을 export하여 저장

## Directory Structure

- `context/` — 사용자 정보(`about-me.md`), 작업 규칙(`rule.md`), 코딩 컨벤션(`conventions.md`), 프로젝트 구조(`project_structure.md`)
- `template/` — plan, phase, handoff, log 템플릿 파일. 프로젝트 파일 생성 시 반드시 이 템플릿을 따른다
- `projects/` — 실제 업무 진행 디렉토리. 경로 형식: `{시작일 yyyyMMdd}_{project name}/`
- `outputs/` — 종료된 업무 정리
- `reference/` — 참고 자료

## Rules

- 프로젝트 파일 생성 시 `template/` 폴더의 템플릿을 사용한다
- 프로젝트 경로는 `context/project_structure.md`를 통해 접근한다
- PM 기획서 경로: `../pm/projects/{project명}/` 하위 requirements, PRD, tech_spec

## Sub-Agents

코드 작업 시 다음 sub-agent를 활용한다:
- **code-writer** — 대상 프로젝트의 CLAUDE.md 또는 REVIEW.md를 참고하여 코드 작성
- **test-code-writer** — 테스트 코드 작성
- **code-verifier** — 작성된 코드 검증

## Coding Conventions (실제 코드 프로젝트에 적용)

- **아키텍처**: DDD + MSA, hexagonal 또는 DDD layered architecture
- **인프라**: EDA 기반 — SNS + SQS 메시지큐 시스템
- **Java**: `final` 키워드 사용
- **테스트**: given/when/then 패턴
  - 통합 테스트: service layer → DB까지 확인
  - 단위 테스트: domain layer → 순수 Java 테스트

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
