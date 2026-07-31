---
title: "[해시] 의상"
date: 2026-06-12 00:03:00 +0900
categories: [CodingTest, Hash]
tags: [코딩테스트, 해시, javascript]
---

<div class="problem-page-header">
  <div class="page-header-left">
    <a href="/posts/coding-test-hash" class="back-link">← 해시 문제 목록</a>
    <div class="page-header-type">문제 3 · 의상</div>
  </div>
  <div class="page-header-right">
    <span style="font-size:0.85rem; padding: 0.3rem 0.8rem; border-radius: 20px; font-weight:600; background:#e3f2fd; color:#1565c0;">Lv.2</span>
  </div>
</div>

<div class="problem-card">
  <div class="problem-body">
    <div class="problem-section">
      <h4>문제 설명</h4>
      <p>스파이가 매일 다른 조합의 의상을 입으려 합니다.</p>
      <p>2차원 배열 <code>clothes</code>가 주어지며, 각 원소는 <code>[의상 이름, 의상 종류]</code>로 이루어져 있습니다. 스파이는 하루에 의상 종류별로 최대 한 벌씩만 입을 수 있고, 하루에 최소 한 종류 이상의 의상을 입어야 합니다.</p>
      <p>서로 다른 의상의 조합의 수를 반환하는 <code>solution</code> 함수를 완성하세요.</p>
    </div>
    <div class="problem-section">
      <h4>제한사항</h4>
      <ul>
        <li>1 ≤ clothes의 길이 ≤ 30</li>
        <li>clothes의 각 행은 [의상 이름, 의상 종류]로 이루어져 있습니다.</li>
        <li>스파이가 가진 의상의 수는 1개 이상 30개 이하입니다.</li>
        <li>같은 이름을 가진 의상은 존재하지 않습니다.</li>
        <li>clothes에는 중복된 의상 이름이 없습니다.</li>
        <li>모든 원소는 문자열로 이루어져 있습니다.</li>
        <li>각 문자열의 길이는 1 이상 20 이하이며 알파벳 소문자와 언더바(_)로만 이루어져 있습니다.</li>
      </ul>
    </div>
    <div class="problem-section">
      <h4>입출력 예</h4>
      <table class="io-table">
        <thead>
          <tr><th>clothes</th><th>return</th></tr>
        </thead>
        <tbody>
          <tr><td>[["yellow_hat","headgear"],["blue_sunglasses","eyewear"],["green_turban","headgear"]]</td><td>5</td></tr>
          <tr><td>[["crow_mask","face"],["blue_sunglasses","face"],["smoky_makeup","face"]]</td><td>3</td></tr>
          <tr><td>[["a","top"]]</td><td>1</td></tr>
        </tbody>
      </table>
    </div>
    <div class="problem-section">
      <h4>입출력 예 설명</h4>
      <p><strong>예 1.</strong> headgear 종류 2벌, eyewear 종류 1벌이 있습니다.</p>
      <ul>
        <li>headgear만 입는 경우: yellow_hat, green_turban → 2가지</li>
        <li>eyewear만 입는 경우: blue_sunglasses → 1가지</li>
        <li>headgear + eyewear를 함께 입는 경우: 2 × 1 = 2가지</li>
        <li>합계: 5가지</li>
      </ul>
      <p><strong>예 2.</strong> face 종류만 3벌 있습니다. 하루에 한 벌만 입을 수 있으므로 3가지입니다.</p>
      <p><strong>예 3.</strong> 의상이 1벌뿐이므로 선택지는 1가지입니다.</p>
    </div>
    <div class="solution-area">
      <h4>내 풀이</h4>

```javascript
function solution(clothes) {

}
```

    </div>
    <details class="answer-toggle">
      <summary>해설 보기</summary>

**핵심 아이디어**

각 종류별로 "안 입는 경우"를 포함해 경우의 수를 곱한 뒤, 전부 안 입는 경우 1가지를 빼면 된다.

```
(종류 A 수 + 1) × (종류 B 수 + 1) × ... - 1
```

`+1`은 "해당 종류를 안 입는 선택지", `-1`은 "아무것도 안 입는 경우"를 제거하는 것이다.

```javascript
function solution(clothes) {
  const count = {};
  for (const [, type] of clothes) {
    count[type] = (count[type] || 0) + 1;
  }
  return Object.values(count).reduce((acc, n) => acc * (n + 1), 1) - 1;
}
```

**예 1 추적**

```
headgear: 2개 → (2+1) = 3 (yellow, green, 안 입음)
eyewear:  1개 → (1+1) = 2 (blue, 안 입음)

3 × 2 = 6 → 6 - 1(전부 안 입음) = 5
```

**시간복잡도**: O(N)

> `for (const [, type] of clothes)`에서 `,`는 첫 번째 원소(의상 이름)를 무시하고 두 번째 원소(의상 종류)만 가져오는 구조 분해 패턴이다.
{: .prompt-tip }

    </details>
  </div>
</div>

<div class="problem-pagination">
  <a href="/posts/coding-test-hash-2">
    <span>이전 문제</span>
    <span>← 중복 없는 문자</span>
  </a>
  <div class="next disabled">
    <span>다음 문제</span>
    <span>—</span>
  </div>
</div>
