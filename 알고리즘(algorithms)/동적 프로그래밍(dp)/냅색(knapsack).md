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
def knapsack(N, W, weights, values):
    dp = [[0] * (W + 1) for _ in range(N + 1)]

    for i in range(1, N + 1):
        for w in range(W + 1):
            # 안 담는 경우
            dp[i][w] = dp[i-1][w]
            # 담는 경우 (무게가 허용될 때)
            if weights[i-1] <= w:
                dp[i][w] = max(dp[i][w], dp[i-1][w - weights[i-1]] + values[i-1])

    return dp[N][W]
```

#### C++
```cpp
int knapsack(int N, int W, vector<int>& weights, vector<int>& values) {
    vector<vector<int>> dp(N + 1, vector<int>(W + 1, 0));

    for (int i = 1; i <= N; i++)
        for (int w = 0; w <= W; w++) {
            dp[i][w] = dp[i-1][w];                 // 안 담는 경우
            if (weights[i-1] <= w)                 // 담는 경우
                dp[i][w] = max(dp[i][w], dp[i-1][w - weights[i-1]] + values[i-1]);
        }
    return dp[N][W];
}
```

### 0/1 냅색 (1차원 최적화)

#### Python
```python
def knapsack(N, W, weights, values):
    dp = [0] * (W + 1)

    for i in range(N):
        for w in range(W, weights[i] - 1, -1):  # 역순으로 순회 (중복 선택 방지)
            dp[w] = max(dp[w], dp[w - weights[i]] + values[i])

    return dp[W]
```

#### C++
```cpp
int knapsack(int N, int W, vector<int>& weights, vector<int>& values) {
    vector<int> dp(W + 1, 0);

    for (int i = 0; i < N; i++)
        for (int w = W; w >= weights[i]; w--)   // 역순으로 순회 (중복 선택 방지)
            dp[w] = max(dp[w], dp[w - weights[i]] + values[i]);

    return dp[W];
}
```

### 완전 냅색 (물건 중복 선택 가능)

#### Python
```python
def unbounded_knapsack(N, W, weights, values):
    dp = [0] * (W + 1)

    for w in range(1, W + 1):
        for i in range(N):
            if weights[i] <= w:
                dp[w] = max(dp[w], dp[w - weights[i]] + values[i])  # 정순으로 순회

    return dp[W]
```

#### C++
```cpp
int unboundedKnapsack(int N, int W, vector<int>& weights, vector<int>& values) {
    vector<int> dp(W + 1, 0);

    for (int w = 1; w <= W; w++)
        for (int i = 0; i < N; i++)
            if (weights[i] <= w)
                dp[w] = max(dp[w], dp[w - weights[i]] + values[i]);   // 정순으로 순회

    return dp[W];
}
```

## 5. 0/1 냅색 vs 완전 냅색 차이

#### Python
```python
# 0/1 냅색: 역순 순회 (이미 선택한 물건 재선택 방지)
for w in range(W, weight - 1, -1):
    dp[w] = max(dp[w], dp[w - weight] + value)

# 완전 냅색: 정순 순회 (같은 물건 여러 번 선택 허용)
for w in range(weight, W + 1):
    dp[w] = max(dp[w], dp[w - weight] + value)
```

#### C++
```cpp
// 0/1 냅색: 역순 순회 (이미 선택한 물건 재선택 방지)
for (int w = W; w >= weight; w--)
    dp[w] = max(dp[w], dp[w - weight] + value);

// 완전 냅색: 정순 순회 (같은 물건 여러 번 선택 허용)
for (int w = weight; w <= W; w++)
    dp[w] = max(dp[w], dp[w - weight] + value);
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
