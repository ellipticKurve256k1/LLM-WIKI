---
title: Himalaya Gmail IMAP SMTP setup
tags:
  - email
  - gmail
  - imap
  - smtp
  - himalaya
aliases:
  - Himalaya Gmail CLI
  - Himalaya IMAP SMTP setup
---

# Himalaya Gmail IMAP SMTP setup

macOS에서 Gmail을 IMAP·SMTP로 다루기 위한 Himalaya CLI 설정 및 검증 기록이다. 이는 Hermes Gateway 이메일 어댑터와 별개인 독립 터미널 이메일 클라이언트다.

## 현재 확인된 상태

- 실행 파일: `/usr/local/bin/himalaya`
- 버전: `himalaya v2.1.0`
- 설정 파일: `~/Library/Application Support/himalaya/config.toml`
- 기본 계정: `gmail` (개인 Gmail 계정)
- 읽기 백엔드: IMAPS `imap.gmail.com:993`
- 발신 백엔드: SMTPS `smtp.gmail.com:465`
- Inbox 별칭: `INBOX`
- IMAP 사서함 목록 조회 성공: Gmail 기본 사서함과 사용자 사서함이 반환됨

> [!success] 검증 범위
> `himalaya mailbox list --json`이 정상 응답했다. 따라서 설치, 설정 파일 로드, 자격 증명 명령 실행, Gmail IMAP 인증 및 사서함 열람까지 확인됐다. 실제 SMTP 발신은 이 기록 시점에 시험하지 않았다.

## 자격 증명 처리

설정은 IMAP·SMTP 인증에 명령 기반 비밀번호 공급자를 사용한다. 비밀번호·앱 비밀번호·토큰·명령 본문은 이 노트나 Git에 기록하지 않는다.

- 설정 파일 권한과 자격 증명 공급자의 보안은 별도로 유지한다.
- 설정을 공유하거나 백업할 때 인증 관련 `passwd`/명령 값을 포함하지 않는다.

## Himalaya v2 명령

설치된 v2.1.0에서는 예전 `folder` 명령 대신 `mailbox`를 사용한다.

```bash
# 설치 및 버전 확인
himalaya --version

# 설정된 계정 확인
himalaya account list

# IMAP 연결 및 사서함 목록 확인
himalaya mailbox list --json

# 특정 계정 지정
himalaya --account gmail mailbox list --json
```

> [!warning] 버전 차이
> 오래된 예제의 `himalaya folder list`는 v2.1.0에서 동작하지 않는다. 이 환경에서는 `himalaya mailbox list`를 사용한다. 자동화 스크립트를 작성할 때도 현재 설치된 명령 도움말(`himalaya <command> --help`)을 우선 확인한다.

## 사용 전 점검

1. `himalaya account list`에서 `gmail`이 기본 계정인지 확인한다.
2. `himalaya mailbox list --json`으로 IMAP 접근을 확인한다.
3. 발신 자동화 전에는 수신자·제목·본문을 검토하고, 테스트 메일은 별도 승인 후 보낸다.
4. SMTP 전송 뒤 오류가 발생해도 무작정 재시도하지 않는다. SMTP는 이미 전달됐지만 사서함 저장 단계가 실패한 경우 중복 발신 위험이 있다.

## 관련 맥락

[[LLMS/zcode-codex-harness-context-analysis]]에는 Codex의 Gmail 플러그인 맥락이 있다. 이 노트의 Himalaya 설정은 그와 별개로 로컬 IMAP·SMTP 프로토콜을 직접 사용하는 터미널 경로다.
