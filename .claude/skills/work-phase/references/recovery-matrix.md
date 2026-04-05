# 재작업 복귀 지점 매트릭스

FAIL 유형에 따라 최소한의 단계만 재실행하여 토큰과 시간을 절약한다.

## 복귀 지점

| FAIL 발생 위치 | 카운터 | 재실행 경로 | 건너뛰는 단계 |
|---|---|---|---|
| 컴파일 실패 (5-a) | 빌드 | code-writer → compile | — |
| 기존 테스트 실패 (5-b) | 빌드 | code-writer → compile → test | — |
| code-verifier FAIL (6) | 검증 | code-writer → build (5) → verifier (6) → **Decision Gate (7)** | test-writer, reviewers, simplifier |
| Decision Gate MODIFY (7) | decision | code-writer → build (5) → verifier (6) → **Gate (7)** | test-writer, reviewers, simplifier |
| Decision Gate REJECT (7) | decision | revert → code-writer → build (5) → verifier (6) → **Gate (7)** | test-writer, reviewers, simplifier |
| reviewer FAIL (9) | 검증 | code-writer → build (5) → verifier (6) → Gate (7) → test-writer 조건부 (8) → reviewers (9) | simplifier |
| review FAIL (11) | 검증 | code-writer → build (5) → verifier (6) → Gate (7) | test-writer, reviewers, simplifier |
| test-writer 실패 A (8) | 테스트 | test-code-writer 재호출 (8) | — |
| test-writer 실패 B (8) | 검증 | code-writer → build (5) → verifier (6) → Gate (7) → test-writer (8) | reviewers, simplifier |

## 핵심 원칙

- 빌드 실패: 가장 짧은 루프 (code-writer → build만 반복)
- 검증/리뷰 실패 시 simplifier는 항상 건너뛴다 (최종 PASS 후 1회만 실행)
- reviewer FAIL 시 test-writer는 조건부 재실행
- review FAIL 시 reviewer는 건너뛰고 verifier로 재검증
- 재작업 커밋은 `fix` 타입, 모든 재작업 시 `phase_fail_{n}.md` 기록

## test-code-writer 조건부 재실행 규칙

재작업 시 test-code-writer는 기본 건너뛰지만, 다음 중 하나라도 해당하면 재실행:

1. code-writer가 수정한 파일이 **3개 이상**
2. **public 메서드 시그니처 변경** (메서드명, 파라미터 타입/수, 반환 타입)
3. **새로운 클래스 추가**
4. reviewer가 **명시적으로 테스트 보완 요구**

조건 확인: `git diff --stat` + `git diff`로 판단. "수정 범위가 한정적"이라는 가정이 깨진 경우를 포착하기 위함.

## 테스트 실패 분류 기준

| 에러 유형 | 분류 | 처리 |
|---|---|---|
| 컴파일 에러, NPE in test setup, mock 에러 | A (테스트 문제) | test-writer 재호출 (테스트 카운터, 최대 2회) |
| AssertionError, Expected X but was Y, 로직 불일치 | B (프로덕션 버그) | code-writer 재호출 (검증 카운터, 에스컬레이션) |
| 판단 어려운 경우 | B로 분류 | 안전한 쪽으로 |
