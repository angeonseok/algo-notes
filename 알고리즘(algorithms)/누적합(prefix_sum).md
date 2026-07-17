# Prefix Sum (누적 합)

> 한 줄 정리: 구간 합을 O(1)에 구하기 위해 미리 합을 저장해두는 방법

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<unordered_map>`

## 1. 언제 쓰는가
- 구간 합을 여러 번 빠르게 구해야 할 때
- 2차원 격자의 부분 합
- 차이 배열(difference array)로 구간 업데이트

## 2. 핵심 아이디어
- prefix[i] = arr[0] + arr[1] + ... + arr[i-1]
- 구간 합 arr[l:r+1] = prefix[r+1] - prefix[l]
- 전처리 O(N), 쿼리 O(1)
- 2차원으로 확장 가능

## 3. 시간/공간 복잡도
| | 복잡도 |
|--|--------|
| 전처리 | O(N) |
| 구간 합 쿼리 | O(1) |
| 공간 | O(N) |

## 4. 기본 코드

### 1차원 누적 합

#### Python
```python
arr = [1, 2, 3, 4, 5]
n = len(arr)

#0번 자리 비워두고 시작해야 빼기가 깔끔해짐
prefix = [0] * (n + 1)
for i in range(n):
    prefix[i + 1] = prefix[i] + arr[i]

#arr[l:r+1] 구간 합 (0-indexed). 빼기 한 번이면 끝
def range_sum(l, r):
    return prefix[r + 1] - prefix[l]

#arr[1] + arr[2] + arr[3] = 9
range_sum(1, 3)
```

#### C++
```cpp
vector<int> arr = {1, 2, 3, 4, 5};
int n = arr.size();

//합 커지면 int로는 터지니까 long long. 0번 자리는 비워두고 시작
vector<long long> prefix(n + 1, 0);
for (int i = 0; i < n; i++)
    prefix[i + 1] = prefix[i] + arr[i];

//arr[l..r] 구간 합 (0-indexed). 빼기 한 번이면 끝 — prefix 전역 가정
long long rangeSum(int l, int r) {
    return prefix[r + 1] - prefix[l];
}

//rangeSum(1, 3) == 9
```

### 2차원 누적 합

#### Python
```python
#0행 0열 패딩해두면 경계 체크 안 해도 되니까 편함
N, M = len(mat), len(mat[0])
prefix = [[0] * (M + 1) for _ in range(N + 1)]

#위 + 왼쪽 더하면 겹치는 부분이 두 번 들어가니까 한 번 빼주자
for i in range(1, N + 1):
    for j in range(1, M + 1):
        prefix[i][j] = (mat[i-1][j-1]
                      + prefix[i-1][j]
                      + prefix[i][j-1]
                      - prefix[i-1][j-1])

#(r1,c1) ~ (r2,c2) 구간 합 (0-indexed)
#위/왼쪽 덜어내고, 두 번 빠진 모서리 다시 더해주기
def range_sum_2d(r1, c1, r2, c2):
    return (prefix[r2+1][c2+1]
          - prefix[r1][c2+1]
          - prefix[r2+1][c1]
          + prefix[r1][c1])
```

#### C++
```cpp
//0행 0열 패딩해두면 경계 체크 안 해도 되니까 편함
int N = mat.size(), M = mat[0].size();
vector<vector<long long>> prefix(N + 1, vector<long long>(M + 1, 0));

//위 + 왼쪽 더하면 겹치는 부분이 두 번 들어가니까 한 번 빼주자
for (int i = 1; i <= N; i++)
    for (int j = 1; j <= M; j++)
        prefix[i][j] = mat[i-1][j-1]
                     + prefix[i-1][j]
                     + prefix[i][j-1]
                     - prefix[i-1][j-1];

//(r1,c1) ~ (r2,c2) 구간 합 (0-indexed)
//위/왼쪽 덜어내고, 두 번 빠진 모서리 다시 더해주기
long long rangeSum2d(int r1, int c1, int r2, int c2) {
    return prefix[r2+1][c2+1]
         - prefix[r1][c2+1]
         - prefix[r2+1][c1]
         + prefix[r1][c1];
}
```

### 차이 배열 (구간 업데이트 O(1))

#### Python
```python
#arr[l:r+1]에 val을 더하는 연산을 여러 번 수행
#diff[r+1]까지 건드리니까 크기는 n+1로 잡자
n = 5
diff = [0] * (n + 1)

#시작에 +val, 끝난 다음칸에 -val만 박기. 실제 더하기는 나중에 한 방에
def range_update(l, r, val):
    diff[l] += val
    diff[r + 1] -= val

range_update(1, 3, 5)
range_update(2, 4, 3)

#업데이트 다 끝나고 누적합 돌려야 진짜 배열이 나옴
result = [0] * n
result[0] = diff[0]
for i in range(1, n):
    result[i] = result[i-1] + diff[i]
```

#### C++
```cpp
//arr[l..r]에 val을 더하는 연산을 여러 번 수행
//diff[r+1]까지 건드리니까 크기는 n+1
int n = 5;
vector<long long> diff(n + 1, 0);

//시작에 +val, 끝난 다음칸에 -val만 박기 (diff 전역 가정)
void rangeUpdate(int l, int r, long long val) {
    diff[l] += val;
    diff[r + 1] -= val;
}

//업데이트 다 끝나고 누적합 돌려야 진짜 배열이 나옴
// vector<long long> result(n);
// result[0] = diff[0];
// for (int i = 1; i < n; i++) result[i] = result[i-1] + diff[i];
```

## 5. 실전 패턴

### 나머지 합 (모듈러 누적 합)

#### Python
```python
from collections import defaultdict

#구간 합이 M으로 나누어 떨어지는 경우의 수
#나머지가 같은 두 지점 사이 구간합은 M의 배수. 그 짝을 세자
def count_divisible(arr, M):
    prefix = 0

    #나머지 0은 시작점 몫으로 미리 하나 깔아두기
    cnt = defaultdict(int)
    cnt[0] = 1
    result = 0

    for v in arr:
        prefix = (prefix + v) % M

        #앞에서 같은 나머지 나온 개수만큼 짝이 생김
        result += cnt[prefix]
        cnt[prefix] += 1

    return result
```

#### C++
```cpp
//구간 합이 M으로 나누어 떨어지는 경우의 수
//나머지가 같은 두 지점 사이 구간합은 M의 배수. 그 짝을 세자
long long countDivisible(vector<int>& arr, int M) {
    long long prefix = 0, result = 0;

    //나머지 0은 시작점 몫으로 미리 하나 깔아두기
    unordered_map<long long,long long> cnt;
    cnt[0] = 1;

    for (int v : arr) {
        //C++ %는 음수 뱉을 수 있으니 M 더했다 다시 나머지
        prefix = ((prefix + v) % M + M) % M;

        //앞에서 같은 나머지 나온 개수만큼 짝이 생김
        result += cnt[prefix];
        cnt[prefix]++;
    }
    return result;
}
```

## 6. 이걸 떠올려야 할 때
- "구간 합을 여러 번 구해야 한다" → 누적 합
- "2차원 부분 합" → 2차원 누적 합
- "구간에 값을 더하는 연산이 많다" → 차이 배열
- "구간 합 % M" → 나머지 누적 합

## 7. 자주 틀리는 포인트
- prefix 배열 크기를 N+1로 해야 인덱스 에러 없음
- 2차원 누적 합 공식에서 포함/제외 범위 헷갈리는 경우 → 그림 그려서 확인
- 차이 배열에서 `diff[r+1]`이 범위 초과 → 크기를 n+1로 선언
- 0-indexed vs 1-indexed 혼용 주의
- C++은 합이 커질 수 있으니 prefix를 `long long`으로
