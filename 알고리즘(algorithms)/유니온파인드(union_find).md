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

#x의 루트를 경로를 압축하면서 찾자
def find(parent, x):
    if parent[x] != x:
        parent[x] = find(parent, parent[x])
    return parent[x]

#서로의 루트가 다를 경우 통합하자(root_b를 root_a 밑으로)
def union(parent, x, y):
    root_a, root_b = find(parent, x), find(parent, y)

    #루트가 같으면 이미 한 집합. 여기서 또 이으면 사이클이니까 False
    if root_a == root_b:
        return False

    parent[root_b] = root_a
    return True

#자기 자신을 부모로 시작해보자
parent = list(range(N + 1))

union(parent, 1, 2)
union(parent, 2, 3)

#1이랑 3이 같은 집합인지 확인 > True
find(parent, 1) == find(parent, 3)
```

#### C++
```cpp
vector<int> parent;

//x의 루트를 경로를 압축하면서 찾자
int find(int x) {
    if (parent[x] != x)
        parent[x] = find(parent[x]);
    return parent[x];
}

//union은 키워드 느낌이라 unite로
//서로의 루트가 다를 경우 통합하자(root_b를 root_a 밑으로)
bool unite(int x, int y) {
    int root_a = find(x), root_b = find(y);

    //루트가 같으면 이미 한 집합. 여기서 또 이으면 사이클이니까 false
    if (root_a == root_b) return false;

    parent[root_b] = root_a;
    return true;
}

//초기화: parent.resize(N + 1); iota(parent.begin(), parent.end(), 0);
```

### 반복문 버전 (재귀 깊이 걱정 없음)

#### Python
```python
#재귀 깊이 걱정되면 while로 타고 올라가자
def find(parent, x):
    #일단 루트부터 찾고
    root = x
    while parent[root] != root:
        root = parent[root]

    #왔던 길 되짚으면서 루트 직접 박기(경로 압축)
    while parent[x] != root:
        parent[x], x = root, parent[x]

    return root
```

#### C++
```cpp
//재귀 깊이 걱정되면 while로 타고 올라가자
int find(int x) {
    //일단 루트부터 찾고
    int root = x;
    while (parent[root] != root) root = parent[root];

    //왔던 길 되짚으면서 루트 직접 박기(경로 압축)
    while (parent[x] != root) {
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
        #자기 자신을 부모로 시작해보자
        self.parent = list(range(n + 1))

        #트리 높이. 낮은 놈을 높은 놈 밑에 붙이려고 들고 있음
        self.rank = [0] * (n + 1)

    #x의 루트를 경로를 압축하면서 찾자
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]

    #서로의 루트가 다를 경우 통합하자
    def union(self, x, y):
        root_a, root_b = self.find(x), self.find(y)

        #이미 한 집합이면 할 거 없음
        if root_a == root_b:
            return False

        #항상 root_a가 더 높은 트리가 되게 맞춰놓자
        if self.rank[root_a] < self.rank[root_b]:
            root_a, root_b = root_b, root_a

        self.parent[root_b] = root_a

        #높이 같았을 때만 붙이면서 한 칸 올라감
        if self.rank[root_a] == self.rank[root_b]:
            self.rank[root_a] += 1

        return True
```

#### C++
```cpp
struct UnionFind {
    //rank는 std::rank랑 충돌나니까 rnk로
    vector<int> parent, rnk;

    UnionFind(int n) : parent(n + 1), rnk(n + 1, 0) {
        //자기 자신을 부모로 시작해보자
        iota(parent.begin(), parent.end(), 0);
    }

    //x의 루트를 경로를 압축하면서 찾자
    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]);
        return parent[x];
    }

    //서로의 루트가 다를 경우 통합하자
    bool unite(int x, int y) {
        int root_a = find(x), root_b = find(y);

        //이미 한 집합이면 할 거 없음
        if (root_a == root_b) return false;

        //항상 root_a가 더 높은 트리가 되게 맞춰놓자
        if (rnk[root_a] < rnk[root_b]) swap(root_a, root_b);
        parent[root_b] = root_a;

        //높이 같았을 때만 붙이면서 한 칸 올라감
        if (rnk[root_a] == rnk[root_b]) rnk[root_a]++;

        return true;
    }
};
```

## 5. 실전 패턴

### 사이클 감지

#### Python
```python
#자기 자신을 부모로 시작해보자
parent = list(range(V + 1))
has_cycle = False

for u, v in edges:
    #union이 False면 이미 한 집합이었다는 뜻 > 사이클
    if not union(parent, u, v):
        has_cycle = True
```

#### C++
```cpp
//parent 초기화 후
bool hasCycle = false;

//e = {u, v}. unite가 false면 이미 한 집합이었다는 뜻 > 사이클
for (auto& e : edges)
    if (!unite(e.first, e.second))
        hasCycle = true;
```

### 연결 요소 개수

#### Python
```python
#자기 자신을 부모로 시작해보자
parent = list(range(V + 1))

for u, v in edges:
    union(parent, u, v)

#서로 다른 루트가 몇 개냐 = 덩어리가 몇 개냐
components = len(set(find(parent, i) for i in range(1, V + 1)))
```

#### C++
```cpp
for (auto& e : edges) unite(e.first, e.second);

//서로 다른 루트가 몇 개냐 = 덩어리가 몇 개냐
set<int> roots;
for (int i = 1; i <= V; i++) roots.insert(find(i));
int components = roots.size();
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
