# Develop Vault

Claude Code 기반의 **업무 관리 워크스페이스**입니다.
PM 기획서를 개발 계획으로 변환하고, phase 단위로 업무를 추적하며, 실제 코드 작성까지 연결합니다.

---

## 빠른 시작

```
1. PM 기획서 준비  →  ../pm/projects/{project명}/ 에 requirements, PRD, tech_spec 배치
2. /work-plan       →  기획서를 분석하여 plan.md 생성
3. /work-start      →  plan 기반으로 phase별 업무 시작
4. /work-review     →  진행 상황 검증 및 변경사항 확인
5. /work-outputs    →  업무 종료, log 정리 및 아카이브
```

---

## 디렉토리 구조

```
develop/
├── CLAUDE.md              # Claude Code 지시사항
├── context/               # 프로젝트 설정 및 규칙
│   ├── about-me.md        #   사용자 정보
│   ├── rule.md            #   작업 규칙 (PM 기획서 경로 등)
│   ├── conventions.md     #   코딩 컨벤션
│   └── project_structure.md  # 대상 프로젝트 구조
├── template/              # 파일 생성 시 사용하는 템플릿
│   ├── plan.md
│   ├── phase_{n}.md
│   ├── handoff_{n}.md
│   └── log.md
├── projects/              # 진행 중인 업무
│   └── {yyyyMMdd}_{project name}/
│       ├── plan.md
│       ├── phase_1.md
│       ├── handoff_1.md
│       └── ...
├── outputs/               # 완료된 업무 아카이브
└── reference/             # 참고 자료
```

---

## 워크플로우 상세

### 1단계: 계획 수립 — `/work-plan`

PM 기획서(requirements, PRD, tech_spec)를 분석하여 `plan.md`를 생성합니다.

- **입력**: `../pm/projects/{project명}/` 하위 기획 문서
- **출력**: `projects/{yyyyMMdd}_{project명}/plan.md`
- plan에는 Goal, Inputs, Done When, Steps(phase 목록)가 포함됩니다.

### 2단계: 업무 진행 — `/work-start`

plan의 각 step을 `phase_{n}.md`와 `handoff_{n}.md`로 분해하여 실제 업무를 수행합니다.

- **phase**: 목표, 작업 목록, 제약사항, 완료 조건 정의
- **handoff**: phase 완료 시 결과 기록 (Finished / Blocked / Changed / Next)
- 코드 작성이 필요하면 sub-agent(code-writer, test-code-writer)가 대상 프로젝트에서 작업합니다.

### 3단계: 검증 — `/work-review`

현재 phase의 진행 상황을 점검합니다.

- 완료된 작업과 미완료 작업 확인
- 코드 변경사항 검증
- 계획 대비 변경된 내용 추적

### 4단계: 종료 — `/work-outputs`

업무가 완료되면 `log.md`에 전체 내용을 정리하고 `outputs/`로 아카이브합니다.

---

## Sub-Agent

코드 작업 시 자동으로 전문 에이전트가 투입됩니다.

| Agent | 역할 | 참고 파일 |
|-------|------|-----------|
| **code-writer** | 코드 작성 | 대상 프로젝트의 `CLAUDE.md`, `REVIEW.md`, `conventions.md` |
| **test-code-writer** | 테스트 코드 작성 | given/when/then 패턴 적용 |
| **code-verifier** | 작성된 코드 검증 | 컨벤션 준수 여부 확인 |

---

## 테스트 관련 스킬

개별 레이어 테스트도 직접 요청할 수 있습니다.

| 명령 | 설명 |
|------|------|
| `/domain-test` | 도메인 엔티티 순수 단위 테스트 (JUnit 5 + AssertJ) |
| `/service-test` | Service 계층 통합 테스트 (@SpringBootTest) |
| `/controller-test` | Controller 엔드포인트 테스트 (@WebMvcTest + MockMvc) |
| `/parallel-test` | 위 3개를 병렬 실행 |

---

## 기타 스킬

| 명령 | 설명 |
|------|------|
| `/code-review` | PR 코드 리뷰 |
| `/skill-creator` | 새로운 스킬 생성 및 수정 |
| `/simplify` | 변경된 코드의 품질/효율성 개선 |

---

## 초기 설정

이 워크스페이스를 처음 사용한다면 `context/` 폴더의 파일을 채워주세요.

1. **`context/about-me.md`** — 본인의 역할, 기술 스택, 선호 사항
2. **`context/rule.md`** — PM 기획서 접근 경로 설정
3. **`context/conventions.md`** — 코딩 컨벤션 (선택)
4. **`context/project_structure.md`** — 대상 코드 프로젝트 경로 (선택)

---

## 코딩 컨벤션 (대상 프로젝트 적용)

- **아키텍처**: DDD + MSA, hexagonal / DDD layered architecture
- **인프라**: EDA 기반 (SNS + SQS)
- **Java**: `final` 키워드 사용
- **테스트**: given/when/then 패턴
  - 통합 테스트: service → DB
  - 단위 테스트: domain → 순수 Java
