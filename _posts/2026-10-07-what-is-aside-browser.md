---
title: AI 브라우저 Aside 알아보기
date: 2026-10-07 18:00:00 +0900
categories: [개발, AI]
tags: [ai, aside, browser, agent, claude-code, automation]
---

Claude Code로 코드 작업은 많이 자동화했지만, Slack 메시지 확인이나 Gmail 회신, 클라우드 콘솔 확인처럼 로그인이 필요한 웹 작업은 여전히 직접 해야 했다. API나 MCP 서버가 없는 서비스는 브라우저를 열 수밖에 없었다. Aside는 이런 작업을 맡길 수 있는 브라우저다.

---

## Aside란

[Aside](https://aside.com)는 Chromium 기반의 AI 브라우저다. 확장 프로그램이 아니라 독립된 데스크톱 앱이고, 브라우저 안에 에이전트가 들어 있다.

에이전트는 내가 로그인해 둔 사이트를 직접 조작한다. 서비스별 연동 없이, 사람처럼 페이지를 열고 클릭하고 입력한다.

주요 기능은 네 가지다.

| 기능 | 설명 |
|------|------|
| 브라우저 에이전트 | 메일, 메신저, 문서, 스프레드시트, 대시보드 등에서 하는 일을 대신 수행 |
| 메모리 | 브라우징 기록을 바탕으로 사람, 프로젝트, 자주 쓰는 사이트 같은 맥락을 정리 |
| Aside Vault | 비밀번호를 모델에 노출하지 않고 자동 입력으로 로그인 |
| 승인 단계 | 결제, 게시, 메시지 전송은 사람이 확인한 뒤 실행 |

작업과 메모리는 기기에 암호화되어 저장된다. 쓰던 ChatGPT나 Claude 구독을 연결해서 쓸 수도 있다.

> 메모리가 있어서 "우리 팀 Slack은 이거고, 배포는 Vercel에서 하고" 같은 설명을 매번 하지 않아도 된다.
{: .prompt-info }

---

## CLI로 쓰기

Aside는 CLI를 제공한다. 설치하면 터미널에서 `aside` 명령으로 브라우저 작업을 맡길 수 있다.

```bash
curl -fsSL https://releases.aside.com/install.sh | bash
```

작업은 자연어로 넘긴다.

```bash
aside "Slack #develop 채널에서 안 읽은 메시지 요약해줘"
aside "GitHub 알림 중 나에게 리뷰 요청된 PR 목록 정리해줘"
aside https://github.com/notifications   # URL만 넘기면 브라우저에서 열기
```

작업은 세션 단위로 실행되고, 중간에 방향을 바꾸거나 이어서 시킬 수 있다.

```bash
aside session steer <id> "Google Flights로 바꿔서 찾아줘"
aside session queue <id> "끝나면 결과를 CSV로 내보내줘"
aside session resume <id> "3단계부터 이어서 진행"
```

DOM이나 스크린샷을 직접 다뤄야 하면 `aside repl`로 Playwright 스타일의 JavaScript를 실행한다. 메모리는 `aside memory search`로 조회할 수 있다.

---

## Claude Code와 연결하는 두 방향

Claude Code와 Aside는 양쪽 방향으로 연결할 수 있는데, 목적이 다르다.

| | Claude Code → Aside | Aside → Claude |
|---|---|---|
| 작업을 이끄는 쪽 | Claude Code | Aside |
| Claude의 역할 | 작업을 지시하고 웹 작업만 Aside에 맡김 | Aside 에이전트의 모델 |
| 맞는 상황 | 터미널에서 코딩하다 웹 작업이 필요할 때 | 브라우저에서 바로 작업을 시킬 때 |
| 연결 방법 | 스킬 또는 MCP 서버 | Aside 설정에서 Claude 구독 연결 |

이 글에서는 첫 번째 방향을 다룬다. 두 방향을 같이 쓰면 Claude Code가 Aside를 부르고, Aside 에이전트도 Claude 모델로 돌아간다.

---

## Claude Code에 Aside 붙이기

방법은 두 가지다.

### 스킬로 추가

`~/.claude/skills/aside-browser/SKILL.md`에 스킬 파일을 둔다. 스킬 본문은 "사용 전에 `aside guide`를 실행해서 최신 사용법을 읽어라"는 안내 정도라서, 실제 사용법은 CLI가 알려준다.

```markdown
---
name: aside-browser
version: 3
description: Read when you need a browser, or have to work across user's logged-in websites (e.g. Gmail, Slack, cloud consoles, etc.), or have to refer personal contexts like memory and browsing history.
---

# Aside

This file is only an entry point. You MUST run `aside guide` to read this skill’s actual instructions before using Aside, then follow them.

(설치·업데이트 안내 생략)
```

description에 브라우저, 로그인된 사이트, 개인 맥락이 들어 있어서, 이런 작업이 필요할 때 Claude가 스킬을 읽고 `aside`를 호출한다.

### MCP 서버로 추가

`aside mcp`는 stdio MCP 서버로 동작한다. Claude Code에 등록하면 `exec`(에이전트 실행)와 `repl`(Playwright 스타일 JavaScript 실행) 도구가 생긴다.

```bash
claude mcp add aside -- aside mcp
```

스킬은 CLI를 Bash로 호출하고, MCP는 도구로 호출한다는 차이가 있다. 둘 중 하나만 붙여도 된다.

### 이렇게 쓴다

- 맥락 조회: "이 이슈 누가 처음 제기했지?" 같은 질문에 Aside 메모리를 먼저 검색
- 로그인이 필요한 확인: Vercel 배포 상태, Sentry 에러, 사내 어드민 페이지
- 작업 마무리: PR을 올린 뒤 Slack 리뷰 요청 메시지 초안 작성

MCP 서버가 없는 사내 도구나 SaaS도 브라우저로 열 수 있으면 다룰 수 있다.

---

## 활용 예시

1. 아침 정리: Slack 멘션, 중요 메일, 캘린더를 훑고 요약
2. 반복 폼 입력: 경비 처리, 휴가 신청처럼 매번 같은 페이지를 거치는 작업
3. 비교 정리: 채용 공고, 항공권, 라이브러리 등을 여러 사이트에서 모아 표로 정리
4. 콘솔 점검: 클라우드 콘솔이나 결제 대시보드에서 이상 여부 확인
5. 원격 실행: `aside --host <기기>`로 다른 컴퓨터의 Aside에 작업 넘기기

---

## 한계

- 속도와 안정성: 페이지를 열고 클릭하는 방식이라 API 호출보다 느리고, 사이트 화면이 바뀌면 실패할 수 있다. 공식 API나 MCP가 있는 서비스는 그쪽을 쓰고, Aside는 연동이 없는 사이트나 여러 사이트를 오가는 작업에 쓰는 게 낫다.
- 계정 권한: 에이전트는 내 계정으로 실제 동작을 한다. 잘못된 지시가 그대로 메시지 전송이나 데이터 변경으로 이어질 수 있다.

> 기본 권한 모드(guard)를 유지하고, 메시지 전송이나 결제는 승인 단계에서 내용을 확인하자.
{: .prompt-warning }

---

## 마무리

코드 작업은 Claude Code, 웹 작업은 Aside로 나눠 연결해두면 터미널에서 처리할 수 있는 일이 늘어난다. 처음에는 알림 요약이나 정보 조회 같은 읽기 작업부터 맡기고, 익숙해지면 폼 입력이나 메시지 초안 작성으로 넓혀가면 된다.
