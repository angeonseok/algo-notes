# LCA (Lowest Common Ancestor, 최소 공통 조상)

> 한 줄 정리: 트리에서 두 노드의 가장 가까운 공통 조상을 찾는 알고리즘

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<queue>`, `<algorithm>` (`swap`)

## 1. 언제 쓰는가
- 트리에서 두 노드의 공통 조상을 구할 때
- 두 노드 사이의 거리를 구할 때
- 트리에서 경로 쿼리가 많을 때

## 2. 핵심 아이디어
- 기본: 두 노드의 깊이를 맞추고 한 칸씩 올라가며 비교 → O(N)
- 희소 테이블(Sparse Table): 2^k번째 조상을 미리 저장 → O(log N)
- depth[v]: 루트에서 v까지의 깊이
- parent[k][v]: v의 2^k번째 조상

## 3. 시간/공간 복잡도
| 방법 | 전처리 | 쿼리 | 공간 |
|------|--------|------|------|
| 기본 (한 칸씩) | O(N) | O(N) | O(N) |
| 희소 테이블 | O(N log N) | O(log N) | O(N log N) |

## 4. 기본 코드

### 기본 구현 (O(N) 쿼리)

#### Python
```python
from collections import deque

def bfs(root, graph, N):
    depth = [-1] * (N + 1)
    parent = [0] * (N + 1)
    depth[root] = 0
    queue = deque([root])

    while queue:
        node = queue.popleft()
        for neighbor in graph[node]:
            if depth[neighbor] == -1:
                depth[neighbor] = depth[node] + 1
                parent[neighbor] = node
                queue.append(neighbor)

    return depth, parent

def lca_basic(u, v, depth, parent):
    while depth[u] > depth[v]:
        u = parent[u]
    while depth[v] > depth[u]:
        v = parent[v]
    while u != v:
        u = parent[u]
        v = parent[v]
    return u
```

#### C++
```cpp
vector<int> depth, par;   // par[v] = v의 직접 부모

void bfs(int root, vector<vector<int>>& graph, int N) {
    depth.assign(N + 1, -1);
    par.assign(N + 1, 0);
    depth[root] = 0;
    queue<int> q;
    q.push(root);
    while (!q.empty()) {
        int node = q.front(); q.pop();
        for (int next : graph[node])
            if (depth[next] == -1) {
                depth[next] = depth[node] + 1;
                par[next] = node;
                q.push(next);
            }
    }
}

int lcaBasic(int u, int v) {
    while (depth[u] > depth[v]) u = par[u];   // 깊이 맞추기
    while (depth[v] > depth[u]) v = par[v];
    while (u != v) { u = par[u]; v = par[v]; } // 같이 올라가기
    return u;
}
```

### 희소 테이블 구현 (O(log N) 쿼리)

#### Python
```python
from collections import deque
import math

def preprocess(root, graph, N):
    LOG = int(math.log2(N)) + 1
    depth = [-1] * (N + 1)
    parent = [[0] * (N + 1) for _ in range(LOG)]

    depth[root] = 0
    queue = deque([root])
    while queue:
        node = queue.popleft()
        for neighbor in graph[node]:
            if depth[neighbor] == -1:
                depth[neighbor] = depth[node] + 1
                parent[0][neighbor] = node
                queue.append(neighbor)

    # parent[k][v] = v의 2^k번째 조상
    for k in range(1, LOG):
        for v in range(1, N + 1):
            parent[k][v] = parent[k-1][parent[k-1][v]]

    return depth, parent, LOG

def lca(u, v, depth, parent, LOG):
    if depth[u] < depth[v]:
        u, v = v, u

    diff = depth[u] - depth[v]
    for k in range(LOG):
        if diff & (1 << k):
            u = parent[k][u]

    if u == v:
        return u

    for k in range(LOG - 1, -1, -1):
        if parent[k][u] != parent[k][v]:
            u = parent[k][u]
            v = parent[k][v]

    return parent[0][u]
```

#### C++
```cpp
int LOG;
vector<int> depth;
vector<vector<int>> parent;   // parent[k][v] = v의 2^k번째 조상

void preprocess(int root, vector<vector<int>>& graph, int N) {
    LOG = 1;
    while ((1 << LOG) < N) LOG++;
    LOG++;   // 여유 한 칸
    depth.assign(N + 1, -1);
    parent.assign(LOG, vector<int>(N + 1, 0));

    depth[root] = 0;
    queue<int> q;
    q.push(root);
    while (!q.empty()) {
        int node = q.front(); q.pop();
        for (int next : graph[node])
            if (depth[next] == -1) {
                depth[next] = depth[node] + 1;
                parent[0][next] = node;
                q.push(next);
            }
    }

    // 희소 테이블 채우기
    for (int k = 1; k < LOG; k++)
        for (int v = 1; v <= N; v++)
            parent[k][v] = parent[k-1][parent[k-1][v]];
}

int lca(int u, int v) {
    if (depth[u] < depth[v]) swap(u, v);   // u가 더 깊도록

    int diff = depth[u] - depth[v];        // 깊이 맞추기
    for (int k = 0; k < LOG; k++)
        if (diff & (1 << k))
            u = parent[k][u];

    if (u == v) return u;

    for (int k = LOG - 1; k >= 0; k--)     // 같아지기 직전까지 같이 올라가기
        if (parent[k][u] != parent[k][v]) {
            u = parent[k][u];
            v = parent[k][v];
        }
    return parent[0][u];
}
```

### 두 노드 사이의 거리

#### Python
```python
def distance(u, v, depth, parent, LOG):
    ancestor = lca(u, v, depth, parent, LOG)
    return depth[u] + depth[v] - 2 * depth[ancestor]
```

#### C++
```cpp
int treeDist(int u, int v) {   // std::distance 와 이름 충돌 피하려고 treeDist
    int ancestor = lca(u, v);
    return depth[u] + depth[v] - 2 * depth[ancestor];
}
```

## 5. 이걸 떠올려야 할 때
- "트리에서 두 노드의 공통 조상" → LCA
- "트리에서 두 노드 사이의 거리" → LCA로 depth[u] + depth[v] - 2 * depth[lca]
- 쿼리가 많으면 → 희소 테이블 O(log N)
- 쿼리가 적으면 → 기본 구현 O(N)으로 충분

## 6. 자주 틀리는 포인트
- LOG 크기를 `log2(N) + 1` 이상으로 잡아야 안전
- 희소 테이블에서 깊이 맞출 때 diff 비트를 낮은 자리부터 체크
- 같이 올라갈 때 `parent[k][u] != parent[k][v]` 조건 → 같아지기 직전까지만
- 루트의 부모는 0 또는 루트 자신으로 초기화
- 1-indexed 노드 번호 확인
