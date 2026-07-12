# Fenwick Tree / BIT (펜윅 트리 / 이진 인덱스 트리)

> 한 줄 정리: 구간 합과 업데이트를 O(log N)에 처리하는 트리, 세그먼트 트리보다 구현이 간단하고 메모리가 적다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<algorithm>` (`sort`, `unique`, `lower_bound`)

## 1. 언제 쓰는가
- 구간 합을 여러 번 조회하고 업데이트도 자주 할 때
- 세그먼트 트리보다 간단하게 구현하고 싶을 때
- 최솟값/최댓값은 필요 없고 합만 구할 때
- 역전 쌍(inversion count) 문제

## 2. 핵심 아이디어
- 비트 연산으로 각 인덱스가 담당하는 구간을 결정
- `i & (-i)`: i의 최하위 비트 → 담당 구간의 크기
- 1-indexed 배열로 구현 (0번 인덱스 사용 안 함)
- 업데이트: 담당 구간을 포함하는 상위 노드들 갱신
- 쿼리: prefix sum을 이용해 구간 합 계산

## 3. 시간/공간 복잡도
| 연산 | 복잡도 |
|------|--------|
| 업데이트 | O(log N) |
| 구간 합 쿼리 | O(log N) |
| 공간 | O(N) |

## 4. 기본 코드

### 함수형 (전역 배열, 코테 권장)

#### Python
```python
N = 0
tree = []

def update(i, delta):
    while i <= N:
        tree[i] += delta
        i += i & (-i)  # 최하위 비트만큼 올라감

def prefix_sum(i):
    total = 0
    while i > 0:
        total += tree[i]
        i -= i & (-i)  # 최하위 비트 제거
    return total

def query(left, right):
    return prefix_sum(right) - prefix_sum(left - 1)

# 사용 (1-indexed)
arr = [1, 2, 3, 4, 5]
N = len(arr)
tree = [0] * (N + 1)

for i, v in enumerate(arr, 1):
    update(i, v)

query(2, 4)   # 9
update(3, 5)  # arr[3] += 5
query(2, 4)   # 14
```

#### C++
```cpp
int N;
vector<long long> tree;   // 크기 N+1, 1-indexed

void update(int i, long long delta) {
    while (i <= N) {
        tree[i] += delta;
        i += i & (-i);   // 최하위 비트만큼 올라감
    }
}

long long prefixSum(int i) {
    long long total = 0;
    while (i > 0) {
        total += tree[i];
        i -= i & (-i);   // 최하위 비트 제거
    }
    return total;
}

long long query(int left, int right) {
    return prefixSum(right) - prefixSum(left - 1);
}

// 사용 (1-indexed)
// vector<int> arr = {1, 2, 3, 4, 5};
// N = arr.size();  tree.assign(N + 1, 0);
// for (int i = 1; i <= N; i++) update(i, arr[i-1]);
// query(2, 4);    // 9
// update(3, 5);   // arr[3] += 5
// query(2, 4);    // 14
```

### 클래스/구조체형 (호출 단순화, 트리 2개 이상 쓸 때 유용)

#### Python
```python
class FenwickTree:
    def __init__(self, arr):
        self.n = len(arr)
        self.tree = [0] * (self.n + 1)  # 1-indexed
        for i, v in enumerate(arr, 1):
            self._update(i, v)

    def _update(self, i, delta):
        while i <= self.n:
            self.tree[i] += delta
            i += i & (-i)

    def _prefix_sum(self, i):
        total = 0
        while i > 0:
            total += self.tree[i]
            i -= i & (-i)
        return total

    def update(self, i, delta):   # 1-indexed, delta만큼 더하기
        self._update(i, delta)

    def query(self, left, right): # 1-indexed, 구간 합
        return self._prefix_sum(right) - self._prefix_sum(left - 1)

# 사용
arr = [1, 2, 3, 4, 5]
ft = FenwickTree(arr)
ft.query(2, 4)   # 9
ft.update(3, 5)  # arr[3] += 5
ft.query(2, 4)   # 14
```

#### C++
```cpp
struct FenwickTree {
    int n;
    vector<long long> tree;   // 1-indexed

    FenwickTree(int n) : n(n), tree(n + 1, 0) {}
    FenwickTree(vector<int>& arr) : n(arr.size()), tree(arr.size() + 1, 0) {
        for (int i = 1; i <= n; i++) update(i, arr[i-1]);
    }

    void update(int i, long long delta) {   // 1-indexed, delta만큼 더하기
        while (i <= n) {
            tree[i] += delta;
            i += i & (-i);
        }
    }

    long long prefixSum(int i) {
        long long total = 0;
        while (i > 0) {
            total += tree[i];
            i -= i & (-i);
        }
        return total;
    }

    long long query(int left, int right) {   // 1-indexed, 구간 합
        return prefixSum(right) - prefixSum(left - 1);
    }
};

// 사용
// vector<int> arr = {1, 2, 3, 4, 5};
// FenwickTree ft(arr);
// ft.query(2, 4);    // 9
// ft.update(3, 5);
// ft.query(2, 4);    // 14
```

### i & (-i) 원리
```
i의 최하위 비트 = 담당하는 구간의 크기
i=1 → 0001 → 1개 담당
i=2 → 0010 → 2개 담당
i=4 → 0100 → 4개 담당
i=6 → 0110 → 2개 담당

update: i += i & (-i)  → 상위 노드로 이동
query:  i -= i & (-i)  → 하위 구간으로 이동
```
> `i & (-i)`는 Python·C++ 모두 2의 보수 기반이라 동일하게 동작.

### 역전 쌍 카운트 (inversion count)

#### Python
```python
def count_inversions(arr):
    n = len(arr)
    sorted_arr = sorted(set(arr))
    rank = {v: i+1 for i, v in enumerate(sorted_arr)}

    ft = FenwickTree([0] * n)
    inversions = 0

    for v in arr:
        r = rank[v]
        inversions += ft.query(r + 1, n) if r + 1 <= n else 0
        ft.update(r, 1)

    return inversions
```

#### C++
```cpp
long long countInversions(vector<int>& arr) {
    int n = arr.size();
    // 좌표 압축
    vector<int> vals = arr;
    sort(vals.begin(), vals.end());
    vals.erase(unique(vals.begin(), vals.end()), vals.end());
    int m = vals.size();

    FenwickTree ft(m);
    long long inversions = 0;

    for (int v : arr) {
        // rank: 1-indexed 압축 좌표
        int r = lower_bound(vals.begin(), vals.end(), v) - vals.begin() + 1;
        if (r + 1 <= m) inversions += ft.query(r + 1, m);
        ft.update(r, 1);
    }
    return inversions;
}
```

## 5. 세그먼트 트리 vs 펜윅 트리
| | 세그먼트 트리 | 펜윅 트리 |
|--|-------------|---------|
| 구현 난이도 | 보통 | 쉬움 |
| 구간 합 쿼리 | O(log N) | O(log N) |
| 업데이트 | O(log N) | O(log N) |
| 최솟값/최댓값 | 가능 | 불가 |
| 구간 업데이트 | 가능 (Lazy) | 제한적 |
| 메모리 | O(4N) | O(N) |
| 적합한 상황 | 최솟값/최댓값, 복잡한 쿼리 | 구간 합만 필요할 때 |

## 6. 이걸 떠올려야 할 때
- "구간 합 + 업데이트" → 펜윅 트리 (세그먼트 트리보다 간단)
- "역전 쌍 개수" → 펜윅 트리
- "최솟값/최댓값도 필요" → 세그먼트 트리로

## 7. 자주 틀리는 포인트
- 0-indexed 배열 사용 불가 → 반드시 1-indexed
- update는 `i += i & (-i)`, query는 `i -= i & (-i)` → 방향 헷갈리지 않기
- 구간 합은 prefix_sum(right) - prefix_sum(left-1) → left-1 빠뜨리는 경우
- 역전 쌍에서 좌표 압축 안 하면 트리 크기가 너무 커짐
- 합이 커질 수 있으니 C++은 `long long` 사용
