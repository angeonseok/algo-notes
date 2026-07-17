# DP 기초 (Dynamic Programming)

> 한 줄 정리: 큰 문제를 작은 문제로 나누되, 겹치는 부분 문제의 결과를 저장해서 중복 계산을 없앤다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<algorithm>` (`min`/`max`)

## 1. 언제 쓰는가
- 완전탐색으로 풀리는데 시간초과가 나는 경우
- 같은 부분 문제가 반복해서 등장할 때
- "최솟값/최댓값", "경우의 수", "가능 여부"를 구하는 문제
- 점화식으로 표현 가능한 문제

## 2. 핵심 아이디어
- 메모이제이션(Memoization): 재귀 + 결과 저장 (top-down)
- 타뷸레이션(Tabulation): 반복문으로 작은 것부터 채움 (bottom-up)
- 점화식이 핵심 → 점화식을 먼저 세우고 코드로 옮기기
- DP가 되려면: 최적 부분 구조 + 겹치는 부분 문제

## 3. Top-down vs Bottom-up
| | Top-down (메모이제이션) | Bottom-up (타뷸레이션) |
|--|------------------------|----------------------|
| 방식 | 재귀 + 캐시 | 반복문 |
| 장점 | 점화식 그대로 구현 | 스택오버플로 없음 |
| 단점 | 재귀 깊이 제한 | 순서 고려 필요 |
| 속도 | 약간 느림 | 약간 빠름 |

## 4. 기본 코드

### Top-down (재귀 + 메모이제이션)

#### Python
```python
from functools import lru_cache
import sys
sys.setrecursionlimit(100000)

#lru_cache가 알아서 결과 저장해주니까 메모 배열 따로 안 만들어도 됨
@lru_cache(maxsize=None)
def dp(n):
    #기저 조건. 여기서 안 끊으면 무한 재귀
    if n <= 1:
        return n

    #점화식 그대로 박기
    return dp(n - 1) + dp(n - 2)
```

#### C++
```cpp
//-1 = 아직 계산 안 한 칸. 실제 답이랑 안 겹치는 값으로 골라야 함
vector<long long> memo(100001, -1);

long long dp(int n) {
    //기저 조건. 여기서 안 끊으면 무한 재귀
    if (n <= 1) return n;

    //이미 구해놨으면 걍 꺼내 쓰기
    if (memo[n] != -1) return memo[n];

    //점화식 그대로 박고 저장까지 한 번에
    return memo[n] = dp(n - 1) + dp(n - 2);
}
```

### Bottom-up (반복문)

#### Python
```python
#작은 것부터 채워 올라가니까 재귀 깊이 걱정 없음
def dp(n):
    if n <= 1:
        return n

    #기저 조건 먼저 깔아두고 시작
    table = [0] * (n + 1)
    table[0] = 0
    table[1] = 1

    #점화식 그대로. 앞의 두 칸은 이미 채워져 있음
    for i in range(2, n + 1):
        table[i] = table[i - 1] + table[i - 2]

    return table[n]
```

#### C++
```cpp
//작은 것부터 채워 올라가니까 재귀 깊이 걱정 없음
long long dp(int n) {
    if (n <= 1) return n;

    //기저 조건 먼저 깔아두고 시작
    vector<long long> table(n + 1, 0);
    table[0] = 0;
    table[1] = 1;

    //점화식 그대로. 앞의 두 칸은 이미 채워져 있음
    for (int i = 2; i <= n; i++)
        table[i] = table[i-1] + table[i-2];

    return table[n];
}
```

### 1차원 DP (계단 오르기)

#### Python
```python
#1칸 또는 2칸씩 오를 수 있을 때 N번째 계단 오르는 경우의 수
def stair(n):
    if n <= 2:
        return n

    #dp[1]은 1칸 하나, dp[2]는 1+1이랑 2 두 가지
    dp = [0] * (n + 1)
    dp[1] = 1
    dp[2] = 2

    #마지막에 1칸 왔거나 2칸 왔거나 둘 중 하나니까 더하면 됨
    for i in range(3, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]

    return dp[n]
```

#### C++
```cpp
//1칸 또는 2칸씩 오를 수 있을 때 N번째 계단 오르는 경우의 수
long long stair(int n) {
    if (n <= 2) return n;

    //dp[1]은 1칸 하나, dp[2]는 1+1이랑 2 두 가지
    vector<long long> dp(n + 1, 0);
    dp[1] = 1;
    dp[2] = 2;

    //마지막에 1칸 왔거나 2칸 왔거나 둘 중 하나니까 더하면 됨
    for (int i = 3; i <= n; i++)
        dp[i] = dp[i-1] + dp[i-2];

    return dp[n];
}
```

### 2차원 DP (격자 최솟값 경로)

#### Python
```python
#왼쪽 또는 위에서만 이동 가능할 때 (0,0) → (N-1,M-1) 최솟값
def min_path(mat):
    N, M = len(mat), len(mat[0])
    dp = [[0] * M for _ in range(N)]
    dp[0][0] = mat[0][0]

    #첫 행/열은 올 수 있는 길이 하나뿐이라 걍 쭉 더하기
    for i in range(1, N):
        dp[i][0] = dp[i-1][0] + mat[i][0]
    for j in range(1, M):
        dp[0][j] = dp[0][j-1] + mat[0][j]

    #위에서 왔거나 왼쪽에서 왔거나. 싼 놈 골라서 현재 칸 값 더하기
    for i in range(1, N):
        for j in range(1, M):
            dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + mat[i][j]

    return dp[N-1][M-1]
```

#### C++
```cpp
//왼쪽 또는 위에서만 이동 가능할 때 (0,0) → (N-1,M-1) 최솟값
int minPath(vector<vector<int>>& mat) {
    int N = mat.size(), M = mat[0].size();
    vector<vector<int>> dp(N, vector<int>(M, 0));
    dp[0][0] = mat[0][0];

    //첫 행/열은 올 수 있는 길이 하나뿐이라 걍 쭉 더하기
    for (int i = 1; i < N; i++) dp[i][0] = dp[i-1][0] + mat[i][0];
    for (int j = 1; j < M; j++) dp[0][j] = dp[0][j-1] + mat[0][j];

    //위에서 왔거나 왼쪽에서 왔거나. 싼 놈 골라서 현재 칸 값 더하기
    for (int i = 1; i < N; i++)
        for (int j = 1; j < M; j++)
            dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + mat[i][j];

    return dp[N-1][M-1];
}
```

## 5. 점화식 세우는 법
```
1. dp[i]의 의미를 정확히 정의
   예: dp[i] = i번째 계단까지 오르는 경우의 수

2. 마지막 선택을 기준으로 경우를 나눔
   예: 마지막에 1칸 올랐다 → dp[i-1]
       마지막에 2칸 올랐다 → dp[i-2]

3. 점화식 완성
   dp[i] = dp[i-1] + dp[i-2]

4. 기저 조건 설정
   dp[1] = 1, dp[2] = 2
```

## 6. 이걸 떠올려야 할 때
- 완전탐색이 시간초과 → DP 고려
- "최소/최대/경우의 수" + 큰 N → DP
- 점화식이 보인다 → DP
- 그리디로 반례가 있다 → DP

## 7. 자주 틀리는 포인트
- dp 배열 크기를 N으로 하면 인덱스 초과 → `N+1`로 선언
- 기저 조건 빠뜨리거나 잘못 설정
- dp[i]의 의미를 명확히 안 정하면 점화식이 꼬임
- 2차원 DP에서 첫 행/열 초기화 빠뜨리는 경우
- Top-down에서 Python `lru_cache`는 리스트 인자 불가(tuple로) / C++은 memo 배열의 초기값(-1 등)이 실제 답과 겹치지 않게 주의
