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

#소문자면 31, 대소문자 섞이면 131
BASE = 31

#누적 해시 h랑 BASE 거듭제곱 p 미리 만들어두기
def build_hash(s):
    n = len(s)
    h = [0] * (n + 1)

    #p는 나중에 자릿수 맞춰 빼줄 때 씀
    p = [1] * (n + 1)

    #앞에서부터 자리 하나씩 밀면서 누적
    for i in range(n):
        h[i+1] = (h[i] * BASE + ord(s[i])) % MOD
        p[i+1] = p[i] * BASE % MOD

    return h, p

#s[l:r+1] 구간 해시를 O(1)로 뽑기 (0-indexed)
def get_hash(h, p, l, r):
    #앞부분 기여분을 자릿수 맞춰 빼주면 구간만 남음
    return (h[r+1] - h[l] * p[r-l+1]) % MOD
```

#### C++
```cpp
const long long MOD  = 1'000'000'007;

//소문자면 31, 대소문자 섞이면 131
const long long BASE = 31;

//h: 누적 해시, p: BASE 거듭제곱 미리 만들어두기
void buildHash(const string& s, vector<long long>& h, vector<long long>& p) {
    int n = s.size();
    h.assign(n + 1, 0);
    p.assign(n + 1, 1);

    //앞에서부터 자리 하나씩 밀면서 누적
    for (int i = 0; i < n; i++) {
        h[i+1] = (h[i] * BASE + s[i]) % MOD;
        p[i+1] = p[i] * BASE % MOD;
    }
}

//s[l..r] 구간 해시를 O(1)로 뽑기 (0-indexed)
long long getHash(vector<long long>& h, vector<long long>& p, int l, int r) {
    //빼다 보면 음수 나올 수 있으니 MOD 한 번 더 더해서 보정
    return ((h[r+1] - h[l] * p[r-l+1]) % MOD + MOD) % MOD;
}
```

### 이중 해시 (충돌 방지)

#### Python
```python
#MOD/BASE 쌍을 두 개 써서 우연히 겹칠 확률 확 낮추기
MOD1, BASE1 = 1_000_000_007, 31
MOD2, BASE2 = 998_244_353,   37

#해시 두 세트를 한 번에 만들기
def build_double_hash(s):
    n = len(s)
    h1 = [0] * (n + 1); p1 = [1] * (n + 1)
    h2 = [0] * (n + 1); p2 = [1] * (n + 1)

    #단일 해시 두 벌을 나란히 누적하는 것뿐
    for i in range(n):
        h1[i+1] = (h1[i] * BASE1 + ord(s[i])) % MOD1
        p1[i+1] = p1[i] * BASE1 % MOD1
        h2[i+1] = (h2[i] * BASE2 + ord(s[i])) % MOD2
        p2[i+1] = p2[i] * BASE2 % MOD2

    return h1, p1, h2, p2

#두 해시를 묶어서 반환. 둘 다 같아야 같은 문자열로 침
def get_double_hash(h1, p1, h2, p2, l, r):
    hash1 = (h1[r+1] - h1[l] * p1[r-l+1]) % MOD1
    hash2 = (h2[r+1] - h2[l] * p2[r-l+1]) % MOD2
    return (hash1, hash2)
```

#### C++
```cpp
//MOD/BASE 쌍을 두 개 써서 우연히 겹칠 확률 확 낮추기
const long long MOD1 = 1'000'000'007, BASE1 = 31;
const long long MOD2 = 998'244'353,   BASE2 = 37;

//해시 두 세트를 한 번에 만들기
void buildDoubleHash(const string& s,
                     vector<long long>& h1, vector<long long>& p1,
                     vector<long long>& h2, vector<long long>& p2) {
    int n = s.size();
    h1.assign(n+1, 0); p1.assign(n+1, 1);
    h2.assign(n+1, 0); p2.assign(n+1, 1);

    //단일 해시 두 벌을 나란히 누적하는 것뿐
    for (int i = 0; i < n; i++) {
        h1[i+1] = (h1[i] * BASE1 + s[i]) % MOD1;
        p1[i+1] = p1[i] * BASE1 % MOD1;
        h2[i+1] = (h2[i] * BASE2 + s[i]) % MOD2;
        p2[i+1] = p2[i] * BASE2 % MOD2;
    }
}

//두 해시를 묶어서 반환. 둘 다 같아야 같은 문자열로 침
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
#해시로 패턴 찾기. 문자 하나하나 비교 안 하고 숫자로 후려침
def rabin_karp(s, p):
    n, m = len(s), len(p)
    ht, pt = build_hash(s)
    hp, pp = build_hash(p)

    #찾을 패턴 해시 미리 뽑아두기
    target = get_hash(hp, pp, 0, m - 1)
    result = []

    #길이 m짜리 창을 밀면서 해시만 대조
    for i in range(n - m + 1):
        if get_hash(ht, pt, i, i + m - 1) == target:
            result.append(i)

    return result
```

#### C++
```cpp
//해시로 패턴 찾기. 문자 하나하나 비교 안 하고 숫자로 후려침
vector<int> rabinKarp(const string& s, const string& p) {
    int n = s.size(), m = p.size();

    //패턴이 더 길면 나올 수가 없음
    if (m > n) return {};

    vector<long long> ht, pt, hp, pp;
    buildHash(s, ht, pt);
    buildHash(p, hp, pp);

    //찾을 패턴 해시 미리 뽑아두기
    long long target = getHash(hp, pp, 0, m - 1);
    vector<int> result;

    //길이 m짜리 창을 밀면서 해시만 대조
    for (int i = 0; i + m - 1 < n; i++)
        if (getHash(ht, pt, i, i + m - 1) == target)
            result.push_back(i);
    return result;
}
```

### 팰린드롬 판별

#### Python
```python
#정방향 해시랑 뒤집은 해시가 같으면 팰린드롬
def is_palindrome(s, l, r, h, p, hr, pr):
    #h, p는 정방향 / hr, pr은 뒤집은 문자열 해시
    forward  = get_hash(h,  p,  l, r)

    #뒤집은 쪽에선 인덱스도 대칭으로 뒤집어서 봐야 함
    backward = get_hash(hr, pr, len(s)-1-r, len(s)-1-l)
    return forward == backward

#뒤집은 문자열로 그냥 똑같이 해시 빌드
def build_reverse_hash(s):
    return build_hash(s[::-1])
```

#### C++
```cpp
//정방향 해시랑 뒤집은 해시가 같으면 팰린드롬
//h,p: 정방향 / hr,pr: 뒤집은 문자열 해시 (모두 미리 빌드)
bool isPalindrome(int n, int l, int r,
                  vector<long long>& h, vector<long long>& p,
                  vector<long long>& hr, vector<long long>& pr) {
    long long forward  = getHash(h,  p,  l, r);

    //뒤집은 쪽에선 인덱스도 대칭으로 뒤집어서 봐야 함
    long long backward = getHash(hr, pr, n-1-r, n-1-l);
    return forward == backward;
}

//뒤집은 해시: string rev(s.rbegin(), s.rend()); buildHash(rev, hr, pr);
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
