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
    #기저 조건. 이게 없으면 무한으로 내려감
    if n <= 0:
        return 0

    #재귀 조건. 나보다 작은 문제한테 넘기고 결과 받아쓰기
    return n + recursive(n - 1)

#파이썬 기본 재귀 제한이 1000이라 깊게 들어가면 터짐. 늘려두자
import sys
sys.setrecursionlimit(100000)
```

#### C++
```cpp
int recursive(int n) {
    //기저 조건. 이게 없으면 무한으로 내려감
    if (n <= 0) return 0;

    //재귀 조건. 나보다 작은 문제한테 넘기고 결과 받아쓰기
    return n + recursive(n - 1);
}

//C++엔 setrecursionlimit 같은 게 없음
//너무 깊으면 컴파일/실행 시 스택 크기를 늘리거나 반복문으로 바꾸자
```

### 팩토리얼

#### Python
```python
def factorial(n):
    #기저 조건. 0!이든 1!이든 1
    if n <= 1:
        return 1

    #n * (n-1)! 로 쪼개기
    return n * factorial(n - 1)
```

#### C++
```cpp
long long factorial(int n) {
    //기저 조건. 0!이든 1!이든 1
    if (n <= 1) return 1;

    //n * (n-1)! 로 쪼개기. 금방 커지니까 long long
    return (long long)n * factorial(n - 1);
}
```

### 피보나치 (메모이제이션)

#### Python
```python
from functools import lru_cache

#lru_cache 붙이면 같은 n 두 번 계산 안 함. 걍 이걸로 메모이제이션 퉁치자
@lru_cache(maxsize=None)
def fib(n):
    #0, 1은 그대로 반환
    if n <= 1:
        return n

    #앞 두 개 더하기
    return fib(n - 1) + fib(n - 2)
```

#### C++
```cpp
//-1로 채워두고 아직 계산 안 한 표시로 씀
vector<long long> memo(100001, -1);

long long fib(int n) {
    //0, 1은 그대로 반환
    if (n <= 1) return n;

    //이미 계산해둔 거면 재활용
    if (memo[n] != -1) return memo[n];

    //계산하면서 memo에 박아두기
    return memo[n] = fib(n - 1) + fib(n - 2);
}
```

### 분할 정복 구조

#### Python
```python
def divide_and_conquer(arr, l, h):
    #원소 하나 남으면 더 쪼갤 거 없음 (기저 조건)
    if l >= h:
        return

    #반으로 가르기
    mid = (l + h) // 2

    #왼쪽 먼저 정렬
    divide_and_conquer(arr, l, mid)

    #오른쪽도 정렬
    divide_and_conquer(arr, mid + 1, h)

    #둘 다 정렬됐으니 합치기
    merge(arr, l, mid, h)
```

#### C++
```cpp
void divideAndConquer(vector<int>& arr, int l, int h) {
    //원소 하나 남으면 더 쪼갤 거 없음 (기저 조건)
    if (l >= h) return;

    //반으로 가르기
    int mid = (l + h) / 2;

    //왼쪽 먼저 정렬
    divideAndConquer(arr, l, mid);

    //오른쪽도 정렬
    divideAndConquer(arr, mid + 1, h);

    //둘 다 정렬됐으니 합치기
    //merge(arr, l, mid, h);
}
```

## 5. 실전 패턴

### 트리 재귀 탐색

#### Python
```python
def dfs(node):
    #빈 노드까지 내려왔으면 높이 0
    if not node:
        return 0

    #양쪽 자식 높이 먼저 재고
    left  = dfs(node.left)
    right = dfs(node.right)

    #더 깊은 쪽 + 나 자신 한 칸 = 트리 높이
    return 1 + max(left, right)
```

#### C++
```cpp
//struct TreeNode { int val; TreeNode *left, *right; };
int dfs(TreeNode* node) {
    //빈 노드까지 내려왔으면 높이 0
    if (!node) return 0;

    //양쪽 자식 높이 먼저 재고
    int left  = dfs(node->left);
    int right = dfs(node->right);

    //더 깊은 쪽 + 나 자신 한 칸 = 트리 높이
    return 1 + max(left, right);
}
```

### 거듭제곱 (빠른 지수)

#### Python
```python
def power(base, exp):
    #0제곱은 1
    if exp == 0:
        return 1

    #짝수면 반만 구해서 제곱 > 한 번에 반토막
    if exp % 2 == 0:
        half = power(base, exp // 2)
        return half * half

    #홀수면 하나 떼서 짝수로 만들기
    return base * power(base, exp - 1)
```

#### C++
```cpp
long long power(long long base, int exp) {
    //0제곱은 1
    if (exp == 0) return 1;

    //짝수면 반만 구해서 제곱 > 한 번에 반토막
    if (exp % 2 == 0) {
        long long half = power(base, exp / 2);
        return half * half;
    }

    //홀수면 하나 떼서 짝수로 만들기
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
