# Dinic (디닉)

> 한 줄 정리: BFS로 레벨 그래프를 만들고 DFS로 막힌 흐름을 찾아 O(V²E)를 달성하는 가장 빠른 최대 유량 알고리즘

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<queue>`, `<algorithm>` (`min`, `fill`), `<climits>`

## 1. 언제 쓰는가
- 그래프가 크고 빠른 최대 유량이 필요할 때
- 에드몬즈-카프가 시간초과날 때
- 플래티넘 이상 네트워크 플로우 문제

## 2. 핵심 아이디어
- BFS로 레벨 그래프 구성 (소스에서 각 정점까지의 거리)
- DFS로 레벨 그래프 위에서 막힌 흐름(blocking flow) 탐색
- 이미 탐색한 간선은 건너뜀 (current edge 최적화)
- 레벨 그래프 재구성 반복

## 3. 시간복잡도
| | 복잡도 |
|--|--------|
| 시간 | O(V²E) |
| 단위 용량 그래프 | O(E√V) |
| 에드몬즈-카프 대비 | 실전에서 훨씬 빠름 |

## 4. 기본 코드

#### Python
```python
from collections import deque

class Dinic:
    def __init__(self, V):
        self.V = V
        self.graph = [[] for _ in range(V)]

    def add_edge(self, u, v, cap):
        # graph[u][i] = [to, cap, rev_idx]
        self.graph[u].append([v, cap, len(self.graph[v])])
        self.graph[v].append([u, 0,   len(self.graph[u]) - 1])  # 역방향

    def bfs(self, s, t):
        self.level = [-1] * self.V
        self.level[s] = 0
        queue = deque([s])
        while queue:
            v = queue.popleft()
            for to, cap, _ in self.graph[v]:
                if cap > 0 and self.level[to] == -1:
                    self.level[to] = self.level[v] + 1
                    queue.append(to)
        return self.level[t] != -1

    def dfs(self, v, t, f):
        if v == t:
            return f
        while self.iter[v] < len(self.graph[v]):
            to, cap, rev = self.graph[v][self.iter[v]]
            if cap > 0 and self.level[v] < self.level[to]:
                d = self.dfs(to, t, min(f, cap))
                if d > 0:
                    self.graph[v][self.iter[v]][1] -= d
                    self.graph[to][rev][1] += d
                    return d
            self.iter[v] += 1
        return 0

    def max_flow(self, s, t):
        flow = 0
        while self.bfs(s, t):
            self.iter = [0] * self.V
            while True:
                f = self.dfs(s, t, float('inf'))
                if f == 0:
                    break
                flow += f
        return flow
```

#### C++
```cpp
struct Dinic {
    struct Edge { int to, cap, rev; };
    int V;
    vector<vector<Edge>> graph;
    vector<int> level, iter;

    Dinic(int V) : V(V), graph(V), level(V), iter(V) {}

    void addEdge(int u, int v, int cap) {
        graph[u].push_back({v, cap, (int)graph[v].size()});
        graph[v].push_back({u, 0,   (int)graph[u].size() - 1});   // 역방향
    }

    bool bfs(int s, int t) {
        fill(level.begin(), level.end(), -1);
        level[s] = 0;
        queue<int> q;
        q.push(s);
        while (!q.empty()) {
            int v = q.front(); q.pop();
            for (auto& e : graph[v])
                if (e.cap > 0 && level[e.to] == -1) {
                    level[e.to] = level[v] + 1;
                    q.push(e.to);
                }
        }
        return level[t] != -1;   // 싱크 도달 가능 여부
    }

    int dfs(int v, int t, int f) {
        if (v == t) return f;
        for (int& i = iter[v]; i < (int)graph[v].size(); i++) {   // current-arc 최적화
            Edge& e = graph[v][i];
            if (e.cap > 0 && level[v] < level[e.to]) {
                int d = dfs(e.to, t, min(f, e.cap));
                if (d > 0) {
                    e.cap -= d;
                    graph[e.to][e.rev].cap += d;
                    return d;
                }
            }
        }
        return 0;
    }

    int maxFlow(int s, int t) {
        int flow = 0;
        while (bfs(s, t)) {
            fill(iter.begin(), iter.end(), 0);   // 매 레벨 그래프마다 초기화
            int f;
            while ((f = dfs(s, t, INT_MAX)) > 0)
                flow += f;
        }
        return flow;
    }
};

// 사용 예시 (0-indexed)
// int V, E; cin >> V >> E;
// int S = 0, T = V - 1;
// Dinic dinic(V);
// for (int i = 0; i < E; i++) { int u,v,c; cin >> u >> v >> c; dinic.addEdge(u, v, c); }
// cout << dinic.maxFlow(S, T) << "\n";
```

## 5. 포드-풀커슨 vs 에드몬즈-카프 vs 디닉
| | 포드-풀커슨 | 에드몬즈-카프 | 디닉 |
|--|-----------|-------------|------|
| 경로 탐색 | DFS | BFS | BFS + DFS |
| 시간복잡도 | O(EF) | O(VE²) | O(V²E) |
| 구현 난이도 | 쉬움 | 보통 | 어려움 |
| 실전 성능 | 느림 | 보통 | 빠름 |
| 권장 상황 | 개념 이해 | 일반적 | 대규모 |

## 6. 이걸 떠올려야 할 때
- 에드몬즈-카프가 시간초과 → 디닉
- 그래프가 매우 크거나 용량이 클 때
- 이분 매칭도 디닉으로 O(E√V)에 풀 수 있음

## 7. 자주 틀리는 포인트
- `iter` 배열 매 BFS(레벨 그래프)마다 초기화 필수 → 안 하면 탐색 안 됨
- 역방향 간선 인덱스 관리 → `addEdge`에서 rev 정확히 저장
- 레벨 그래프에서 `level[v] < level[to]` 조건 빠뜨리면 오답
- 0-indexed vs 1-indexed 혼용 주의
- C++ `for (int& i = iter[v]; ...)` 는 참조로 current-arc를 유지하는 핵심 (값 복사하면 최적화 깨짐)
