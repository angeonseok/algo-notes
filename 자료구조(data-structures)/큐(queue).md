# Queue (큐)

> 한 줄 정리: 먼저 넣은 것을 먼저 꺼내는 구조 (FIFO)

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<queue>`, `<vector>`, `<utility>` (`pair`)

## 1. 언제 쓰는가
- FIFO(First In First Out) 구조가 필요할 때
- BFS 구현할 때
- 순서대로 처리해야 할 때
- 최단 거리/최소 횟수 문제
- 여러 작업을 순서대로 처리하는 시뮬레이션

## 2. 핵심 아이디어
- 가장 먼저 넣은 데이터를 먼저 꺼낸다.
- enqueue → 삽입 (`append` / `push`)
- dequeue → 제거 (`popleft` / `pop`)
- Python은 `collections.deque` (list 쓰면 안 됨), C++은 `queue<T>`

## 3. 시간복잡도
- enqueue: O(1)
- dequeue: O(1) → deque / queue 사용 시
- dequeue: O(n) → Python list의 pop(0)은 느림

## 4. 기본 코드

#### Python
```python
from collections import deque

queue = deque()

#enqueue
queue.append(1)
queue.append(2)

#dequeue
front = queue.popleft()

#front 확인
front = queue[0]

#empty 체크
if not queue:
    print("empty")
```

#### C++
```cpp
queue<int> q;

//enqueue
q.push(1);
q.push(2);

//pop()은 값을 안 돌려주니까 front()로 먼저 읽고 pop
int front = q.front();
q.pop();

//empty 체크
if (q.empty()) {
    //empty
}
```

## 5. 실전 패턴

### BFS 기본

#### Python
```python
from collections import deque

#시작점에서 연결된 노드 전부 훑기
def bfs(graph, start):
    queue = deque([start])
    visited = set([start])

    #BFS는 큐를 활용
    while queue:
        node = queue.popleft()

        #꺼낼 때 말고 넣을 때 방문 체크해야 큐에 중복 안 쌓임
        for v in graph[node]:
            if v not in visited:
                visited.add(v)
                queue.append(v)
```

#### C++
```cpp
//시작점에서 연결된 노드 전부 훑기
void bfs(vector<vector<int>>& graph, int start) {
    queue<int> q;
    q.push(start);
    vector<bool> visited(graph.size(), false);
    visited[start] = true;

    //BFS는 큐를 활용
    while (!q.empty()) {
        int node = q.front(); q.pop();

        //꺼낼 때 말고 넣을 때 방문 체크해야 큐에 중복 안 쌓임
        for (int v : graph[node]) {
            if (!visited[v]) {
                visited[v] = true;
                q.push(v);
            }
        }
    }
}
```

### BFS 최단 거리

#### Python
```python
from collections import deque

#시작점 > 각 노드까지 최단 거리 구하기
def bfs(graph, start):
    #(노드, 거리) 형태로 큐에 담기
    queue = deque([(start, 0)])
    visited = set([start])

    while queue:
        node, d = queue.popleft()

        #한 칸 더 갔으니까 d + 1 달아서 넣자
        for v in graph[node]:
            if v not in visited:
                visited.add(v)
                queue.append((v, d + 1))
```

#### C++
```cpp
//시작점 > 각 노드까지 최단 거리 구하기
void bfs(vector<vector<int>>& graph, int start) {
    //(노드, 거리) 형태로 큐에 담기
    queue<pair<int,int>> q;
    q.push({start, 0});
    vector<bool> visited(graph.size(), false);
    visited[start] = true;

    while (!q.empty()) {
        int node = q.front().first, d = q.front().second;
        q.pop();

        //한 칸 더 갔으니까 d + 1 달아서 넣자
        for (int v : graph[node]) {
            if (!visited[v]) {
                visited[v] = true;
                q.push({v, d + 1});
            }
        }
    }
}
```

### 2차원 배열 BFS

#### Python
```python
from collections import deque

#상하좌우 4방향
dx = [0, 0, 1, -1]
dy = [1, -1, 0, 0]

#(sx, sy)에서 시작해서 격자 퍼져나가기
def bfs(mat, sx, sy):
    queue = deque([(sx, sy)])
    visited = [[False] * len(mat[0]) for _ in range(len(mat))]
    visited[sx][sy] = True

    while queue:
        x, y = queue.popleft()

        #4방향 다 조사
        for i in range(4):
            nx, ny = x + dx[i], y + dy[i]

            #범위부터 보고 visited 봐야 인덱스 안 터짐
            if 0 <= nx < len(mat) and 0 <= ny < len(mat[0]) and not visited[nx][ny]:
                visited[nx][ny] = True
                queue.append((nx, ny))
```

#### C++
```cpp
//상하좌우 4방향
int dx[] = {0, 0, 1, -1};
int dy[] = {1, -1, 0, 0};

//(sx, sy)에서 시작해서 격자 퍼져나가기
void bfs(vector<vector<int>>& mat, int sx, int sy) {
    int N = mat.size(), M = mat[0].size();
    queue<pair<int,int>> q;
    q.push({sx, sy});
    vector<vector<bool>> visited(N, vector<bool>(M, false));
    visited[sx][sy] = true;

    while (!q.empty()) {
        int x = q.front().first, y = q.front().second;
        q.pop();

        //4방향 다 조사
        for (int i = 0; i < 4; i++) {
            int nx = x + dx[i], ny = y + dy[i];

            //범위부터 보고 visited 봐야 인덱스 안 터짐
            if (0 <= nx && nx < N && 0 <= ny && ny < M && !visited[nx][ny]) {
                visited[nx][ny] = true;
                q.push({nx, ny});
            }
        }
    }
}
```

## 6. 이걸 떠올려야 할 때
- "최단 거리/최소 횟수를 구하라" → BFS
- "레벨별로 처리해야 한다" → BFS
- "순서대로 처리해야 한다" → 큐
- "동시에 여러 시작점에서 퍼져나간다" → 멀티 소스 BFS (큐에 시작점 여러 개 넣기)

## 7. 자주 틀리는 포인트
- Python `list.pop(0)`은 O(n)이라 시간초과 → `deque` / C++ `queue` 사용
- visited 체크를 dequeue할 때 하면 같은 노드가 큐에 중복으로 쌓임 → enqueue할 때 바로 체크
- 2차원 BFS에서 범위 체크 순서 틀리면 IndexError → `0 <= nx < N and 0 <= ny < M` 먼저
- **C++ `queue::pop()`은 void** → `front()`로 값을 먼저 읽고 `pop()`
