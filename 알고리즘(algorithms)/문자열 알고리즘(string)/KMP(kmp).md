# KMP (Knuth-Morris-Pratt)

> 한 줄 정리: 문자열에서 패턴을 O(N+M)에 찾는 알고리즘, 실패 함수로 불필요한 비교를 건너뛴다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<string>`

## 1. 언제 쓰는가
- 긴 문자열에서 특정 패턴이 등장하는 위치를 모두 찾을 때
- 단순 탐색 O(NM)이 시간초과날 때
- 패턴 매칭이 반복적으로 필요할 때

## 2. 핵심 아이디어
- 실패 함수(failure function): 패턴의 접두사이면서 접미사인 최장 길이
- 불일치 발생 시 처음부터 다시 비교하지 않고 실패 함수로 이동
- 전처리 O(M) + 탐색 O(N) = O(N+M)

## 3. 시간/공간 복잡도
| | 복잡도 |
|--|--------|
| 전처리 (실패 함수) | O(M) |
| 탐색 | O(N) |
| 전체 | O(N + M) |

## 4. 기본 코드

### 실패 함수 (failure function)

#### Python
```python
def get_failure(pattern):
    M = len(pattern)
    failure = [0] * M
    j = 0

    for i in range(1, M):
        while j > 0 and pattern[i] != pattern[j]:
            j = failure[j - 1]  # 이전 실패 함수로 이동
        if pattern[i] == pattern[j]:
            j += 1
            failure[i] = j

    return failure
```

#### C++
```cpp
vector<int> getFailure(const string& pattern) {
    int M = pattern.size();
    vector<int> failure(M, 0);
    int j = 0;

    for (int i = 1; i < M; i++) {
        while (j > 0 && pattern[i] != pattern[j])
            j = failure[j - 1];        // 이전 실패 함수로 이동
        if (pattern[i] == pattern[j])
            failure[i] = ++j;
    }
    return failure;
}
```

### KMP 탐색

#### Python
```python
def kmp(text, pattern):
    N, M = len(text), len(pattern)
    failure = get_failure(pattern)
    result = []
    j = 0

    for i in range(N):
        while j > 0 and text[i] != pattern[j]:
            j = failure[j - 1]  # 실패 함수로 이동
        if text[i] == pattern[j]:
            j += 1
        if j == M:  # 패턴 완전 매칭
            result.append(i - M + 1)  # 시작 인덱스
            j = failure[j - 1]        # 다음 탐색 위치

    return result

# 사용
print(kmp("aabaabaab", "aab"))  # [0, 3, 6]
```

#### C++
```cpp
vector<int> kmp(const string& text, const string& pattern) {
    int N = text.size(), M = pattern.size();
    vector<int> failure = getFailure(pattern);
    vector<int> result;
    int j = 0;

    for (int i = 0; i < N; i++) {
        while (j > 0 && text[i] != pattern[j])
            j = failure[j - 1];        // 실패 함수로 이동
        if (text[i] == pattern[j])
            j++;
        if (j == M) {                  // 패턴 완전 매칭
            result.push_back(i - M + 1);   // 시작 인덱스
            j = failure[j - 1];            // 다음 탐색 위치
        }
    }
    return result;
}
// kmp("aabaabaab", "aab") → {0, 3, 6}
```

## 5. 실전 패턴

### 패턴 등장 횟수만 필요할 때

#### Python
```python
def kmp_count(text, pattern):
    failure = get_failure(pattern)
    count = 0
    j = 0
    for c in text:
        while j > 0 and c != pattern[j]:
            j = failure[j - 1]
        if c == pattern[j]:
            j += 1
        if j == len(pattern):
            count += 1
            j = failure[j - 1]
    return count
```

#### C++
```cpp
int kmpCount(const string& text, const string& pattern) {
    vector<int> failure = getFailure(pattern);
    int M = pattern.size(), count = 0, j = 0;
    for (char c : text) {
        while (j > 0 && c != pattern[j])
            j = failure[j - 1];
        if (c == pattern[j])
            j++;
        if (j == M) {
            count++;
            j = failure[j - 1];
        }
    }
    return count;
}
```

### 문자열 주기 찾기

#### Python
```python
def string_period(s):
    failure = get_failure(s)
    n = len(s)
    period = n - failure[n - 1]
    if n % period == 0:
        return period  # s는 길이 period인 문자열의 반복
    return n           # 주기 없음
```

#### C++
```cpp
int stringPeriod(const string& s) {
    vector<int> failure = getFailure(s);
    int n = s.size();
    int period = n - failure[n - 1];
    if (n % period == 0) return period;   // 길이 period 문자열의 반복
    return n;                             // 주기 없음
}
```

## 6. 이걸 떠올려야 할 때
- "문자열에서 패턴 찾기" + N, M이 크면 → KMP
- "문자열의 주기" → 실패 함수 활용
- 단순 `in`(Python) / `string::find`(C++)은 최악 O(NM) → 큰 입력엔 KMP

## 7. 자주 틀리는 포인트
- 실패 함수에서 `i`는 1부터 시작 (`failure[0]`은 항상 0)
- 매칭 후 `j = failure[j-1]`로 이동해야 다음 탐색 가능 (빠뜨리면 중복 매칭 누락)
- 결과 인덱스가 0-indexed인지 1-indexed인지 문제에 따라 조정
