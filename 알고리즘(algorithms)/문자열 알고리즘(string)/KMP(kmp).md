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
#패턴의 접두사이면서 접미사인 최장 길이를 미리 구해두자
def get_pi(p):
    m = len(p)
    pi = [0] * m
    j = 0

    #pi[0]은 무조건 0이니까 1부터 시작
    for i in range(1, m):
        #안 맞으면 처음부터 다시 말고 이전 실패 함수로 점프
        while j > 0 and p[i] != p[j]:
            j = pi[j - 1]

        #맞으면 길이 늘려서 박기
        if p[i] == p[j]:
            j += 1
            pi[i] = j

    return pi
```

#### C++
```cpp
//패턴의 접두사이면서 접미사인 최장 길이를 미리 구해두자
vector<int> getPi(const string& p) {
    int m = p.size();
    vector<int> pi(m, 0);
    int j = 0;

    //pi[0]은 무조건 0이니까 1부터 시작
    for (int i = 1; i < m; i++) {
        //안 맞으면 처음부터 다시 말고 이전 실패 함수로 점프
        while (j > 0 && p[i] != p[j])
            j = pi[j - 1];

        //맞으면 길이 늘려서 박기
        if (p[i] == p[j])
            pi[i] = ++j;
    }
    return pi;
}
```

### KMP 탐색

#### Python
```python
#s 안에서 p 나오는 시작 위치 전부 모으기
def kmp(s, p):
    n, m = len(s), len(p)
    pi = get_pi(p)
    result = []
    j = 0

    for i in range(n):
        #안 맞아도 i는 안 되돌림. j만 실패 함수로 점프
        while j > 0 and s[i] != p[j]:
            j = pi[j - 1]

        #맞으면 한 칸 전진
        if s[i] == p[j]:
            j += 1

        #j가 m까지 갔으면 패턴 하나 완성
        if j == m:
            #찾은 놈의 시작 인덱스 박기
            result.append(i - m + 1)

            #겹치는 매칭도 잡아야 하니까 실패 함수로 되돌리기
            j = pi[j - 1]

    return result

#써먹기 > [0, 3, 6] 나옴
print(kmp("aabaabaab", "aab"))
```

#### C++
```cpp
//s 안에서 p 나오는 시작 위치 전부 모으기
vector<int> kmp(const string& s, const string& p) {
    int n = s.size(), m = p.size();
    vector<int> pi = getPi(p);
    vector<int> result;
    int j = 0;

    for (int i = 0; i < n; i++) {
        //안 맞아도 i는 안 되돌림. j만 실패 함수로 점프
        while (j > 0 && s[i] != p[j])
            j = pi[j - 1];

        //맞으면 한 칸 전진
        if (s[i] == p[j])
            j++;

        //j가 m까지 갔으면 패턴 하나 완성
        if (j == m) {
            //찾은 놈의 시작 인덱스 박기
            result.push_back(i - m + 1);

            //겹치는 매칭도 잡아야 하니까 실패 함수로 되돌리기
            j = pi[j - 1];
        }
    }
    return result;
}

//kmp("aabaabaab", "aab") → {0, 3, 6}
```

## 5. 실전 패턴

### 패턴 등장 횟수만 필요할 때

#### Python
```python
#위치는 필요없고 몇 번 나오는지만 셀 때
def kmp_count(s, p):
    pi = get_pi(p)
    cnt = 0
    j = 0

    #인덱스 안 쓰니까 걍 문자만 돌리자
    for c in s:
        #안 맞으면 실패 함수로 점프
        while j > 0 and c != p[j]:
            j = pi[j - 1]

        if c == p[j]:
            j += 1

        #하나 찾았으면 세고 바로 되돌리기
        if j == len(p):
            cnt += 1
            j = pi[j - 1]

    return cnt
```

#### C++
```cpp
//위치는 필요없고 몇 번 나오는지만 셀 때
int kmpCount(const string& s, const string& p) {
    vector<int> pi = getPi(p);
    int m = p.size(), cnt = 0, j = 0;

    //인덱스 안 쓰니까 걍 문자만 돌리자
    for (char c : s) {
        //안 맞으면 실패 함수로 점프
        while (j > 0 && c != p[j])
            j = pi[j - 1];

        if (c == p[j])
            j++;

        //하나 찾았으면 세고 바로 되돌리기
        if (j == m) {
            cnt++;
            j = pi[j - 1];
        }
    }
    return cnt;
}
```

### 문자열 주기 찾기

#### Python
```python
#s가 어떤 조각의 반복인지, 그 조각 길이 구하기
def string_period(s):
    pi = get_pi(s)
    n = len(s)

    #전체 길이 - 최장 접두접미 = 한 주기 길이
    period = n - pi[n - 1]

    #딱 나눠떨어져야 진짜 반복. s는 길이 period짜리의 반복
    if n % period == 0:
        return period

    #안 나눠떨어지면 주기 없음
    return n
```

#### C++
```cpp
//s가 어떤 조각의 반복인지, 그 조각 길이 구하기
int stringPeriod(const string& s) {
    vector<int> pi = getPi(s);
    int n = s.size();

    //전체 길이 - 최장 접두접미 = 한 주기 길이
    int period = n - pi[n - 1];

    //딱 나눠떨어져야 진짜 반복. 길이 period짜리의 반복
    if (n % period == 0) return period;

    //안 나눠떨어지면 주기 없음
    return n;
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
