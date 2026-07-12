# Brute Force (완전 탐색)

> 한 줄 정리: 가능한 모든 경우를 다 시도해보는 방법, 느리지만 확실하다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<algorithm>` (`sort`, `next_permutation`, `min`, `fill`), `<climits>`

## 1. 언제 쓰는가
- 경우의 수가 충분히 작을 때 (N ≤ 20 이하)
- 최적해를 반드시 구해야 할 때
- 다른 풀이가 떠오르지 않을 때 → 일단 완전탐색으로 구현 후 최적화
- 순열/조합을 바로 열거할 수 있을 때 (Python itertools / C++ next_permutation)

## 2. 핵심 아이디어
- 모든 경우를 다 탐색 → 정답을 반드시 찾음
- 시간복잡도가 크기 때문에 N의 크기 먼저 확인
- 구현이 간단한 만큼 실수가 적음 → 시간 내에 통과되면 굳이 최적화 안 해도 됨

## 3. 시간복잡도
| 방법 | 복잡도 | 가능한 N |
|------|--------|---------|
| 순열 | O(N!) | N ≤ 8 |
| 조합/부분집합 | O(2^N) | N ≤ 20 |
| 이중 반복문 | O(N²) | N ≤ 5,000 |
| 삼중 반복문 | O(N³) | N ≤ 500 |

## 4. 기본 코드

### 중첩 반복문

#### Python
```python
# 모든 쌍 탐색 O(N²)
for i in range(n):
    for j in range(i + 1, n):
        # i, j 조합 처리
        pass
```

#### C++
```cpp
// 모든 쌍 탐색 O(N²)
for (int i = 0; i < n; i++)
    for (int j = i + 1; j < n; j++) {
        // i, j 조합 처리
    }
```

### 순열

#### Python
```python
from itertools import permutations

arr = [1, 2, 3]

# 전체 순열 O(N!)
for p in permutations(arr):
    print(p)  # (1,2,3), (1,3,2), ...

# r개 선택 순열
for p in permutations(arr, 2):
    print(p)  # (1,2), (1,3), (2,1), ...
```

#### C++
```cpp
vector<int> arr = {1, 2, 3};

// 전체 순열 O(N!) — 반드시 정렬된 상태에서 시작
sort(arr.begin(), arr.end());
do {
    // arr 사용: 1 2 3 → 1 3 2 → ...
} while (next_permutation(arr.begin(), arr.end()));
```

### 조합

#### Python
```python
from itertools import combinations, combinations_with_replacement

arr = [1, 2, 3]

for c in combinations(arr, 2):
    print(c)  # (1,2), (1,3), (2,3)

for c in combinations_with_replacement(arr, 2):
    print(c)  # (1,1), (1,2), (1,3), (2,2), ...
```

#### C++
```cpp
// C++엔 combinations 내장이 없음 → 선택 마스크에 next_permutation 트릭
vector<int> arr = {1, 2, 3};
int n = arr.size(), r = 2;

vector<bool> pick(n, false);
fill(pick.end() - r, pick.end(), true);   // [F F T] 형태(가장 작은 배열)에서 시작
do {
    vector<int> comb;
    for (int i = 0; i < n; i++)
        if (pick[i]) comb.push_back(arr[i]);
    // comb 사용: (1,2), (1,3), (2,3)
} while (next_permutation(pick.begin(), pick.end()));
// (복잡한 조건이면 백트래킹 파일의 combination 참고)
```

### 부분집합 (비트마스크)

#### Python
```python
arr = [1, 2, 3]
n = len(arr)

for mask in range(1 << n):  # 0 ~ 2^n - 1
    subset = []
    for i in range(n):
        if mask & (1 << i):
            subset.append(arr[i])
    print(subset)
```

#### C++
```cpp
vector<int> arr = {1, 2, 3};
int n = arr.size();

for (int mask = 0; mask < (1 << n); mask++) {   // 0 ~ 2^n - 1
    vector<int> subset;
    for (int i = 0; i < n; i++)
        if (mask & (1 << i))
            subset.push_back(arr[i]);
    // subset 사용
}
```

## 5. 실전 패턴

### 최솟값/최댓값 완전탐색

#### Python
```python
from itertools import permutations

result = float('inf')
for p in permutations(arr):
    result = min(result, calculate(p))
```

#### C++
```cpp
sort(arr.begin(), arr.end());
long long result = LLONG_MAX;
do {
    result = min(result, calculate(arr));
} while (next_permutation(arr.begin(), arr.end()));
```

### 2차원 그리드 완전탐색

#### Python
```python
for i in range(N):
    for j in range(M):
        if grid[i][j] == target:
            bfs(i, j)
```

#### C++
```cpp
for (int i = 0; i < N; i++)
    for (int j = 0; j < M; j++)
        if (grid[i][j] == target)
            bfs(i, j);
```

### 모든 부분집합 합 탐색

#### Python
```python
from itertools import combinations

arr = [1, 2, 3, 4]
target = 5

for r in range(1, len(arr) + 1):
    for c in combinations(arr, r):
        if sum(c) == target:
            print(c)
```

#### C++
```cpp
vector<int> arr = {1, 2, 3, 4};
int target = 5, n = arr.size();

for (int mask = 1; mask < (1 << n); mask++) {
    int sum = 0;
    for (int i = 0; i < n; i++)
        if (mask & (1 << i)) sum += arr[i];
    if (sum == target) { /* 해당 부분집합 */ }
}
```

## 6. 이걸 떠올려야 할 때
- "N이 작다" → 완전탐색 가능
- "모든 경우", "최소/최대" → 완전탐색 후 최적화 고려
- 순열/조합을 직접 구현해야 한다면 → 백트래킹 파일 참고

## 7. 자주 틀리는 포인트
- N 크기 확인 안 하고 완전탐색 → 시간초과
- C++ `next_permutation`은 **정렬된 상태**에서 시작해야 모든 순열을 얻음
- 부분집합 비트마스크에서 `1 << n`이 전체 경우의 수 (0 포함)
- `combinations`(Python)는 인덱스 기준이라 중복 값 있어도 다른 원소로 취급
