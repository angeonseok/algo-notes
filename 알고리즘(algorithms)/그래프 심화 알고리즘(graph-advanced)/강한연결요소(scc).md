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

#DFS 두 번 돌려서 SCC 묶음들 뽑아내기
def kosaraju(graph, reverse_graph, V):
    visited = [False] * (V + 1)

    #DFS 끝나는 순서대로 쌓아두자
    order = []

    #원본 그래프로 훑으면서 종료 순서 기록
    def dfs1(v):
        visited[v] = True
        for i in graph[v]:
            if not visited[i]:
                dfs1(i)
        order.append(v)

    for v in range(1, V + 1):
        if not visited[v]:
            dfs1(v)

    #2차 탐색용으로 방문 기록 초기화
    visited = [False] * (V + 1)
    sccs = []

    #역방향으로 훑었을 때 한 덩어리로 묶이는 놈들이 곧 SCC
    def dfs2(v, scc):
        visited[v] = True
        scc.append(v)
        for i in reverse_graph[v]:
            if not visited[i]:
                dfs2(i, scc)

    #늦게 끝난 놈부터 꺼내야 하니까 pop
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

//DFS 끝나는 순서대로 쌓아두자
vector<int> order;
vector<vector<int>> sccs;

//원본 그래프로 훑으면서 종료 순서 기록
void dfs1(int v) {
    visited[v] = true;
    for (int i : graph[v])
        if (!visited[i]) dfs1(i);

    //돌아 나올 때 찍어야 종료 시점이 됨
    order.push_back(v);
}

//역방향으로 훑었을 때 한 덩어리로 묶이는 놈들이 곧 SCC
void dfs2(int v, vector<int>& scc) {
    visited[v] = true;
    scc.push_back(v);
    for (int i : reverse_graph[v])
        if (!visited[i]) dfs2(i, scc);
}

//DFS 두 번 돌려서 SCC 묶음들 뽑아내기
vector<vector<int>> kosaraju(int V) {
    visited.assign(V + 1, false);
    order.clear();
    for (int v = 1; v <= V; v++)
        if (!visited[v]) dfs1(v);

    //2차 탐색용으로 방문 기록 초기화
    visited.assign(V + 1, false);
    sccs.clear();

    //늦게 끝난 놈부터 봐야 하니까 종료 역순으로
    for (int i = order.size() - 1; i >= 0; i--) {
        int v = order[i];
        if (!visited[v]) {
            vector<int> scc;
            dfs2(v, scc);
            sccs.push_back(scc);
        }
    }
    return sccs;
}

//입력: graph[u].push_back(v); reverse_graph[v].push_back(u);
```

### 타잔 알고리즘 (DFS 1번, 구현 까다로움)

#### Python
```python
import sys
sys.setrecursionlimit(100000)

#DFS 한 번으로 SCC 뽑기
def tarjan(graph, V):
    order = [0]
    visited = [0] * (V + 1)

    #low는 역행 간선 타고 닿을 수 있는 제일 빠른 방문 번호
    low = [0] * (V + 1)
    on_stack = [False] * (V + 1)
    stack = []
    sccs = []

    def dfs(v):
        #방문 번호 찍고 일단 스택에 넣어두기
        order[0] += 1
        visited[v] = low[v] = order[0]
        stack.append(v)
        on_stack[v] = True

        for i in graph[v]:
            #아직 안 가본 놈이면 내려갔다 와서 low 당겨오기
            if not visited[i]:
                dfs(i)
                low[v] = min(low[v], low[i])

            #스택에 남아있는 놈만 같은 SCC 후보. 이미 끝난 SCC는 건드리면 안 됨
            elif on_stack[i]:
                low[v] = min(low[v], visited[i])

        #더 위로 못 올라가면 v가 SCC 루트. 스택에서 v까지 걷어내자
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

//disc는 방문 순서, low는 역행 간선 타고 닿을 수 있는 제일 빠른 순서
vector<int> disc, low;
vector<bool> on_stack;
stack<int> stk;
int timer = 0;
vector<vector<int>> sccs;

//DFS 한 번으로 SCC 뽑기
void tarjanDfs(int v) {
    //방문 번호 찍고 일단 스택에 넣어두기
    disc[v] = low[v] = ++timer;
    stk.push(v);
    on_stack[v] = true;

    for (int i : graph[v]) {
        //아직 안 가본 놈이면 내려갔다 와서 low 당겨오기
        if (disc[i] == 0) {
            tarjanDfs(i);
            low[v] = min(low[v], low[i]);
        } else if (on_stack[i]) {
            //스택 위에 있는 놈만 같은 SCC 후보. 이미 끝난 SCC는 건드리면 안 됨
            low[v] = min(low[v], disc[i]);
        }
    }

    //더 위로 못 올라가면 v가 SCC 루트. 스택에서 v까지 걷어내자
    if (low[v] == disc[v]) {
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

//disc.assign(V+1, 0); low.assign(V+1, 0); on_stack.assign(V+1, false);
//for (int v = 1; v <= V; v++) if (disc[v] == 0) tarjanDfs(v);
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
#각 정점이 어느 SCC 소속인지 번호 붙여주기
scc_id = [0] * (V + 1)
for i, scc in enumerate(sccs):
    for v in scc:
        scc_id[v] = i

#SCC 하나를 노드 하나로 보면 사이클 없는 DAG가 나옴
dag = defaultdict(set)
for v in range(1, V + 1):
    for i in graph[v]:
        #같은 SCC 안 간선은 압축되면서 사라지니까 스킵
        if scc_id[v] != scc_id[i]:
            dag[scc_id[v]].add(scc_id[i])
```

#### C++
```cpp
//각 정점이 어느 SCC 소속인지 번호 붙여주기
vector<int> scc_id(V + 1);
for (int i = 0; i < (int)sccs.size(); i++)
    for (int v : sccs[i])
        scc_id[v] = i;

//SCC 하나를 노드 하나로 보면 사이클 없는 DAG가 나옴
vector<set<int>> dag(sccs.size());
for (int v = 1; v <= V; v++)
    for (int i : graph[v])
        //같은 SCC 안 간선은 압축되면서 사라지니까 스킵
        if (scc_id[v] != scc_id[i])
            dag[scc_id[v]].insert(scc_id[i]);
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
