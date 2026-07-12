# Segment Tree (세그먼트 트리)

> 한 줄 정리: 구간 합/최솟값/최댓값을 O(log N)에 조회하고 업데이트하는 트리 자료구조

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<functional>` (`function`), `<algorithm>` (`min`/`max`), `<climits>`

## 1. 언제 쓰는가
- 구간 합/최솟값/최댓값을 여러 번 조회할 때
- 값이 동적으로 업데이트되는 상황
- 누적 합은 업데이트가 O(N)이라 느릴 때 → 세그먼트 트리로 대체

## 2. 핵심 아이디어
- 배열을 트리 형태로 저장, 각 노드가 구간의 합/최솟값을 저장
- 리프 노드: 원소 하나
- 내부 노드: 자식 노드들의 합/최솟값
- 1-indexed 배열로 구현, 부모 i → 자식 2i, 2i+1

## 3. 시간/공간 복잡도
| 연산 | 복잡도 |
|------|--------|
| 빌드 | O(N) |
| 업데이트 | O(log N) |
| 구간 쿼리 | O(log N) |
| 공간 | O(4N) |

## 4. 기본 코드

### 함수형 (전역 배열, 코테 권장)

#### Python
```python
import sys
sys.setrecursionlimit(100000)

N = 0
tree = []

def build(arr, node, start, end):
    if start == end:
        tree[node] = arr[start]
    else:
        mid = (start + end) // 2
        build(arr, node*2,   start, mid)
        build(arr, node*2+1, mid+1, end)
        tree[node] = tree[node*2] + tree[node*2+1]

def update(node, start, end, idx, val):
    if start == end:
        tree[node] = val
    else:
        mid = (start + end) // 2
        if idx <= mid: update(node*2,   start, mid,   idx, val)
        else:          update(node*2+1, mid+1, end,   idx, val)
        tree[node] = tree[node*2] + tree[node*2+1]

def query(node, start, end, left, right):
    if right < start or end < left: return 0
    if left <= start and end <= right: return tree[node]
    mid = (start + end) // 2
    return (query(node*2,   start, mid,   left, right) +
            query(node*2+1, mid+1, end,   left, right))

# 사용
arr = [1, 2, 3, 4, 5]
N = len(arr)
tree = [0] * (4 * N)
build(arr, 1, 0, N-1)

query(1, 0, N-1, 1, 3)   # arr[1:4] 구간 합 = 9
update(1, 0, N-1, 2, 10)  # arr[2] = 10
query(1, 0, N-1, 1, 3)   # 16
```

#### C++
```cpp
int N;
vector<long long> tree;   // 크기 4*N

void build(vector<int>& arr, int node, int start, int end) {
    if (start == end) {
        tree[node] = arr[start];
    } else {
        int mid = (start + end) / 2;
        build(arr, node*2,   start, mid);
        build(arr, node*2+1, mid+1, end);
        tree[node] = tree[node*2] + tree[node*2+1];
    }
}

void update(int node, int start, int end, int idx, long long val) {
    if (start == end) {
        tree[node] = val;
    } else {
        int mid = (start + end) / 2;
        if (idx <= mid) update(node*2,   start, mid, idx, val);
        else            update(node*2+1, mid+1, end, idx, val);
        tree[node] = tree[node*2] + tree[node*2+1];
    }
}

long long query(int node, int start, int end, int left, int right) {
    if (right < start || end < left) return 0;              // 완전히 벗어남
    if (left <= start && end <= right) return tree[node];   // 완전히 포함
    int mid = (start + end) / 2;
    return query(node*2,   start, mid, left, right)
         + query(node*2+1, mid+1, end, left, right);
}

// 사용
// vector<int> arr = {1, 2, 3, 4, 5};
// N = arr.size();  tree.assign(4 * N, 0);
// build(arr, 1, 0, N-1);
// query(1, 0, N-1, 1, 3);    // 9
// update(1, 0, N-1, 2, 10);
// query(1, 0, N-1, 1, 3);    // 16
```

### 클래스/구조체형 (호출 단순화, 트리 2개 이상 쓸 때 유용)

#### Python
```python
import sys

class SegmentTree:
    def __init__(self, arr, func=sum, default=0):
        self.n       = len(arr)
        self.func    = func      # sum, min, max 등 연산 함수
        self.default = default   # 범위 벗어날 때 반환값
        self.tree    = [default] * (4 * self.n)
        self._build(arr, 1, 0, self.n - 1)

    def _build(self, arr, node, start, end):
        if start == end:
            self.tree[node] = arr[start]
        else:
            mid = (start + end) // 2
            self._build(arr, node*2,   start, mid)
            self._build(arr, node*2+1, mid+1, end)
            self.tree[node] = self.func([self.tree[node*2], self.tree[node*2+1]])

    def _update(self, node, start, end, idx, val):
        if start == end:
            self.tree[node] = val
        else:
            mid = (start + end) // 2
            if idx <= mid: self._update(node*2,   start, mid,   idx, val)
            else:          self._update(node*2+1, mid+1, end,   idx, val)
            self.tree[node] = self.func([self.tree[node*2], self.tree[node*2+1]])

    def _query(self, node, start, end, left, right):
        if right < start or end < left: return self.default
        if left <= start and end <= right: return self.tree[node]
        mid = (start + end) // 2
        return self.func([self._query(node*2,   start, mid,   left, right),
                          self._query(node*2+1, mid+1, end,   left, right)])

    def update(self, idx, val):
        self._update(1, 0, self.n - 1, idx, val)

    def query(self, left, right):
        return self._query(1, 0, self.n - 1, left, right)

# 사용
arr = [1, 2, 3, 4, 5]
st_sum = SegmentTree(arr, func=sum, default=0)            # 구간 합
st_min = SegmentTree(arr, func=min, default=sys.maxsize)  # 구간 최솟값
st_max = SegmentTree(arr, func=max, default=0)            # 구간 최댓값
st_sum.query(1, 3)   # 9
st_sum.update(2, 10) # arr[2] = 10
st_sum.query(1, 3)   # 16
```

#### C++
```cpp
struct SegmentTree {
    int n;
    vector<long long> tree;
    function<long long(long long,long long)> func;   // sum, min, max 등
    long long defaultVal;

    SegmentTree(vector<int>& arr,
                function<long long(long long,long long)> f =
                    [](long long a, long long b){ return a + b; },
                long long def = 0)
        : n(arr.size()), tree(4 * arr.size(), def), func(f), defaultVal(def) {
        build(arr, 1, 0, n - 1);
    }

    void build(vector<int>& arr, int node, int start, int end) {
        if (start == end) { tree[node] = arr[start]; return; }
        int mid = (start + end) / 2;
        build(arr, node*2, start, mid);
        build(arr, node*2+1, mid+1, end);
        tree[node] = func(tree[node*2], tree[node*2+1]);
    }

    void _update(int node, int start, int end, int idx, long long val) {
        if (start == end) { tree[node] = val; return; }
        int mid = (start + end) / 2;
        if (idx <= mid) _update(node*2, start, mid, idx, val);
        else            _update(node*2+1, mid+1, end, idx, val);
        tree[node] = func(tree[node*2], tree[node*2+1]);
    }

    long long _query(int node, int start, int end, int left, int right) {
        if (right < start || end < left) return defaultVal;
        if (left <= start && end <= right) return tree[node];
        int mid = (start + end) / 2;
        return func(_query(node*2, start, mid, left, right),
                    _query(node*2+1, mid+1, end, left, right));
    }

    void update(int idx, long long val) { _update(1, 0, n - 1, idx, val); }
    long long query(int left, int right) { return _query(1, 0, n - 1, left, right); }
};

// 사용
// vector<int> arr = {1, 2, 3, 4, 5};
// SegmentTree st_sum(arr);   // 기본: 구간 합
// SegmentTree st_min(arr, [](long long a, long long b){ return min(a,b); }, LLONG_MAX);
// SegmentTree st_max(arr, [](long long a, long long b){ return max(a,b); }, LLONG_MIN);
// st_sum.query(1, 3);  // 9
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
- "구간 합 + 업데이트가 많다" → 세그먼트 트리 or 펜윅 트리
- "구간 최솟값/최댓값" → 세그먼트 트리 (펜윅은 불가)
- 누적 합은 업데이트 O(N) → 업데이트 많으면 세그먼트 트리

## 7. 자주 틀리는 포인트
- 트리 크기를 4*N으로 잡아야 안전 (2*N이면 부족할 수 있음)
- query에서 범위 벗어남 조건 (`right < start or end < left`) 순서 헷갈리는 경우
- update 후 부모 노드 갱신 빠뜨리는 경우
- 0-indexed vs 1-indexed 혼용 주의
- C++은 `std::function` 대신 템플릿을 쓰면 더 빠름 (호출 오버헤드 제거)
