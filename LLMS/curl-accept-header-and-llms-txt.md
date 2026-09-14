---
title: curl Accept 헤더와 llms.txt
tags:
  - llm
  - http
  - curl
  - mime
aliases:
  - accept header
  - content negotiation
  - llms.txt
  - text/markdown
created_date: 2026-09-14
---

# curl Accept 헤더와 llms.txt

```bash
curl -H "Accept: text/markdown" https://example.com
```

`curl`로 HTTP 요청을 보내면서, 서버에게 **Markdown 형식의 응답을 선호한다**고 전달하는 명령어다.

* `curl`: URL로 HTTP 요청 전송
* `-H`: HTTP Header 추가
* `Accept`: 클라이언트가 받고 싶은 응답 형식을 서버에 전달
* `text/markdown`: Markdown 형식 선호
* `text/plain`: 일반 텍스트 형식 선호

예:

```bash
curl -H "Accept: text/markdown" https://example.com
```

→ 가능하면 Markdown으로 응답 요청

```bash
curl -H "Accept: text/plain" https://example.com
```

→ 일반 텍스트로 응답 요청

> [!note]
> `Accept`는 **강제가 아니라 요청**이다. 서버가 해당 형식을 지원하거나 `Accept` 헤더에 따라 응답을 다르게 처리해야 실제 결과가 달라진다. 이 콘텐츠 협상(content negotiation)은 [[LLMS/api-base-url-endpoints|API 엔드포인트]]를 다룰 때와 같은 HTTP 계층의 동작이다.

## llms.txt와의 관계

파일명이 `llms.txt`여도 내용은 Markdown 문법으로 작성될 수 있다. 즉 다음 세 가지는 서로 별개다.

```text
파일명          llms.txt
내용 문법       Markdown
Content-Type    text/plain 또는 text/markdown
```

`.txt`라는 확장자가 있다고 해서 반드시 `text/plain`이어야 하는 것은 아니다.

## Case sensitivity

HTTP 헤더 이름과 MIME 타입은 대소문자를 구분하지 않는다.

```text
Accept
accept
ACCEPT
```

은 같은 의미다.

```text
text/markdown
TEXT/MARKDOWN
```

도 의미상 동일하다. 다만 일반적으로 소문자를 사용한다.

반면 URL 경로는 서버에 따라 대소문자를 구분할 수 있으므로 `/llms.txt`와 `/LLMS.txt`는 다른 주소로 취급될 수 있다.

## 실제 응답 형식 확인

```bash
curl -I -H "Accept: text/markdown" https://example.com/llms.txt
```

응답 헤더의 `Content-Type: text/markdown` 또는 `Content-Type: text/plain`을 보면 서버가 실제로 어떤 형식으로 응답했는지 확인할 수 있다.

## 관련 노트

* curl 기본 사용법은 [[bash-ssh/bash-commands]] 참고
* LLM 사이트가 콘텐츠를 노출하는 방식(엔드포인트 구조)은 [[LLMS/api-base-url-endpoints]] 참고
