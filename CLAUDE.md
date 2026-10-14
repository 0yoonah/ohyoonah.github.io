# ohyoonah.github.io

개인 기술 블로그. Jekyll + Chirpy 테마, GitHub Pages로 배포.

## 프로젝트 구조

- `_posts/` — 블로그 포스트 (파일명: `YYYY-MM-DD-title.md`)
- `_tabs/` — 사이드바 탭 페이지 (about, archives, categories, tags)
- `_data/contact.yml` — 사이드바 소셜 링크
- `_data/authors.yml` — 포스트 작성자 정보
- `assets/img/` — 이미지 파일
- `_config.yml` — 사이트 설정

## 사이트 정보

- 제목: ohyoonah.dev
- 부제목: 개발 기록
- 소유자: ohyoonah
- GitHub: https://github.com/ohyoonah
- 이메일: dhdbsdk33@gmail.com

## 로컬 실행

```bash
bundle exec jekyll serve
```

## 배포

`origin`(0yoonah/ohyoonah.github.io)의 `master`에만 푸시한다. `.github/workflows/pages-deploy.yml`이 사이트를 빌드해서 ohyoonah/ohyoonah.github.io 저장소의 `gh-pages` 브랜치로 올리고, 그 저장소가 https://ohyoonah.github.io 로 서비스한다.

- ohyoonah 저장소에는 소스를 직접 푸시하지 않는다 (`gh-pages`는 배포 결과만 담는다)
- 배포 키: 0yoonah 저장소 시크릿 `OHYOONAH_DEPLOY_KEY` ↔ ohyoonah 저장소 Deploy keys(쓰기 권한)
