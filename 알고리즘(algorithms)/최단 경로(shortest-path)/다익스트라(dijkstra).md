# Dijkstra (다익스트라)

> 한 줄 정리: 가중치가 양수인 그래프에서 한 정점에서 모든 정점까지의 최단 거리를 구한다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<queue>`, `<utility>`, `<climits>`, `<algorithm>`, `<functional>` (`greater`)

## 1. 언제 쓰는가
- 가중치가 있는 그래프에서 최단 경로
- 가중치가 모두 양수일 때
- 단일 출발점 → 모든 정점 최단 거리

## 2. 핵심 아이디어
- 방문하지 않은 정점 중 거리가 가장 짧은 것부터 처리
- 힙으로 구현하면 O((V + E) log V)
- 음수 가중치 있으면 사용 불가 → 벨만포드 사용

## 3. 시간/공간 복잡도
| 구현 | 시간 |
|------|------|
| 힙 사용 | O((V + E) log V) |
| 힙 미사용 | O(V²) |

## 4. 기본 코드

### 힙 기반 다익스트라

#### Python
```python
import heapq
import sys

#시작점 > 모든 정점까지 최단거리 구하기
def dijkstra(graph, start, V):
    INF = sys.maxsize
    dist = [INF] * (V + 1)
    dist[start] = 0

    #(거리, 정점) 형태로 힙에 담기
    pq = [(0, start)]

    #다익스트라는 힙을 활용
    while pq:
        d, u = heapq.heappop(pq)

        #dist가 최신 정보. d가 더 크면 구버전이니 스킵
        if d > dist[u]:
            continue

        #거리정보 갱신
        for v, w in graph[u]:
            nd = d + w

            #갱신 가능하면 힙에 다시 넣자
            if nd < dist[v]:
                dist[v] = nd
                heapq.heappush(pq, (nd, v))

    return dist
```

#### C++
```cpp
vector<long long> dijkstra(vector<vector<pair<int,int>>>& graph, int start, int V) {
    const long long INF = LLONG_MAX;
    vector<long long> dist(V + 1, INF);
    dist[start] = 0;

    //(거리, 정점) 최소 힙
    priority_queue<pair<long long,int>, vector<pair<long long,int>>, greater<>> pq;
    pq.push({0, start});

    while (!pq.empty()) {
        long long d = pq.top().first;
        int u = pq.top().second;
        pq.pop();

        //이미 처리된 노드면 스킵
        if (d > dist[u]) continue;

        //거리정보 갱신
        for (auto& e : graph[u]) {
            int v = e.first, w = e.second;
            long long nd = d + w;

            //갱신 가능하면 힙에 다시 넣자
            if (nd < dist[v]) {
                dist[v] = nd;
                pq.push({nd, v});
            }
        }
    }
    return dist;
}
```

입력 예시:
```cpp
int V, E;
cin >> V >> E;
vector<vector<pair<int,int>>> graph(V + 1);
for (int i = 0; i < E; i++) {
    int u, v, w;
    cin >> u >> v >> w;

    //방향 그래프니까 한쪽만
    graph[u].push_back({v, w});
}
```

### 경로 복원

#### Python
```python
#최단거리뿐만 아니라 실제로 어디를 거쳐갔는지까지 뽑기
def dijkstra_path(graph, start, end, V):
    INF = sys.maxsize
    dist = [INF] * (V + 1)

    #prev에 직전 노드를 남겨둬야 나중에 거꾸로 되짚음
    prev = [-1] * (V + 1)
    dist[start] = 0

    #(거리, 정점) 형태로 힙에 담기
    pq = [(0, start)]

    #다익스트라는 힙을 활용
    while pq:
        d, u = heapq.heappop(pq)

        #dist가 최신 정보. d가 더 크면 구버전이니 스킵
        if d > dist[u]:
            continue

        #거리정보 갱신
        for v, w in graph[u]:
            nd = d + w

            #갱신 가능하면 힙에 다시 넣고, 누가 갱신했는지도 남기자
            if nd < dist[v]:
                dist[v] = nd
                prev[v] = u
                heapq.heappush(pq, (nd, v))

    #end에서 prev 타고 start까지 거꾸로 올라가기
    path = []
    cur = end
    while cur != -1:
        path.append(cur)
        cur = prev[cur]
    return dist[end], list(reversed(path))
```

#### C++
```cpp
//최단거리뿐만 아니라 실제로 어디를 거쳐갔는지까지 뽑기
pair<long long, vector<int>> dijkstraPath(vector<vector<pair<int,int>>>& graph,
                                          int start, int end, int V) {
    const long long INF = LLONG_MAX;
    vector<long long> dist(V + 1, INF);

    //prev에 직전 노드를 남겨둬야 나중에 거꾸로 되짚음
    vector<int> prev(V + 1, -1);
    dist[start] = 0;

    //(거리, 정점) 최소 힙
    priority_queue<pair<long long,int>, vector<pair<long long,int>>, greater<>> pq;
    pq.push({0, start});

    while (!pq.empty()) {
        long long d = pq.top().first;
        int u = pq.top().second;
        pq.pop();

        //이미 처리된 노드면 스킵
        if (d > dist[u]) continue;

        //거리정보 갱신
        for (auto& e : graph[u]) {
            int v = e.first, w = e.second;
            long long nd = d + w;

            //갱신 가능하면 힙에 다시 넣고, 누가 갱신했는지도 남기자
            if (nd < dist[v]) {
                dist[v] = nd;
                prev[v] = u;
                pq.push({nd, v});
            }
        }
    }

    //end에서 prev 타고 start까지 거꾸로 올라가기
    vector<int> path;
    for (int cur = end; cur != -1; cur = prev[cur])
        path.push_back(cur);
    reverse(path.begin(), path.end());
    return {dist[end], path};
}
```

## 5. 이걸 떠올려야 할 때
- "가중치 있는 최단 경로" + "음수 없음" → 다익스트라
- "특정 노드까지 최단 거리" → 다익스트라 후 dist[target]
- 가중치가 모두 1이면 → BFS로 충분

## 6. 자주 틀리는 포인트
- `if cost > dist[node]: continue` 빠뜨리면 시간초과
- 음수 가중치 있을 때 다익스트라 쓰면 오답
- 노드 번호가 1-indexed인지 확인 → dist 크기 V+1로
- 양방향 그래프면 양쪽 모두 추가
- C++은 거리 합이 커질 수 있으니 `long long`, INF는 `LLONG_MAX`

## 7. 세 알고리즘 비교
| | 다익스트라 | 벨만포드 | 플로이드 워셜 |
|--|-----------|---------|-------------|
| 출발점 | 단일 | 단일 | 모든 쌍 |
| 음수 가중치 | 불가 | 가능 | 가능 |
| 음수 사이클 감지 | 불가 | 가능 | 가능 |
| 시간복잡도 | O((V+E) log V) | O(VE) | O(V³) |
| 적합한 V 크기 | 대규모 | 중간 | V ≤ 500 |
