---
title: "[해시] 중복 없는 문자"
date: 2026-06-12 00:02:00 +0900
categories: [CodingTest, Hash]
tags: [코딩테스트, 해시, javascript]
---

<div class="problem-page-header">
  <div class="page-header-left">
    <a href="/posts/coding-test-hash" class="back-link">← 해시 문제 목록</a>
    <div class="page-header-type">문제 2 · 중복 없는 문자</div>
  </div>
  <div class="page-header-right">
    <span style="font-size:0.85rem; padding: 0.3rem 0.8rem; border-radius: 20px; font-weight:600; background:#e8f5e9; color:#2e7d32;">Lv.1</span>
  </div>
</div>

<div class="problem-card">
  <div class="problem-body">
    <div class="problem-section">
      <h4>문제 설명</h4>
      <p>소문자 알파벳으로만 이루어진 문자열 <code>s</code>가 주어집니다. <code>s</code>에서 딱 한 번만 등장하는 문자를 앞에서부터 찾아 반환하세요.</p>
      <p>한 번만 등장하는 문자가 여러 개라면 가장 먼저 등장하는 문자를 반환합니다. 한 번만 등장하는 문자가 없다면 <code>"none"</code>을 반환합니다.</p>
    </div>
    <div class="problem-section">
      <h4>제한사항</h4>
      <ul>
        <li>1 ≤ s의 길이 ≤ 100,000</li>
        <li>s는 소문자 알파벳으로만 이루어져 있습니다.</li>
      </ul>
    </div>
    <div class="problem-section">
      <h4>입출력 예</h4>
      <table class="io-table">
        <thead>
          <tr><th>s</th><th>return</th></tr>
        </thead>
        <tbody>
          <tr><td>"aabcdd"</td><td>"b"</td></tr>
          <tr><td>"aabb"</td><td>"none"</td></tr>
          <tr><td>"abcd"</td><td>"a"</td></tr>
          <tr><td>"zzzza"</td><td>"a"</td></tr>
          <tr><td>"abcabc"</td><td>"none"</td></tr>
        </tbody>
      </table>
    </div>
    <div class="problem-section">
      <h4>입출력 예 설명</h4>
      <p><strong>예 1.</strong> "aabcdd"에서 한 번만 등장하는 문자는 b, c입니다. 이 중 앞에 먼저 등장하는 "b"를 반환합니다.</p>
      <p><strong>예 2.</strong> "aabb"에서 모든 문자는 2번씩 등장합니다. 한 번만 등장하는 문자가 없으므로 "none"을 반환합니다.</p>
      <p><strong>예 3.</strong> "abcd"에서 모든 문자가 한 번씩 등장합니다. 가장 먼저 등장하는 "a"를 반환합니다.</p>
      <p><strong>예 4.</strong> "zzzza"에서 한 번만 등장하는 문자는 "a"입니다.</p>
    </div>
    <div class="solution-area">
      <h4>내 풀이</h4>

```javascript
function solution(s) {

}
```

    </div>
    <details class="answer-toggle">
      <summary>해설 보기</summary>

**핵심 아이디어**

두 번 순회한다. 첫 번째 순회에서 각 문자의 등장 횟수를 해시맵에 저장하고, 두 번째 순회에서 원래 순서를 유지하며 카운트가 1인 첫 번째 문자를 반환한다.

```javascript
function solution(s) {
  const count = {};
  for (const ch of s) count[ch] = (count[ch] || 0) + 1;

  for (const ch of s) {
    if (count[ch] === 1) return ch;
  }

  return "none";
}
```

**왜 두 번 순회하는가?**

첫 번째 순회만으로는 "이 문자가 나중에 또 나오는지"를 알 수 없다. 전체를 한 번 훑어서 카운트를 완성한 뒤, 다시 앞에서부터 카운트가 1인 문자를 찾는 방식이 필요하다.

**시간복잡도**: O(N)

    </details>
  </div>
</div>

<div class="problem-pagination">
  <a href="/posts/coding-test-hash-1">
    <span>이전 문제</span>
    <span>← 두 배열의 공통 원소</span>
  </a>
  <a href="/posts/coding-test-hash-3" class="next">
    <span>다음 문제</span>
    <span>의상 →</span>
  </a>
</div>
