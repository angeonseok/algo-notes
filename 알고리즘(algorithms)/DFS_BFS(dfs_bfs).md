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
#재귀 호출 스택이 알아서 스택 역할 해줌
def dfs(graph, node, visited):
    #들어오자마자 방문 체크해야 무한루프 안 돎
    visited.add(node)
    print(node)

    #안 가본 이웃으로 계속 깊게 파고들자
    for i in graph[node]:
        if i not in visited:
            dfs(graph, i, visited)

visited = set()
dfs(graph, start, visited)
```

#### C++
```cpp
vector<vector<int>> graph;
vector<bool> visited;

//재귀 호출 스택이 알아서 스택 역할 해줌
void dfs(int node) {
    //들어오자마자 방문 체크해야 무한루프 안 돎
    visited[node] = true;
    cout << node << "\n";

    //안 가본 이웃으로 계속 깊게 파고들자
    for (int i : graph[node])
        if (!visited[i])
            dfs(i);
}
//visited.assign(V + 1, false); dfs(start);
```

### DFS - 스택 (반복문)

#### Python
```python
#재귀 깊어지면 터지니까 스택 직접 들고 돌자
def dfs(graph, start):
    stack = [start]
    visited = set([start])

    while stack:
        #pop이 마지막에 넣은 놈부터 꺼내니까 깊이 우선
        node = stack.pop()
        print(node)

        #넣을 때 바로 방문 체크. 꺼낼 때 하면 중복으로 쌓임
        for i in graph[node]:
            if i not in visited:
                visited.add(i)
                stack.append(i)
```

#### C++
```cpp
//재귀 깊어지면 터지니까 스택 직접 들고 돌자
void dfs(int start, int V) {
    stack<int> st;
    st.push(start);
    vector<bool> visited(V + 1, false);
    visited[start] = true;

    while (!st.empty()) {
        //마지막에 넣은 놈부터 꺼내니까 깊이 우선
        int node = st.top(); st.pop();
        cout << node << "\n";

        //넣을 때 바로 방문 체크. 꺼낼 때 하면 중복으로 쌓임
        for (int i : graph[node])
            if (!visited[i]) {
                visited[i] = true;
                st.push(i);
            }
    }
}
```

### BFS

#### Python
```python
from collections import deque

#가까운 놈부터 퍼지려면 큐를 쓰자
def bfs(graph, start):
    queue = deque([start])
    visited = set([start])

    while queue:
        #먼저 넣은 놈부터 꺼내니까 너비 우선
        node = queue.popleft()
        print(node)

        #넣을 때 바로 방문 체크. 꺼낼 때 하면 중복으로 쌓임
        for i in graph[node]:
            if i not in visited:
                visited.add(i)
                queue.append(i)
```

#### C++
```cpp
//가까운 놈부터 퍼지려면 큐를 쓰자
void bfs(int start, int V) {
    queue<int> q;
    q.push(start);
    vector<bool> visited(V + 1, false);
    visited[start] = true;

    while (!q.empty()) {
        //먼저 넣은 놈부터 꺼내니까 너비 우선
        int node = q.front(); q.pop();
        cout << node << "\n";

        //넣을 때 바로 방문 체크. 꺼낼 때 하면 중복으로 쌓임
        for (int i : graph[node])
            if (!visited[i]) {
                visited[i] = true;
                q.push(i);
            }
    }
}
```

### BFS 최단 거리

#### Python
```python
from collections import deque

#가중치 없으면 BFS 퍼지는 순서가 곧 최단거리
def bfs(graph, start):
    #(노드, 거리) 같이 들고 다니자
    queue = deque([(start, 0)])
    visited = set([start])

    while queue:
        node, dist = queue.popleft()

        #한 칸 더 갔으니 dist+1
        for i in graph[node]:
            if i not in visited:
                visited.add(i)
                queue.append((i, dist + 1))
```

#### C++
```cpp
//가중치 없으면 BFS 퍼지는 순서가 곧 최단거리
void bfs(int start, int V) {
    //(노드, 거리) 같이 들고 다니자
    queue<pair<int,int>> q;
    q.push({start, 0});
    vector<bool> visited(V + 1, false);
    visited[start] = true;

    while (!q.empty()) {
        int node = q.front().first, dist = q.front().second;
        q.pop();

        //한 칸 더 갔으니 dist+1
        for (int i : graph[node])
            if (!visited[i]) {
                visited[i] = true;
                q.push({i, dist + 1});
            }
    }
}
```

### 2차원 그리드 BFS

#### Python
```python
from collections import deque

#상하좌우 네 방향 미리 빼놓자
dx = [0, 0, 1, -1]
dy = [1, -1, 0, 0]

def bfs(mat, sx, sy):
    N, M = len(mat), len(mat[0])

    #(x, y, 거리) 같이 들고 다니자
    queue = deque([(sx, sy, 0)])
    visited = [[False] * M for _ in range(N)]
    visited[sx][sy] = True

    while queue:
        x, y, dist = queue.popleft()

        #네 방향 다 뻗어보기
        for i in range(4):
            nx, ny = x + dx[i], y + dy[i]

            #격자 밖으로 안 나가고 아직 안 가본 칸만
            if 0 <= nx < N and 0 <= ny < M and not visited[nx][ny]:
                visited[nx][ny] = True
                queue.append((nx, ny, dist + 1))
```

#### C++
```cpp
//상하좌우 네 방향 미리 빼놓자
int dx[] = {0, 0, 1, -1};
int dy[] = {1, -1, 0, 0};

void bfs(vector<vector<int>>& mat, int sx, int sy) {
    int N = mat.size(), M = mat[0].size();

    //(x, y, 거리) 같이 들고 다니자
    queue<tuple<int,int,int>> q;
    q.push({sx, sy, 0});
    vector<vector<bool>> visited(N, vector<bool>(M, false));
    visited[sx][sy] = true;

    while (!q.empty()) {
        //C++14엔 구조적 바인딩 없으니까 tie로 풀기
        int x, y, dist;
        tie(x, y, dist) = q.front();
        q.pop();

        //네 방향 다 뻗어보기
        for (int i = 0; i < 4; i++) {
            int nx = x + dx[i], ny = y + dy[i];

            //격자 밖으로 안 나가고 아직 안 가본 칸만
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

#시작점이 여러 개면 걍 다 큐에 처음부터 때려박고 동시에 퍼뜨리자
def multi_bfs(mat, sources):
    queue = deque()
    visited = set()

    #모든 시작점을 거리 0으로 깔고 시작
    for src in sources:
        queue.append((src, 0))
        visited.add(src)

    while queue:
        node, dist = queue.popleft()

        #한 칸 더 갔으니 dist+1
        for i in graph[node]:
            if i not in visited:
                visited.add(i)
                queue.append((i, dist + 1))
```

#### C++
```cpp
//시작점이 여러 개면 걍 다 큐에 처음부터 때려박고 동시에 퍼뜨리자
void multiBfs(vector<int>& sources, int V) {
    //(노드, 거리)
    queue<pair<int,int>> q;
    vector<bool> visited(V + 1, false);

    //모든 시작점을 거리 0으로 깔고 시작
    for (int src : sources) {
        q.push({src, 0});
        visited[src] = true;
    }

    while (!q.empty()) {
        int node = q.front().first, dist = q.front().second;
        q.pop();

        //한 칸 더 갔으니 dist+1
        for (int i : graph[node])
            if (!visited[i]) {
                visited[i] = true;
                q.push({i, dist + 1});
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
