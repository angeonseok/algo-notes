# DFS / BFS (깊이 우선 탐색 / 너비 우선 탐색)

> 한 줄 정리: 그래프/트리를 탐색하는 두 가지 방법, DFS는 깊게 파고들고 BFS는 가까운 것부터 퍼져나간다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<queue>`, `<stack>`, `<tuple>`, `<utility>`

## 1. 언제 쓰는가

### DFS
- 경로가 존재하는지 확인
- 모든 경우를 탐색해야 할 때 (백트래킹)
- 연결 요소 개수, 사이클 감지
- 트리 순회

### BFS
- 최단 거리/최소 횟수
- 레벨별로 처리해야 할 때
- 가중치 없는 그래프에서 최단 경로
- 멀티 소스 탐색 (시작점이 여러 개)

## 2. 핵심 차이

| | DFS | BFS |
|--|-----|-----|
| 자료구조 | 스택 (재귀) | 큐 |
| 탐색 방식 | 깊이 우선 | 너비 우선 |
| 최단 거리 | 보장 안 됨 | 보장됨 (가중치 없을 때) |
| 메모리 | O(깊이) | O(너비) |
| 적합한 문제 | 경로 탐색, 백트래킹 | 최단 거리, 레벨 탐색 |

## 3. 시간/공간 복잡도
| | 시간 | 공간 |
|--|------|------|
| DFS | O(V + E) | O(V) |
| BFS | O(V + E) | O(V) |

## 4. 기본 코드

### DFS - 재귀

#### Python
```python
def dfs(graph, node, visited):
    visited.add(node)
    print(node)

    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited)

visited = set()
dfs(graph, start, visited)
```

#### C++
```cpp
vector<vector<int>> graph;
vector<bool> visited;

void dfs(int node) {
    visited[node] = true;
    cout << node << "\n";
    for (int next : graph[node])
        if (!visited[next])
            dfs(next);
}
// visited.assign(V + 1, false); dfs(start);
```

### DFS - 스택 (반복문)

#### Python
```python
def dfs(graph, start):
    stack = [start]
    visited = set([start])

    while stack:
        node = stack.pop()
        print(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                stack.append(neighbor)
```

#### C++
```cpp
void dfs(int start, int V) {
    stack<int> st;
    st.push(start);
    vector<bool> visited(V + 1, false);
    visited[start] = true;

    while (!st.empty()) {
        int node = st.top(); st.pop();
        cout << node << "\n";
        for (int next : graph[node])
            if (!visited[next]) {
                visited[next] = true;
                st.push(next);
            }
    }
}
```

### BFS

#### Python
```python
from collections import deque

def bfs(graph, start):
    queue = deque([start])
    visited = set([start])

    while queue:
        node = queue.popleft()
        print(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
```

#### C++
```cpp
void bfs(int start, int V) {
    queue<int> q;
    q.push(start);
    vector<bool> visited(V + 1, false);
    visited[start] = true;

    while (!q.empty()) {
        int node = q.front(); q.pop();
        cout << node << "\n";
        for (int next : graph[node])
            if (!visited[next]) {
                visited[next] = true;
                q.push(next);
            }
    }
}
```

### BFS 최단 거리

#### Python
```python
from collections import deque

def bfs(graph, start):
    queue = deque([(start, 0)])  # (노드, 거리)
    visited = set([start])

    while queue:
        node, dist = queue.popleft()
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, dist + 1))
```

#### C++
```cpp
void bfs(int start, int V) {
    queue<pair<int,int>> q;   // (노드, 거리)
    q.push({start, 0});
    vector<bool> visited(V + 1, false);
    visited[start] = true;

    while (!q.empty()) {
        int node = q.front().first, dist = q.front().second;
        q.pop();
        for (int next : graph[node])
            if (!visited[next]) {
                visited[next] = true;
                q.push({next, dist + 1});
            }
    }
}
```

### 2차원 그리드 BFS

#### Python
```python
from collections import deque

dx = [0, 0, 1, -1]
dy = [1, -1, 0, 0]

def bfs(grid, sx, sy):
    N, M = len(grid), len(grid[0])
    queue = deque([(sx, sy, 0)])  # (x, y, 거리)
    visited = [[False] * M for _ in range(N)]
    visited[sx][sy] = True

    while queue:
        x, y, dist = queue.popleft()
        for i in range(4):
            nx, ny = x + dx[i], y + dy[i]
            if 0 <= nx < N and 0 <= ny < M and not visited[nx][ny]:
                visited[nx][ny] = True
                queue.append((nx, ny, dist + 1))
```

#### C++
```cpp
int dx[] = {0, 0, 1, -1};
int dy[] = {1, -1, 0, 0};

void bfs(vector<vector<int>>& grid, int sx, int sy) {
    int N = grid.size(), M = grid[0].size();
    queue<tuple<int,int,int>> q;   // (x, y, 거리)
    q.push({sx, sy, 0});
    vector<vector<bool>> visited(N, vector<bool>(M, false));
    visited[sx][sy] = true;

    while (!q.empty()) {
        int x, y, dist;
        tie(x, y, dist) = q.front();   // C++14: 구조적 바인딩 대신 tie
        q.pop();
        for (int i = 0; i < 4; i++) {
            int nx = x + dx[i], ny = y + dy[i];
            if (0 <= nx && nx < N && 0 <= ny && ny < M && !visited[nx][ny]) {
                visited[nx][ny] = true;
                q.push({nx, ny, dist + 1});
            }
        }
    }
}
```

### 멀티 소스 BFS (시작점 여러 개)

#### Python
```python
from collections import deque

def multi_bfs(grid, sources):
    queue = deque()
    visited = set()

    # 모든 시작점을 큐에 넣고 시작
    for src in sources:
        queue.append((src, 0))
        visited.add(src)

    while queue:
        node, dist = queue.popleft()
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, dist + 1))
```

#### C++
```cpp
void multiBfs(vector<int>& sources, int V) {
    queue<pair<int,int>> q;   // (노드, 거리)
    vector<bool> visited(V + 1, false);

    // 모든 시작점을 큐에 넣고 시작
    for (int src : sources) {
        q.push({src, 0});
        visited[src] = true;
    }

    while (!q.empty()) {
        int node = q.front().first, dist = q.front().second;
        q.pop();
        for (int next : graph[node])
            if (!visited[next]) {
                visited[next] = true;
                q.push({next, dist + 1});
            }
    }
}
```

## 5. 이걸 떠올려야 할 때
- "최단 거리/최소 횟수" → BFS
- "모든 경로 탐색", "경우의 수" → DFS
- "연결 요소 개수" → DFS or BFS 둘 다
- "동시에 여러 곳에서 퍼져나감" → 멀티 소스 BFS
- "레벨별로 처리" → BFS

## 6. 자주 틀리는 포인트
- visited 체크를 dequeue할 때 하면 같은 노드가 큐에 중복으로 쌓임 → enqueue할 때 바로 체크
- DFS 재귀에서 Python은 `sys.setrecursionlimit`, C++은 깊은 그래프에서 스택 오버플로 → 반복(스택) 전환
- 2차원 BFS에서 범위 체크 순서 틀리면 IndexError → `0 <= nx < N and 0 <= ny < M` 먼저
- DFS 스택 방식은 재귀 방식과 탐색 순서가 다를 수 있음
