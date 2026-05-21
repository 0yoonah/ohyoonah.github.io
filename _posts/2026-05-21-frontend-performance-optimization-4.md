---
title: 프론트엔드 성능 최적화, 어디서부터 시작할까 (4) — 렌더 블로킹 제거와 LCP 개선
date: 2026-05-21 03:00:00 +0900
categories: [개발, 성능]
tags: [performance, frontend, lighthouse, lcp, render-blocking, font]
---

[3편](/posts/frontend-performance-optimization-3)에서 1차 수정 대상으로 네 가지를 골랐다. 이번 글에서는 그중 두 가지, 렌더 블로킹 제거와 LCP 이미지 수정을 적용한 과정을 기록한다.

---

## 렌더 블로킹 제거

3편 Lighthouse 진단에서 렌더 블로킹 리소스로 잡힌 항목들이다.

| 리소스 | 블로킹 시간 |
|--------|------------|
| Google Fonts (Noto Sans KR) | 2,551ms |
| Sentry (raven.js) | 905ms |
| Pretendard CSS | 756ms |

코드를 파고들다 보니 수치 이면에 더 구체적인 문제가 있었다.

#### Pretendard CDN이 중복으로 선언되어 있었다

CSS 파일 두 곳에서 Pretendard CDN을 import하고 있었다.

```css
@import url('https://cdn.jsdelivr.net/gh/orioncactus/pretendard/dist/web/static/pretendard.css');
```

그런데 별도 폰트 파일에서 이미 서버에 올려둔 woff2 파일을 `font-display: swap`으로 로드하고 있었다. CDN import가 완전히 중복이었다. 두 곳에서 같은 폰트를 불러오니 불필요한 네트워크 요청이 발생하고, CDN 쪽은 렌더를 블로킹하고 있었다.

해결은 간단했다. 중복된 CDN import를 제거하고 기존에 올려둔 폰트 파일만 사용하도록 했다.

#### Noto Sans KR을 제거했다

글로벌 스타일에 Google Fonts에서 Noto Sans KR을 불러오는 import가 있었다.

```css
/* 제거 */
@import url(//fonts.googleapis.com/earlyaccess/notosanskr.css);
```

Lighthouse unused CSS 분석에서 1위로 잡힌 항목으로 203.9 KB가 낭비되고 있었고, Google Fonts CDN 요청이 렌더를 2.5초 블로킹하고 있었다. import 한 줄이 두 문제의 원인이었다.

Pretendard가 이미 한국어를 커버하기 때문에 Noto Sans KR은 폴백으로도 필요 없다. font-family 선언에서도 함께 제거했다.

```css
/* 변경 전 */
font-family: 'Pretendard', 'Noto Sans KR', Roboto;

/* 변경 후 */
font-family: 'Pretendard', Roboto;
```

#### Sentry 스크립트에 defer를 추가했다

```html
<!-- 변경 전 -->
<script src="https://cdn.ravenjs.com/3.23.1/raven.min.js" crossorigin="anonymous"></script>

<!-- 변경 후 -->
<script src="https://cdn.ravenjs.com/3.23.1/raven.min.js" crossorigin="anonymous" defer></script>
```

Sentry는 에러 추적 도구로 초기 렌더와 무관하다. defer 없이 동기 로드하면 HTML 파싱을 멈추고 외부 CDN 응답을 기다린다. `defer`를 붙이면 HTML 파싱과 병렬로 다운로드되고 파싱 완료 후에 실행된다.

초기화는 `window.Raven`을 참조하는 코드에서 이루어지기 때문에 defer로도 정상 동작한다.

---

## LCP 이미지 수정

Lighthouse가 LCP 요소로 지목한 배너 이미지에 `loading="lazy"`가 붙어 있었다.

```tsx
/* 변경 전 */
<img src={bannerUrl} loading="lazy" decoding="async" />

/* 변경 후 */
<img src={bannerUrl} fetchpriority="high" decoding="async" />
```

`loading="lazy"`는 뷰포트 진입 전까지 이미지 로드를 지연하는 속성이다. 스크롤 없이 바로 보이는 배너 이미지가 LCP 요소인데 lazy loading이 걸려 있으면, 브라우저가 가장 중요한 이미지를 의도적으로 늦게 불러오는 모순이 생긴다.

`fetchpriority="high"`를 추가하면 브라우저가 다른 리소스보다 이 이미지를 먼저 요청한다.

---

## 수정 후 측정

localhost:3333(로컬 프로덕션 빌드) 기준으로 수정 전후를 비교했다.

| 지표 | 수정 전 | 수정 후 | 변화 |
|------|---------|---------|------|
| FCP | 41.6s | 40.0s | -1.6s |
| LCP | 82.9s | 75.7s | **-7.2s** |
| TBT | 2,200ms | 2,190ms | -10ms |
| CLS | 1.042 | 0.973 | -0.069 |

LCP가 7.2초 줄었다. 렌더 블로킹 폰트 제거와 LCP 이미지 우선순위 조정이 함께 적용된 결과다. TBT와 CLS는 이번 수정과 직접 관련이 없어서 변화가 거의 없다.

> localhost:3333은 프록시 레이턴시 때문에 FCP/LCP 절대 수치가 실서버보다 크게 나온다. 수치 자체보다 전후 변화량을 기준으로 보는 게 맞다. 실서버 배포 후 측정에서 더 뚜렷한 차이를 확인할 예정이다.
{: .prompt-info }

---

다음 편에서는 번들 분석 결과를 바탕으로 코드 스플리팅을 적용한다.
