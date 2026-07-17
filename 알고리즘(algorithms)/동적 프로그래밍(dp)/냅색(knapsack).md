# Knapsack (냅색)

> 한 줄 정리: 무게 제한이 있는 배낭에 가치 합을 최대화하는 물건을 담는 문제

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<algorithm>` (`max`)

## 1. 언제 쓰는가
- 용량 제한 안에서 최대 가치를 구해야 할 때
- "N개 중 일부를 선택해서 조건을 만족하는 최적값"
- 0/1 냅색: 각 물건을 한 번만 선택
- 완전 냅색: 각 물건을 여러 번 선택 가능

## 2. 핵심 아이디어
- dp[i][w] = i번째 물건까지 고려했을 때 무게 w 이하로 담을 수 있는 최대 가치
- 각 물건마다 "담는다 / 안 담는다" 두 가지 선택
- 1차원 배열로 최적화 가능 (공간 O(W))

## 3. 시간/공간 복잡도
| | 복잡도 |
|--|--------|
| 시간 | O(N × W) |
| 공간 | O(N × W) → 1차원 최적화 시 O(W) |

## 4. 기본 코드

### 0/1 냅색 (2차원)

#### Python
```python
#0/1 냅색. 물건 하나씩 보면서 담을지 말지만 정하면 됨
def knapsack(N, W, w, v):
    #dp[i][j] = i번째까지 봤을 때 무게 j 이하로 담는 최대 가치
    dp = [[0] * (W + 1) for _ in range(N + 1)]

    for i in range(1, N + 1):
        for j in range(W + 1):
            #일단 안 담고 이전 줄 결과 그대로 들고오기
            dp[i][j] = dp[i-1][j]

            #무게 한도 안 걸리는 놈이면 담아본 결과랑 비교해서 큰 쪽
            if w[i-1] <= j:
                dp[i][j] = max(dp[i][j], dp[i-1][j - w[i-1]] + v[i-1])

    return dp[N][W]
```

#### C++
```cpp
//0/1 냅색. 물건 하나씩 보면서 담을지 말지만 정하면 됨
int knapsack(int N, int W, vector<int>& w, vector<int>& v) {
    //dp[i][j] = i번째까지 봤을 때 무게 j 이하로 담는 최대 가치
    vector<vector<int>> dp(N + 1, vector<int>(W + 1, 0));

    for (int i = 1; i <= N; i++)
        for (int j = 0; j <= W; j++) {
            //일단 안 담고 이전 줄 결과 그대로 들고오기
            dp[i][j] = dp[i-1][j];

            //무게 한도 안 걸리는 놈이면 담아본 결과랑 비교해서 큰 쪽
            if (w[i-1] <= j)
                dp[i][j] = max(dp[i][j], dp[i-1][j - w[i-1]] + v[i-1]);
        }
    return dp[N][W];
}
```

### 0/1 냅색 (1차원 최적화)

#### Python
```python
#0/1 냅색 1차원. 어차피 이전 줄만 쓰니까 한 줄로 우려먹기
def knapsack(N, W, w, v):
    #우리는 W값에 따라 결과가 달라진다
    dp = [0] * (W + 1)

    for i in range(N):
        #역순으로 가야 이번 물건이 두 번 담기지 않음
        for j in range(W, w[i] - 1, -1):
            #딱 맞게 넣거나, 이전 결과 + 무게 한도 안걸리는 놈 추가
            dp[j] = max(dp[j], dp[j - w[i]] + v[i])

    return dp[W]
```

#### C++
```cpp
//0/1 냅색 1차원. 어차피 이전 줄만 쓰니까 한 줄로 우려먹기
int knapsack(int N, int W, vector<int>& w, vector<int>& v) {
    //우리는 W값에 따라 결과가 달라진다
    vector<int> dp(W + 1, 0);

    for (int i = 0; i < N; i++)
        //역순으로 가야 이번 물건이 두 번 담기지 않음
        //딱 맞게 넣거나, 이전 결과 + 무게 한도 안걸리는 놈 추가
        for (int j = W; j >= w[i]; j--)
            dp[j] = max(dp[j], dp[j - w[i]] + v[i]);

    return dp[W];
}
```

### 완전 냅색 (물건 중복 선택 가능)

#### Python
```python
#완전 냅색. 같은 물건 몇 번이든 담아도 되는 버전
def unbounded_knapsack(N, W, w, v):
    dp = [0] * (W + 1)

    #정순이라 dp[j - w[i]]에 이번 물건이 이미 들어있을 수 있음. 그게 노림수
    for j in range(1, W + 1):
        for i in range(N):
            if w[i] <= j:
                dp[j] = max(dp[j], dp[j - w[i]] + v[i])

    return dp[W]
```

#### C++
```cpp
//완전 냅색. 같은 물건 몇 번이든 담아도 되는 버전
int unboundedKnapsack(int N, int W, vector<int>& w, vector<int>& v) {
    vector<int> dp(W + 1, 0);

    //정순이라 dp[j - w[i]]에 이번 물건이 이미 들어있을 수 있음. 그게 노림수
    for (int j = 1; j <= W; j++)
        for (int i = 0; i < N; i++)
            if (w[i] <= j)
                dp[j] = max(dp[j], dp[j - w[i]] + v[i]);

    return dp[W];
}
```

## 5. 0/1 냅색 vs 완전 냅색 차이

#### Python
```python
#0/1 냅색: 역순 순회 (이미 선택한 물건 재선택 방지)
for j in range(W, w - 1, -1):
    dp[j] = max(dp[j], dp[j - w] + v)

#완전 냅색: 정순 순회 (같은 물건 여러 번 선택 허용)
for j in range(w, W + 1):
    dp[j] = max(dp[j], dp[j - w] + v)
```

#### C++
```cpp
//0/1 냅색: 역순 순회 (이미 선택한 물건 재선택 방지)
for (int j = W; j >= w; j--)
    dp[j] = max(dp[j], dp[j - w] + v);

//완전 냅색: 정순 순회 (같은 물건 여러 번 선택 허용)
for (int j = w; j <= W; j++)
    dp[j] = max(dp[j], dp[j - w] + v);
```

## 6. 이걸 떠올려야 할 때
- "N개 중 선택, 합이 W 이하, 최대/최소" → 냅색
- "동전으로 금액 만들기" → 완전 냅색 (동전 중복 사용 가능)
- "부분집합 합" → 0/1 냅색 변형

## 7. 자주 틀리는 포인트
- 0/1 냅색에서 정순 순회하면 같은 물건이 중복 선택됨 → 반드시 역순
- 완전 냅색에서 역순 순회하면 중복 선택이 안 됨 → 반드시 정순
- dp 배열 크기를 W로 하면 인덱스 초과 → `W+1`로 선언
- 무게/가치 인덱스 헷갈리지 않게 변수명 명확히
