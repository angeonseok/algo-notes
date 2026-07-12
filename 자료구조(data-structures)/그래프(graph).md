# Graph (그래프)

> 한 줄 정리: 노드(정점)와 엣지(간선)로 이루어진 자료구조, 표현 방식 선택이 성능을 가른다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<iostream>`, `<utility>` (`pair`)

## 1. 언제 쓰는가
- 노드 간의 관계를 표현해야 할 때
- 경로 탐색, 연결 여부 확인
- 지도, 네트워크, 소셜 관계 등 모델링

## 2. 핵심 아이디어
- 그래프 = 정점(V) + 간선(E)
- 표현 방식은 크게 두 가지: 인접 행렬 vs 인접 리스트
- 어떤 방식을 쓰느냐에 따라 시간/공간 복잡도가 달라짐

## 3. 시간복잡도
| 연산 | 인접 행렬 | 인접 리스트 |
|------|-----------|-------------|
| 간선 존재 확인 | O(1) | O(degree) |
| 전체 간선 순회 | O(V²) | O(V + E) |
| 공간 | O(V²) | O(V + E) |

## 4. 기본 코드

### 인접 행렬

#### Python
```python
# V개의 정점, 0-indexed
V = 5
graph = [[0] * V for _ in range(V)]

# 간선 추가 (양방향)
graph[0][1] = 1
graph[1][0] = 1

# 가중치 있는 경우
graph[0][1] = 3
graph[1][0] = 3

# 간선 존재 확인
if graph[u][v]:
    print("연결됨")
```

#### C++
```cpp
// V개의 정점, 0-indexed
int V = 5;
vector<vector<int>> graph(V, vector<int>(V, 0));

// 간선 추가 (양방향)
graph[0][1] = 1;
graph[1][0] = 1;

// 가중치 있는 경우
graph[0][1] = 3;
graph[1][0] = 3;

// 간선 존재 확인
if (graph[u][v]) {
    // 연결됨
}
```

### 인접 리스트

#### Python
```python
from collections import defaultdict

# 방법 1: defaultdict
graph = defaultdict(list)
graph[0].append(1)
graph[1].append(0)

# 방법 2: 직접 초기화
V = 5
graph = [[] for _ in range(V)]
graph[0].append(1)
graph[1].append(0)

# 가중치 있는 경우 (튜플로)
graph[0].append((1, 3))  # (이웃 노드, 가중치)
graph[1].append((0, 3))
```

#### C++
```cpp
// 크기 V로 초기화한 인접 리스트
int V = 5;
vector<vector<int>> graph(V);
graph[0].push_back(1);
graph[1].push_back(0);

// 가중치 있는 경우 (pair로: 이웃 노드, 가중치)
vector<vector<pair<int,int>>> wgraph(V);
wgraph[0].push_back({1, 3});
wgraph[1].push_back({0, 3});
```

### 입력 받기 (코테 기준)

#### Python
```python
import sys
from collections import defaultdict
input = sys.stdin.readline

V, E = map(int, input().split())
graph = defaultdict(list)

for _ in range(E):
    u, v = map(int, input().split())
    graph[u].append(v)
    graph[v].append(u)  # 양방향이면 추가

# 가중치 있는 경우
for _ in range(E):
    u, v, w = map(int, input().split())
    graph[u].append((v, w))
    graph[v].append((u, w))
```

#### C++
```cpp
int V, E;
cin >> V >> E;
vector<vector<int>> graph(V + 1);   // 1-indexed 대비 +1

for (int i = 0; i < E; i++) {
    int u, v;
    cin >> u >> v;
    graph[u].push_back(v);
    graph[v].push_back(u);   // 양방향이면 추가
}

// 가중치 있는 경우
vector<vector<pair<int,int>>> wgraph(V + 1);
for (int i = 0; i < E; i++) {
    int u, v, w;
    cin >> u >> v >> w;
    wgraph[u].push_back({v, w});
    wgraph[v].push_back({u, w});
}
```

> 입력이 많으면 C++은 `ios_base::sync_with_stdio(false); cin.tie(NULL);` 로 `cin` 가속.

## 5. 인접 행렬 vs 인접 리스트 선택 기준
| 상황 | 선택 |
|------|------|
| 정점 수 적고 간선 많음 (밀집 그래프) | 인접 행렬 |
| 정점 수 많고 간선 적음 (희소 그래프) | 인접 리스트 |
| 두 노드의 연결 여부를 자주 확인 | 인접 행렬 |
| 특정 노드의 이웃을 자주 순회 | 인접 리스트 |
| 코테 일반적인 경우 | 인접 리스트 |

## 6. 이걸 떠올려야 할 때
- 정점 수 V가 1000 이하면 인접 행렬도 무방
- V가 10만 이상이면 반드시 인접 리스트
- 가중치 있으면 튜플/pair로 저장
- 방향 그래프면 단방향으로만 추가

## 7. 자주 틀리는 포인트
- 양방향 그래프인데 단방향으로만 추가하는 실수
- 인접 행렬을 큰 V에 쓰면 메모리 초과 (V=10만이면 10^10 칸)
- 노드 번호가 1-indexed인지 0-indexed인지 확인 안 하는 실수
- 가중치 그래프에서 `graph[u].append(v)` 대신 `(v, w)` / `{v, w}` 써야 함
