# String Hashing (문자열 해싱)

> 한 줄 정리: 문자열을 숫자로 변환해 부분 문자열 비교를 O(1)에 처리한다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<string>`, `<utility>` (`pair`)

## 1. 언제 쓰는가
- 두 부분 문자열이 같은지 O(1)에 비교
- 긴 문자열에서 패턴 탐색 (라빈-카프)
- 팰린드롬 판별
- 문자열 집합에서 중복 탐색

## 2. 핵심 아이디어
- 문자열 → 해시값(정수)으로 변환
- 같은 문자열 → 같은 해시값
- 누적 해시로 부분 문자열 해시를 O(1)에 계산
- 충돌 방지를 위해 큰 소수(MOD)와 BASE 사용

## 3. 시간/공간 복잡도
| | 복잡도 |
|--|--------|
| 전처리 | O(N) |
| 부분 문자열 해시 | O(1) |
| 공간 | O(N) |

## 4. 기본 코드

### 단일 해시

#### Python
```python
MOD  = 1_000_000_007
BASE = 31  # 소문자면 31, 대소문자 포함이면 131

def build_hash(s):
    n = len(s)
    h = [0] * (n + 1)
    p = [1] * (n + 1)  # BASE의 거듭제곱

    for i in range(n):
        h[i+1] = (h[i] * BASE + ord(s[i])) % MOD
        p[i+1] = p[i] * BASE % MOD

    return h, p

def get_hash(h, p, l, r):
    # s[l:r+1]의 해시값 (0-indexed)
    return (h[r+1] - h[l] * p[r-l+1]) % MOD
```

#### C++
```cpp
const long long MOD  = 1'000'000'007;
const long long BASE = 31;   // 소문자면 31, 대소문자 포함이면 131

// h: 누적 해시, p: BASE 거듭제곱
void buildHash(const string& s, vector<long long>& h, vector<long long>& p) {
    int n = s.size();
    h.assign(n + 1, 0);
    p.assign(n + 1, 1);
    for (int i = 0; i < n; i++) {
        h[i+1] = (h[i] * BASE + s[i]) % MOD;
        p[i+1] = p[i] * BASE % MOD;
    }
}

// s[l..r]의 해시값 (0-indexed)
long long getHash(vector<long long>& h, vector<long long>& p, int l, int r) {
    return ((h[r+1] - h[l] * p[r-l+1]) % MOD + MOD) % MOD;   // 음수 방지
}
```

### 이중 해시 (충돌 방지)

#### Python
```python
MOD1, BASE1 = 1_000_000_007, 31
MOD2, BASE2 = 998_244_353,   37

def build_double_hash(s):
    n = len(s)
    h1 = [0] * (n + 1); p1 = [1] * (n + 1)
    h2 = [0] * (n + 1); p2 = [1] * (n + 1)

    for i in range(n):
        h1[i+1] = (h1[i] * BASE1 + ord(s[i])) % MOD1
        p1[i+1] = p1[i] * BASE1 % MOD1
        h2[i+1] = (h2[i] * BASE2 + ord(s[i])) % MOD2
        p2[i+1] = p2[i] * BASE2 % MOD2

    return h1, p1, h2, p2

def get_double_hash(h1, p1, h2, p2, l, r):
    hash1 = (h1[r+1] - h1[l] * p1[r-l+1]) % MOD1
    hash2 = (h2[r+1] - h2[l] * p2[r-l+1]) % MOD2
    return (hash1, hash2)
```

#### C++
```cpp
const long long MOD1 = 1'000'000'007, BASE1 = 31;
const long long MOD2 = 998'244'353,   BASE2 = 37;

void buildDoubleHash(const string& s,
                     vector<long long>& h1, vector<long long>& p1,
                     vector<long long>& h2, vector<long long>& p2) {
    int n = s.size();
    h1.assign(n+1, 0); p1.assign(n+1, 1);
    h2.assign(n+1, 0); p2.assign(n+1, 1);
    for (int i = 0; i < n; i++) {
        h1[i+1] = (h1[i] * BASE1 + s[i]) % MOD1;
        p1[i+1] = p1[i] * BASE1 % MOD1;
        h2[i+1] = (h2[i] * BASE2 + s[i]) % MOD2;
        p2[i+1] = p2[i] * BASE2 % MOD2;
    }
}

pair<long long,long long> getDoubleHash(vector<long long>& h1, vector<long long>& p1,
                                        vector<long long>& h2, vector<long long>& p2,
                                        int l, int r) {
    long long hash1 = ((h1[r+1] - h1[l] * p1[r-l+1]) % MOD1 + MOD1) % MOD1;
    long long hash2 = ((h2[r+1] - h2[l] * p2[r-l+1]) % MOD2 + MOD2) % MOD2;
    return {hash1, hash2};
}
```

### 라빈-카프 (패턴 탐색)

#### Python
```python
def rabin_karp(text, pattern):
    N, M = len(text), len(pattern)
    h_text, p_text = build_hash(text)
    h_pat,  p_pat  = build_hash(pattern)

    target = get_hash(h_pat, p_pat, 0, M - 1)
    result = []

    for i in range(N - M + 1):
        if get_hash(h_text, p_text, i, i + M - 1) == target:
            result.append(i)

    return result
```

#### C++
```cpp
vector<int> rabinKarp(const string& text, const string& pattern) {
    int N = text.size(), M = pattern.size();
    if (M > N) return {};
    vector<long long> ht, pt, hp, pp;
    buildHash(text, ht, pt);
    buildHash(pattern, hp, pp);

    long long target = getHash(hp, pp, 0, M - 1);
    vector<int> result;
    for (int i = 0; i + M - 1 < N; i++)
        if (getHash(ht, pt, i, i + M - 1) == target)
            result.push_back(i);
    return result;
}
```

### 팰린드롬 판별

#### Python
```python
def is_palindrome(s, l, r, h, p, hr, pr):
    # h, p: 정방향 해시 / hr, pr: 역방향 해시
    forward  = get_hash(h,  p,  l, r)
    backward = get_hash(hr, pr, len(s)-1-r, len(s)-1-l)
    return forward == backward

# 역방향 해시 빌드
def build_reverse_hash(s):
    return build_hash(s[::-1])
```

#### C++
```cpp
// h,p: 정방향 해시 / hr,pr: 역방향 해시 (모두 미리 빌드)
bool isPalindrome(int n, int l, int r,
                  vector<long long>& h, vector<long long>& p,
                  vector<long long>& hr, vector<long long>& pr) {
    long long forward  = getHash(h,  p,  l, r);
    long long backward = getHash(hr, pr, n-1-r, n-1-l);
    return forward == backward;
}
// 역방향 해시: string rev(s.rbegin(), s.rend()); buildHash(rev, hr, pr);
```

## 5. 이걸 떠올려야 할 때
- "부분 문자열 비교를 O(1)에" → 문자열 해싱
- "긴 문자열에서 패턴 탐색" → 라빈-카프 (KMP 대안)
- "팰린드롬 판별" → 정방향 + 역방향 해시 비교
- 충돌이 걱정되면 → 이중 해시 사용

## 6. 자주 틀리는 포인트
- 해시값이 음수 나올 수 있음 → `% MOD` 후 `(+ MOD) % MOD` 처리 (특히 C++)
- 단일 해시는 충돌 가능 → 중요한 문제엔 이중 해시
- BASE를 너무 작게 잡으면 충돌 증가
- `p` 배열 크기를 N+1로 잡아야 인덱스 오류 없음
- C++은 `h[i] * BASE`가 `long long` 범위 안에 들도록 매 단계 `% MOD`
