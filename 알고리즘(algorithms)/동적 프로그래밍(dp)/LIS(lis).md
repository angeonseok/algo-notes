# LIS (Longest Increasing Subsequence, 최장 증가 부분 수열)

> 한 줄 정리: 수열에서 값이 증가하는 가장 긴 부분 수열의 길이

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<algorithm>` (`max`, `max_element`, `lower_bound`, `reverse`)

## 1. 언제 쓰는가
- 수열에서 증가하는 가장 긴 부분 수열을 구할 때
- O(N²) DP로 기본 구현, O(N log N)으로 최적화 가능
- 인내심 게임, 박스 쌓기 등 변형 문제

## 2. 핵심 아이디어
- dp[i] = i번째 원소로 끝나는 LIS의 길이
- 모든 j < i에 대해 arr[j] < arr[i]이면 dp[i] = max(dp[i], dp[j] + 1)
- O(N log N): 이진 탐색으로 LIS 배열을 유지

## 3. 시간/공간 복잡도
| 방법 | 시간 | 공간 |
|------|------|------|
| DP | O(N²) | O(N) |
| DP + 이진 탐색 | O(N log N) | O(N) |

## 4. 기본 코드

### O(N²) DP

#### Python
```python
def lis(arr):
    N = len(arr)
    dp = [1] * N

    for i in range(1, N):
        for j in range(i):
            if arr[j] < arr[i]:
                dp[i] = max(dp[i], dp[j] + 1)

    return max(dp)
```

#### C++
```cpp
int lis(vector<int>& arr) {
    int N = arr.size();
    vector<int> dp(N, 1);

    for (int i = 1; i < N; i++)
        for (int j = 0; j < i; j++)
            if (arr[j] < arr[i])
                dp[i] = max(dp[i], dp[j] + 1);

    return *max_element(dp.begin(), dp.end());
}
```

### O(N log N) 이진 탐색

#### Python
```python
import bisect

def lis(arr):
    tails = []  # tails[i] = 길이 i+1인 LIS의 마지막 원소 중 최솟값

    for v in arr:
        idx = bisect.bisect_left(tails, v)
        if idx == len(tails):
            tails.append(v)
        else:
            tails[idx] = v  # 더 작은 값으로 교체 (실제 수열은 아님)

    return len(tails)
```

#### C++
```cpp
int lis(vector<int>& arr) {
    vector<int> tails;   // tails[i] = 길이 i+1 LIS의 마지막 원소 중 최솟값
    for (int v : arr) {
        auto it = lower_bound(tails.begin(), tails.end(), v);
        if (it == tails.end()) tails.push_back(v);
        else *it = v;    // 더 작은 값으로 교체 (실제 수열은 아님)
    }
    return tails.size();
}
```
> 순증가(>)가 아니라 비감소(≥) LIS가 필요하면 `bisect_right` / `upper_bound`로 바꾼다.

### LIS 실제 수열 복원 (O(N log N))

#### Python
```python
import bisect

def lis_sequence(arr):
    N = len(arr)
    tails = []
    indices = []   # 각 원소가 삽입된 tails 인덱스
    predecessors = [-1] * N  # 이전 원소 인덱스

    for i, v in enumerate(arr):
        idx = bisect.bisect_left(tails, v)
        if idx == len(tails):
            tails.append(v)
        else:
            tails[idx] = v
        indices.append(idx)
        if idx > 0:
            for j in range(i - 1, -1, -1):
                if indices[j] == idx - 1 and arr[j] < v:
                    predecessors[i] = j
                    break

    # 역추적
    result = []
    idx = len(tails) - 1
    for i in range(N - 1, -1, -1):
        if indices[i] == idx:
            result.append(arr[i])
            idx -= 1
            if idx < 0:
                break

    return list(reversed(result))
```

#### C++
```cpp
// tails에 대응하는 인덱스와 parent 링크로 O(N log N)에 복원
vector<int> lisSequence(vector<int>& arr) {
    int N = arr.size();
    vector<int> tails;      // 값
    vector<int> tailIdx;    // tails에 대응하는 arr 인덱스
    vector<int> parent(N, -1);

    for (int i = 0; i < N; i++) {
        int idx = lower_bound(tails.begin(), tails.end(), arr[i]) - tails.begin();
        if (idx == (int)tails.size()) {
            tails.push_back(arr[i]);
            tailIdx.push_back(i);
        } else {
            tails[idx] = arr[i];
            tailIdx[idx] = i;
        }
        parent[i] = (idx > 0) ? tailIdx[idx - 1] : -1;   // 삽입 시점의 직전 길이 끝 원소
    }

    // 역추적: 마지막 길이의 끝 인덱스부터 parent를 따라간다
    vector<int> result;
    for (int cur = tailIdx[tails.size() - 1]; cur != -1; cur = parent[cur])
        result.push_back(arr[cur]);
    reverse(result.begin(), result.end());
    return result;
}
```

## 5. 이걸 떠올려야 할 때
- "증가하는 부분 수열의 최대 길이" → LIS
- N ≤ 1000이면 O(N²) DP로 충분
- N > 10000이면 O(N log N) 필요
- "박스 쌓기", "인내심 게임" 유형 → LIS 변형

## 6. 자주 틀리는 포인트
- O(N log N)의 tails 배열은 실제 LIS가 아님 → 수열 복원 시 parent 링크 별도 관리
- 비감소(≥) vs 순증가(>) 조건 혼동 → `bisect_left`/`lower_bound` vs `bisect_right`/`upper_bound`
- dp 초기값을 0이 아닌 1로 설정 (자기 자신 포함)
