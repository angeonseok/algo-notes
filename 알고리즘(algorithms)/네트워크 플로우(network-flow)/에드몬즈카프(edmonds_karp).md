# Edmonds-Karp (에드몬즈-카프)

> 한 줄 정리: 포드-풀커슨에서 BFS로 증가 경로를 찾아 O(VE²)를 보장하는 알고리즘

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<queue>`, `<algorithm>` (`min`), `<climits>`

## 1. 언제 쓰는가
- 일반적인 최대 유량 문제
- 포드-풀커슨보다 안정적인 시간복잡도가 필요할 때
- 디닉보다 구현이 간단할 때

## 2. 핵심 아이디어
- 포드-풀커슨과 동일하지만 DFS 대신 BFS로 최단 증가 경로 탐색
- BFS → 항상 최단 경로 선택 → O(VE²) 보장
- 역방향 간선으로 흐름 취소 가능

## 3. 시간복잡도
| | 복잡도 |
|--|--------|
| 시간 | O(VE²) |
| 포드-풀커슨 대비 | F에 무관하게 보장 |

## 4. 기본 코드

#### Python
```python
from collections import deque, defaultdict

#BFS로 증가경로 찾아 흘리기를 더 못 흘릴 때까지 반복
def edmonds_karp(graph, cap, s, t, V):
    #소스 > 싱크 최단 증가경로 하나 찾아오기
    def bfs():
        #visited가 부모 기록도 겸함. s 부모는 None
        visited = {s: None}
        queue = deque([s])
        while queue:
            v = queue.popleft()

            #싱크 찍었으면 부모 타고 거꾸로 경로 복원
            if v == t:
                path = []
                while v is not None:
                    path.append(v)
                    v = visited[v]
                return list(reversed(path))

            #용량 남은 간선만 타고 진행
            for u in graph[v]:
                if u not in visited and cap[v][u] > 0:
                    visited[u] = v
                    queue.append(u)
        return None

    total = 0
    while True:
        path = bfs()

        #더 이상 경로 없으면 끝
        if not path:
            break

        #경로에서 제일 좁은 목이 흘릴 수 있는 최대치
        flow = min(cap[path[i]][path[i+1]] for i in range(len(path)-1))

        #흘린 만큼 정방향 깎고 역방향 채우기(나중에 취소 가능하게)
        for i in range(len(path) - 1):
            cap[path[i]][path[i+1]] -= flow
            cap[path[i+1]][path[i]] += flow

        total += flow

    return total
```

#### C++
```cpp
vector<vector<int>> graph;

//cap[u][v]는 남은 용량
vector<vector<int>> cap;

//BFS로 증가경로 찾아 흘리기를 더 못 흘릴 때까지 반복
int edmondsKarp(int s, int t, int V) {
    int total = 0;
    while (true) {
        //parent가 방문 여부 겸 경로 복원용. s는 자기 자신을 부모로
        vector<int> parent(V, -1);
        parent[s] = s;
        queue<int> q;
        q.push(s);

        //용량 남은 간선만 타고 싱크 찾기
        while (!q.empty() && parent[t] == -1) {
            int v = q.front(); q.pop();
            for (int u : graph[v])
                if (parent[u] == -1 && cap[v][u] > 0) {
                    parent[u] = v;
                    q.push(u);
                }
        }

        //싱크 못 찍었으면 더 이상 경로 없으니 끝
        if (parent[t] == -1) break;

        //싱크에서 부모 거꾸로 타면서 제일 좁은 목 찾기
        int flow = INT_MAX;
        for (int v = t; v != s; v = parent[v])
            flow = min(flow, cap[parent[v]][v]);

        //흘린 만큼 정방향 깎고 역방향 채우기(취소용)
        for (int v = t; v != s; v = parent[v]) {
            cap[parent[v]][v] -= flow;
            cap[v][parent[v]] += flow;
        }
        total += flow;
    }
    return total;
}

//입력: graph에 u→v, v→u 둘 다 추가(중복 없이), cap[u][v] += c
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
- 일반적인 최대 유량 문제 → 에드몬즈-카프
- 포드-풀커슨이 시간초과 → 에드몬즈-카프로 대체
- 그래프가 매우 크면 → 디닉 고려

## 7. 자주 틀리는 포인트
- 역방향 간선 추가 빠뜨리는 경우
- 중복 간선 처리 → cap 합산, graph는 중복 없이
- BFS 경로 복원 시 부모 배열 방향(자식 → 부모) 헷갈리는 경우
