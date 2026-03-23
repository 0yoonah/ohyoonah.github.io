---
name: post
description: >
  새 블로그 포스트 파일을 생성하는 스킬.
  사용자가 "포스트 작성", "글 써줘", "포스팅해줘", "새 글 만들어줘" 등을 언급하면 이 스킬을 사용할 것.
---

오늘 날짜를 기준으로 `_posts/` 폴더에 새 블로그 포스트 파일을 생성해줘.

사용자가 제목, 카테고리, 태그, 내용을 입력하면 아래 형식으로 파일을 만들어줘.

## 파일명 규칙
`YYYY-MM-DD-{title을-영어-소문자-kebab-case로}.md`

## frontmatter 형식
```yaml
---
title: {제목}
date: YYYY-MM-DD HH:MM:SS +0900
categories: [{상위 카테고리}, {하위 카테고리}]
tags: [{태그1}, {태그2}]
---
```

## 규칙
- categories는 최대 2개 (Chirpy 제한)
- tags는 소문자
- 내용이 없으면 frontmatter만 작성하고 빈 본문으로 생성
- 사용자가 제목만 주면 나머지는 추론하거나 비워서 생성

$ARGUMENTS
