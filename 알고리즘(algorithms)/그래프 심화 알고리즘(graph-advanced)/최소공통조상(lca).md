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

#루트에서 훑으면서 각 노드 깊이랑 부모 채워두기
def bfs(root, graph, N):
    depth = [-1] * (N + 1)
    parent = [0] * (N + 1)
    depth[root] = 0
    queue = deque([root])

    while queue:
        node = queue.popleft()
        for i in graph[node]:
            #깊이가 -1이면 아직 안 가본 놈. 트리니까 이게 곧 자식
            if depth[i] == -1:
                depth[i] = depth[node] + 1
                parent[i] = node
                queue.append(i)

    return depth, parent

#깊이 맞추고 한 칸씩 같이 올라가기
def lca_basic(u, v, depth, parent):
    #깊은 놈을 얕은 놈 높이까지 끌어올리자
    while depth[u] > depth[v]:
        u = parent[u]
    while depth[v] > depth[u]:
        v = parent[v]

    #높이 같아졌으면 만날 때까지 나란히 올리기
    while u != v:
        u = parent[u]
        v = parent[v]
    return u
```

#### C++
```cpp
//parent[v]는 v의 직접 부모
vector<int> depth, parent;

//루트에서 훑으면서 각 노드 깊이랑 부모 채워두기
void bfs(int root, vector<vector<int>>& graph, int N) {
    depth.assign(N + 1, -1);
    parent.assign(N + 1, 0);
    depth[root] = 0;
    queue<int> q;
    q.push(root);
    while (!q.empty()) {
        int node = q.front(); q.pop();

        //깊이가 -1이면 아직 안 가본 놈. 트리니까 이게 곧 자식
        for (int i : graph[node])
            if (depth[i] == -1) {
                depth[i] = depth[node] + 1;
                parent[i] = node;
                q.push(i);
            }
    }
}

//깊이 맞추고 한 칸씩 같이 올라가기
int lcaBasic(int u, int v) {
    //깊은 놈을 얕은 놈 높이까지 끌어올리자
    while (depth[u] > depth[v]) u = parent[u];
    while (depth[v] > depth[u]) v = parent[v];

    //높이 같아졌으면 만날 때까지 나란히 올리기
    while (u != v) { u = parent[u]; v = parent[v]; }
    return u;
}
```

### 희소 테이블 구현 (O(log N) 쿼리)

#### Python
```python
from collections import deque
import math

#조상 테이블 미리 채워두면 쿼리를 log로 후려칠 수 있음
def preprocess(root, graph, N):
    #2^LOG가 N을 덮어야 어떤 깊이든 점프 가능
    LOG = int(math.log2(N)) + 1
    depth = [-1] * (N + 1)
    parent = [[0] * (N + 1) for _ in range(LOG)]

    #일단 BFS로 depth랑 직접 부모(2^0)부터 채우기
    depth[root] = 0
    queue = deque([root])
    while queue:
        node = queue.popleft()
        for i in graph[node]:
            if depth[i] == -1:
                depth[i] = depth[node] + 1
                parent[0][i] = node
                queue.append(i)

    #2^k칸 위 = 2^(k-1)칸 두 번 올라간 곳. 낮은 k부터 채워야 함
    for k in range(1, LOG):
        for v in range(1, N + 1):
            parent[k][v] = parent[k-1][parent[k-1][v]]

    return depth, parent, LOG

#조상 테이블로 점프하면서 LCA 찾기
def lca(u, v, depth, parent, LOG):
    #u를 더 깊은 놈으로 고정해두면 케이스가 하나로 줄음
    if depth[u] < depth[v]:
        u, v = v, u

    #깊이 차를 2진수로 쪼개서 켜진 비트만큼 점프
    diff = depth[u] - depth[v]
    for k in range(LOG):
        if diff & (1 << k):
            u = parent[k][u]

    #끌어올렸더니 v면 v가 조상이었던 것
    if u == v:
        return u

    #조상이 갈리는 동안만 올리기. 큰 폭부터 해야 안 넘어감
    for k in range(LOG - 1, -1, -1):
        if parent[k][u] != parent[k][v]:
            u = parent[k][u]
            v = parent[k][v]

    #둘 다 LCA 바로 밑에 멈춰있으니 한 칸만 더
    return parent[0][u]
```

#### C++
```cpp
int LOG;
vector<int> depth;

//parent[k][v]는 v의 2^k번째 조상
vector<vector<int>> parent;

//조상 테이블 미리 채워두면 쿼리를 log로 후려칠 수 있음
void preprocess(int root, vector<vector<int>>& graph, int N) {
    //2^LOG가 N을 덮어야 어떤 깊이든 점프 가능
    LOG = 1;
    while ((1 << LOG) < N) LOG++;

    //빠듯하면 터지니까 여유 한 칸
    LOG++;
    depth.assign(N + 1, -1);
    parent.assign(LOG, vector<int>(N + 1, 0));

    //일단 BFS로 depth랑 직접 부모(2^0)부터 채우기
    depth[root] = 0;
    queue<int> q;
    q.push(root);
    while (!q.empty()) {
        int node = q.front(); q.pop();
        for (int i : graph[node])
            if (depth[i] == -1) {
                depth[i] = depth[node] + 1;
                parent[0][i] = node;
                q.push(i);
            }
    }

    //2^k칸 위 = 2^(k-1)칸 두 번 올라간 곳. 낮은 k부터 채워야 함
    for (int k = 1; k < LOG; k++)
        for (int v = 1; v <= N; v++)
            parent[k][v] = parent[k-1][parent[k-1][v]];
}

//조상 테이블로 점프하면서 LCA 찾기
int lca(int u, int v) {
    //u를 더 깊은 놈으로 고정해두면 케이스가 하나로 줄음
    if (depth[u] < depth[v]) swap(u, v);

    //깊이 차를 2진수로 쪼개서 켜진 비트만큼 점프
    int diff = depth[u] - depth[v];
    for (int k = 0; k < LOG; k++)
        if (diff & (1 << k))
            u = parent[k][u];

    //끌어올렸더니 v면 v가 조상이었던 것
    if (u == v) return u;

    //조상이 갈리는 동안만 올리기. 큰 폭부터 해야 안 넘어감
    for (int k = LOG - 1; k >= 0; k--)
        if (parent[k][u] != parent[k][v]) {
            u = parent[k][u];
            v = parent[k][v];
        }

    //둘 다 LCA 바로 밑에 멈춰있으니 한 칸만 더
    return parent[0][u];
}
```

### 두 노드 사이의 거리

#### Python
```python
#u > 루트 > v로 세면 LCA 위쪽이 두 번 겹치니까 2번 빼주기
def distance(u, v, depth, parent, LOG):
    ancestor = lca(u, v, depth, parent, LOG)
    return depth[u] + depth[v] - 2 * depth[ancestor]
```

#### C++
```cpp
//u > 루트 > v로 세면 LCA 위쪽이 두 번 겹치니까 2번 빼주기
//이름은 std::distance랑 충돌나서 treeDist로
int treeDist(int u, int v) {
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
