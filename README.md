# codex_review

Codex CLI + Web 인터페이스를 병행하여 사용하는 환경에서  
**spec → 구현 → review → 토론** 흐름을 안정적으로 관리하기 위한 Bash 도구 세트입니다.

---

# 목적

이 도구의 목적은 다음 문제를 해결하는 것입니다.

- 코딩 머신 ↔ 웹 인터페이스 머신 분리 작업
- 작업 명세(spec) 동기화
- git diff + commit 메시지 기반 리뷰 패킷 생성
- 현재 작업 트리의 파일 내용을 별도 dump로 전송
- 토큰 절약을 위한 commit 메시지 중심 토론

---

# 구성 요소

```text
codex_review/
 ├── bin/
 │    ├── pull_spec
 │    ├── make_review
 │    ├── send_review
 │    └── send_dump
 └── README.md
```

설정 파일:

```text
~/.config/codex_review/config
```

---

# 개념 구조

## spec.md

- 현재 작업 계약서
- 프로젝트 루트에 위치
- 항상 하나만 존재
- `pull_spec`로 overwrite됨
- git이 변경 이력을 관리

## review 파일

- Git 기준 작업 결과 패킷
- commit 메시지 + stat + diff 포함
- timestamp prefix로 누적 관리
- 기본적으로 git에 포함하지 않음

## dump 파일

- 파일시스템 기준 작업 컨텍스트 패킷
- 특정 디렉토리의 파일 본문 전체를 포함
- timestamp prefix로 누적 관리
- 기본적으로 git에 포함하지 않음

---

# 설치

## 1. 저장소 클론

```bash
git clone <your_repo> ~/project/codex_review
```

## 2. 실행 권한 부여

```bash
chmod +x ~/project/codex_review/bin/*
```

## 3. PATH 추가 (권장)

```bash
export PATH="$HOME/project/codex_review/bin:$PATH"
```

`.bashrc` 또는 `.zshrc`에 추가 추천.

---

# 설정

파일:

```text
~/.config/codex_review/config
```

예시:

```bash
CODEX_REVIEW_TARGET_USER="shinjaehun"
CODEX_REVIEW_TARGET_HOST="192.168.0.1"
CODEX_REVIEW_TARGET_DIR="~/review_diffs"

CODEX_REVIEW_SPEC_REMOTE="~/review_diffs/spec.md"

CODEX_REVIEW_PROJECT_DIR="$HOME/project/suksuk_project"
CODEX_REVIEW_SPEC_LOCAL="$CODEX_REVIEW_PROJECT_DIR/spec.md"

CODEX_REVIEW_OUT_DIR="/tmp"
CODEX_REVIEW_DUMP_OUT_DIR="/tmp"
```

설명:

- `CODEX_REVIEW_TARGET_USER` / `CODEX_REVIEW_TARGET_HOST`  
  원격 리뷰 머신 SSH 접속 정보
- `CODEX_REVIEW_TARGET_DIR`  
  review / dump 파일을 업로드할 원격 디렉토리
- `CODEX_REVIEW_SPEC_REMOTE`  
  원격 spec.md 경로
- `CODEX_REVIEW_PROJECT_DIR`  
  로컬 프로젝트 루트
- `CODEX_REVIEW_SPEC_LOCAL`  
  로컬 spec.md 경로
- `CODEX_REVIEW_OUT_DIR`  
  `make_review` 출력 디렉토리
- `CODEX_REVIEW_DUMP_OUT_DIR`  
  `send_dump`가 생성하는 dump 파일 저장 디렉토리

---

# SSH 준비 (최초 1회)

로컬에서:

```bash
ssh-keygen
ssh-copy-id shinjaehun@192.168.0.1
```

테스트:

```bash
ssh shinjaehun@192.168.0.1
```

비밀번호 없이 접속되면 정상입니다.

---

# 작업 흐름

## 시작

```bash
cd ~/project/suksuk_project
pull_spec
```

동작:

- 원격의 `spec.md`를 로컬 프로젝트 루트로 overwrite
- 오늘 작업 명세 확정

---

## Codex 작업

Codex CLI에서:

```text
spec.md 기준으로 작업 수행
```

---

## 작업 종료: Git 기준 review 보내기

```bash
send_review HEAD
```

또는

```bash
send_review b9e0c65..HEAD
```

동작:

1. `make_review` 실행
2. review 파일 생성
   - timestamp 포함
   - commit 메시지 항상 포함
3. 원격으로 scp 전송

---

## 작업 종료: 현재 파일 내용 dump 보내기

```bash
send_dump app/controllers
```

또는

```bash
send_dump app/views -m 3
send_dump app/controllers -ed concerns
```

동작:

1. 지정 디렉토리 기준 파일 탐색
2. 파일 내용을 하나의 dump 파일로 생성
3. 원격으로 scp 전송

---

# 명령어 설명

## pull_spec

원격의 `spec.md`를 로컬 프로젝트 루트로 가져옵니다.

기본 사용:

```bash
pull_spec
```

직접 경로 지정:

```bash
pull_spec ~/review_diffs/spec.md /path/to/project/spec.md
```

---

## make_review

특정 ref 또는 range의 review 파일을 로컬에 생성합니다.

기본 사용:

```bash
make_review HEAD
make_review 9b59ee1
make_review main...feature/my-branch
```

출력:

- review 파일 경로를 stdout에 출력

---

## send_review

`make_review`를 실행한 뒤 생성된 review 파일을 원격으로 전송합니다.

기본 사용:

```bash
send_review HEAD
send_review b9e0c65..HEAD
```

적합한 상황:

- 커밋 하나 리뷰받고 싶을 때
- 브랜치 diff를 전달하고 싶을 때
- 무엇이 바뀌었는지 중심으로 보여주고 싶을 때

---

## send_dump

특정 디렉토리의 파일 내용을 dump 파일로 만든 뒤 원격으로 전송합니다.

기본 사용:

```bash
send_dump <directory>
```

예시:

```bash
send_dump app
send_dump src -ed bin,__pycache__
send_dump app -m 1
send_dump app -ed bin,tmp -m 2
```

옵션:

- `-ed`, `--exclude-dir <csv>`  
  제외할 디렉토리 이름 목록  
  예: `bin,__pycache__,node_modules`

- `-m`, `--maxdepth <n>`  
  `find -maxdepth`와 동일한 의미의 탐색 깊이 제한

적합한 상황:

- 아직 커밋 안 한 파일까지 보여주고 싶을 때
- 특정 디렉토리 전체 맥락을 전달하고 싶을 때
- LLM에게 실제 파일 본문을 읽히고 싶을 때

---

# send_review vs send_dump

핵심 기준:

- `send_review` = **git 기준**
- `send_dump` = **파일시스템 기준**

## send_review를 쓰는 경우

- 커밋 diff를 리뷰받고 싶다
- 변경 의도와 패치를 함께 보여주고 싶다
- Git 이력 중심으로 설명하고 싶다

예:

```bash
send_review HEAD
send_review main...feature/my-branch
```

## send_dump를 쓰는 경우

- 아직 커밋 안 한 새 파일/수정 파일도 보여주고 싶다
- 특정 디렉토리 구조와 실제 본문을 함께 보여주고 싶다
- diff보다 파일 전체 맥락이 중요하다

예:

```bash
send_dump app/controllers -ed concerns
send_dump app/views -m 3
```

## 둘 다 쓰는 경우

- diff도 중요하고 실제 파일 본문도 같이 필요할 때만 사용

예:

```bash
send_review HEAD
send_dump app/services
```

---

# review 파일 구조

```text
=== REF or RANGE ===

=== COMMIT (subject + body) ===

=== LOG ===

=== STAT ===

=== DIFF ===
```

---

# dump 파일 구조

```text
==== ./path/to/file ====
<file contents>

==== ./another/file ====
<file contents>
```

---

# 토큰 절약 전략

웹 인터페이스에서:

1. 먼저 review 파일의 commit 메시지 섹션만 복사
2. 필요하면 `STAT` 추가
3. 그래도 필요하면 특정 파일 diff만 복사
4. diff만으로 부족하면 `send_dump`로 필요한 디렉토리 본문만 전달

즉:

- 전체 diff를 항상 붙일 필요는 없음
- 파일 전체를 항상 붙일 필요도 없음
- review와 dump를 상황에 따라 구분해서 사용

---

# spec 위치

프로젝트 루트:

```text
$HOME/project/suksuk_project/spec.md
```

항상 이 파일이 현재 작업의 SSOT입니다.

---

# review 파일 위치

기본:

```text
/tmp
```

원하면 config에서 변경 가능:

```bash
CODEX_REVIEW_OUT_DIR="$HOME/review_diffs_local"
```

---

# dump 파일 위치

기본:

```text
/tmp
```

원하면 config에서 변경 가능:

```bash
CODEX_REVIEW_DUMP_OUT_DIR="$HOME/review_dumps_local"
```

---

# Troubleshooting

## pull_spec 에러

- SSH 키 확인
- remote spec 경로 확인

```bash
ssh shinjaehun@192.168.0.1
ls ~/review_diffs
```

---

## send_review 실패

- 원격 디렉토리 권한 확인
- scp 직접 테스트

```bash
scp test.txt shinjaehun@192.168.0.1:~/review_diffs
```

---

## send_dump 결과가 비어 있음

예를 들어 `app/views`에서:

```bash
send_dump . -m 1
```

결과가 0KB일 수 있습니다.

이유:

- 현재 디렉토리 바로 아래에 파일이 없고
- 하위 디렉토리만 존재하면
- `-m 1`에서는 파일이 잡히지 않음

이 경우 탐색 깊이를 늘려서 확인합니다.

예:

```bash
send_dump . -m 3
```

---

## exclude-dir가 기대대로 동작하지 않음

`-ed`는 디렉토리 이름 exact match 기준입니다.

예를 들어 실제 디렉토리가 `concerns`인데

```bash
send_dump . -ed concern
```

처럼 입력하면 제외되지 않습니다.

올바른 예:

```bash
send_dump . -ed concerns
```

---

# 철학

- spec = 입력(계약)
- commit message = 의도
- diff = 구현 증거
- review = Git 기준 교환 패킷
- dump = 파일 내용 기준 교환 패킷
- git = 과거 기록 저장소

---

# 권장 운영 방식

로컬: 구현 전용  
원격: 설계 + 토론 전용

기본 루프:

```text
spec → 구현 → review or dump → 토론 → 다음 spec
```

상황에 따라:

- 변경사항 리뷰면 `send_review`
- 파일 본문 컨텍스트면 `send_dump`
- 둘 다 필요하면 둘 다 사용
