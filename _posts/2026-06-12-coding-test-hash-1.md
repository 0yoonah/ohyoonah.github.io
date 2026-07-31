---
title: "[해시] 두 배열의 공통 원소"
date: 2026-06-12 00:01:00 +0900
categories: [CodingTest, Hash]
tags: [코딩테스트, 해시, javascript]
---

<div class="problem-page-header">
  <div class="page-header-left">
    <a href="/posts/coding-test-hash" class="back-link">← 해시 문제 목록</a>
    <div class="page-header-type">문제 1 · 두 배열의 공통 원소</div>
  </div>
  <div class="page-header-right">
    <span style="font-size:0.85rem; padding: 0.3rem 0.8rem; border-radius: 20px; font-weight:600; background:#e8f5e9; color:#2e7d32;">Lv.1</span>
  </div>
</div>

<div class="problem-card">
  <div class="problem-body">
    <div class="problem-section">
      <h4>문제 설명</h4>
      <p>두 정수 배열 <code>arr1</code>, <code>arr2</code>가 주어집니다. 두 배열에 공통으로 존재하는 원소들을 오름차순으로 정렬하여 반환하는 <code>solution</code> 함수를 완성하세요.</p>
    </div>
    <div class="problem-section">
      <h4>제한사항</h4>
      <ul>
        <li>1 ≤ arr1의 길이 ≤ 100,000</li>
        <li>1 ≤ arr2의 길이 ≤ 100,000</li>
        <li>1 ≤ 각 원소 ≤ 1,000,000</li>
        <li>arr1, arr2 각각에는 중복된 원소가 없습니다.</li>
        <li>공통 원소가 없는 경우 빈 배열을 반환합니다.</li>
      </ul>
    </div>
    <div class="problem-section">
      <h4>입출력 예</h4>
      <table class="io-table">
        <thead>
          <tr><th>arr1</th><th>arr2</th><th>return</th></tr>
        </thead>
        <tbody>
          <tr><td>[1, 2, 3, 4, 5]</td><td>[3, 4, 5, 6, 7]</td><td>[3, 4, 5]</td></tr>
          <tr><td>[1, 2, 3]</td><td>[4, 5, 6]</td><td>[]</td></tr>
          <tr><td>[5, 100, 23, 7]</td><td>[7, 23, 100, 9]</td><td>[7, 23, 100]</td></tr>
          <tr><td>[1]</td><td>[1]</td><td>[1]</td></tr>
        </tbody>
      </table>
    </div>
    <div class="problem-section">
      <h4>입출력 예 설명</h4>
      <p><strong>예 1.</strong> arr1과 arr2에 공통으로 존재하는 원소는 3, 4, 5이므로 오름차순으로 정렬하여 [3, 4, 5]를 반환합니다.</p>
      <p><strong>예 2.</strong> 공통 원소가 없으므로 빈 배열 []을 반환합니다.</p>
      <p><strong>예 3.</strong> 공통 원소는 7, 23, 100이므로 오름차순 정렬하여 [7, 23, 100]을 반환합니다.</p>
    </div>
    <div class="solution-area">
      <h4>내 풀이</h4>

```javascript
function solution(arr1, arr2) {

}
```

    </div>
    <details class="answer-toggle">
      <summary>해설 보기</summary>

**핵심 아이디어**

`arr2`를 `Set`으로 변환해두면 `arr1`의 각 원소가 `arr2`에 있는지를 O(1)로 확인할 수 있다. `filter()` 안에서 `arr2.includes()`를 쓰면 O(N²)이 되므로 `Set` 사용이 핵심이다.

```javascript
function solution(arr1, arr2) {
  const set = new Set(arr2);
  return arr1.filter(n => set.has(n)).sort((a, b) => a - b);
}
```

**시간복잡도**: O(N log N) — 정렬이 지배

> `sort()` 에 비교 함수를 넘기지 않으면 숫자가 문자열 기준으로 정렬된다. 반드시 `(a, b) => a - b`를 써야 한다.
{: .prompt-warning }

    </details>
  </div>
</div>

<div class="problem-pagination">
  <div class="disabled">
    <span>이전 문제</span>
    <span>—</span>
  </div>
  <a href="/posts/coding-test-hash-2" class="next">
    <span>다음 문제</span>
    <span>중복 없는 문자 →</span>
  </a>
</div>
