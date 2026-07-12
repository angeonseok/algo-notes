# Binary Search (이분 탐색)

> 한 줄 정리: 정렬된 배열에서 절반씩 줄여가며 탐색, O(log N)으로 빠르게 찾는다

## 1. 언제 쓰는가
- 정렬된 배열에서 특정 값을 찾을 때
- "가능한 최솟값/최댓값"을 구할 때 (매개변수 탐색)
- 범위 내에서 조건을 만족하는 경계값을 찾을 때
- 선형 탐색이 시간초과날 때

## 2. 핵심 아이디어
- 탐색 범위를 절반씩 줄임 → O(log N)
- 반드시 정렬된 상태여야 함
- `lo`, `hi`, `mid` 세 포인터로 범위 관리
- 매개변수 탐색: "정답이 될 수 있는 값"을 이분탐색으로 찾기

## 3. 시간/공간 복잡도
| | 복잡도 |
|--|--------|
| 시간 | O(log N) |
| 공간 | O(1) |

## 4. 기본 코드

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<algorithm>`

### 기본형 (특정 값 탐색)

#### Python
```python
def binary_search(arr, target):
    lo, hi = 0, len(arr) - 1

    while lo <= hi:
        mid = (lo + hi) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1

    return -1  # 없음
```

#### C++
```cpp
int binarySearch(vector<int>& arr, int target) {
    int lo = 0, hi = (int)arr.size() - 1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;   // 오버플로 방지
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;  // 없음
}
```

### bisect / lower_bound (실전 권장)

#### Python
```python
import bisect

arr = [1, 2, 4, 4, 5, 7]

# target 이상인 첫 번째 인덱스
bisect.bisect_left(arr, 4)   # 2

# target 초과인 첫 번째 인덱스
bisect.bisect_right(arr, 4)  # 4

# 4의 개수
cnt = bisect.bisect_right(arr, 4) - bisect.bisect_left(arr, 4)  # 2

# 정렬 유지하며 삽입
bisect.insort(arr, 3)  # [1, 2, 3, 4, 4, 5, 7]
```

#### C++
```cpp
vector<int> arr = {1, 2, 4, 4, 5, 7};

// target 이상인 첫 번째 인덱스 (bisect_left)
int left = lower_bound(arr.begin(), arr.end(), 4) - arr.begin();   // 2

// target 초과인 첫 번째 인덱스 (bisect_right)
int right = upper_bound(arr.begin(), arr.end(), 4) - arr.begin();  // 4

// 4의 개수
int cnt = upper_bound(arr.begin(), arr.end(), 4)
        - lower_bound(arr.begin(), arr.end(), 4);                  // 2

// 정렬 유지하며 삽입
arr.insert(lower_bound(arr.begin(), arr.end(), 3), 3);  // {1,2,3,4,4,5,7}
```

### 매개변수 탐색 (Parametric Search)

#### Python
```python
# "조건을 만족하는 최솟값" 찾기
def parametric_search(lo, hi):
    result = hi  # 또는 lo

    while lo <= hi:
        mid = (lo + hi) // 2
        if is_possible(mid):   # 조건 함수
            result = mid
            hi = mid - 1       # 더 작은 값 탐색 (최솟값)
        else:
            lo = mid + 1

    return result

# "조건을 만족하는 최댓값" 찾기
def parametric_search_max(lo, hi):
    result = lo

    while lo <= hi:
        mid = (lo + hi) // 2
        if is_possible(mid):
            result = mid
            lo = mid + 1       # 더 큰 값 탐색 (최댓값)
        else:
            hi = mid - 1

    return result
```

#### C++
```cpp
// "조건을 만족하는 최솟값" 찾기
int parametricSearch(int lo, int hi) {
    int result = hi;  // 또는 lo

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (isPossible(mid)) {   // 조건 함수
            result = mid;
            hi = mid - 1;        // 더 작은 값 탐색 (최솟값)
        } else {
            lo = mid + 1;
        }
    }
    return result;
}

// "조건을 만족하는 최댓값" 찾기
int parametricSearchMax(int lo, int hi) {
    int result = lo;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (isPossible(mid)) {
            result = mid;
            lo = mid + 1;        // 더 큰 값 탐색 (최댓값)
        } else {
            hi = mid - 1;
        }
    }
    return result;
}
```

## 5. 실전 패턴

### 특정 값의 범위 찾기

#### Python
```python
import bisect

def count_in_range(arr, lo, hi):
    # arr에서 lo 이상 hi 이하인 원소 개수
    return bisect.bisect_right(arr, hi) - bisect.bisect_left(arr, lo)
```

#### C++
```cpp
// arr에서 lo 이상 hi 이하인 원소 개수
int countInRange(vector<int>& arr, int lo, int hi) {
    return upper_bound(arr.begin(), arr.end(), hi)
         - lower_bound(arr.begin(), arr.end(), lo);
}
```

### 매개변수 탐색 예시 (랜선 자르기)

#### Python
```python
# N개의 랜선을 K개 이상 만들 수 있는 최대 길이
def solution(lines, K):
    def is_possible(length):
        return sum(l // length for l in lines) >= K

    lo, hi = 1, max(lines)
    result = 0
    while lo <= hi:
        mid = (lo + hi) // 2
        if is_possible(mid):
            result = mid
            lo = mid + 1
        else:
            hi = mid - 1
    return result
```

#### C++
```cpp
// N개의 랜선을 K개 이상 만들 수 있는 최대 길이
int solution(vector<int>& lines, int K) {
    auto isPossible = [&](long long length) {
        long long cnt = 0;
        for (int l : lines) cnt += l / length;
        return cnt >= K;
    };

    int lo = 1, hi = *max_element(lines.begin(), lines.end());
    int result = 0;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (isPossible(mid)) {
            result = mid;
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return result;
}
```

## 6. 이걸 떠올려야 할 때
- "정렬된 배열에서 탐색" → 이분탐색
- "최솟값/최댓값을 구하라" + 단조 증가/감소 조건 → 매개변수 탐색
- "가능한 값의 범위가 크다" (1 ~ 10억) → 매개변수 탐색
- `bisect_left` vs `bisect_right` — 같은 값 여러 개일 때 왼쪽/오른쪽 경계

## 7. 자주 틀리는 포인트
- 정렬 안 된 배열에 이분탐색 → 틀린 결과
- `lo <= hi` vs `lo < hi` 조건 헷갈림 → 값 탐색은 `<=`, 경계 탐색은 상황마다 다름
- 매개변수 탐색에서 `lo`, `hi` 범위를 너무 좁게 잡는 경우
- `mid = (lo + hi) // 2` → Python은 오버플로 없지만 C++/Java 등은 `lo + hi`가 int 범위를 넘을 수 있어 `lo + (hi - lo) / 2` 필수
