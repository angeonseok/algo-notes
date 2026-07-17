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
#두 문자열의 최장 공통 부분수열 길이
def lcs(str1, str2):
    #dp[i][j] = str1 앞 i글자, str2 앞 j글자의 LCS 길이. 0행 0열은 패딩
    N, M = len(str1), len(str2)
    dp = [[0] * (M + 1) for _ in range(N + 1)]

    for i in range(1, N + 1):
        for j in range(1, M + 1):
            #글자 같으면 대각선에서 하나 이어붙이기
            if str1[i-1] == str2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1

            #다르면 한 글자 버린 두 경우 중 더 긴 쪽
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])

    return dp[N][M]
```

#### C++
```cpp
//두 문자열의 최장 공통 부분수열 길이
int lcs(const string& str1, const string& str2) {
    //dp[i][j] = str1 앞 i글자, str2 앞 j글자의 LCS 길이. 0행 0열은 패딩
    int N = str1.size(), M = str2.size();
    vector<vector<int>> dp(N + 1, vector<int>(M + 1, 0));

    for (int i = 1; i <= N; i++)
        for (int j = 1; j <= M; j++) {
            //글자 같으면 대각선에서 하나 이어붙이기
            if (str1[i-1] == str2[j-1])
                dp[i][j] = dp[i-1][j-1] + 1;

            //다르면 한 글자 버린 두 경우 중 더 긴 쪽
            else
                dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
        }
    return dp[N][M];
}
```

### LCS 실제 수열 복원

#### Python
```python
#길이만 말고 실제 공통 수열까지 뽑기
def lcs_sequence(str1, str2):
    #일단 평소처럼 dp 테이블 다 채워두기
    N, M = len(str1), len(str2)
    dp = [[0] * (M + 1) for _ in range(N + 1)]

    for i in range(1, N + 1):
        for j in range(1, M + 1):
            if str1[i-1] == str2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])

    #오른쪽 끝에서 거꾸로 따라 올라가며 글자 주워담기
    result = []
    i, j = N, M
    while i > 0 and j > 0:
        #글자 같았던 칸이면 그놈이 LCS 멤버. 대각선으로 이동
        if str1[i-1] == str2[j-1]:
            result.append(str1[i-1])
            i -= 1
            j -= 1

        #아니면 값 물려준 쪽으로 되돌아가기
        elif dp[i-1][j] > dp[i][j-1]:
            i -= 1
        else:
            j -= 1

    #뒤에서부터 담았으니 뒤집어야 순서 맞음
    return ''.join(reversed(result))
```

#### C++
```cpp
//길이만 말고 실제 공통 수열까지 뽑기
string lcsSequence(const string& str1, const string& str2) {
    //일단 평소처럼 dp 테이블 다 채워두기
    int N = str1.size(), M = str2.size();
    vector<vector<int>> dp(N + 1, vector<int>(M + 1, 0));

    for (int i = 1; i <= N; i++)
        for (int j = 1; j <= M; j++) {
            if (str1[i-1] == str2[j-1])
                dp[i][j] = dp[i-1][j-1] + 1;
            else
                dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
        }

    //오른쪽 끝에서 거꾸로 따라 올라가며 글자 주워담기
    string result;
    int i = N, j = M;
    while (i > 0 && j > 0) {
        //글자 같았던 칸이면 그놈이 LCS 멤버. 대각선으로 이동
        if (str1[i-1] == str2[j-1]) {
            result += str1[i-1];
            i--; j--;

        //아니면 값 물려준 쪽으로 되돌아가기
        } else if (dp[i-1][j] > dp[i][j-1]) {
            i--;
        } else {
            j--;
        }
    }

    //뒤에서부터 담았으니 뒤집어야 순서 맞음
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
