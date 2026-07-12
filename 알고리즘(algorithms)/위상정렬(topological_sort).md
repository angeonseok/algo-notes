# Topological Sort (위상 정렬)

> 한 줄 정리: 방향 그래프에서 순서가 있는 작업들을 순서대로 나열하는 방법

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<queue>`, `<algorithm>` (`max`)

## 1. 언제 쓰는가
- 선수 과목, 작업 순서처럼 의존 관계가 있는 문제
- 사이클 없는 방향 그래프(DAG)에서 순서 결정
- 사이클 존재 여부 확인

## 2. 핵심 아이디어
- 진입 차수(in-degree)가 0인 노드부터 처리
- 처리 후 연결된 노드의 진입 차수 감소
- 진입 차수가 0이 된 노드를 큐에 추가
- 처리된 노드 수가 V보다 적으면 사이클 존재

## 3. 시간/공간 복잡도
| | 복잡도 |
|--|--------|
| 시간 | O(V + E) |
| 공간 | O(V + E) |

## 4. 기본 코드

### 칸 알고리즘 (BFS 기반)

#### Python
```python
from collections import deque

def topological_sort(graph, in_degree, V):
    queue = deque()

    for i in range(1, V + 1):
        if in_degree[i] == 0:
            queue.append(i)

    result = []
    while queue:
        node = queue.popleft()
        result.append(node)

        for neighbor in graph[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)

    # 사이클 감지
    if len(result) != V:
        return None  # 사이클 존재

    return result
```

#### C++
```cpp
vector<int> topologicalSort(vector<vector<int>>& graph, vector<int>& in_degree, int V) {
    queue<int> q;
    for (int i = 1; i <= V; i++)
        if (in_degree[i] == 0)
            q.push(i);

    vector<int> result;
    while (!q.empty()) {
        int node = q.front(); q.pop();
        result.push_back(node);

        for (int next : graph[node]) {
            in_degree[next]--;
            if (in_degree[next] == 0)
                q.push(next);
        }
    }

    if ((int)result.size() != V) return {};   // 사이클 존재
    return result;
}
// 입력: graph[u].push_back(v); in_degree[v]++;
```

### 최장 경로 (위상 정렬 + DP)

#### Python
```python
def longest_path(graph, in_degree, V):
    queue = deque()
    dp = [0] * (V + 1)

    for i in range(1, V + 1):
        if in_degree[i] == 0:
            queue.append(i)

    while queue:
        node = queue.popleft()
        for neighbor, weight in graph[node]:
            dp[neighbor] = max(dp[neighbor], dp[node] + weight)
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)

    return max(dp)
```

#### C++
```cpp
long long longestPath(vector<vector<pair<int,int>>>& graph, vector<int>& in_degree, int V) {
    queue<int> q;
    vector<long long> dp(V + 1, 0);

    for (int i = 1; i <= V; i++)
        if (in_degree[i] == 0)
            q.push(i);

    while (!q.empty()) {
        int node = q.front(); q.pop();
        for (auto& e : graph[node]) {          // e = (이웃, 가중치)
            int next = e.first, weight = e.second;
            dp[next] = max(dp[next], dp[node] + weight);
            in_degree[next]--;
            if (in_degree[next] == 0)
                q.push(next);
        }
    }

    long long ans = 0;
    for (int i = 1; i <= V; i++) ans = max(ans, dp[i]);
    return ans;
}
```

## 5. 이걸 떠올려야 할 때
- "순서가 정해진 작업", "선수 과목" → 위상 정렬
- "사이클 존재 여부" → 위상 정렬 후 처리 노드 수 확인
- "DAG에서 최장/최단 경로" → 위상 정렬 + DP

## 6. 자주 틀리는 포인트
- 사이클 있는 그래프에 위상 정렬 → 일부 노드 처리 못함 → 결과 길이로 감지
- 진입 차수 배열 초기화 빠뜨리는 경우
- 위상 정렬 결과가 유일하지 않을 수 있음 → 여러 정답 가능
- 사전순 결과가 필요하면 큐 대신 `priority_queue`(최소 힙) 사용
