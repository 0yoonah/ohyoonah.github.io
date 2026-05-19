---
title: CLAUDE.md vs AGENTS.md — 무엇이 다른가
date: 2026-05-19 00:00:00 +0900
categories: [AI, Claude]
tags: [claude, claude-code, agents, ai, llm]
---

AI 코딩 어시스턴트를 쓰다 보면 루트 디렉터리에 `CLAUDE.md`나 `AGENTS.md`를 두게 된다. 둘 다 AI에게 프로젝트 컨텍스트를 주는 파일인데, 어떤 차이가 있을까.

---

## 왜 이 파일들이 생겼나

AI 코딩 도구들은 저장소를 처음 열 때 프로젝트가 뭘 하는 곳인지 모른다. 어떤 언어를 쓰는지, 어떤 컨벤션을 따르는지, 건드리면 안 되는 파일이 어딘지. 매번 프롬프트로 알려주는 건 불편하다.

그래서 나온 것이 "AI에게 주는 지시 파일"이다. 저장소 루트에 마크다운 파일을 두면, 도구가 대화 시작 전에 읽어 컨텍스트를 채운다.

---

## CLAUDE.md

`CLAUDE.md`는 Anthropic의 Claude Code가 읽는 파일이다.

Claude Code를 실행하면 현재 디렉터리부터 상위 디렉터리까지 올라가며 `CLAUDE.md`를 찾는다. 홈 디렉터리(`~/.claude/CLAUDE.md`)에 두면 글로벌 설정으로 쓸 수 있다. 하위 디렉터리에도 놓을 수 있어서, 모노레포에서 패키지별로 다른 지시를 줄 수도 있다.

```
project-root/
├── CLAUDE.md          # 프로젝트 전체 지시
├── packages/
│   ├── web/
│   │   └── CLAUDE.md  # 웹 패키지 전용 지시
│   └── api/
│       └── CLAUDE.md  # API 패키지 전용 지시
```

이름에서 알 수 있듯 Claude Code 전용이다. 다른 AI 도구는 이 파일을 읽지 않는다.

---

## AGENTS.md

`AGENTS.md`는 OpenAI가 Codex CLI를 출시하면서 도입한 파일이다.

목적은 같다. 프로젝트 컨텍스트와 지시를 AI에게 전달하는 것. 다만 "특정 도구 전용"이 아니라 **여러 AI 에이전트가 공통으로 읽는 표준**을 목표로 했다.

구조도 비슷하다. 마크다운으로 작성하고, 저장소 루트에 두거나 서브 디렉터리에 배치할 수 있다. 내용도 기술 스택, 컨벤션, 빌드/테스트 명령어, 주의사항 같은 것들로 채운다.

---

## Claude Code는 어느 쪽을 읽나

둘 다 읽는다. 단, 우선순위가 있다.

같은 디렉터리에 `CLAUDE.md`와 `AGENTS.md`가 모두 있으면 **`CLAUDE.md`가 더 높은 우선순위**를 갖는다. `CLAUDE.md`가 없는 경우에 `AGENTS.md`를 읽는다. 이 방식 덕분에 `AGENTS.md`만 쓰는 저장소에서도 Claude Code가 컨텍스트를 얻을 수 있다.

| | CLAUDE.md | AGENTS.md |
|---|---|---|
| 읽는 도구 | Claude Code | OpenAI Codex 등 여러 AI 도구 |
| 목적 | Claude Code 전용 지시 | 범용 AI 에이전트 지시 |
| 제안 주체 | Anthropic | OpenAI |
| Claude Code 우선순위 | 높음 | 낮음 (CLAUDE.md 없을 때) |

---

## 어느 쪽을 써야 하나

**Claude Code만 쓴다면**: `CLAUDE.md` 하나면 충분하다.

**여러 AI 도구를 혼용한다면**: 공통 내용은 `AGENTS.md`에 두고, Claude Code 전용 설정만 `CLAUDE.md`에 따로 두는 방식이 합리적이다. Claude Code는 `CLAUDE.md`를 먼저 읽고, 다른 도구들은 `AGENTS.md`를 읽는다. 두 파일이 공존해도 충돌하지 않는다.

팀 안에서 사람마다 쓰는 도구가 다르다면, `AGENTS.md`에 공통 컨벤션을 관리하는 것이 낫다. 어떤 도구를 쓰든 같은 지시를 받는다.

---

## 마치며

두 파일의 형식과 내용은 거의 같다. 차이는 **누가 읽느냐**다.

`CLAUDE.md`는 Claude Code를 위한 파일이고, `AGENTS.md`는 도구에 상관없이 AI 에이전트 전반이 읽을 수 있도록 설계된 파일이다. AI 코딩 도구가 다양해지면서 `AGENTS.md` 같은 범용 표준의 필요성이 생겼고, Claude Code는 두 파일 모두를 지원하는 방향을 택했다.

어떤 파일을 쓰든, 내용을 잘 채우는 것이 더 중요하다. 빈 `CLAUDE.md`보다 잘 쓴 `AGENTS.md`가 낫다.
