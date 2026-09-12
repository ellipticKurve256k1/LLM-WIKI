---
title: 기본 obsidian-vault setup
type: reference
tags:
  - obsidian
  - settings
  - setup
  - reference
aliases:
  - Obsidian Vault 기본 설정
  - Obsidian Vault Setup
---

# 기본 obsidian-vault setup

## 개요

Obsidian Vault를 일관된 환경으로 구성하기 위한 기본 설정값이다. 설정 적용 여부를 점검할 때는 [[obsidian-vault-required-settings-checklist]]도 함께 참고한다.

## 앱 및 편집기

| 항목 | 설정값 |
| --- | --- |
| 기본 보기 모드 | Reading View (`preview`) |
| 기본 편집 모드 | Source Mode |
| Vim 모드 | 활성화 |
| 편집 모드 상태 표시 | 활성화 |
| 줄 번호 | 표시 |
| 내부 링크 자동 업데이트 | 활성화 |

### PDF 내보내기

| 항목 | 설정값 |
| --- | --- |
| 파일명 포함 | 활성화 |
| 용지 크기 | A4 |
| 방향 | 세로 |
| 여백 | `0` |
| 축소 비율 | 100% |

## 외형

| 항목 | 설정값 |
| --- | --- |
| 테마 | `PLN` 1.18.1 |
| 색상 모드 식별자 | `obsidian` |
| 강조색 | `#f7931a` / `RGB(247, 147, 26)` |
| 본문 글꼴 | `Operator Mono` |
| 인터페이스 글꼴 | `Operator Mono` |
| 기본 글꼴 크기 | 14 |
| 리본 메뉴 | 숨김 |
| CSS 스니펫 | `font-sidebar` 활성화 |

### `font-sidebar` 스니펫

```css
.nav-folder-title,
.nav-file-title {
  font-size: 12px;
}

.nav-folder-title,
.nav-file-title {
  line-height: 1.4;
}
```

## 플러그인

### 커뮤니티 플러그인

| 플러그인 | 버전 | 주요 설정 |
| --- | --- | --- |
| Terminal | 3.23.0 | macOS 통합 터미널에서 `/bin/zsh --login` 사용 |
| Calendar | 1.5.10 | 시스템 로케일 기준 주 시작일, 주간 노트 비활성화 |

Terminal의 기본 프로필은 별도로 고정하지 않는다. 새 터미널은 가로 분할로 열고, 새 인스턴스는 고정하며, 렌더러는 `webgl`을 사용한다.

### 활성화된 코어 플러그인

- File Explorer
- Global Search
- Quick Switcher
- Graph View
- Backlinks
- Canvas
- Outgoing Links
- Tags
- Properties
- Page Preview
- Daily Notes
- Templates
- Note Composer
- Command Palette
- Editor Status
- Bookmarks
- Outline
- Word Count
- File Recovery
- Sync
- Bases

### 비활성화된 코어 플러그인

- Footnotes
- Slash Commands
- Markdown Importer
- Unique Note Creator
- Random Note
- Slides
- Audio Recorder
- Workspaces
- Publish
- Web Viewer

## 단축키

| 단축키 | 동작 |
| --- | --- |
| `Cmd + J` | Vault 루트에서 통합 터미널 열기 |
| `Cmd + D` | 오늘의 Daily Note 생성 |
| `Cmd + Shift + S` | 왼쪽 사이드바 전환 |
| `Cmd + Shift + P` | 오른쪽 사이드바 전환 |
| `Cmd + Shift + E` | Source/Preview 모드 전환 |
| `Cmd + M` | 파일명 변경 |
| `Cmd + Shift + .` | 인용문 전환 |
| `Alt + P` | 탭 고정 전환 |
| `Cmd + R` | 기본 앱으로 열기 |
| `Cmd + Shift + O` | Smart Composer 새 채팅 |

`Cmd + D` 충돌을 방지하기 위해 기본 `Delete paragraph` 단축키는 제거한다.

## Daily Notes 및 템플릿

| 항목 | 설정값 |
| --- | --- |
| Daily Note 파일명 형식 | `YYYY-MM-DD` |
| Daily Note 폴더 | `01_Areas/Personal-Managements/Daily-Logs/2026/05_May` |
| Daily Note 템플릿 | `02_Resources/Templates/dlog-template` |
| 기본 템플릿 폴더 | `02_Resources/Templates` |

> [!warning] Daily Note 폴더 확인
> Daily Note 폴더가 `2026/05_May`로 고정되어 있다. 월별 폴더를 사용한다면 현재 월에 맞게 갱신해야 한다.

## 속성 타입

| 속성 | 타입 |
| --- | --- |
| `aliases` | Aliases |
| `cssclasses` | Multi-text |
| `tags` | Tags |
| `priority` | Number |
| `done` | Checkbox |
| `modified_date` | Date |

## 그래프

| 항목 | 설정값 |
| --- | --- |
| 태그 표시 | 비활성화 |
| 첨부 파일 표시 | 비활성화 |
| 해결되지 않은 링크 숨김 | 비활성화 |
| 고립된 노트 표시 | 활성화 |
| 화살표 표시 | 비활성화 |
| 링크 거리 | 250 |
| 반발 강도 | 10 |

## 확인이 필요한 항목

- `Cmd + R`은 Reveal in Finder가 아니라 `Open with default app` 명령에 연결되어 있다.
- `Cmd + Shift + O`의 Smart Composer 단축키가 남아 있지만 Smart Composer는 활성화된 커뮤니티 플러그인 목록에 없다.
- 동기화 충돌본보다 현재 설정 파일을 우선한다. 현재 설정에는 `font-sidebar`, Terminal, Calendar가 모두 활성화되어 있다.
- Obsidian CLI 실행 파일은 `/Applications/Obsidian.app/Contents/MacOS/obsidian`에 있다.

## 관련 설정 파일

- `.obsidian/app.json`
- `.obsidian/appearance.json`
- `.obsidian/core-plugins.json`
- `.obsidian/community-plugins.json`
- `.obsidian/hotkeys.json`
- `.obsidian/daily-notes.json`
- `.obsidian/templates.json`
- `.obsidian/types.json`
- `.obsidian/graph.json`
- `.obsidian/plugins/calendar/data.json`
- `.obsidian/plugins/terminal/data.json`

