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

#왼쪽 애들 하나씩 밀어넣으면서 매칭 최대로 늘리기
def bipartite_matching(graph, left_size, right_size):
    #match_left[i]는 i랑 짝지어진 오른쪽 놈, match_right[j]는 그 반대
    match_left  = [-1] * (left_size + 1)
    match_right = [-1] * (right_size + 1)

    #v가 들어갈 자리 찾아보고 성공하면 True
    def dfs(v, visited):
        for u in graph[v]:
            #이번 턴에 이미 찔러본 놈이면 스킵
            if visited[u]:
                continue
            visited[u] = True

            #u가 비었거나, u 주인이 다른 데로 비켜줄 수 있으면 뺏자
            if match_right[u] == -1 or dfs(match_right[u], visited):
                match_left[v] = u
                match_right[u] = v
                return True
        return False

    result = 0
    for v in range(1, left_size + 1):
        #visited는 매 시도마다 새로. 안 그러면 경로를 못 찾음
        visited = [False] * (right_size + 1)

        if dfs(v, visited):
            result += 1

    return result, match_left, match_right
```

#### C++
```cpp
//왼쪽 정점에서 뻗어나가는 오른쪽 정점들
vector<vector<int>> graph;
vector<int> match_left, match_right;
vector<bool> visited;

//v가 들어갈 자리 찾아보고 성공하면 true
bool dfs(int v) {
    for (int u : graph[v]) {
        //이번 턴에 이미 찔러본 놈이면 스킵
        if (visited[u]) continue;
        visited[u] = true;

        //u가 비었거나, u 주인이 다른 데로 비켜줄 수 있으면 뺏자
        if (match_right[u] == -1 || dfs(match_right[u])) {
            match_left[v] = u;
            match_right[u] = v;
            return true;
        }
    }
    return false;
}

//왼쪽 애들 하나씩 밀어넣으면서 매칭 최대로 늘리기
int bipartiteMatching(int left_size, int right_size) {
    match_left.assign(left_size + 1, -1);
    match_right.assign(right_size + 1, -1);

    int result = 0;
    for (int v = 1; v <= left_size; v++) {
        //visited는 매 시도마다 새로. 안 그러면 경로를 못 찾음
        visited.assign(right_size + 1, false);

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

#두 가지 색으로 칠해지면 이분 그래프
def is_bipartite(graph, V):
    color = [-1] * (V + 1)

    #끊어진 덩어리가 있을 수 있으니 모든 정점에서 시작해보기
    for start in range(1, V + 1):
        #이미 칠해진 놈이면 스킵
        if color[start] != -1:
            continue
        color[start] = 0
        queue = deque([start])

        while queue:
            v = queue.popleft()
            for i in graph[v]:
                #안 칠해진 이웃은 나랑 반대색으로
                if color[i] == -1:
                    color[i] = 1 - color[v]
                    queue.append(i)

                #이웃이 나랑 같은 색이면 두 색으로는 안 갈라짐
                elif color[i] == color[v]:
                    return False

    return True
```

#### C++
```cpp
//두 가지 색으로 칠해지면 이분 그래프
bool isBipartite(vector<vector<int>>& graph, int V) {
    vector<int> color(V + 1, -1);

    //끊어진 덩어리가 있을 수 있으니 모든 정점에서 시작해보기
    for (int start = 1; start <= V; start++) {
        //이미 칠해진 놈이면 스킵
        if (color[start] != -1) continue;
        color[start] = 0;
        queue<int> q;
        q.push(start);
        while (!q.empty()) {
            int v = q.front(); q.pop();
            for (int i : graph[v]) {
                //안 칠해진 이웃은 나랑 반대색으로
                if (color[i] == -1) {
                    color[i] = 1 - color[v];
                    q.push(i);
                } else if (color[i] == color[v]) {
                    //이웃이 나랑 같은 색이면 두 색으로는 안 갈라짐
                    return false;
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
