## 기술 스택 프로필

기술 스택별 코딩 규칙은 `context/conventions/{profile}/` 하위에서 관리한다.
plan.md의 `## Profile` 필드로 사용할 프로필을 지정한다.

### 사용 가능한 프로필

| Profile            | 기술 스택                                 | 경로                                   |
| ------------------ | ------------------------------------- | ------------------------------------ |
| `java-spring`      | Java/Kotlin + Spring Boot + DDD + EDA | `context/conventions/java-spring/`      |

### 프로필 구조

각 프로필 디렉토리에는 다음 파일이 포함된다:
- `code-writer.md` — 코드 작성 규칙 (아키텍처, 코딩 표준, 빌드 명령어, 품질 체크리스트)
- `test-code-writer.md` — 테스트 작성 규칙 (테스트 구조, 단위/통합 기준, 빌드 명령어, 품질 체크리스트)

### 새 프로필 추가 방법

1. `context/conventions/{프로필명}/` 디렉토리 생성
2. `code-writer.md` 작성 (필수 섹션: Identity, Architecture, Coding Standards, Build Commands, Quality Checklist)
3. `test-code-writer.md` 작성 (필수 섹션: Identity, Test Structure, Unit/Integration Tests, Build Commands, Quality Checklist)
4. 이 파일의 사용 가능한 프로필 테이블에 추가
