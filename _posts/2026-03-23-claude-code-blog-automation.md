---
title: Claude Code Skills로 Jekyll 블로그 포스팅 자동화하기
date: 2026-03-23 00:00:00 +0900
categories: [AI, Claude]
tags: [claude, ai, claude-code, automation, jekyll]
---

## 왜 자동화인가

블로그에 글을 쓰는 것보다 글 쓸 환경을 만드는 데 더 많은 시간을 쓰는 사람들이 있다. 나도 그 중 하나였다. Jekyll + Chirpy 테마로 블로그를 세팅하고 나서 깨달은 건, 포스트 하나 만들려면 `YYYY-MM-DD-title.md` 파일을 손으로 만들고, frontmatter를 복붙하고, 날짜를 고치는 반복 작업이 생긴다는 거다.

Claude Code의 **Skills** 기능을 이용하면 이 작업을 자연어 명령 하나로 끝낼 수 있다.

## Claude Code Skills

Skills는 프로젝트 내 `.claude/skills/{skill-name}/SKILL.md` 파일에 프롬프트를 작성해두면, `/skill-name` 슬래시 커맨드로 호출할 수 있는 기능이다.

```
.claude/
└── skills/
    └── post/
        └── SKILL.md   ← 포스트 생성 로직을 여기에 작성
```

`SKILL.md`에는 파일명 규칙, frontmatter 형식, 동작 규칙 등을 자연어로 기술한다. Claude가 이 지시를 읽고 실제 파일을 생성해준다.

## 사용 방법

터미널에서 Claude Code를 열고 슬래시 커맨드를 입력한다.

```
/post
```

그러면 Claude가 제목, 카테고리, 태그, 내용을 물어본다. 원하는 정보를 입력하면 `_posts/` 디렉토리에 올바른 파일명과 frontmatter가 채워진 마크다운 파일이 생성된다.

```
1. 제목
2. 카테고리 (최대 2개)
3. 태그
4. 내용 (없으면 빈 본문)
```

제목만 입력해도 나머지는 Claude가 추론해서 채워준다.

## SKILL.md 작성 예시

```markdown
오늘 날짜를 기준으로 `_posts/` 폴더에 새 블로그 포스트 파일을 생성해줘.

## 파일명 규칙
`YYYY-MM-DD-{title을-영어-소문자-kebab-case로}.md`

## frontmatter 형식
---
title: {제목}
date: YYYY-MM-DD HH:MM:SS +0900
categories: [{상위 카테고리}, {하위 카테고리}]
tags: [{태그1}, {태그2}]
---

## 규칙
- categories는 최대 2개 (Chirpy 제한)
- tags는 소문자
- 내용이 없으면 frontmatter만 작성하고 빈 본문으로 생성
- 사용자가 제목만 주면 나머지는 추론하거나 비워서 생성
```

## CLAUDE.md로 컨텍스트 제공

프로젝트 루트에 `CLAUDE.md`를 두면 Claude가 대화 시작 시 이 파일을 읽어 프로젝트 컨텍스트를 파악한다. 블로그 구조, 사이트 정보, 배포 방법 등을 여기에 기술해두면 별도 설명 없이도 일관된 작업이 가능하다.

```markdown
# ohyoonah.github.io

개인 기술 블로그. Jekyll + Chirpy 테마, GitHub Pages로 배포.

## 프로젝트 구조
- `_posts/` — 블로그 포스트 (파일명: `YYYY-MM-DD-title.md`)
...
```

## 결과

이제 블로그 포스트 작성 흐름은 이렇다.

1. Claude Code 실행
2. `/post` 입력
3. 제목/카테고리/태그 입력
4. 생성된 파일에 내용 작성
5. 커밋 & 푸시

반복 작업은 Claude에게 맡기고, 글 쓰는 데만 집중할 수 있는 환경이 만들어졌다.
