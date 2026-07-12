# Recursion (재귀)

> 한 줄 정리: 함수가 자기 자신을 호출하는 방식, 큰 문제를 같은 구조의 작은 문제로 쪼갠다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<algorithm>` (`max`)

## 1. 언제 쓰는가
- 문제가 동일한 구조의 더 작은 문제로 나뉠 때
- 트리/그래프 탐색
- 분할 정복 (병합 정렬, 퀵 정렬)
- 백트래킹의 기반
- 점화식으로 표현 가능한 문제

## 2. 핵심 아이디어
- 기저 조건(base case): 재귀를 멈추는 조건 → 반드시 있어야 함
- 재귀 조건(recursive case): 자기 자신을 호출하는 부분
- 스택 프레임이 쌓임 → 깊이가 깊어지면 스택 오버플로
- Python 기본 재귀 제한: 1000 → `sys.setrecursionlimit` 필요 / C++은 콜스택 크기에 의존

## 3. 시간복잡도
- 문제마다 다름
- 팩토리얼: O(N)
- 피보나치 (메모이제이션 없음): O(2^N)
- 피보나치 (메모이제이션): O(N)

## 4. 기본 코드

### 재귀 기본 구조

#### Python
```python
def recursive(n):
    # 기저 조건
    if n <= 0:
        return 0
    # 재귀 조건
    return n + recursive(n - 1)

# 재귀 제한 설정
import sys
sys.setrecursionlimit(100000)
```

#### C++
```cpp
int recursive(int n) {
    if (n <= 0) return 0;          // 기저 조건
    return n + recursive(n - 1);   // 재귀 조건
}
// C++엔 setrecursionlimit이 없다. 재귀가 매우 깊으면
// 컴파일/실행 시 스택 크기를 늘리거나 반복문으로 변환한다.
```

### 팩토리얼

#### Python
```python
def factorial(n):
    if n <= 1:      # 기저 조건
        return 1
    return n * factorial(n - 1)
```

#### C++
```cpp
long long factorial(int n) {
    if (n <= 1) return 1;                    // 기저 조건
    return (long long)n * factorial(n - 1);
}
```

### 피보나치 (메모이제이션)

#### Python
```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)
```

#### C++
```cpp
vector<long long> memo(100001, -1);   // -1 = 미계산

long long fib(int n) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];
    return memo[n] = fib(n - 1) + fib(n - 2);
}
```

### 분할 정복 구조

#### Python
```python
def divide_and_conquer(arr, lo, hi):
    if lo >= hi:    # 기저 조건
        return

    mid = (lo + hi) // 2
    divide_and_conquer(arr, lo, mid)      # 왼쪽 절반
    divide_and_conquer(arr, mid + 1, hi)  # 오른쪽 절반
    merge(arr, lo, mid, hi)               # 합치기
```

#### C++
```cpp
void divideAndConquer(vector<int>& arr, int lo, int hi) {
    if (lo >= hi) return;                   // 기저 조건

    int mid = (lo + hi) / 2;
    divideAndConquer(arr, lo, mid);         // 왼쪽 절반
    divideAndConquer(arr, mid + 1, hi);     // 오른쪽 절반
    // merge(arr, lo, mid, hi);             // 합치기
}
```

## 5. 실전 패턴

### 트리 재귀 탐색

#### Python
```python
def dfs(node):
    if not node:
        return 0
    left  = dfs(node.left)
    right = dfs(node.right)
    return 1 + max(left, right)  # 트리 높이
```

#### C++
```cpp
// struct TreeNode { int val; TreeNode *left, *right; };
int dfs(TreeNode* node) {
    if (!node) return 0;
    int left  = dfs(node->left);
    int right = dfs(node->right);
    return 1 + max(left, right);   // 트리 높이
}
```

### 거듭제곱 (빠른 지수)

#### Python
```python
def power(base, exp):
    if exp == 0:
        return 1
    if exp % 2 == 0:
        half = power(base, exp // 2)
        return half * half
    return base * power(base, exp - 1)
```

#### C++
```cpp
long long power(long long base, int exp) {
    if (exp == 0) return 1;
    if (exp % 2 == 0) {
        long long half = power(base, exp / 2);
        return half * half;
    }
    return base * power(base, exp - 1);
}
```

## 6. 이걸 떠올려야 할 때
- "트리/그래프 구조" → 재귀로 탐색
- "큰 문제 = 작은 문제 + 작은 문제" → 분할 정복
- "점화식이 보인다" → 재귀 + 메모이제이션 (= DP)

## 7. 자주 틀리는 포인트
- 기저 조건 빠뜨리면 무한 재귀 → RecursionError / 스택 오버플로
- Python 재귀 제한 1000 → `sys.setrecursionlimit(100000)` 필수
- 재귀 깊이가 깊으면 스택 오버플로 → 반복문으로 전환 고려
- 메모이제이션: Python `lru_cache`는 mutable 인자 불가(tuple로) / C++은 memo 초기값이 실제 답과 안 겹치게
