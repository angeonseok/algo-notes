# Prime Factorization (소인수분해)

> 한 줄 정리: 어떤 수를 소수들의 곱으로 분해한다, √N까지만 나눠보면 충분하다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<map>`, `<algorithm>`

## 1. 언제 쓰는가
- 약수 개수/약수의 합 구할 때
- 두 수의 GCD/LCM을 소인수로 분석할 때
- 특정 수가 어떤 소수로 이루어졌는지 확인
- 제곱수 판별

## 2. 핵심 아이디어
- 2부터 √N까지 나눠보며 소인수 추출
- √N까지 나눠도 안 떨어지면 나머지가 소수
- 각 소인수의 지수까지 함께 구하면 약수 개수 계산 가능

## 3. 시간복잡도
| | 복잡도 |
|--|--------|
| 시간 | O(√N) |

## 4. 기본 코드

### 소인수 목록 구하기

#### Python
```python
#작은 놈부터 나눠떨어지는 만큼 계속 빼내자
def prime_factors(n):
    factors = []
    d = 2

    #√n까지만 봐도 충분
    while d * d <= n:
        #더 안 나눠질 때까지 같은 d로 계속 빼기
        while n % d == 0:
            factors.append(d)
            n //= d
        d += 1

    #다 나누고 1보다 크게 남았으면 걔가 소수
    if n > 1:
        factors.append(n)
    return factors

prime_factors(12)   #[2, 2, 3]
prime_factors(100)  #[2, 2, 5, 5]
```

#### C++
```cpp
//작은 놈부터 나눠떨어지는 만큼 계속 빼내자
vector<long long> primeFactors(long long n) {
    vector<long long> factors;

    //√n까지만 봐도 충분
    for (long long d = 2; d * d <= n; d++) {
        //더 안 나눠질 때까지 같은 d로 계속 빼기
        while (n % d == 0) {
            factors.push_back(d);
            n /= d;
        }
    }

    //다 나누고 1보다 크게 남았으면 걔가 소수
    if (n > 1) factors.push_back(n);
    return factors;
}
//primeFactors(12)  → {2, 2, 3}
//primeFactors(100) → {2, 2, 5, 5}
```

### 소인수와 지수 함께 구하기

#### Python
```python
#지수까지 필요하면 리스트 말고 딕셔너리에 세면서 담자
def prime_factorization(n):
    factors = {}
    d = 2

    #√n까지만 봐도 충분
    while d * d <= n:
        #뺀 횟수가 곧 지수
        while n % d == 0:
            factors[d] = factors.get(d, 0) + 1
            n //= d
        d += 1

    #다 나누고 1보다 크게 남았으면 걔가 소수
    if n > 1:
        factors[n] = factors.get(n, 0) + 1
    return factors

prime_factorization(12)   #{2: 2, 3: 1}  → 2² × 3
prime_factorization(100)  #{2: 2, 5: 2}  → 2² × 5²
```

#### C++
```cpp
//지수까지 필요하면 세면서 담자
map<long long, int> primeFactorization(long long n) {
    //소수 → 지수. map이라 알아서 정렬됨
    map<long long, int> factors;

    //√n까지만 봐도 충분
    for (long long d = 2; d * d <= n; d++) {
        //뺀 횟수가 곧 지수
        while (n % d == 0) {
            factors[d]++;
            n /= d;
        }
    }

    //다 나누고 1보다 크게 남았으면 걔가 소수
    if (n > 1) factors[n]++;
    return factors;
}
//primeFactorization(12)  → {2:2, 3:1}  → 2² × 3
//primeFactorization(100) → {2:2, 5:2}  → 2² × 5²
```

### 약수 개수 구하기

#### Python
```python
#n = p1^a1 * p2^a2 * ... 면 약수 개수 = (a1+1)(a2+1)...
#각 소수를 0~ai개 쓸 수 있으니까 지수마다 +1 해서 곱하기
def count_divisors(n):
    factors = prime_factorization(n)
    result = 1

    #소수가 뭔지는 상관없고 지수만 있으면 됨
    for exp in factors.values():
        result *= (exp + 1)
    return result

count_divisors(12)  #(2+1)(1+1) = 6 → 1,2,3,4,6,12
```

#### C++
```cpp
//n = p1^a1 * p2^a2 * ... 면 약수 개수 = (a1+1)(a2+1)...
//각 소수를 0~ai개 쓸 수 있으니까 지수마다 +1 해서 곱하기
long long countDivisors(long long n) {
    map<long long, int> factors = primeFactorization(n);
    long long result = 1;

    //C++14엔 구조적 바인딩 없으니까 .second로 지수 꺼내기
    for (auto& kv : factors)
        result *= (kv.second + 1);
    return result;
}
//countDivisors(12) → (2+1)(1+1) = 6 → 1,2,3,4,6,12
```

### 약수 전체 구하기

#### Python
```python
#약수는 짝지어 나오니까 √n까지만 돌면서 둘 다 줍자
def get_divisors(n):
    divisors = []

    #i가 약수면 n//i도 약수
    for i in range(1, int(n**0.5) + 1):
        if n % i == 0:
            divisors.append(i)

            #완전제곱수면 i랑 n//i가 같으니 중복 방지
            if i != n // i:
                divisors.append(n // i)

    #짝으로 주워서 순서 엉켜있음
    return sorted(divisors)

get_divisors(12)  #[1, 2, 3, 4, 6, 12]
```

#### C++
```cpp
//약수는 짝지어 나오니까 √n까지만 돌면서 둘 다 줍자
vector<long long> getDivisors(long long n) {
    vector<long long> divisors;

    //sqrt 쓰면 오차나니까 i*i로 비교. i가 약수면 n/i도 약수
    for (long long i = 1; i * i <= n; i++) {
        if (n % i == 0) {
            divisors.push_back(i);

            //완전제곱수면 i랑 n/i가 같으니 중복 방지
            if (i != n / i) divisors.push_back(n / i);
        }
    }

    //짝으로 주워서 순서 엉켜있음
    sort(divisors.begin(), divisors.end());
    return divisors;
}
//getDivisors(12) → {1, 2, 3, 4, 6, 12}
```

## 5. 이걸 떠올려야 할 때
- "약수 개수" → 소인수분해 후 지수 곱
- "약수 전체" → O(√N) 순회
- "소수인지 판별" → 소인수분해 후 인수가 자기 자신뿐이면 소수
- "제곱수 판별" → 모든 지수가 짝수면 제곱수

## 6. 자주 틀리는 포인트
- √N까지 나눈 후 `if n > 1` 체크 빠뜨리면 큰 소인수 누락
- 약수 개수 공식에서 지수에 +1 빠뜨리는 경우
- 약수 구할 때 `i == n // i`인 경우 (완전제곱수) 중복 추가 주의
- C++에서 `int(n**0.5)` 대신 `i * i <= n` 사용 → 부동소수점 오차로 인한 경계 실수 방지
