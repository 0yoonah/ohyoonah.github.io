---
title: 프론트엔드 성능 최적화, 어디서부터 시작할까 (2) — 번들 최적화
date: 2026-05-21 01:00:00 +0900
categories: [개발, 성능]
tags: [performance, frontend, webpack, code-splitting, lazy-loading, tree-shaking]
---

[1편](/posts/frontend-performance-optimization-1)에서는 무엇을 측정해야 하는지, 어떤 도구로 현황을 파악하는지 정리했다. 이번 글에서는 번들 크기를 실제로 줄이는 방법을 다룬다.

---

## 번들 크기가 왜 문제인가

SPA(Single Page Application)는 기본적으로 애플리케이션의 모든 코드를 하나의 번들로 묶어 내려받는다. 페이지가 하나뿐이라면 괜찮지만, 기능이 늘어날수록 번들 크기도 같이 커진다.

사용자가 로그인 페이지에 접속했을 때 대시보드, 설정, 통계 페이지의 코드까지 전부 내려받는 셈이다. 그 페이지들은 아직 열지도 않았는데.

번들 크기를 줄이는 접근은 크게 세 가지다.

- **코드 스플리팅**: 번들을 여러 청크로 나눠 필요할 때만 불러온다
- **트리 쉐이킹**: 실제로 쓰지 않는 코드를 빌드에서 제거한다
- **라이브러리 교체**: 무거운 라이브러리를 가벼운 대안으로 바꾼다

---

## 코드 스플리팅

코드 스플리팅은 번들을 여러 청크(chunk)로 나누는 것이다. 초기 로딩에는 꼭 필요한 코드만 내려받고, 나머지는 필요한 시점에 불러온다.

#### 라우트 기반 스플리팅

가장 효과적인 방법이다. 페이지 단위로 코드를 나눠서, 해당 페이지에 진입할 때만 그 코드를 불러온다.

React에서는 `React.lazy`와 `Suspense`를 조합해서 구현한다.

```tsx
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

`import()`는 동적 임포트(dynamic import)로, 함수를 호출하는 시점에 해당 모듈을 비동기로 불러온다. Webpack은 이 구문을 보고 해당 코드를 별도 청크로 분리한다.

> `Suspense`의 `fallback`은 청크가 로딩되는 동안 보여줄 UI다. 너무 오래 보이면 오히려 이상해 보이므로, 스켈레톤이나 스피너를 적절히 사용하는 것이 좋다.
{: .prompt-tip }

#### 컴포넌트 기반 스플리팅

라우트 외에도 무거운 컴포넌트를 필요할 때만 불러오는 방식으로 쓸 수 있다.

```tsx
const HeavyChart = lazy(() => import('./components/HeavyChart'));

function Dashboard() {
  const [showChart, setShowChart] = useState(false);

  return (
    <div>
      <button onClick={() => setShowChart(true)}>차트 보기</button>
      {showChart && (
        <Suspense fallback={<div>차트 로딩 중...</div>}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}
```

차트 라이브러리처럼 무겁지만 초기 화면에 필요 없는 컴포넌트에 적합하다.

---

## 트리 쉐이킹

트리 쉐이킹은 실제로 사용하지 않는 코드를 번들에서 제거하는 최적화다. 나무를 흔들어 죽은 잎을 떨어뜨리는 것에서 이름이 붙었다.

Webpack과 같은 번들러가 ES Module(`import`/`export`) 구문을 분석해서 어떤 코드가 실제로 참조되는지 파악하고, 참조되지 않는 코드는 빌드 결과에서 제외한다.

#### import 방식이 중요하다

트리 쉐이킹이 제대로 동작하려면 import 방식이 맞아야 한다.

```js
// 전체를 불러오면 트리 쉐이킹이 안 된다
import _ from 'lodash';
const result = _.get(obj, 'a.b');

// 필요한 함수만 불러와야 한다
import { get } from 'lodash-es';
const result = get(obj, 'a.b');
```

`lodash`는 CommonJS 모듈이라 트리 쉐이킹이 제한적이다. `lodash-es`는 ES Module 버전이라 개별 함수만 번들에 포함된다.

#### side effect 설정 확인

라이브러리가 `package.json`에 `"sideEffects": false`를 명시하고 있으면 번들러가 더 공격적으로 트리 쉐이킹을 수행한다. 직접 만든 유틸 파일도 side effect가 없다면 명시해두는 것이 좋다.

```json
// package.json
{
  "sideEffects": ["*.css", "*.scss"]
}
```

CSS 파일은 import 자체가 side effect이므로 예외로 지정한다.

---

## 라이브러리 교체 검토

번들 분석을 해보면 특정 라이브러리 하나가 전체의 상당 부분을 차지하는 경우가 있다. 이 경우 더 가벼운 대안으로 교체하는 것이 효과적이다.

자주 등장하는 케이스들이다.

**moment.js → day.js**

`moment.js`는 번들 크기가 약 67KB(gzip)다. `day.js`는 동일한 API를 제공하면서 2KB 수준이다.

```bash
# moment.js 제거
npm uninstall moment

# day.js 설치
npm install dayjs
```

API가 거의 동일해서 마이그레이션 비용이 크지 않다.

**lodash → 네이티브 또는 lodash-es**

ES2020 이후로 네이티브 JS가 lodash의 많은 기능을 대체할 수 있다. `Array.flat()`, `Object.fromEntries()`, `Optional chaining` 등으로 의존성 자체를 없앨 수 있는 경우도 많다.

전체를 교체하기 어렵다면 `lodash-es`로 전환해서 트리 쉐이킹이 되도록 하는 것이 차선이다.

> 라이브러리 교체는 신중하게 해야 한다. 동작이 미묘하게 다른 경우가 있어서, 교체 전후로 충분히 테스트하는 것이 필요하다.
{: .prompt-warning }

---

## 번들 최적화 전후 비교

최적화를 진행했을 때 어느 정도 효과를 기대할 수 있는지 실제 수치로 확인하고 기록하는 것이 중요하다.

webpack-bundle-analyzer로 최적화 전후 스크린샷을 찍어두거나, 빌드 결과의 번들 크기 변화를 기록해두면 팀 내에서 공유하기도 쉽다.

```bash
# 빌드 결과 크기 확인
ls -lh dist/static/js/
```

Lighthouse에서 "사용하지 않는 JavaScript 제거" 항목의 절감 예상치가 줄어드는 것도 하나의 지표가 된다.

---

번들 크기를 줄이면 초기 로딩 속도는 확실히 개선된다. 3편에서는 번들 크기와 별개로 렌더링 자체를 최적화하는 방법, 불필요한 리렌더링을 줄이는 React 메모이제이션을 다룰 예정이다.
