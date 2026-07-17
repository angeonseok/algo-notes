# Bitmask (비트마스킹)

> 한 줄 정리: 정수의 비트를 이용해 집합/상태를 표현하고 연산하는 방법

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<algorithm>` (`min`), `<climits>`

## 1. 언제 쓰는가
- 부분집합을 표현할 때
- 방문 상태를 정수 하나로 압축할 때
- DP with 비트마스크 (외판원 순회 등)
- 집합 연산을 빠르게 처리할 때

## 2. 핵심 아이디어
- N개의 원소 → N비트 정수로 상태 표현
- 각 비트가 해당 원소의 포함 여부를 나타냄
- 전체 부분집합 수: 2^N
- 비트 연산이 O(1)이라 상태 처리가 빠름

## 3. 비트 연산 정리
| 연산 | 코드 | 의미 |
|------|------|------|
| i번 비트 켜기 | `mask \| (1 << i)` | i번 원소 추가 |
| i번 비트 끄기 | `mask & ~(1 << i)` | i번 원소 제거 |
| i번 비트 확인 | `mask & (1 << i)` | i번 원소 포함 여부 |
| i번 비트 토글 | `mask ^ (1 << i)` | 있으면 제거, 없으면 추가 |
| 전체 비트 켜기 | `(1 << N) - 1` | 모든 원소 포함 |
| 비트 수 세기 | Python `bin(mask).count('1')` / C++ `__builtin_popcount(mask)` | 포함된 원소 수 |

## 4. 기본 코드

### 부분집합 순회

#### Python
```python
n = 3
arr = ['a', 'b', 'c']

#0 ~ 2^n - 1 돌리면 부분집합 전부 나옴
for mask in range(1 << n):
    subset = []

    #켜져 있는 비트 자리의 원소만 주워담기
    for i in range(n):
        if mask & (1 << i):
            subset.append(arr[i])

    print(subset)
```

#### C++
```cpp
int n = 3;
vector<char> arr = {'a', 'b', 'c'};

//0 ~ 2^n - 1 돌리면 부분집합 전부 나옴
for (int mask = 0; mask < (1 << n); mask++) {
    vector<char> subset;

    //켜져 있는 비트 자리의 원소만 주워담기
    for (int i = 0; i < n; i++)
        if (mask & (1 << i))
            subset.push_back(arr[i]);

    //subset 사용
}
```

### 비트 연산 예시

#### Python
```python
#아무것도 없는 상태에서 시작
mask = 0b0000

#OR로 켜기 > 0001, 0101
mask |= (1 << 0)
mask |= (1 << 2)

#AND 걸어서 0 아니면 들어있는 놈
if mask & (1 << 0):
    print("0번 포함")

#반전시켜서 AND = 그 자리만 끄기 > 0001
mask &= ~(1 << 2)

#켜진 비트 세기 > 1
print(bin(mask).count('1'))
```

#### C++
```cpp
//아무것도 없는 상태에서 시작
int mask = 0b0000;

//OR로 켜기 > 0001, 0101
mask |= (1 << 0);
mask |= (1 << 2);

//AND 걸어서 0 아니면 들어있는 놈
if (mask & (1 << 0)) { /* 0번 포함 */ }

//반전시켜서 AND = 그 자리만 끄기 > 0001
mask &= ~(1 << 2);

//켜진 비트 세기 > 1
int cnt = __builtin_popcount(mask);
```

## 5. 실전 패턴

### DP + 비트마스크 (외판원 순회 TSP)

#### Python
```python
import sys

#모든 도시 한 번씩 돌고 시작점으로 돌아오는 최소 비용
def tsp(graph, n):
    INF = sys.maxsize

    #dp[mask][i] = mask 상태에서 i번 도시에 있을 때 최소 비용
    dp = [[INF] * n for _ in range(1 << n)]

    #0번 도시에서 시작하니까 0번만 켜진 mask = 1
    dp[1][0] = 0

    #방문 상태를 작은 놈부터 채워나가자
    for mask in range(1 << n):
        for v in range(n):
            #아직 도달 못한 상태니까 스킵
            if dp[mask][v] == INF:
                continue

            #v에 서있으려면 mask에 v가 켜져 있어야 말이 됨
            if not (mask & (1 << v)):
                continue

            #v에서 갈 수 있는 다음 도시 찾기
            for next_v in range(n):
                #이미 방문한 놈이니까 스킵
                if mask & (1 << next_v):
                    continue

                #간선 없으면 못 가니까 스킵
                if graph[v][next_v] == 0:
                    continue

                #next_v 켜서 다음 상태로 넘기기
                new_mask = mask | (1 << next_v)
                dp[new_mask][next_v] = min(
                    dp[new_mask][next_v],
                    dp[mask][v] + graph[v][next_v]
                )

    #전부 켜진 상태에서 0번으로 돌아오는 비용까지 더한 게 답
    full = (1 << n) - 1
    return min(dp[full][i] + graph[i][0] for i in range(n) if graph[i][0])
```

#### C++
```cpp
//모든 도시 한 번씩 돌고 시작점으로 돌아오는 최소 비용
int tsp(vector<vector<int>>& graph, int n) {
    const int INF = INT_MAX;

    //dp[mask][i] = mask 상태에서 i번 도시에 있을 때 최소 비용
    vector<vector<int>> dp(1 << n, vector<int>(n, INF));

    //0번 도시에서 시작하니까 0번만 켜진 mask = 1
    dp[1][0] = 0;

    //방문 상태를 작은 놈부터 채워나가자
    for (int mask = 0; mask < (1 << n); mask++)
        for (int v = 0; v < n; v++) {
            //아직 도달 못한 상태니까 스킵
            if (dp[mask][v] == INF) continue;

            //v에 서있으려면 mask에 v가 켜져 있어야 말이 됨
            if (!(mask & (1 << v))) continue;

            //v에서 갈 수 있는 다음 도시 찾기
            for (int next_v = 0; next_v < n; next_v++) {
                //이미 방문한 놈이니까 스킵
                if (mask & (1 << next_v)) continue;

                //간선 없으면 못 가니까 스킵
                if (graph[v][next_v] == 0) continue;

                //next_v 켜서 다음 상태로 넘기기
                int new_mask = mask | (1 << next_v);
                dp[new_mask][next_v] = min(dp[new_mask][next_v],
                                           dp[mask][v] + graph[v][next_v]);
            }
        }

    //전부 켜진 상태에서 0번으로 돌아오는 비용까지 더한 게 답
    int full = (1 << n) - 1, ans = INF;
    for (int i = 0; i < n; i++)
        if (graph[i][0])
            ans = min(ans, dp[full][i] + graph[i][0]);
    return ans;
}
```

### 집합 연산

#### Python
```python
#A = {1, 3}, B = {2, 3}
A = 0b1010
B = 0b1100

#합집합 > 0b1110 = {1, 2, 3}
A | B

#교집합 > 0b1000 = {3}
A & B

#대칭차 > 0b0110 = {1, 2}
A ^ B

#차집합 > 0b0010 = {1}
A & ~B
```

#### C++
```cpp
//A = {1, 3}, B = {2, 3}
int A = 0b1010;
int B = 0b1100;

//합집합 > 0b1110
A | B;

//교집합 > 0b1000
A & B;

//대칭차 > 0b0110
A ^ B;

//차집합 > 0b0010
A & ~B;
```

## 6. 이걸 떠올려야 할 때
- "N ≤ 20인 부분집합 탐색" → 비트마스크
- "방문 상태를 정수로 표현" → 비트마스크 DP
- "외판원 순회(TSP)" → 비트마스크 DP 대표 문제
- "집합 연산을 빠르게" → 비트 연산

## 7. 자주 틀리는 포인트
- `1 << N`이 전체 경우의 수, 인덱스는 `0 ~ (1<<N) - 1`
- Python `~`는 부호 있는 무한 정밀도라 `& ~(1<<i)` 시 주의 → C++은 고정 폭이라 그대로 안전
- N이 크면 (N > 20) 메모리/시간 초과 → 비트마스크 DP는 N ≤ 20 이하에서만
- C++ `1 << N`은 N ≥ 31이면 int 오버플로 → `1LL << N` 사용
