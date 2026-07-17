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
#정렬된 arr에서 target 위치 찾기
def binary_search(arr, target):
    l, h = 0, len(arr) - 1

    while l <= h:
        mid = (l + h) // 2

        #찾았으면 바로 반환
        if arr[mid] == target:
            return mid

        #mid가 작으면 target은 오른쪽에 있음
        elif arr[mid] < target:
            l = mid + 1

        #아니면 왼쪽
        else:
            h = mid - 1

    #범위 다 좁혔는데 없으면 없는 거
    return -1
```

#### C++
```cpp
//정렬된 arr에서 target 위치 찾기
int binarySearch(vector<int>& arr, int target) {
    int l = 0, h = (int)arr.size() - 1;

    while (l <= h) {
        //l + h 하면 int 넘칠 수 있으니까 이렇게 (오버플로 방지)
        int mid = l + (h - l) / 2;

        //찾았으면 바로 반환, mid가 작으면 오른쪽, 크면 왼쪽
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) l = mid + 1;
        else h = mid - 1;
    }

    //범위 다 좁혔는데 없으면 없는 거
    return -1;
}
```

### bisect / lower_bound (실전 권장)

#### Python
```python
import bisect

arr = [1, 2, 4, 4, 5, 7]

#target 이상인 첫 번째 인덱스 > 2
bisect.bisect_left(arr, 4)

#target 초과인 첫 번째 인덱스 > 4
bisect.bisect_right(arr, 4)

#오른쪽 경계 - 왼쪽 경계 = 4의 개수 > 2
cnt = bisect.bisect_right(arr, 4) - bisect.bisect_left(arr, 4)

#정렬 유지하며 삽입 > [1, 2, 3, 4, 4, 5, 7]
bisect.insort(arr, 3)
```

#### C++
```cpp
vector<int> arr = {1, 2, 4, 4, 5, 7};

//target 이상인 첫 번째 인덱스 (bisect_left) > 2
int left = lower_bound(arr.begin(), arr.end(), 4) - arr.begin();

//target 초과인 첫 번째 인덱스 (bisect_right) > 4
int right = upper_bound(arr.begin(), arr.end(), 4) - arr.begin();

//오른쪽 경계 - 왼쪽 경계 = 4의 개수 > 2
int cnt = upper_bound(arr.begin(), arr.end(), 4)
        - lower_bound(arr.begin(), arr.end(), 4);

//정렬 유지하며 삽입 > {1,2,3,4,4,5,7}
arr.insert(lower_bound(arr.begin(), arr.end(), 3), 3);
```

### 매개변수 탐색 (Parametric Search)

#### Python
```python
#"조건을 만족하는 최솟값" 찾기
def parametric_search(l, h):
    #일단 아무거나 박아두고 되는 놈 나올 때마다 갱신 (또는 l)
    ans = h

    while l <= h:
        mid = (l + h) // 2

        #is_possible이 조건 함수. 되는 값이면 답 후보로 챙겨두고
        if is_possible(mid):
            ans = mid

            #더 작은 것도 되나 왼쪽 뒤져보자
            h = mid - 1

        #안 되면 너무 작았다는 뜻
        else:
            l = mid + 1

    return ans

#"조건을 만족하는 최댓값" 찾기
def parametric_search_max(l, h):
    ans = l

    while l <= h:
        mid = (l + h) // 2

        #되는 값이면 답 후보로 챙겨두고
        if is_possible(mid):
            ans = mid

            #더 큰 것도 되나 오른쪽 뒤져보자
            l = mid + 1

        #안 되면 너무 컸다는 뜻
        else:
            h = mid - 1

    return ans
```

#### C++
```cpp
//"조건을 만족하는 최솟값" 찾기
int parametricSearch(int l, int h) {
    //일단 아무거나 박아두고 되는 놈 나올 때마다 갱신 (또는 l)
    int ans = h;

    while (l <= h) {
        int mid = l + (h - l) / 2;

        //isPossible이 조건 함수. 되는 값이면 답 후보로 챙겨두고
        if (isPossible(mid)) {
            ans = mid;

            //더 작은 것도 되나 왼쪽 뒤져보자
            h = mid - 1;

        //안 되면 너무 작았다는 뜻
        } else {
            l = mid + 1;
        }
    }

    return ans;
}

//"조건을 만족하는 최댓값" 찾기
int parametricSearchMax(int l, int h) {
    int ans = l;

    while (l <= h) {
        int mid = l + (h - l) / 2;

        //되는 값이면 답 후보로 챙겨두고
        if (isPossible(mid)) {
            ans = mid;

            //더 큰 것도 되나 오른쪽 뒤져보자
            l = mid + 1;

        //안 되면 너무 컸다는 뜻
        } else {
            h = mid - 1;
        }
    }

    return ans;
}
```

## 5. 실전 패턴

### 특정 값의 범위 찾기

#### Python
```python
import bisect

#arr에서 l 이상 h 이하인 원소 개수
def count_in_range(arr, l, h):
    #양쪽 경계 인덱스 구해서 빼면 그 사이 개수
    return bisect.bisect_right(arr, h) - bisect.bisect_left(arr, l)
```

#### C++
```cpp
//arr에서 l 이상 h 이하인 원소 개수
int countInRange(vector<int>& arr, int l, int h) {
    //양쪽 경계 인덱스 구해서 빼면 그 사이 개수
    return upper_bound(arr.begin(), arr.end(), h)
         - lower_bound(arr.begin(), arr.end(), l);
}
```

### 매개변수 탐색 예시 (랜선 자르기)

#### Python
```python
#N개의 랜선을 K개 이상 만들 수 있는 최대 길이
def solution(lines, K):
    #이 길이로 잘랐을 때 K개 이상 나오나?
    def is_possible(length):
        return sum(i // length for i in lines) >= K

    #길이는 1부터 제일 긴 랜선까지. 답이 이 범위 안에 있음
    l, h = 1, max(lines)
    ans = 0

    while l <= h:
        mid = (l + h) // 2

        #길이가 길수록 개수는 줄어듦 > 되면 더 길게 가보자
        if is_possible(mid):
            ans = mid
            l = mid + 1

        #안 되면 너무 길게 잘랐다는 뜻
        else:
            h = mid - 1

    return ans
```

#### C++
```cpp
//N개의 랜선을 K개 이상 만들 수 있는 최대 길이
int solution(vector<int>& lines, int K) {
    //이 길이로 잘랐을 때 K개 이상 나오나?
    auto isPossible = [&](long long length) {
        long long cnt = 0;
        for (int i : lines) cnt += i / length;
        return cnt >= K;
    };

    //길이는 1부터 제일 긴 랜선까지. 답이 이 범위 안에 있음
    int l = 1, h = *max_element(lines.begin(), lines.end());
    int ans = 0;

    while (l <= h) {
        int mid = l + (h - l) / 2;

        //길이가 길수록 개수는 줄어듦 > 되면 더 길게 가보자
        if (isPossible(mid)) {
            ans = mid;
            l = mid + 1;

        //안 되면 너무 길게 잘랐다는 뜻
        } else {
            h = mid - 1;
        }
    }

    return ans;
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
