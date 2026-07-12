# Bipartite Matching (이분 매칭)

> 한 줄 정리: 두 그룹 사이에서 최대한 많은 쌍을 매칭하는 알고리즘

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<queue>`

## 1. 언제 쓰는가
- 두 집합 사이의 최대 매칭 수를 구할 때
- 일대일 대응이 가능한 최대 쌍의 수
- 작업-사람 배정, 좌석 배정 문제
- 최소 버텍스 커버 (쾨닉 정리)

## 2. 핵심 아이디어
- 이분 그래프: 정점을 두 그룹으로 나눌 수 있고 간선이 그룹 사이에만 존재
- DFS로 증가 경로를 찾아 매칭을 하나씩 늘림
- 이미 매칭된 정점도 다른 경로가 있으면 재배정 가능
- 시간복잡도: O(V * E)

## 3. 시간복잡도
| | 복잡도 |
|--|--------|
| 시간 | O(V × E) |
| 공간 | O(V + E) |

## 4. 기본 코드

### 이분 매칭 기본 구현

#### Python
```python
from collections import defaultdict

def bipartite_matching(graph, left_size, right_size):
    match_left  = [-1] * (left_size + 1)   # match_left[i]  = i와 매칭된 오른쪽 정점
    match_right = [-1] * (right_size + 1)  # match_right[j] = j와 매칭된 왼쪽 정점

    def dfs(v, visited):
        for u in graph[v]:
            if visited[u]:
                continue
            visited[u] = True
            # u가 매칭 안 됐거나, u의 매칭 상대가 다른 곳으로 갈 수 있으면
            if match_right[u] == -1 or dfs(match_right[u], visited):
                match_left[v] = u
                match_right[u] = v
                return True
        return False

    result = 0
    for v in range(1, left_size + 1):
        visited = [False] * (right_size + 1)
        if dfs(v, visited):
            result += 1

    return result, match_left, match_right
```

#### C++
```cpp
vector<vector<int>> graph;   // 왼쪽 정점 → 연결된 오른쪽 정점들
vector<int> match_left, match_right;
vector<bool> visited;

bool dfs(int v) {
    for (int u : graph[v]) {
        if (visited[u]) continue;
        visited[u] = true;
        // u가 매칭 안 됐거나, u의 매칭 상대가 다른 곳으로 갈 수 있으면
        if (match_right[u] == -1 || dfs(match_right[u])) {
            match_left[v] = u;
            match_right[u] = v;
            return true;
        }
    }
    return false;
}

int bipartiteMatching(int left_size, int right_size) {
    match_left.assign(left_size + 1, -1);
    match_right.assign(right_size + 1, -1);

    int result = 0;
    for (int v = 1; v <= left_size; v++) {
        visited.assign(right_size + 1, false);   // 매 정점마다 초기화
        if (dfs(v)) result++;
    }
    return result;
}
```

## 5. 실전 패턴

### 매칭 결과 확인

#### Python
```python
for i in range(1, left_size + 1):
    if match_left[i] != -1:
        print(f"{i} - {match_left[i]}")
```

#### C++
```cpp
for (int i = 1; i <= left_size; i++)
    if (match_left[i] != -1)
        cout << i << " - " << match_left[i] << "\n";
```

### 이분 그래프 판별

#### Python
```python
from collections import deque

def is_bipartite(graph, V):
    color = [-1] * (V + 1)

    for start in range(1, V + 1):
        if color[start] != -1:
            continue
        color[start] = 0
        queue = deque([start])

        while queue:
            v = queue.popleft()
            for neighbor in graph[v]:
                if color[neighbor] == -1:
                    color[neighbor] = 1 - color[v]
                    queue.append(neighbor)
                elif color[neighbor] == color[v]:
                    return False  # 같은 색 → 이분 그래프 아님

    return True
```

#### C++
```cpp
bool isBipartite(vector<vector<int>>& graph, int V) {
    vector<int> color(V + 1, -1);
    for (int start = 1; start <= V; start++) {
        if (color[start] != -1) continue;
        color[start] = 0;
        queue<int> q;
        q.push(start);
        while (!q.empty()) {
            int v = q.front(); q.pop();
            for (int next : graph[v]) {
                if (color[next] == -1) {
                    color[next] = 1 - color[v];
                    q.push(next);
                } else if (color[next] == color[v]) {
                    return false;   // 같은 색 → 이분 그래프 아님
                }
            }
        }
    }
    return true;
}
```

## 6. 이걸 떠올려야 할 때
- "두 그룹 사이 최대 매칭" → 이분 매칭
- "사람-작업 최대 배정" → 이분 매칭
- "최소 버텍스 커버" → 최대 매칭 수와 같음 (쾨닉 정리)
- "이분 그래프인지 판별" → BFS로 2-색칠

## 7. 자주 틀리는 포인트
- visited 배열을 매 DFS마다 초기화해야 함 → for 루프 안에서 초기화
- match_right 재배정 가능 여부 체크 빠뜨리면 최적 매칭 못 찾음
- 이분 그래프가 아닌 그래프에 적용하면 오답
- left/right 인덱스 범위 혼용 주의
