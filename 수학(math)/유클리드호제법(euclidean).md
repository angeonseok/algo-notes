# Euclidean Algorithm (유클리드 호제법)

> 한 줄 정리: 두 수의 최대공약수(GCD)를 O(log N)에 구한다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<algorithm>` (`__gcd`)

## 1. 언제 쓰는가
- 최대공약수(GCD), 최소공배수(LCM) 구할 때
- 분수 약분
- 두 수가 서로소인지 확인

## 2. 핵심 아이디어
- gcd(a, b) = gcd(b, a % b)
- a % b = 0이면 b가 GCD
- LCM = a * b // GCD

## 3. 시간복잡도
| | 복잡도 |
|--|--------|
| 시간 | O(log N) |

## 4. 기본 코드

### 기본형

#### Python
```python
def gcd(a, b):
    while b:
        a, b = b, a % b
    return a

def lcm(a, b):
    return a * b // gcd(a, b)
```

#### C++
```cpp
long long gcd(long long a, long long b) {
    while (b) {
        long long t = a % b;
        a = b;
        b = t;
    }
    return a;
}

long long lcm(long long a, long long b) {
    return a / gcd(a, b) * b;   // 오버플로 방지: 먼저 나눈 뒤 곱하기
}
```

### 내장/실전 권장

#### Python
```python
from math import gcd
from math import lcm  # Python 3.9+

gcd(12, 8)   # 4
lcm(12, 8)   # 24

# 여러 수의 GCD/LCM
from functools import reduce
nums = [12, 8, 6]
reduce(gcd, nums)  # 2
reduce(lcm, nums)  # 24
```

#### C++
```cpp
// C++14: GCC 내장 __gcd 사용 (std::gcd / std::lcm 은 C++17부터 <numeric>)
__gcd(12, 8);   // 4

// 여러 수의 GCD/LCM
vector<long long> nums = {12, 8, 6};
long long g = 0;
for (long long x : nums) g = __gcd(g, x);          // 2  (gcd(0, x) = x)
long long l = 1;
for (long long x : nums) l = l / __gcd(l, x) * x;  // 24
```

### 서로소 확인

#### Python
```python
def is_coprime(a, b):
    return gcd(a, b) == 1
```

#### C++
```cpp
bool isCoprime(long long a, long long b) {
    return __gcd(a, b) == 1;
}
```

## 5. 이걸 떠올려야 할 때
- "최대공약수/최소공배수" → 유클리드 호제법
- "분수를 기약분수로" → GCD로 분자/분모 나누기
- "두 수가 서로소" → gcd == 1

## 6. 자주 틀리는 포인트
- LCM 구할 때 `a * b // gcd(a, b)` 순서 중요 → `(a * b) // gcd` 먼저 곱하면 오버플로 가능 → `a // gcd(a, b) * b` 권장
- Python 3.9 미만에서 `math.lcm` 없음 → 직접 구현
- C++: `std::gcd`/`std::lcm`은 C++17부터 → C++14에선 `__gcd`(GCC) 또는 직접 구현
