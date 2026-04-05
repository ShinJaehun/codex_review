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

codex_review/
 ├── bin/
 │    ├── pull_spec
 │    ├── make_review
 │    ├── send_review
 │    └── send_dump
 └── README.md

설정 파일:

~/.config/codex_review/config

---

# 개념 구조

## spec.md

- 현재 작업 계약서
- 프로젝트 루트에 위치
- 항상 하나만 존재
- pull_spec로 overwrite됨
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

git clone <your_repo> ~/project/codex_review

## 2. 실행 권한 부여

chmod +x ~/project/codex_review/bin/*

## 3. PATH 추가 (권장)

export PATH="$HOME/project/codex_review/bin:$PATH"

---

# 설정

파일:

~/.config/codex_review/config

예시:

CODEX_REVIEW_TARGET_USER="shinjaehun"
CODEX_REVIEW_TARGET_HOST="192.168.0.1"
CODEX_REVIEW_TARGET_DIR="~/review_diffs"

CODEX_REVIEW_SPEC_REMOTE="~/review_diffs/spec.md"

CODEX_REVIEW_PROJECT_DIR="$HOME/project/suksuk_project"
CODEX_REVIEW_SPEC_LOCAL="$CODEX_REVIEW_PROJECT_DIR/spec.md"

CODEX_REVIEW_OUT_DIR="/tmp"
CODEX_REVIEW_DUMP_OUT_DIR="/tmp"

---

# 작업 흐름

## 시작

cd ~/project/suksuk_project
pull_spec

## 작업 종료 (review)

send_review HEAD

## 작업 종료 (dump)

send_dump app/controllers

---

# 핵심 구분

send_review = git 기준  
send_dump = 파일시스템 기준

---

# 철학

- spec = 입력
- commit message = 의도
- diff = 구현 증거
- review = Git 기반 패킷
- dump = 파일 내용 기반 패킷

---

# 권장 운영 방식

spec → 구현 → review or dump → 토론 → 다음 spec
