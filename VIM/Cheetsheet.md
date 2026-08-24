---
title: VIM cheetsheet
tags: [vim, cheetseet]
type: reference
priority: 3
finished: true
created_date: 2026-04-21
---

# Cheetsheet

## Abstract

Vim Operator + Motion / Text Object Cheat Sheet

## 기본 구조

`operator + motion`
`operator + text object`

Example:

```vim
dw: 단어 삭제
cw: 단어 변경
yiw: 현재 단어 복사
ci": 따옴표 안 내용 변경
dap: 문단 전체 삭제
```

1) Operators

핵심 Operators

|   |   |   |   |
|---|---|---|---|
|Operator|의미|예시|설명|
|d|delete|dw|범위 삭제|
|c|change|cw|범위 삭제 후 insert mode|
|y|yank|yw|범위 복사|
|>|indent|>j|들여쓰기|
|<|outdent|<j|내어쓰기|
|=|auto-indent|=ap|자동 정렬|
|gU|uppercase|gUw|대문자화|
|gu|lowercase|guw|소문자화|
|g~|toggle case|g~w|대소문자 반전|
|!|filter|!ap|외부 명령으로 범위 처리|

줄 단위 Operators

|   |   |
|---|---|
|명령|의미|
|dd|현재 줄 삭제|
|cc|현재 줄 변경|
|yy|현재 줄 복사|
|>>|현재 줄 들여쓰기|
|<<|현재 줄 내어쓰기|
|==|현재 줄 자동 정렬|

2) Motions

단어 이동

|   |   |
|---|---|
|Motion|의미|
|w|다음 단어 시작까지|
|W|공백 기준 다음 단어 시작까지|
|e|현재/다음 단어 끝까지|
|E|공백 기준 단어 끝까지|
|b|이전 단어 시작까지|
|B|공백 기준 이전 단어 시작까지|
|ge|이전 단어 끝으로|
|gE|공백 기준 이전 단어 끝으로|

```vim
예시:
- dw
- ce
- yb
```
  
줄 내 이동

|   |   |
|---|---|
|Motion|의미|
|0|줄 시작|
|^|첫 non-blank 문자|
|$|줄 끝|
|g_|마지막 non-blank 문자|

```vim
예시:
d$
c^
y0
```

문자 찾기

|   |   |
|---|---|
|Motion|의미|
|f<char>|다음 <char>까지|
|F<char>|이전 <char>까지|
|t<char>|다음 <char> 직전까지|
|T<char>|이전 <char> 직후까지|
|;|직전 찾기 반복|
|,|직전 찾기 반대로 반복|

```vim
예시:

- df)
- ct"
- yF,
```

위/아래 이동

|   |   |
|---|---|
|Motion|의미|
|j|아래 줄|
|k|위 줄|
|+|다음 줄 첫 non-blank|
|-|이전 줄 첫 non-blank|
|_|현재 줄 첫 non-blank|

```vim
예시:
dj
ck
y_
```

문장 / 문단 / 블록

|   |   |
|---|---|
|Motion|의미|
|)|다음 문장|
|(|이전 문장|
|}|다음 문단|
|{|이전 문단|
|%|짝 괄호/블록으로 이동|
|gg|파일 시작|
|G|파일 끝|
|nG|n번째 줄로 이동|

```vim
예시:
- d}
- c)
- y%
- dG
```
  

검색 이동

|   |   |
|---|---|
|Motion|의미|
|/pattern|아래 방향 검색|
|?pattern|위 방향 검색|
|n|다음 검색 결과|
|N|반대 방향 결과|
|*|현재 단어 아래 방향 검색|
|#|현재 단어 위 방향 검색|

예시:

d/foo

c?bar

y*

  

3) Text Objects

기본 개념

- i = inside
- a = around

즉:

- iw = 단어 내부
- aw = 단어 전체
- i" = 따옴표 안쪽
- a" = 따옴표 포함 전체
---
  

단어 계열

|   |   |
|---|---|
|Object|의미|
|iw|현재 단어 내부|
|aw|현재 단어 전체|
|iW|공백 기준 단어 내부|
|aW|공백 기준 단어 전체|

```vim
예시:
- diw
- ciw
- yaw
```
  
따옴표 계열

|   |   |
|---|---|
|Object|의미|
|i"|" 안쪽|
|a"|" 포함|
|i'|' 안쪽|
|a'|' 포함|
|i`|백틱 안쪽|
|a`|백틱 포함|

예시:

ci"

da'

yi`

  

괄호 / 브래킷 / 브레이스

|   |   |
|---|---|
|Object|의미|
|i( / ib|() 안쪽|
|a( / ab|() 포함|
|i[|[] 안쪽|
|a[|[] 포함|
|i{ / iB|{} 안쪽|
|a{ / aB|{} 포함|
|i<|<> 안쪽|
|a<|<> 포함|

예시:

ci(

da[

yi{


문단 / 문장 / 태그

|   |   |
|---|---|
|Object|의미|
|ip|문단 내부|
|ap|문단 전체|
|is|문장 내부|
|as|문장 전체|
|it|태그 안쪽|
|at|태그 포함|

예시:

cip

dap

cit

  
4) 자주 쓰는 조합

---

Delete

dw      " 단어 삭제

dd      " 줄 삭제

diw     " 현재 단어 삭제

di"     " 따옴표 안 삭제

d$      " 커서부터 줄 끝까지 삭제

d0      " 커서부터 줄 시작까지 삭제

df,     " 다음 쉼표까지 삭제

dap     " 문단 삭제

---  

Change

cw      " 단어 변경

cc      " 줄 변경

ciw     " 현재 단어 변경

ci"     " 따옴표 안 변경

ci(     " 괄호 안 변경

C       " 커서부터 줄 끝까지 변경

 --- 

Yank

yw      " 단어 복사

yy      " 줄 복사

yiw     " 현재 단어 복사

yap     " 문단 복사

y$      " 커서부터 줄 끝까지 복사

 --- 

Indent / Format

>>      " 현재 줄 들여쓰기

<<      " 현재 줄 내어쓰기

>ap     " 문단 들여쓰기

=ap     " 문단 정렬

=%      " 블록 정렬

 --- 

Case

gUw     " 단어 대문자화

guiw    " 현재 단어 소문자화

gUap    " 문단 대문자화

g~$     " 줄 끝까지 대소문자 반전

 --- 

5) Count와 함께 쓰기

숫자를 앞에 붙여 반복 가능.

d3w     " 단어 3개 삭제

3dd     " 3줄 삭제

5yy     " 5줄 복사

2>>     " 2줄 들여쓰기

 --- 

6) Visual Mode와 결합

v...d   " 선택 영역 삭제

v...c   " 선택 영역 변경

v...y   " 선택 영역 복사

v...>   " 선택 영역 들여쓰기

v...=   " 선택 영역 정렬

예시:

viw

c

또는

vip=

  

7) 실전 암기 우선순위

Operator 먼저

d c y > < =

Motion 먼저

w b e 0 ^ $ f t % gg G

Text Object 먼저

iw aw i" a" i' a' i( a( i[ a[ i{ a{ ip ap

  

8) 가장 많이 쓰는 실전 패턴

단어

diw

ciw

yiw

따옴표 / 괄호

ci"

ci'

ci(

di[

da{

줄

dd

cc

yy

D

C

문단

dap

cip

yap

=ap

  

9) 한 줄 요약

operator = 무엇을 할지

motion / text object = 어디에 할지

예시:

dw      " delete + word

ci"     " change + inside quote

yap     " yank + around paragraph

  

10) 초압축 치트시트

d/c/y           " delete / change / yank

w/e/b           " word motions

0/^/$           " line motions

f/F/t/T         " find char

gg/G/%          " file/block motions

  

iw/aw           " word object

i"/a"           " quote object

i(/a(           " paren object

i[/a[           " bracket object

i{/a{           " brace object

ip/ap           " paragraph object
