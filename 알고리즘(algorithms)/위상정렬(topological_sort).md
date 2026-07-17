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

#의존관계 순서대로 노드 나열하기
def topological_sort(graph, indegree, V):
    q = deque()

    #진입차수 0 = 나보다 먼저 할 게 없는 놈. 얘네부터 시작
    for i in range(1, V + 1):
        if indegree[i] == 0:
            q.append(i)

    result = []
    while q:
        node = q.popleft()
        result.append(node)

        #나 처리했으니까 나만 기다리던 놈들 차수 하나씩 까주자
        for v in graph[node]:
            indegree[v] -= 1

            #기다릴 게 없어졌으면 얘도 큐에
            if indegree[v] == 0:
                q.append(v)

    #사이클 있으면 서로 물고 있어서 큐에 못 들어감 > 개수가 모자람
    if len(result) != V:
        return None

    return result
```

#### C++
```cpp
//의존관계 순서대로 노드 나열하기
vector<int> topologicalSort(vector<vector<int>>& graph, vector<int>& indegree, int V) {
    queue<int> q;

    //진입차수 0 = 나보다 먼저 할 게 없는 놈. 얘네부터 시작
    for (int i = 1; i <= V; i++)
        if (indegree[i] == 0)
            q.push(i);

    vector<int> result;
    while (!q.empty()) {
        int node = q.front(); q.pop();
        result.push_back(node);

        //나 처리했으니까 나만 기다리던 놈들 차수 하나씩 까주자
        for (int v : graph[node]) {
            indegree[v]--;

            //기다릴 게 없어졌으면 얘도 큐에
            if (indegree[v] == 0)
                q.push(v);
        }
    }

    //사이클 있으면 서로 물고 있어서 큐에 못 들어감 > 개수가 모자람
    if ((int)result.size() != V) return {};

    return result;
}

//입력: graph[u].push_back(v); indegree[v]++;
```

### 최장 경로 (위상 정렬 + DP)

#### Python
```python
#DAG라 순서대로 훑으면 앞쪽 dp는 이미 확정. 그대로 이어붙이면 됨
def longest_path(graph, indegree, V):
    q = deque()

    #dp[i] = i까지 오는 최장거리
    dp = [0] * (V + 1)

    #진입차수 0인 놈들부터 시작
    for i in range(1, V + 1):
        if indegree[i] == 0:
            q.append(i)

    while q:
        node = q.popleft()

        for v, w in graph[node]:
            #나 거쳐서 가는 게 더 길면 갱신하자
            dp[v] = max(dp[v], dp[node] + w)

            indegree[v] -= 1

            #들어오는 간선 다 봤으면 dp 확정 > 큐에
            if indegree[v] == 0:
                q.append(v)

    return max(dp)
```

#### C++
```cpp
//DAG라 순서대로 훑으면 앞쪽 dp는 이미 확정. 그대로 이어붙이면 됨
long long longestPath(vector<vector<pair<int,int>>>& graph, vector<int>& indegree, int V) {
    queue<int> q;

    //dp[i] = i까지 오는 최장거리
    vector<long long> dp(V + 1, 0);

    //진입차수 0인 놈들부터 시작
    for (int i = 1; i <= V; i++)
        if (indegree[i] == 0)
            q.push(i);

    while (!q.empty()) {
        int node = q.front(); q.pop();

        //e = (이웃, 가중치)
        for (auto& e : graph[node]) {
            int v = e.first, w = e.second;

            //나 거쳐서 가는 게 더 길면 갱신하자
            dp[v] = max(dp[v], dp[node] + w);

            indegree[v]--;

            //들어오는 간선 다 봤으면 dp 확정 > 큐에
            if (indegree[v] == 0)
                q.push(v);
        }
    }

    //제일 긴 놈 하나 뽑기
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
