# Kruskal (크루스칼)

> 한 줄 정리: 가중치가 작은 간선부터 선택해서 MST를 만든다, 유니온 파인드로 사이클 감지

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<algorithm>` (`sort`)

## 1. 언제 쓰는가
- 최소 신장 트리(MST) 구할 때
- 간선 수가 적은 희소 그래프
- 모든 노드를 최소 비용으로 연결

## 2. 핵심 아이디어
- 모든 간선을 가중치 기준 오름차순 정렬
- 사이클이 생기지 않는 간선만 선택 (유니온 파인드)
- V-1개의 간선을 선택하면 완료
- 간선 중심 알고리즘

## 3. 시간/공간 복잡도
| | 복잡도 |
|--|--------|
| 시간 | O(E log E) |
| 공간 | O(V) |

## 4. 기본 코드

#### Python
```python
#싼 간선부터 집어먹으면서 MST 만들기
def kruskal(V, edges):
    #싼 간선부터 봐야 하니까 가중치 기준 정렬
    edges.sort(key=lambda x: x[2])

    #자기 자신을 부모로 시작해보자
    parent = list(range(V + 1))

    #x의 루트를 경로를 압축하면서 찾자
    def find(x):
        if parent[x] != x:
            parent[x] = find(parent[x])
        return parent[x]

    #루트가 같으면 이미 연결된 놈들이라 사이클. 다르면 통합하고 True
    def union(x, y):
        rootA, rootB = find(x), find(y)
        if rootA == rootB:
            return False
        parent[rootB] = rootA
        return True

    total = 0
    cnt = 0

    #사이클 안 나는 간선만 골라 담기
    for u, v, w in edges:
        if union(u, v):
            total += w
            cnt += 1

            #V-1개 채웠으면 트리 완성이니 그만
            if cnt == V - 1:
                break

    #V-1개를 못 채웠으면 끊긴 그래프니까 -1
    return total if cnt == V - 1 else -1

#입력 예시
V, E = map(int, input().split())
edges = []
for _ in range(E):
    u, v, w = map(int, input().split())
    edges.append((u, v, w))
```

#### C++
```cpp
vector<int> parent;

//x의 루트를 경로를 압축하면서 찾자
int find(int x) {
    if (parent[x] != x) parent[x] = find(parent[x]);
    return parent[x];
}

//루트가 같으면 이미 연결된 놈들이라 사이클. 다르면 통합하고 true
//union은 키워드 느낌이라 unite
bool unite(int x, int y) {
    int rootA = find(x), rootB = find(y);
    if (rootA == rootB) return false;
    parent[rootB] = rootA;
    return true;
}

struct Edge { int u, v, w; };

//싼 간선부터 집어먹으면서 MST 만들기
long long kruskal(int V, vector<Edge>& edges) {
    //싼 간선부터 봐야 하니까 가중치 기준 오름차순 정렬
    sort(edges.begin(), edges.end(),
         [](const Edge& a, const Edge& b){ return a.w < b.w; });

    //자기 자신을 부모로 시작해보자
    parent.resize(V + 1);
    for (int i = 0; i <= V; i++) parent[i] = i;

    long long total = 0;
    int cnt = 0;

    //사이클 안 나는 간선만 골라 담기
    for (auto& e : edges) {
        if (unite(e.u, e.v)) {
            total += e.w;
            cnt++;

            //V-1개 채웠으면 트리 완성이니 그만
            if (cnt == V - 1) break;
        }
    }

    //V-1개를 못 채웠으면 끊긴 그래프니까 -1
    return (cnt == V - 1) ? total : -1;
}

//입력 예시
// int V, E; cin >> V >> E;
// vector<Edge> edges(E);
// for (int i = 0; i < E; i++) cin >> edges[i].u >> edges[i].v >> edges[i].w;
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
- 간선 수가 적으면 → 크루스칼
- 간선 수가 많으면 → 프림

## 7. 자주 틀리는 포인트
- 연결되지 않는 그래프에서 MST 불가 → count == V-1 확인
- 간선 정렬 기준을 가중치로 해야 함
- 유니온 파인드에서 경로 압축 빠뜨리면 느려짐
- C++은 가중치 합이 커질 수 있어 `total`을 `long long`으로
