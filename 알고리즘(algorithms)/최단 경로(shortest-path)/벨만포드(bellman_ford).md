# Bellman-Ford (벨만포드)

> 한 줄 정리: 음수 가중치가 있어도 최단 거리를 구할 수 있고, 음수 사이클 감지도 가능하다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<climits>`, `<iostream>`

## 1. 언제 쓰는가
- 음수 가중치가 있는 그래프에서 최단 경로
- 음수 사이클 존재 여부 확인
- 단일 출발점 → 모든 정점 최단 거리

## 2. 핵심 아이디어
- 모든 간선을 V-1번 반복해서 완화(relax)
- V번째에도 완화가 일어나면 → 음수 사이클 존재
- 다익스트라보다 느리지만 음수 가중치 처리 가능

## 3. 시간/공간 복잡도
| | 복잡도 |
|--|--------|
| 시간 | O(V × E) |
| 공간 | O(V) |

## 4. 기본 코드

### 벨만포드 기본

#### Python
```python
import sys

#음수 가중치 있어도 최단거리 구하기
def bellman_ford(edges, start, V):
    INF = sys.maxsize
    dist = [INF] * (V + 1)
    dist[start] = 0

    #최단경로는 간선 V-1개를 넘을 수 없으니까 V-1번만 돌리자
    for _ in range(V - 1):
        #모든 간선 훑으면서 갱신 가능하면 갱신
        for u, v, w in edges:
            if dist[u] != INF and dist[u] + w < dist[v]:
                dist[v] = dist[u] + w

    #V번째에도 또 줄어들면 음수 사이클 있는 거니까 None 박기
    for u, v, w in edges:
        if dist[u] != INF and dist[u] + w < dist[v]:
            return None

    return dist
```

#### C++
```cpp
struct Edge { int u, v, w; };

//음수 가중치 있어도 최단거리 구하기. 음수 사이클이면 빈 벡터 반환
vector<long long> bellmanFord(vector<Edge>& edges, int start, int V) {
    const long long INF = LLONG_MAX;
    vector<long long> dist(V + 1, INF);
    dist[start] = 0;

    //최단경로는 간선 V-1개를 넘을 수 없으니까 V-1번만 돌리자
    for (int iter = 0; iter < V - 1; iter++)
        for (auto& e : edges)
            if (dist[e.u] != INF && dist[e.u] + e.w < dist[e.v])
                dist[e.v] = dist[e.u] + e.w;

    //V번째에도 또 줄어들면 음수 사이클 있는 거니까 빈 벡터 박기
    for (auto& e : edges)
        if (dist[e.u] != INF && dist[e.u] + e.w < dist[e.v])
            return {};

    return dist;
}
```

입력 예시:
```cpp
int V, E;
cin >> V >> E;
vector<Edge> edges(E);
for (int i = 0; i < E; i++)
    cin >> edges[i].u >> edges[i].v >> edges[i].w;
```

## 5. 다익스트라 vs 벨만포드
| | 다익스트라 | 벨만포드 |
|--|-----------|---------|
| 음수 가중치 | 불가 | 가능 |
| 음수 사이클 감지 | 불가 | 가능 |
| 시간복잡도 | O((V+E) log V) | O(V × E) |
| 적합한 상황 | 양수 가중치 | 음수 가중치 포함 |

## 6. 이걸 떠올려야 할 때
- "음수 가중치 있는 최단 경로" → 벨만포드
- "음수 사이클 존재 여부" → 벨만포드
- 양수만 있으면 다익스트라가 훨씬 빠름

## 7. 자주 틀리는 포인트
- 반복을 V번 하면 안 됨 → V-1번 완화 후 V번째에 감지
- `dist[u] != INF` 체크 안 하면 INF + w 오버플로우 가능 (C++은 `long long`이라도 주의)
- 음수 사이클이 있을 때 dist가 -INF로 수렴하는 노드 처리

## 8. 세 알고리즘 비교
| | 다익스트라 | 벨만포드 | 플로이드 워셜 |
|--|-----------|---------|-------------|
| 출발점 | 단일 | 단일 | 모든 쌍 |
| 음수 가중치 | 불가 | 가능 | 가능 |
| 음수 사이클 감지 | 불가 | 가능 | 가능 |
| 시간복잡도 | O((V+E) log V) | O(VE) | O(V³) |
| 적합한 V 크기 | 대규모 | 중간 | V ≤ 500 |
