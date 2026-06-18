---
title: "[알고리즘] 코딩테스트 기본기 정리"
date: 2026-06-18 02:00:00 +0900
categories: [Study, Algorithm]
tags: [코딩테스트, 시간복잡도, 공간복잡도, 스택, 큐, 재귀, 백트래킹, dfs, bfs, 이진탐색, dp, 트리, 힙, 그리디, 정렬, javascript]
---

배열/문자열/해시에 이어 코딩테스트에서 꼭 알아야 할 핵심 개념들을 한 곳에 정리했다.

---

## 시간복잡도와 공간복잡도

### Big-O 표기법

입력 크기 N이 커질수록 알고리즘이 얼마나 느려지는지를 나타낸다. 상수와 낮은 차수 항은 무시하고 가장 지배적인 항만 남긴다.

```
3N² + 5N + 100  →  O(N²)
```

| 표기 | 이름 | 예시 |
|------|------|------|
| O(1) | 상수 | 배열 인덱스 접근, 해시맵 조회 |
| O(log N) | 로그 | 이진 탐색 |
| O(N) | 선형 | 배열 순회 |
| O(N log N) | 선형 로그 | 정렬 |
| O(N²) | 이차 | 이중 반복문 |
| O(2^N) | 지수 | 부분집합 탐색 |
| O(N!) | 팩토리얼 | 순열 탐색 |

**반복문 패턴으로 복잡도 파악하기**

```javascript
for (let i = 0; i < n; i++) { ... }                        // O(N)
for (let i = 0; i < n; i++) { for (let j ...) { ... } }   // O(N²)
for (let i = n; i > 0; i = Math.floor(i / 2)) { ... }     // O(log N)
```

**N 범위별 허용 복잡도**

| N 범위 | 허용 복잡도 |
|--------|------------|
| N ≤ 11 | O(N!) |
| N ≤ 25 | O(2^N) |
| N ≤ 500 | O(N³) |
| N ≤ 3,000 | O(N²) |
| N ≤ 100,000 | O(N log N) |
| N ≤ 1,000,000 | O(N) |

> 문제의 N 제한을 먼저 확인하고 허용 복잡도 안에서 접근 방식을 선택하자.
{: .prompt-tip }

**공간복잡도** — 알고리즘이 사용하는 메모리 양. 새 배열을 만들면 O(N), N×N 행렬이면 O(N²). 시간복잡도와 보통 트레이드오프 관계다.

---

## 스택과 큐

**스택 (LIFO)** — 나중에 넣은 게 먼저 나온다.

```javascript
const stack = [];
stack.push(1); stack.push(2);
stack.pop();              // 2
stack[stack.length - 1];  // peek: 1
```

**큐 (FIFO)** — 먼저 넣은 게 먼저 나온다.

```javascript
const queue = [];
queue.push(1); queue.push(2);
queue.shift(); // 1
```

> `shift()`는 O(N)이다. 성능이 중요한 경우 head 포인터로 대신한다.
{: .prompt-warning }

**언제 쓰나**
- 스택: DFS, 괄호 유효성, 실행 취소
- 큐: BFS, 작업 대기열

**괄호 유효성 검사**

```javascript
function isValid(s) {
  const stack = [];
  const map = { ')': '(', ']': '[', '}': '{' };
  for (const ch of s) {
    if ('([{'.includes(ch)) stack.push(ch);
    else if (stack.pop() !== map[ch]) return false;
  }
  return stack.length === 0;
}
```

---

## 재귀와 백트래킹

재귀는 자기 자신을 호출하는 것. **기저 조건**이 없으면 스택 오버플로우가 난다.

```javascript
function factorial(n) {
  if (n <= 1) return 1;        // 기저 조건
  return n * factorial(n - 1); // 재귀 호출
}
```

**백트래킹** — 가능한 경우를 탐색하다가 조건 불만족 시 되돌아오는 기법. 핵심은 **선택 → 탐색 → 취소**.

```javascript
function backtrack(state) {
  if (완료조건) { 결과저장; return; }
  for (const choice of 가능한선택들) {
    if (유효하지않으면) continue; // 가지치기
    선택적용(state);
    backtrack(state);
    선택취소(state); // ← 핵심
  }
}
```

**조합 예제**

```javascript
function combinations(nums, k) {
  const result = [];
  function backtrack(start, current) {
    if (current.length === k) { result.push([...current]); return; }
    for (let i = start; i < nums.length; i++) {
      current.push(nums[i]);
      backtrack(i + 1, current);
      current.pop(); // 선택 취소
    }
  }
  backtrack(0, []);
  return result;
}
```

> 순열은 O(N!), 조합은 O(2^N). 가지치기 없으면 완전탐색과 다를 게 없다.
{: .prompt-warning }

---

## DFS / BFS

그래프 탐색의 두 축. 구현 전에 그래프부터 표현한다.

```javascript
// 인접 리스트 (대부분의 문제에서 이 방식)
const graph = { 1: [2, 3], 2: [1, 4], 3: [1], 4: [2] };
```

**DFS** — 깊이 우선. 재귀 또는 스택.

```javascript
function dfs(graph, node, visited = new Set()) {
  visited.add(node);
  for (const next of graph[node] || []) {
    if (!visited.has(next)) dfs(graph, next, visited);
  }
}
```

**BFS** — 너비 우선. 큐.

```javascript
function bfs(graph, start) {
  const visited = new Set([start]);
  const queue = [start];
  while (queue.length) {
    const node = queue.shift();
    for (const next of graph[node] || []) {
      if (!visited.has(next)) { visited.add(next); queue.push(next); }
    }
  }
}
```

**언제 뭘 쓸까**

| | DFS | BFS |
|--|-----|-----|
| **최단 거리** | ❌ | ✅ |
| **경로/조합 탐색** | ✅ | △ |
| **레벨 단위 처리** | ❌ | ✅ |

> **최단 거리 = BFS**, **경로 탐색 = DFS** 로 기억하면 된다.
{: .prompt-tip }

**2차원 그리드 탐색** (섬의 개수, 미로 등)

```javascript
const dx = [0, 0, 1, -1];
const dy = [1, -1, 0, 0];

function dfsGrid(grid, x, y, visited) {
  visited[x][y] = true;
  for (let d = 0; d < 4; d++) {
    const nx = x + dx[d], ny = y + dy[d];
    if (nx < 0 || ny < 0 || nx >= grid.length || ny >= grid[0].length) continue;
    if (visited[nx][ny] || grid[nx][ny] === 0) continue;
    dfsGrid(grid, nx, ny, visited);
  }
}
```

---

## 이진 탐색

정렬된 배열에서 탐색 범위를 절반씩 줄여 O(log N)으로 찾는다.

```javascript
function binarySearch(arr, target) {
  let left = 0, right = arr.length - 1;
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    if (arr[mid] === target) return mid;
    else if (arr[mid] < target) left = mid + 1;
    else right = mid - 1;
  }
  return -1;
}
```

> `left = mid`로 갱신하면 무한 루프가 날 수 있다. 반드시 `mid + 1` / `mid - 1`.
{: .prompt-warning }

**매개변수 탐색** — "최솟값/최댓값을 구하라" 유형을 "X가 가능한가?" Yes/No 함수로 변환해 이진 탐색한다.

```javascript
function solve(arr, m) {
  let left = 최솟값, right = 최댓값, answer = 0;
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    if (isPossible(mid)) { answer = mid; left = mid + 1; }
    else right = mid - 1;
  }
  return answer;
}
```

> 문제에서 "최솟값", "최댓값", "~이상인 최소" 같은 표현이 나오면 이진 탐색을 의심하자.
{: .prompt-tip }

---

## 동적 프로그래밍 (DP)

큰 문제를 작은 부분 문제로 나눠 풀고, 결과를 저장해서 재사용하는 기법.

**적용 조건**
1. 겹치는 부분 문제 (같은 계산이 반복됨)
2. 최적 부분 구조 (부분 문제의 최적해가 전체 최적해를 구성)

**Top-Down (메모이제이션)** — 재귀 + 캐싱

```javascript
function fib(n, memo = {}) {
  if (n <= 1) return n;
  if (memo[n] !== undefined) return memo[n];
  memo[n] = fib(n - 1, memo) + fib(n - 2, memo);
  return memo[n];
}
```

**Bottom-Up (타뷸레이션)** — 반복문 + 배열

```javascript
function fib(n) {
  const dp = [0, 1];
  for (let i = 2; i <= n; i++) dp[i] = dp[i - 1] + dp[i - 2];
  return dp[n];
}
```

**풀이 순서**: 상태 정의 → 점화식 도출 → 초깃값 설정

**계단 오르기** (1칸 또는 2칸)

```javascript
// dp[i] = dp[i-1] + dp[i-2]
function climbStairs(n) {
  const dp = [0, 1, 2];
  for (let i = 3; i <= n; i++) dp[i] = dp[i - 1] + dp[i - 2];
  return dp[n];
}
```

**배낭 문제** (무게/가치가 있는 N개 물건, 용량 W)

```javascript
// dp[i][w] = i번째까지 고려, 무게 w 이하의 최대 가치
function knapsack(weights, values, W) {
  const n = weights.length;
  const dp = Array.from({ length: n + 1 }, () => new Array(W + 1).fill(0));
  for (let i = 1; i <= n; i++) {
    for (let w = 0; w <= W; w++) {
      dp[i][w] = dp[i - 1][w];
      if (w >= weights[i - 1])
        dp[i][w] = Math.max(dp[i][w], dp[i - 1][w - weights[i - 1]] + values[i - 1]);
    }
  }
  return dp[n][W];
}
```

> "최대/최소/경우의 수" + 이전 상태에 의존 → DP를 의심하자.
{: .prompt-tip }

---

## 트리

노드(node)가 간선(edge)으로 연결된 계층적 자료구조. 사이클이 없는 연결 그래프다.

```
        1        ← 루트 (root)
       / \
      2   3      ← 내부 노드
     / \
    4   5        ← 리프 (leaf, 자식 없음)
```

- **깊이(depth)**: 루트에서 해당 노드까지의 거리
- **높이(height)**: 해당 노드에서 가장 깊은 리프까지의 거리
- **이진 트리**: 자식이 최대 2개인 트리

### 트리 순회

DFS 기반 순회 3가지와 BFS 기반 레벨 순회가 있다.

```javascript
// 노드 정의
class TreeNode {
  constructor(val, left = null, right = null) {
    this.val = val; this.left = left; this.right = right;
  }
}
```

**전위 순회 (Preorder)** — 루트 → 왼쪽 → 오른쪽

```javascript
function preorder(node, result = []) {
  if (!node) return result;
  result.push(node.val);       // 루트 먼저
  preorder(node.left, result);
  preorder(node.right, result);
  return result;
}
// 결과: [1, 2, 4, 5, 3]
```

**중위 순회 (Inorder)** — 왼쪽 → 루트 → 오른쪽

```javascript
function inorder(node, result = []) {
  if (!node) return result;
  inorder(node.left, result);
  result.push(node.val);       // 중간
  inorder(node.right, result);
  return result;
}
// 결과: [4, 2, 5, 1, 3]
// BST를 중위 순회하면 오름차순 정렬된 값이 나온다
```

**후위 순회 (Postorder)** — 왼쪽 → 오른쪽 → 루트

```javascript
function postorder(node, result = []) {
  if (!node) return result;
  postorder(node.left, result);
  postorder(node.right, result);
  result.push(node.val);       // 루트 마지막
  return result;
}
// 결과: [4, 5, 2, 3, 1]
```

**레벨 순회 (Level Order)** — BFS로 위에서 아래, 왼쪽에서 오른쪽

```javascript
function levelOrder(root) {
  if (!root) return [];
  const result = [], queue = [root];
  while (queue.length) {
    const level = [];
    const size = queue.length; // 현재 레벨 노드 수
    for (let i = 0; i < size; i++) {
      const node = queue.shift();
      level.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    result.push(level);
  }
  return result;
}
// 결과: [[1], [2, 3], [4, 5]]
```

### 이진 탐색 트리 (BST)

왼쪽 자식 < 부모 < 오른쪽 자식이 항상 성립하는 이진 트리. 탐색/삽입/삭제가 평균 O(log N).

```javascript
function searchBST(root, target) {
  if (!root) return null;
  if (root.val === target) return root;
  return target < root.val
    ? searchBST(root.left, target)
    : searchBST(root.right, target);
}
```

> BST를 중위 순회하면 오름차순 정렬된 결과가 나온다.
{: .prompt-tip }

---

## 힙 (Heap) / 우선순위 큐

**완전 이진 트리** 기반 자료구조로, 최솟값(최소 힙) 또는 최댓값(최대 힙)을 O(1)로 확인할 수 있다. 삽입/삭제는 O(log N).

- **최소 힙**: 부모 ≤ 자식. `pop()`하면 항상 최솟값이 나온다.
- **최대 힙**: 부모 ≥ 자식. `pop()`하면 항상 최댓값이 나온다.

JavaScript에는 내장 힙이 없어서 직접 구현하거나 배열로 시뮬레이션해야 한다.

### 최소 힙 구현

```javascript
class MinHeap {
  constructor() { this.heap = []; }

  push(val) {
    this.heap.push(val);
    this._bubbleUp(this.heap.length - 1);
  }

  pop() {
    const min = this.heap[0];
    const last = this.heap.pop();
    if (this.heap.length) {
      this.heap[0] = last;
      this._sinkDown(0);
    }
    return min;
  }

  peek() { return this.heap[0]; }
  size() { return this.heap.length; }

  _bubbleUp(i) {
    while (i > 0) {
      const parent = Math.floor((i - 1) / 2);
      if (this.heap[parent] <= this.heap[i]) break;
      [this.heap[parent], this.heap[i]] = [this.heap[i], this.heap[parent]];
      i = parent;
    }
  }

  _sinkDown(i) {
    const n = this.heap.length;
    while (true) {
      let smallest = i;
      const left = 2 * i + 1, right = 2 * i + 2;
      if (left < n && this.heap[left] < this.heap[smallest]) smallest = left;
      if (right < n && this.heap[right] < this.heap[smallest]) smallest = right;
      if (smallest === i) break;
      [this.heap[smallest], this.heap[i]] = [this.heap[i], this.heap[smallest]];
      i = smallest;
    }
  }
}
```

### 주요 사용 패턴

**Top-K 문제** — N개 중 K번째로 작은 값 찾기

```javascript
function kthSmallest(nums, k) {
  const heap = new MinHeap();
  for (const n of nums) heap.push(n);
  let result;
  for (let i = 0; i < k; i++) result = heap.pop();
  return result;
}
```

> 최댓값 힙이 필요하면 값에 `-1`을 곱해서 최소 힙에 넣고, 꺼낼 때 다시 `-1`을 곱하는 방법을 자주 쓴다.
{: .prompt-tip }

---

## 정렬

JavaScript 내장 `sort()`는 기본적으로 **문자열 기준**으로 정렬한다. 숫자 배열은 반드시 비교 함수를 넘겨야 한다.

```javascript
[10, 9, 2, 1].sort()              // [1, 10, 2, 9] ← 잘못됨
[10, 9, 2, 1].sort((a, b) => a - b) // [1, 2, 9, 10] ← 오름차순
[10, 9, 2, 1].sort((a, b) => b - a) // [10, 9, 2, 1] ← 내림차순
```

**비교 함수 규칙**

| 반환값 | 동작 |
|--------|------|
| 음수 | a를 b 앞에 |
| 양수 | b를 a 앞에 |
| 0 | 순서 유지 |

**다중 조건 정렬** — 1순위 같으면 2순위로 비교

```javascript
// 길이 오름차순, 같으면 알파벳 오름차순
words.sort((a, b) => a.length - b.length || a.localeCompare(b));

// 점수 내림차순, 같으면 이름 오름차순
arr.sort((a, b) => b.score - a.score || a.name.localeCompare(b.name));
```

**객체 배열 정렬**

```javascript
const people = [
  { name: 'Alice', age: 30 },
  { name: 'Bob', age: 25 },
];
people.sort((a, b) => a.age - b.age); // 나이 오름차순
```

> JavaScript의 `sort()`는 O(N log N)이고 안정 정렬(stable sort)이다. 같은 값의 원래 순서가 유지된다.
{: .prompt-tip }

---

## 그리디 (Greedy)

매 순간 **현재 상태에서 가장 좋아 보이는 선택**을 반복해서 최적해를 구하는 방법이다.

**DP와의 차이**

| | 그리디 | DP |
|--|--------|-----|
| **접근** | 현재 최선만 선택 | 모든 경우 저장 후 비교 |
| **속도** | 빠름 | 상대적으로 느림 |
| **적용 조건** | 탐욕적 선택 속성 필요 | 겹치는 부분 문제 |

그리디가 항상 정답을 보장하지는 않는다. **탐욕적 선택이 전체 최적해로 이어진다**는 게 수학적으로 성립할 때만 쓸 수 있다.

> "항상 최적해가 나올까?"를 먼저 따져봐야 한다. 확신이 없으면 DP가 안전하다.
{: .prompt-warning }

**거스름돈 문제**

가장 큰 단위 동전부터 최대한 사용하면 동전 개수가 최소가 된다.

```javascript
function minCoins(amount, coins) {
  coins.sort((a, b) => b - a); // 큰 단위부터
  let count = 0;
  for (const coin of coins) {
    count += Math.floor(amount / coin);
    amount %= coin;
  }
  return amount === 0 ? count : -1; // -1: 거슬러줄 수 없음
}

minCoins(1260, [500, 100, 50, 10]); // 6 (500×2 + 100×2 + 50×1 + 10×1)
```

> 동전 단위가 배수 관계일 때만 그리디가 성립한다. [500, 400, 100] 같은 단위면 틀릴 수 있다.
{: .prompt-warning }

**회의실 배정**

회의 시간이 겹치지 않게 가장 많은 회의를 배정하는 문제. **끝나는 시간이 빠른 순**으로 정렬 후 탐욕적으로 선택한다.

```javascript
function maxMeetings(meetings) {
  meetings.sort((a, b) => a[1] - b[1]); // 종료 시간 오름차순
  let count = 1;
  let lastEnd = meetings[0][1];

  for (let i = 1; i < meetings.length; i++) {
    if (meetings[i][0] >= lastEnd) { // 시작 >= 이전 종료
      count++;
      lastEnd = meetings[i][1];
    }
  }
  return count;
}

// [[시작, 종료], ...]
maxMeetings([[1,4],[3,5],[0,6],[5,7],[3,8],[5,9],[6,10],[8,11],[8,12],[2,13],[12,14]]);
// 4
```

**그리디 적용 신호**

- 정렬 후 순서대로 처리하면 최적해가 나오는 구조
- 선택을 번복할 필요가 없는 문제
- "최소 횟수", "최대 개수"처럼 단순 최적화를 요구하는 경우

---

## 한눈에 보기

| 주제 | 핵심 자료구조 | 시간복잡도 | 대표 유형 |
|------|-------------|-----------|----------|
| 스택 | 배열 | O(1) push/pop | 괄호, 모노토닉 스택 |
| 큐 | 배열 | O(1) push / O(N) shift | BFS, 작업 대기열 |
| 재귀 | 콜 스택 | 문제마다 다름 | 분할 정복, 트리 |
| 백트래킹 | 재귀 | O(N!) 최악 | 순열, 조합, N-Queens |
| DFS | 스택/재귀 | O(V+E) | 연결 요소, 경로 탐색 |
| BFS | 큐 | O(V+E) | 최단 거리, 레벨 탐색 |
| 이진 탐색 | — | O(log N) | 정렬 배열 탐색, 최적화 |
| DP | 배열 | 문제마다 다름 | 최적값, 경우의 수 |
| 트리 순회 | 재귀/큐 | O(N) | 전위/중위/후위/레벨 |
| 힙 | 배열 | O(log N) push/pop | Top-K, 우선순위 처리 |
| 정렬 | — | O(N log N) | 커스텀 정렬, 다중 조건 |
| 그리디 | — | 문제마다 다름 | 거스름돈, 회의실 배정 |
