# Sieve of Eratosthenes (에라토스테네스의 체)

> 한 줄 정리: N 이하의 모든 소수를 O(N log log N)에 한 번에 구한다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`

## 1. 언제 쓰는가
- N 이하 소수를 전부 구해야 할 때
- 소수 판별을 여러 번 해야 할 때
- 소수 개수를 구해야 할 때

## 2. 핵심 아이디어
- 2부터 시작해서 배수를 모두 지워나감
- 지워지지 않은 수가 소수
- √N까지만 체크하면 충분

## 3. 시간/공간 복잡도
| | 복잡도 |
|--|--------|
| 시간 | O(N log log N) |
| 공간 | O(N) |

## 4. 기본 코드

### 기본형

#### Python
```python
#일단 다 소수라고 깔고 시작, 아닌 놈들을 지워나가자
def sieve(n):
    is_prime = [True] * (n + 1)

    #0이랑 1은 소수 아니니까 먼저 빼기
    is_prime[0] = is_prime[1] = False

    #√n까지만 봐도 충분
    for i in range(2, int(n**0.5) + 1):
        #이미 지워진 놈은 걔 배수도 다 지워졌으니 스킵
        if is_prime[i]:
            #i*i 미만은 더 작은 소수가 이미 지웠으니까 i*i부터 시작
            for j in range(i*i, n + 1, i):
                is_prime[j] = False

    #살아남은 놈들이 소수
    return [i for i in range(2, n + 1) if is_prime[i]]

primes = sieve(100)  #100 이하 소수 목록
```

#### C++
```cpp
//일단 다 소수라고 깔고 시작, 아닌 놈들을 지워나가자
vector<int> sieve(int n) {
    //vector<bool>은 비트 압축이라 메모리 8배 절약됨
    vector<bool> is_prime(n + 1, true);

    //0이랑 1은 소수 아니니까 먼저 빼기
    if (n >= 0) is_prime[0] = false;
    if (n >= 1) is_prime[1] = false;

    //√n까지만 봐도 충분
    for (int i = 2; (long long)i * i <= n; i++) {
        //이미 지워진 놈은 걔 배수도 다 지워졌으니 스킵
        if (is_prime[i]) {
            //i*i 미만은 더 작은 소수가 이미 지웠으니까 i*i부터, i*i가 int 넘칠 수 있어서 long long
            for (long long j = (long long)i * i; j <= n; j += i)
                is_prime[j] = false;
        }
    }

    //살아남은 놈들이 소수
    vector<int> primes;
    for (int i = 2; i <= n; i++)
        if (is_prime[i]) primes.push_back(i);
    return primes;
}
//sieve(100) → 100 이하 소수 목록
```

### 소수 판별만 할 때

#### Python
```python
#체 깔기 아까운 단발성 판별이면 걍 √n까지 나눠보자
def is_prime(n):
    #0, 1은 소수 아님
    if n < 2:
        return False

    #약수가 있으면 √n 이하에 반드시 하나는 있음
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False
    return True
```

#### C++
```cpp
//체 깔기 아까운 단발성 판별이면 걍 √n까지 나눠보자
bool isPrime(long long n) {
    //0, 1은 소수 아님
    if (n < 2) return false;

    //약수가 있으면 √n 이하에 반드시 하나는 있음. sqrt 쓰면 오차나니까 i*i로 비교
    for (long long i = 2; i * i <= n; i++)
        if (n % i == 0) return false;
    return true;
}
```

## 5. 이걸 떠올려야 할 때
- "N 이하 소수 전부" → 에라토스테네스의 체
- "단일 수 소수 판별" → O(√N) 판별
- N이 크면 (10^6 이상) → 체로 전처리 후 O(1) 조회

## 6. 자주 틀리는 포인트
- `range(i*i, ...)` 대신 `range(i*2, ...)`로 하면 느려짐
- N이 매우 크면 (10^9 이상) 메모리 초과 → 밀러-라빈 소수 판별법 고려
- 1은 소수가 아님 → `is_prime[1] = False` 명시
- C++에서 `i * i`가 int 범위를 넘길 수 있음 → `long long` 캐스팅 (N이 수억일 때)
