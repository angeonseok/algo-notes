# SCC (Strongly Connected Components, 강한 연결 요소)

> 한 줄 정리: 방향 그래프에서 서로 왕복 가능한 정점들의 묶음을 찾는 알고리즘

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<algorithm>` (`min`), `<stack>`, `<set>`

## 1. 언제 쓰는가
- 방향 그래프에서 순환 구조를 찾을 때
- 2-SAT 문제
- 그래프를 DAG로 압축할 때
- 사이클이 있는 방향 그래프 분석

## 2. 핵심 아이디어
- SCC: 집합 내 임의의 두 정점 u, v에 대해 u→v, v→u 경로가 모두 존재
- 같은 SCC 내 정점들은 하나의 노드로 압축 가능 → DAG로 변환
- 구현 방법: 코사라주(DFS 2번), 타잔(DFS 1번)

## 3. 시간복잡도
| 방법 | 복잡도 |
|------|--------|
| 코사라주 | O(V + E) |
| 타잔 | O(V + E) |

## 4. 기본 코드

### 코사라주 알고리즘 (구현 쉬움, 코테 권장)

#### Python
```python
import sys
from collections import defaultdict
sys.setrecursionlimit(100000)

def kosaraju(graph, reverse_graph, V):
    visited = [False] * (V + 1)
    order = []  # 종료 순서

    def dfs1(v):
        visited[v] = True
        for neighbor in graph[v]:
            if not visited[neighbor]:
                dfs1(neighbor)
        order.append(v)

    for v in range(1, V + 1):
        if not visited[v]:
            dfs1(v)

    visited = [False] * (V + 1)
    sccs = []

    def dfs2(v, scc):
        visited[v] = True
        scc.append(v)
        for neighbor in reverse_graph[v]:
            if not visited[neighbor]:
                dfs2(neighbor, scc)

    while order:
        v = order.pop()
        if not visited[v]:
            scc = []
            dfs2(v, scc)
            sccs.append(scc)

    return sccs
```

#### C++
```cpp
vector<vector<int>> graph, reverse_graph;
vector<bool> visited;
vector<int> order;             // 종료 순서
vector<vector<int>> sccs;

void dfs1(int v) {
    visited[v] = true;
    for (int next : graph[v])
        if (!visited[next]) dfs1(next);
    order.push_back(v);        // 종료 시점 기록
}

void dfs2(int v, vector<int>& scc) {
    visited[v] = true;
    scc.push_back(v);
    for (int next : reverse_graph[v])
        if (!visited[next]) dfs2(next, scc);
}

vector<vector<int>> kosaraju(int V) {
    visited.assign(V + 1, false);
    order.clear();
    for (int v = 1; v <= V; v++)
        if (!visited[v]) dfs1(v);

    visited.assign(V + 1, false);
    sccs.clear();
    for (int i = order.size() - 1; i >= 0; i--) {   // 종료 역순
        int v = order[i];
        if (!visited[v]) {
            vector<int> scc;
            dfs2(v, scc);
            sccs.push_back(scc);
        }
    }
    return sccs;
}
// 입력: graph[u].push_back(v); reverse_graph[v].push_back(u);
```

### 타잔 알고리즘 (DFS 1번, 구현 까다로움)

#### Python
```python
import sys
sys.setrecursionlimit(100000)

def tarjan(graph, V):
    order = [0]
    visited = [0] * (V + 1)
    low = [0] * (V + 1)
    on_stack = [False] * (V + 1)
    stack = []
    sccs = []

    def dfs(v):
        order[0] += 1
        visited[v] = low[v] = order[0]
        stack.append(v)
        on_stack[v] = True

        for neighbor in graph[v]:
            if not visited[neighbor]:
                dfs(neighbor)
                low[v] = min(low[v], low[neighbor])
            elif on_stack[neighbor]:
                low[v] = min(low[v], visited[neighbor])

        if low[v] == visited[v]:
            scc = []
            while True:
                u = stack.pop()
                on_stack[u] = False
                scc.append(u)
                if u == v:
                    break
            sccs.append(scc)

    for v in range(1, V + 1):
        if not visited[v]:
            dfs(v)

    return sccs
```

#### C++
```cpp
vector<vector<int>> graph;
vector<int> disc, low;      // disc: 방문 순서, low: 도달 가능한 최소 순서
vector<bool> on_stack;
stack<int> stk;
int timer = 0;
vector<vector<int>> sccs;

void tarjanDfs(int v) {
    disc[v] = low[v] = ++timer;
    stk.push(v);
    on_stack[v] = true;

    for (int next : graph[v]) {
        if (disc[next] == 0) {           // 미방문
            tarjanDfs(next);
            low[v] = min(low[v], low[next]);
        } else if (on_stack[next]) {     // 스택 위에 있는 정점만
            low[v] = min(low[v], disc[next]);
        }
    }

    if (low[v] == disc[v]) {             // v가 SCC의 루트
        vector<int> scc;
        while (true) {
            int u = stk.top(); stk.pop();
            on_stack[u] = false;
            scc.push_back(u);
            if (u == v) break;
        }
        sccs.push_back(scc);
    }
}
// disc.assign(V+1, 0); low.assign(V+1, 0); on_stack.assign(V+1, false);
// for (int v = 1; v <= V; v++) if (disc[v] == 0) tarjanDfs(v);
```

## 5. 코사라주 vs 타잔
| | 코사라주 | 타잔 |
|--|---------|------|
| DFS 횟수 | 2번 | 1번 |
| 역방향 그래프 | 필요 | 불필요 |
| 구현 난이도 | 쉬움 | 어려움 |
| 코테 권장 | ✓ | |

## 6. SCC 압축 → DAG 변환

#### Python
```python
scc_id = [0] * (V + 1)
for i, scc in enumerate(sccs):
    for v in scc:
        scc_id[v] = i

dag = defaultdict(set)
for v in range(1, V + 1):
    for neighbor in graph[v]:
        if scc_id[v] != scc_id[neighbor]:  # 다른 SCC 간 간선만
            dag[scc_id[v]].add(scc_id[neighbor])
```

#### C++
```cpp
// 각 정점이 속한 SCC 번호
vector<int> scc_id(V + 1);
for (int i = 0; i < (int)sccs.size(); i++)
    for (int v : sccs[i])
        scc_id[v] = i;

// SCC 간 간선으로 DAG 구성
vector<set<int>> dag(sccs.size());
for (int v = 1; v <= V; v++)
    for (int next : graph[v])
        if (scc_id[v] != scc_id[next])   // 다른 SCC 간 간선만
            dag[scc_id[v]].insert(scc_id[next]);
```

## 7. 이걸 떠올려야 할 때
- "방향 그래프에서 순환 구조" → SCC
- "2-SAT" → SCC 필수
- "그래프를 단순화해서 위상 정렬" → SCC로 DAG 압축 후 위상 정렬

## 8. 자주 틀리는 포인트
- 코사라주에서 역방향 그래프 빠뜨리는 경우
- 타잔에서 `on_stack` 체크 안 하면 이미 완성된 SCC를 참조
- 재귀 깊이 → Python `sys.setrecursionlimit`, C++은 스택 크기 주의
- SCC 압축 후 같은 SCC 내 간선은 제거해야 함
