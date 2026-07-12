# LCS (Longest Common Subsequence, 최장 공통 부분 수열)

> 한 줄 정리: 두 문자열에서 공통으로 등장하는 가장 긴 부분 수열의 길이

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<string>`, `<algorithm>` (`max`, `reverse`)

## 1. 언제 쓰는가
- 두 문자열의 공통 부분을 구해야 할 때
- 편집 거리(Edit Distance)와 연관된 문제
- DNA 서열 비교, diff 알고리즘

## 2. 핵심 아이디어
- dp[i][j] = s1[0:i]와 s2[0:j]의 LCS 길이
- s1[i] == s2[j]이면 dp[i][j] = dp[i-1][j-1] + 1
- 다르면 dp[i][j] = max(dp[i-1][j], dp[i][j-1])

## 3. 시간/공간 복잡도
| | 복잡도 |
|--|--------|
| 시간 | O(N × M) |
| 공간 | O(N × M) |

## 4. 기본 코드

### LCS 길이

#### Python
```python
def lcs(s1, s2):
    N, M = len(s1), len(s2)
    dp = [[0] * (M + 1) for _ in range(N + 1)]

    for i in range(1, N + 1):
        for j in range(1, M + 1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])

    return dp[N][M]
```

#### C++
```cpp
int lcs(const string& s1, const string& s2) {
    int N = s1.size(), M = s2.size();
    vector<vector<int>> dp(N + 1, vector<int>(M + 1, 0));

    for (int i = 1; i <= N; i++)
        for (int j = 1; j <= M; j++) {
            if (s1[i-1] == s2[j-1])
                dp[i][j] = dp[i-1][j-1] + 1;
            else
                dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
        }
    return dp[N][M];
}
```

### LCS 실제 수열 복원

#### Python
```python
def lcs_sequence(s1, s2):
    N, M = len(s1), len(s2)
    dp = [[0] * (M + 1) for _ in range(N + 1)]

    for i in range(1, N + 1):
        for j in range(1, M + 1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])

    # 역추적
    result = []
    i, j = N, M
    while i > 0 and j > 0:
        if s1[i-1] == s2[j-1]:
            result.append(s1[i-1])
            i -= 1
            j -= 1
        elif dp[i-1][j] > dp[i][j-1]:
            i -= 1
        else:
            j -= 1

    return ''.join(reversed(result))
```

#### C++
```cpp
string lcsSequence(const string& s1, const string& s2) {
    int N = s1.size(), M = s2.size();
    vector<vector<int>> dp(N + 1, vector<int>(M + 1, 0));

    for (int i = 1; i <= N; i++)
        for (int j = 1; j <= M; j++) {
            if (s1[i-1] == s2[j-1])
                dp[i][j] = dp[i-1][j-1] + 1;
            else
                dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
        }

    // 역추적
    string result;
    int i = N, j = M;
    while (i > 0 && j > 0) {
        if (s1[i-1] == s2[j-1]) {
            result += s1[i-1];
            i--; j--;
        } else if (dp[i-1][j] > dp[i][j-1]) {
            i--;
        } else {
            j--;
        }
    }
    reverse(result.begin(), result.end());
    return result;
}
```

## 5. 이걸 떠올려야 할 때
- "두 문자열의 공통 부분" → LCS
- "삭제/삽입 최소 횟수" → LCS 응용 (편집 거리)
- 수열에도 적용 가능 (문자열뿐 아니라 숫자 배열도)

## 6. 자주 틀리는 포인트
- dp 배열 인덱스가 1-based → `s1[i-1]`, `s2[j-1]`로 접근
- 부분 문자열(Substring)과 부분 수열(Subsequence) 혼동 → LCS는 연속 안 해도 됨
- 실제 수열 복원할 때 역추적 방향 헷갈리는 경우
- C++에서 문자열 뒤집기 잊지 말기 (`reverse`) — 역추적은 뒤에서부터 쌓임
