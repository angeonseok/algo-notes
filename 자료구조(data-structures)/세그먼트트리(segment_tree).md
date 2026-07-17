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
    #리프 노드면 걍 원본 값 박기
    if start == end:
        tree[node] = arr[start]
    else:
        mid = (start + end) // 2

        #왼쪽 자식은 [start ~ mid], 오른쪽 자식은 [mid+1 ~ end]
        build(arr, node*2,   start, mid)
        build(arr, node*2+1, mid+1, end)

        #현재 노드는 자식 둘의 합
        tree[node] = tree[node*2] + tree[node*2+1]

def update(node, start, end, idx, val):
    #리프까지 내려왔으면 값 갱신
    if start == end:
        tree[node] = val
    else:
        mid = (start + end) // 2

        #바꾸려는 idx가 왼쪽 구간에 있으면 왼쪽으로, 아니면 오른쪽으로
        if idx <= mid: update(node*2,   start, mid,   idx, val)
        else:          update(node*2+1, mid+1, end,   idx, val)

        #자식 바뀌었으니까 현재 노드 합도 다시 계산
        tree[node] = tree[node*2] + tree[node*2+1]

def query(node, start, end, left, right):
    #아예 안 겹치면 합에 영향 없으니까 0
    if right < start or end < left: return 0

    #현재 구간이 찾는 범위 안에 완전히 들어오면 바로 사용
    if left <= start and end <= right: return tree[node]

    #일부만 겹치면 반 갈라서 계속 내려감
    mid = (start + end) // 2
    return (query(node*2,   start, mid,   left, right) +
            query(node*2+1, mid+1, end,   left, right))

#사용
arr = [1, 2, 3, 4, 5]
N = len(arr)

#세그트리 배열. 보통 넉넉하게 4배 잡음
tree = [0] * (4 * N)

#1번 노드가 arr 전체 구간 담당하게 시작
build(arr, 1, 0, N-1)

#arr[1:4] 구간 합 = 9
query(1, 0, N-1, 1, 3)

#arr[2]를 10으로 바꾸면 이번엔 16
update(1, 0, N-1, 2, 10)
query(1, 0, N-1, 1, 3)
```

#### C++
```cpp
int N;

//세그트리 배열. 보통 넉넉하게 4배 잡음
vector<long long> tree;

void build(vector<int>& arr, int node, int start, int end) {
    //리프 노드면 걍 원본 값 박기
    if (start == end) {
        tree[node] = arr[start];
    } else {
        int mid = (start + end) / 2;

        //왼쪽 자식은 [start ~ mid], 오른쪽 자식은 [mid+1 ~ end]
        build(arr, node*2,   start, mid);
        build(arr, node*2+1, mid+1, end);

        //현재 노드는 자식 둘의 합
        tree[node] = tree[node*2] + tree[node*2+1];
    }
}

void update(int node, int start, int end, int idx, long long val) {
    //리프까지 내려왔으면 값 갱신
    if (start == end) {
        tree[node] = val;
    } else {
        int mid = (start + end) / 2;

        //바꾸려는 idx가 왼쪽 구간에 있으면 왼쪽으로, 아니면 오른쪽으로
        if (idx <= mid) update(node*2,   start, mid, idx, val);
        else            update(node*2+1, mid+1, end, idx, val);

        //자식 바뀌었으니까 현재 노드 합도 다시 계산
        tree[node] = tree[node*2] + tree[node*2+1];
    }
}

long long query(int node, int start, int end, int left, int right) {
    //아예 안 겹치면 합에 영향 없으니까 0
    if (right < start || end < left) return 0;

    //현재 구간이 찾는 범위 안에 완전히 들어오면 바로 사용
    if (left <= start && end <= right) return tree[node];

    //일부만 겹치면 반 갈라서 계속 내려감
    int mid = (start + end) / 2;
    return query(node*2,   start, mid, left, right)
         + query(node*2+1, mid+1, end, left, right);
}

//사용
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

        #sum, min, max 등 연산 함수를 갈아끼울 수 있게
        self.func    = func

        #범위 벗어날 때 반환값
        self.default = default

        #세그트리 배열. 보통 넉넉하게 4배 잡음
        self.tree    = [default] * (4 * self.n)

        #1번 노드가 arr 전체 구간 담당하게 시작
        self._build(arr, 1, 0, self.n - 1)

    def _build(self, arr, node, start, end):
        #리프 노드면 걍 원본 값 박기
        if start == end:
            self.tree[node] = arr[start]
        else:
            mid = (start + end) // 2

            #왼쪽 자식은 [start ~ mid], 오른쪽 자식은 [mid+1 ~ end]
            self._build(arr, node*2,   start, mid)
            self._build(arr, node*2+1, mid+1, end)

            #현재 노드는 자식 둘을 func로 합친 값
            self.tree[node] = self.func([self.tree[node*2], self.tree[node*2+1]])

    def _update(self, node, start, end, idx, val):
        #리프까지 내려왔으면 값 갱신
        if start == end:
            self.tree[node] = val
        else:
            mid = (start + end) // 2

            #바꾸려는 idx가 왼쪽 구간에 있으면 왼쪽으로, 아니면 오른쪽으로
            if idx <= mid: self._update(node*2,   start, mid,   idx, val)
            else:          self._update(node*2+1, mid+1, end,   idx, val)

            #자식 바뀌었으니까 현재 노드도 다시 계산
            self.tree[node] = self.func([self.tree[node*2], self.tree[node*2+1]])

    def _query(self, node, start, end, left, right):
        #아예 안 겹치면 결과에 영향 없으니까 default
        if right < start or end < left: return self.default

        #현재 구간이 찾는 범위 안에 완전히 들어오면 바로 사용
        if left <= start and end <= right: return self.tree[node]

        #일부만 겹치면 반 갈라서 계속 내려감
        mid = (start + end) // 2
        return self.func([self._query(node*2,   start, mid,   left, right),
                          self._query(node*2+1, mid+1, end,   left, right)])

    #밖에서는 루트(1번)부터 시작하는 거 신경 안 쓰게 감싸두자
    def update(self, idx, val):
        self._update(1, 0, self.n - 1, idx, val)

    def query(self, left, right):
        return self._query(1, 0, self.n - 1, left, right)

#사용
arr = [1, 2, 3, 4, 5]

#func랑 default만 갈아끼우면 합/최솟값/최댓값 다 됨
st_sum = SegmentTree(arr, func=sum, default=0)
st_min = SegmentTree(arr, func=min, default=sys.maxsize)
st_max = SegmentTree(arr, func=max, default=0)

#9 → arr[2]를 10으로 갱신 → 16
st_sum.query(1, 3)
st_sum.update(2, 10)
st_sum.query(1, 3)
```

#### C++
```cpp
struct SegmentTree {
    int n;
    vector<long long> tree;

    //sum, min, max 등 연산 함수를 갈아끼울 수 있게
    function<long long(long long,long long)> func;

    //범위 벗어날 때 반환값
    long long defaultVal;

    //기본은 구간 합. func랑 def만 바꾸면 최솟값/최댓값도 됨
    SegmentTree(vector<int>& arr,
                function<long long(long long,long long)> f =
                    [](long long a, long long b){ return a + b; },
                long long def = 0)
        : n(arr.size()), tree(4 * arr.size(), def), func(f), defaultVal(def) {
        build(arr, 1, 0, n - 1);
    }

    void build(vector<int>& arr, int node, int start, int end) {
        //리프 노드면 걍 원본 값 박기
        if (start == end) { tree[node] = arr[start]; return; }
        int mid = (start + end) / 2;

        //자식 둘 채우고
        build(arr, node*2, start, mid);
        build(arr, node*2+1, mid+1, end);

        //현재 노드는 자식 둘을 func로 합친 값
        tree[node] = func(tree[node*2], tree[node*2+1]);
    }

    void _update(int node, int start, int end, int idx, long long val) {
        //리프까지 내려왔으면 값 갱신
        if (start == end) { tree[node] = val; return; }
        int mid = (start + end) / 2;

        //바꾸려는 idx가 왼쪽 구간에 있으면 왼쪽으로, 아니면 오른쪽으로
        if (idx <= mid) _update(node*2, start, mid, idx, val);
        else            _update(node*2+1, mid+1, end, idx, val);

        //자식 바뀌었으니까 현재 노드도 다시 계산
        tree[node] = func(tree[node*2], tree[node*2+1]);
    }

    long long _query(int node, int start, int end, int left, int right) {
        //아예 안 겹치면 결과에 영향 없으니까 default
        if (right < start || end < left) return defaultVal;

        //완전히 포함되면 바로 사용
        if (left <= start && end <= right) return tree[node];

        //일부만 겹치면 반 갈라서 계속 내려감
        int mid = (start + end) / 2;
        return func(_query(node*2, start, mid, left, right),
                    _query(node*2+1, mid+1, end, left, right));
    }

    //밖에서는 루트(1번)부터 시작하는 거 신경 안 쓰게 감싸두자
    void update(int idx, long long val) { _update(1, 0, n - 1, idx, val); }
    long long query(int left, int right) { return _query(1, 0, n - 1, left, right); }
};

//사용
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
