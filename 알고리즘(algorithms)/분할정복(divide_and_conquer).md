# Divide and Conquer (분할 정복)

> 한 줄 정리: 문제를 작은 부분으로 나누고 각각 해결한 뒤 합치는 방법

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`

## 1. 언제 쓰는가
- 문제를 동일한 구조의 더 작은 문제로 나눌 수 있을 때
- 부분 문제의 결과를 합쳐서 전체 문제를 풀 수 있을 때
- 큰 입력을 반씩 줄여나갈 수 있을 때
- 거듭제곱, 행렬 곱, 최근접 점 쌍 등

## 2. 핵심 아이디어
- 분할(Divide): 문제를 더 작은 부분으로 나눔
- 정복(Conquer): 각 부분을 재귀적으로 해결
- 합치기(Combine): 부분 결과를 합쳐 전체 해답 도출
- 매 단계에서 절반씩 줄어들면 → O(log N) or O(N log N)

## 3. 시간복잡도
| 문제 | 복잡도 |
|------|--------|
| 이진 탐색 | O(log N) |
| 병합 정렬 | O(N log N) |
| 거듭제곱 | O(log N) |
| 행렬 거듭제곱 | O(M³ log N) |

- 마스터 정리: T(N) = aT(N/b) + f(N) 형태로 복잡도 분석 가능

## 4. 기본 코드

### 분할 정복 기본 구조
```
Result divideAndConquer(Problem p) {
    if (충분히 작으면) return solveDirect(p);   // 기저 조건
    (left, right) = split(p);                   // 분할
    L = divideAndConquer(left);                 // 정복
    R = divideAndConquer(right);
    return combine(L, R);                        // 합치기
}
```

### 거듭제곱 (빠른 지수)

#### Python
```python
#지수를 반씩 접어서 O(log N)에 거듭제곱 구하기
def power(base, exp, mod=None):
    #더 쪼갤 게 없으면 1 박고 끝
    if exp == 0:
        return 1

    #짝수면 반만 구해서 제곱하면 되니까 개이득
    if exp % 2 == 0:
        half = power(base, exp // 2, mod)
        result = half * half

    #홀수면 하나 떼서 짝수로 만들어놓자
    else:
        result = base * power(base, exp - 1, mod)

    #mod 있으면 매 단계 나머지 취해서 숫자 안 불어나게
    return result % mod if mod else result

#써먹기 > 1024, 24(mod 1000)
power(2, 10)
power(2, 10, 1000)
```

#### C++
```cpp
//지수를 반씩 접어서 O(log N)에 거듭제곱 구하기
long long power(long long base, long long exp, long long mod = 0) {
    //더 쪼갤 게 없으면 1 박고 끝
    if (exp == 0) return 1;

    long long result;

    //짝수면 반만 구해서 제곱하면 되니까 개이득
    if (exp % 2 == 0) {
        long long half = power(base, exp / 2, mod);
        result = half * half;
    }

    //홀수면 하나 떼서 짝수로 만들어놓자
    else {
        result = base * power(base, exp - 1, mod);
    }

    //mod 있으면 매 단계 나머지 취해서 숫자 안 불어나게
    return mod ? result % mod : result;
}

//써먹기 > power(2, 10) == 1024, power(2, 10, 1000) == 24
//mod 없이 큰 지수 넣으면 오버플로 > mod 넣거나 매 단계 mod 필요
```

### 행렬 거듭제곱 (피보나치 O(log N))

#### Python
```python
#행렬 곱은 걍 정의대로 3중 루프
def mat_mul(A, B, mod):
    n = len(A)
    C = [[0] * n for _ in range(n)]

    #커지기 전에 매번 mod 때려주자
    for i in range(n):
        for j in range(n):
            for k in range(n):
                C[i][j] = (C[i][j] + A[i][k] * B[k][j]) % mod

    return C

#숫자 거듭제곱이랑 똑같은 원리, 곱셈만 행렬 곱으로 바꾼 것
def mat_pow(M, exp, mod):
    n = len(M)

    #단위 행렬로 시작해야 1 곱하는 효과
    result = [[1 if i == j else 0 for j in range(n)] for i in range(n)]

    #exp 비트 훑으면서 켜진 자리만 곱해서 모으기
    while exp:
        if exp % 2:
            result = mat_mul(result, M, mod)

        #M을 계속 제곱해두면 M^1, M^2, M^4... 준비됨
        M = mat_mul(M, M, mod)
        exp //= 2

    return result

#[[1,1],[1,0]]^n 하면 피보나치가 튀어나옴
def fib(n, mod=1000000007):
    #0, 1은 그대로 반환
    if n <= 1:
        return n

    M = [[1, 1], [1, 0]]

    #n-1 제곱의 [0][0]이 fib(n)
    result = mat_pow(M, n - 1, mod)
    return result[0][0]
```

#### C++
```cpp
typedef vector<vector<long long>> Matrix;

//행렬 곱은 걍 정의대로 3중 루프
Matrix matMul(Matrix& A, Matrix& B, long long mod) {
    int n = A.size();
    Matrix C(n, vector<long long>(n, 0));

    //커지기 전에 매번 mod 때려주자
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            for (int k = 0; k < n; k++)
                C[i][j] = (C[i][j] + A[i][k] * B[k][j]) % mod;

    return C;
}

//숫자 거듭제곱이랑 똑같은 원리, 곱셈만 행렬 곱으로 바꾼 것
Matrix matPow(Matrix M, long long exp, long long mod) {
    int n = M.size();
    Matrix result(n, vector<long long>(n, 0));

    //단위 행렬로 시작해야 1 곱하는 효과
    for (int i = 0; i < n; i++) result[i][i] = 1;

    //exp 비트 훑으면서 켜진 자리만 곱해서 모으기
    while (exp) {
        if (exp & 1) result = matMul(result, M, mod);

        //M을 계속 제곱해두면 M^1, M^2, M^4... 준비됨
        M = matMul(M, M, mod);
        exp >>= 1;
    }
    return result;
}

//{{1,1},{1,0}}^n 하면 피보나치가 튀어나옴
long long fib(long long n, long long mod = 1000000007) {
    //0, 1은 그대로 반환
    if (n <= 1) return n;

    Matrix M = {{1, 1}, {1, 0}};

    //n-1 제곱의 [0][0]이 fib(n)
    Matrix result = matPow(M, n - 1, mod);
    return result[0][0];
}
```

### 쿼드 트리 / 색종이 분할

#### Python
```python
#(r, c)에서 시작하는 size짜리 정사각형 처리
def quad(mat, r, c, size):
    #왼쪽 위 색을 기준으로 잡고 비교하자
    color = mat[r][c]

    #하나라도 색 다르면 압축 실패
    for i in range(r, r + size):
        for j in range(c, c + size):
            if mat[i][j] != color:
                #반으로 갈라서 4등분 각각 다시 내려보내기
                half = size // 2
                quad(mat, r,        c,        half)
                quad(mat, r,        c + half, half)
                quad(mat, r + half, c,        half)
                quad(mat, r + half, c + half, half)
                return

    #끝까지 왔으면 전부 같은 색이니까 하나로 압축
    result.append(color)
```

#### C++
```cpp
vector<int> result;

//(r, c)에서 시작하는 size짜리 정사각형 처리
void quad(vector<vector<int>>& mat, int r, int c, int size) {
    //왼쪽 위 색을 기준으로 잡고 비교하자
    int color = mat[r][c];

    //하나라도 색 다르면 압축 실패
    for (int i = r; i < r + size; i++)
        for (int j = c; j < c + size; j++)
            if (mat[i][j] != color) {
                //반으로 갈라서 4등분 각각 다시 내려보내기
                int half = size / 2;
                quad(mat, r,        c,        half);
                quad(mat, r,        c + half, half);
                quad(mat, r + half, c,        half);
                quad(mat, r + half, c + half, half);
                return;
            }

    //끝까지 왔으면 전부 같은 색이니까 하나로 압축
    result.push_back(color);
}
```

## 5. 이걸 떠올려야 할 때
- "N이 매우 크고 거듭제곱/반복이 필요" → 빠른 지수
- "피보나치를 매우 큰 N까지" → 행렬 거듭제곱
- "격자를 4등분해서 처리" → 쿼드 트리
- "절반씩 나눠서 각각 풀고 합칠 수 있다" → 분할 정복

## 6. 자주 틀리는 포인트
- 기저 조건 빠뜨리면 무한 재귀
- 거듭제곱에서 mod 연산 빠뜨리면 오버플로 → 중간마다 mod 적용 (C++은 특히 `long long`)
- 행렬 거듭제곱에서 단위 행렬 초기화 빠뜨리는 경우
- 쿼드 트리에서 범위 계산 실수 → r, c, size 꼼꼼히 체크
