# Prim (프림)

> 한 줄 정리: 현재 MST에서 가장 가까운 정점을 하나씩 추가해서 MST를 만든다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<queue>`, `<utility>`, `<functional>` (`greater`)

## 1. 언제 쓰는가
- 최소 신장 트리(MST) 구할 때
- 간선 수가 많은 밀집 그래프
- 다익스트라와 구조가 비슷해 익숙하면 편함

## 2. 핵심 아이디어
- 임의의 시작 정점에서 출발
- 현재 MST와 연결된 간선 중 가중치가 가장 작은 것 선택
- 힙으로 구현하면 O((V+E) log V)
- 정점 중심 알고리즘

## 3. 시간/공간 복잡도
| | 복잡도 |
|--|--------|
| 시간 | O((V + E) log V) |
| 공간 | O(V + E) |

## 4. 기본 코드

#### Python
```python
import heapq
import sys

def prim(graph, start, V):
    INF = sys.maxsize
    visited = [False] * (V + 1)
    heap = [(0, start)]  # (가중치, 노드)
    total = 0

    while heap:
        weight, node = heapq.heappop(heap)

        if visited[node]:
            continue

        visited[node] = True
        total += weight

        for neighbor, w in graph[node]:
            if not visited[neighbor]:
                heapq.heappush(heap, (w, neighbor))

    return total

# 입력 예시
from collections import defaultdict

V, E = map(int, input().split())
graph = defaultdict(list)
for _ in range(E):
    u, v, w = map(int, input().split())
    graph[u].append((v, w))
    graph[v].append((u, w))  # 양방향
```

#### C++
```cpp
long long prim(vector<vector<pair<int,int>>>& graph, int start, int V) {
    vector<bool> visited(V + 1, false);
    // 최소 힙: (가중치, 노드)
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    pq.push({0, start});
    long long total = 0;

    while (!pq.empty()) {
        int weight = pq.top().first;
        int node = pq.top().second;
        pq.pop();

        if (visited[node]) continue;
        visited[node] = true;
        total += weight;

        for (auto& e : graph[node]) {   // e = (이웃, 가중치)
            int next = e.first, w = e.second;
            if (!visited[next])
                pq.push({w, next});
        }
    }
    return total;
}

// 입력 예시
// int V, E; cin >> V >> E;
// vector<vector<pair<int,int>>> graph(V + 1);
// for (int i = 0; i < E; i++) {
//     int u, v, w; cin >> u >> v >> w;
//     graph[u].push_back({v, w});
//     graph[v].push_back({u, w});   // 양방향
// }
```

## 5. 크루스칼 vs 프림
| | 크루스칼 | 프림 |
|--|---------|------|
| 방식 | 간선 중심 | 정점 중심 |
| 시간복잡도 | O(E log E) | O((V+E) log V) |
| 적합한 그래프 | 희소 그래프 (E 작음) | 밀집 그래프 (E 큼) |
| 구현 난이도 | 쉬움 | 보통 |

## 6. 이걸 떠올려야 할 때
- "모든 노드를 최소 비용으로 연결" → MST
- 간선 수가 많으면 → 프림
- 다익스트라 구조 익숙하면 → 프림이 편함

## 7. 자주 틀리는 포인트
- 다익스트라와 구조가 비슷하지만 dist 배열 없이 visited만 사용
- 양방향 그래프 입력 빠뜨리는 경우
- visited 체크 안 하면 같은 노드 중복 처리
- C++ `priority_queue`는 기본이 최대 힙 → 최소 힙은 `greater<>` 필수
