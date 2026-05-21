---
title: 프론트엔드 성능 최적화, 어디서부터 시작할까 (3) — 실제 프로젝트에 적용하기
date: 2026-05-21 02:00:00 +0900
categories: [개발, 성능]
tags: [performance, frontend, lighthouse, web-vitals]
---

[1편](/posts/frontend-performance-optimization-1)에서 측정 방법을, [2편](/posts/frontend-performance-optimization-2)에서 번들 최적화 기법을 정리했다. 이번 글부터는 실제 서비스에 직접 적용하면서 기록한다.

---

## 측정 환경 구성

최적화 효과를 빠르게 확인하려면 로컬에서 프로덕션과 유사한 조건으로 측정할 수 있는 환경이 필요하다.

단순히 `yarn dev`로 측정하면 안 되는 이유가 있다. 개발 서버는 소스맵, HMR, 미니파이 없는 코드가 포함되어서 수치가 실서버와 크게 다르게 나온다. 그렇다고 매번 실서버에 배포해서 확인하면 반복이 너무 느리다.

해결 방법으로 두 가지 스크립트를 추가했다.

```bash
yarn build:local   # LOCAL_PROD=true — 프로덕션 빌드, 로컬 서빙용 publicPath
yarn serve:local   # Express 서버로 dist/ 서빙 + API 요청을 실서버로 프록시
```

`serve:local`은 정적 파일은 빌드 결과물에서 서빙하면서, API 요청은 실서버로 포워딩한다. 로컬에서 실행하면서도 실제 데이터로 페이지가 그려진다.

측정은 `http://localhost:3333`에서 Lighthouse를 돌린다.

환경별로 어떤 용도로 쓸지 정리해두었다.

| 상황 | 방법 |
|------|------|
| 개발 중 빠른 동작 확인 | `yarn dev` (localhost:8000) |
| 최적화 단위 완료 후 수치 측정 | `yarn build:local` + `yarn serve:local` |
| 최종 확인 | 실서버 배포 후 측정 |

> localhost:3333에서의 FCP/LCP는 프록시 레이턴시 때문에 실서버보다 높게 나온다. 절대 수치보다는 **최적화 전후 변화량**과 **TBT**를 기준으로 비교한다.
{: .prompt-warning }

---

## 베이스라인 측정

최적화 전 현재 상태를 수치로 기록해둔다. 실서버 기준이다.

Lighthouse를 처음 돌렸을 때 Performance 점수가 2점이었다. SEO는 100점이다. 검색 노출에는 공을 들였지만 정작 들어온 사용자가 어떤 경험을 하는지는 신경 쓰지 못했다는 뜻이기도 하다.

![Lighthouse 베이스라인 측정 결과](/assets/img/lighthouse-baseline.png)

| 지표 | 현재 | 기준(Good) |
|------|------|-----------|
| FCP | 19.2s | 1.8s 이하 |
| LCP | 34.6s | 2.5s 이하 |
| TBT | 2,680ms | 200ms 이하 |
| CLS | 0.965 | 0.1 이하 |

전 항목이 기준치를 크게 벗어나 있다. 특히 FCP와 LCP는 기준의 10배 이상이다.

---

## 무엇이 문제인가

Lighthouse Insights와 Diagnostics 항목을 분석했다. 문제는 여러 개지만, Core Web Vitals에 직접 영향을 주고 개선 방법이 명확한 것들을 1차 수정 대상으로 골랐다.

#### 렌더 블로킹 리소스 — FCP 9.8초 지연

페이지 첫 렌더를 막는 리소스가 여러 개다.

| 리소스 | 블로킹 시간 |
|--------|------------|
| Google Fonts (Noto Sans KR) | 2,551ms |
| 메인 CSS | 1,205ms |
| Sentry (raven.js) | 905ms |
| Pretendard CSS | 756ms |

폰트 CSS가 가장 크다. `fonts.googleapis.com`에서 Noto Sans KR을 불러오는 방식이 렌더를 2.5초 동안 막고 있다. `font-display: swap`을 적용하거나 폰트를 로컬로 내려받는 방향으로 개선할 수 있다.

Sentry 스크립트도 렌더 블로킹으로 잡혔다. `async` 또는 `defer`를 붙이면 렌더를 막지 않는다.

#### LCP 이미지에 lazy loading이 걸려 있다

LCP 요소로 판정된 배너 이미지의 HTML이 이렇다.

```html
<img src="banner.png" loading="lazy" decoding="async" />
```

LCP 이미지에 `loading="lazy"`가 붙어 있으면 브라우저가 해당 이미지를 뒤늦게 로드한다. LCP가 느릴 수밖에 없다. 게다가 `fetchpriority="high"`도 없어서 우선순위가 낮게 처리된다.

수정 방법은 간단하다.

```html
<img src="banner.png" fetchpriority="high" decoding="async" />
```

LCP 요소에는 `loading="lazy"`를 제거하고 `fetchpriority="high"`를 추가한다.

#### Unused JavaScript — 1,848 KiB 절감 가능

코드 스플리팅이 적용되지 않아 사용하지 않는 JS가 초기 로딩에 포함되어 있다. 파일별 현황이다.

| 파일 | 전체 크기 | 미사용 | 비율 |
|------|-----------|--------|------|
| 메인 번들 | 1,271 KB | 760 KB | 60% |
| 비동기 청크 A | 386 KB | 234 KB | 61% |
| 비동기 청크 B | 306 KB | 212 KB | 69% |
| 진입점 번들 | 285 KB | 161 KB | 56% |

메인 번들 하나만 해도 1.2MB에 60%가 미사용이다. `React.lazy`와 `dynamic import`로 라우트 단위 코드 스플리팅을 적용하면 초기 번들 크기를 크게 줄일 수 있다.

#### CLS — 0.965

레이아웃 이동의 주요 원인은 메인 콘텐츠 영역이다. API 응답으로 채워지는 동적 콘텐츠들이 늦게 로딩되면서 레이아웃을 밀어내고 있다. 스켈레톤 UI를 도입하거나 컨테이너에 고정 높이를 지정하는 방식으로 개선할 수 있다.

---

다음 글에서는 각 항목을 실제로 적용하고 전후 수치를 비교한다.
