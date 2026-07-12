# Union-Find (유니온 파인드 / 서로소 집합)

> 한 줄 정리: 여러 노드의 연결 여부를 빠르게 확인하고 합치는 자료구조

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<numeric>` (`iota`), `<set>`, `<utility>` (`swap`)

## 1. 언제 쓰는가
- 두 노드가 같은 집합(연결 요소)에 속하는지 확인
- 사이클 감지
- 크루스칼 알고리즘 (MST)
- 동적으로 연결 관계가 추가될 때

## 2. 핵심 아이디어
- Find: 노드의 루트(대표) 찾기
- Union: 두 집합 합치기
- 경로 압축(Path Compression): Find 시 루트를 직접 연결 → O(α(N)) ≈ O(1)
- 랭크 기반 합치기: 트리 높이 최소화

## 3. 시간/공간 복잡도
| 연산 | 복잡도 |
|------|--------|
| Find (경로 압축) | O(α(N)) ≈ O(1) |
| Union | O(α(N)) ≈ O(1) |
| 공간 | O(N) |

- α(N): 아커만 함수의 역함수, 사실상 상수

## 4. 기본 코드

### 함수형 구현 (코테 표준)

#### Python
```python
import sys
sys.setrecursionlimit(100000)

def find(parent, x):
    if parent[x] != x:
        parent[x] = find(parent, parent[x])  # 경로 압축: 루트를 직접 연결
    return parent[x]

def union(parent, x, y):
    rx, ry = find(parent, x), find(parent, y)
    if rx == ry:
        return False  # 이미 같은 집합 → 사이클
    parent[ry] = rx   # ry를 rx 아래로 합치기
    return True

# 초기화 및 사용
parent = list(range(N + 1))  # parent[i] = i (자기 자신이 루트)

union(parent, 1, 2)
union(parent, 2, 3)
find(parent, 1) == find(parent, 3)  # True (같은 집합)
```

#### C++
```cpp
vector<int> parent;

int find(int x) {
    if (parent[x] != x)
        parent[x] = find(parent[x]);   // 경로 압축: 루트를 직접 연결
    return parent[x];
}

bool unite(int x, int y) {             // union은 키워드 느낌이라 unite
    int rx = find(x), ry = find(y);
    if (rx == ry) return false;        // 이미 같은 집합 → 사이클
    parent[ry] = rx;                   // ry를 rx 아래로 합치기
    return true;
}

// 초기화: parent.resize(N + 1); iota(parent.begin(), parent.end(), 0);
```

### 반복문 버전 (재귀 깊이 걱정 없음)

#### Python
```python
def find(parent, x):
    root = x
    while parent[root] != root:  # 루트 찾기
        root = parent[root]
    while parent[x] != root:     # 경로 압축
        parent[x], x = root, parent[x]
    return root
```

#### C++
```cpp
int find(int x) {
    int root = x;
    while (parent[root] != root) root = parent[root];   // 루트 찾기
    while (parent[x] != root) {                         // 경로 압축
        int next = parent[x];
        parent[x] = root;
        x = next;
    }
    return root;
}
```

### 참고: 클래스/구조체 버전 (경로 압축 + 랭크)

#### Python
```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n + 1))
        self.rank = [0] * (n + 1)

    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]

    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False
        if self.rank[rx] < self.rank[ry]:
            rx, ry = ry, rx
        self.parent[ry] = rx
        if self.rank[rx] == self.rank[ry]:
            self.rank[rx] += 1
        return True
```

#### C++
```cpp
struct UnionFind {
    vector<int> parent, rnk;   // rank는 std::rank와 충돌해 rnk

    UnionFind(int n) : parent(n + 1), rnk(n + 1, 0) {
        iota(parent.begin(), parent.end(), 0);   // parent[i] = i
    }

    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }

    bool unite(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        if (rnk[rx] < rnk[ry]) swap(rx, ry);
        parent[ry] = rx;
        if (rnk[rx] == rnk[ry]) rnk[rx]++;
        return true;
    }
};
```

## 5. 실전 패턴

### 사이클 감지

#### Python
```python
parent = list(range(V + 1))
has_cycle = False

for u, v in edges:
    if not union(parent, u, v):  # 이미 같은 집합 → 사이클
        has_cycle = True
```

#### C++
```cpp
// parent 초기화 후
bool hasCycle = false;
for (auto& e : edges)                        // e = {u, v}
    if (!unite(e.first, e.second))           // 이미 같은 집합 → 사이클
        hasCycle = true;
```

### 연결 요소 개수

#### Python
```python
parent = list(range(V + 1))
for u, v in edges:
    union(parent, u, v)

# 루트가 자기 자신인 노드 수 = 연결 요소 수
components = len(set(find(parent, i) for i in range(1, V + 1)))
```

#### C++
```cpp
for (auto& e : edges) unite(e.first, e.second);

set<int> roots;
for (int i = 1; i <= V; i++) roots.insert(find(i));
int components = roots.size();   // 서로 다른 루트 수
```

## 6. 이걸 떠올려야 할 때
- "두 노드가 연결되어 있는가" → 유니온 파인드
- "사이클 존재 여부" → 유니온 파인드
- "크루스칼 MST" → 유니온 파인드 필수
- 간선이 동적으로 추가될 때 → 유니온 파인드 (BFS/DFS보다 빠름)

## 7. 자주 틀리는 포인트
- 경로 압축 없으면 트리가 편향되어 O(N)으로 느려짐
- `find` 재귀 깊이 → Python `sys.setrecursionlimit`, C++은 반복문 버전 고려
- union 시 find 결과로 합쳐야 함 → `parent[y] = x` 대신 `parent[ry] = rx`
- C++에서 `rank`는 `std::rank`(type trait)와 이름 충돌 가능 → `rnk` 등으로
