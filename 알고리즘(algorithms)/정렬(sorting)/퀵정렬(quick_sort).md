# Quick Sort (퀵 정렬)

> 한 줄 정리: 피벗을 기준으로 작은 값/큰 값으로 나눠 재귀 정렬, 평균 O(n log n)으로 실전에서 가장 빠르다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<utility>` (`swap`), `<cstdlib>` (`rand`)

## 1. 언제 쓰는가
- 평균적으로 가장 빠른 정렬이 필요할 때
- 제자리 정렬(추가 메모리 최소화)이 필요할 때
- Python 기본 정렬(Tim Sort) / C++ `sort`(Introsort)로 대체 가능하지만 개념은 필수

## 2. 핵심 아이디어
- 피벗(pivot) 하나를 선택
- 피벗보다 작은 값은 왼쪽, 큰 값은 오른쪽으로 분리
- 각 부분을 재귀적으로 반복
- 피벗 선택이 성능을 좌우 → 랜덤 or 중앙값 선택이 유리

## 3. 시간/공간 복잡도
| 케이스 | 시간복잡도 |
|--------|-----------|
| 최선 | O(n log n) |
| 평균 | O(n log n) |
| 최악 (이미 정렬됨) | O(n²) |
| 공간복잡도 | O(log n) |

- unstable 정렬 (같은 값의 순서가 바뀔 수 있음)
- 최악을 피하려면 랜덤 피벗 사용

## 4. 기본 코드

### 기본형 (직관적인 방식)

#### Python
```python
def quick_sort(arr):
    if len(arr) <= 1:
        return arr

    pivot = arr[len(arr) // 2]  # 중앙값 피벗
    left   = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right  = [x for x in arr if x > pivot]

    return quick_sort(left) + middle + quick_sort(right)
```

#### C++
```cpp
vector<int> quickSort(vector<int>& arr) {
    if (arr.size() <= 1) return arr;

    int pivot = arr[arr.size() / 2];   // 중앙값 피벗
    vector<int> left, mid, right;
    for (int x : arr) {
        if (x < pivot)      left.push_back(x);
        else if (x == pivot) mid.push_back(x);
        else                right.push_back(x);
    }
    left = quickSort(left);
    right = quickSort(right);
    left.insert(left.end(), mid.begin(), mid.end());
    left.insert(left.end(), right.begin(), right.end());
    return left;
}
```

### 제자리 정렬 (in-place, 추가 메모리 최소화)

#### Python
```python
import random

def quick_sort(arr, lo, hi):
    if lo >= hi:
        return
    pivot_idx = partition(arr, lo, hi)
    quick_sort(arr, lo, pivot_idx - 1)
    quick_sort(arr, pivot_idx + 1, hi)

def partition(arr, lo, hi):
    # 랜덤 피벗으로 최악 케이스 방지
    rand_idx = random.randint(lo, hi)
    arr[rand_idx], arr[hi] = arr[hi], arr[rand_idx]

    pivot = arr[hi]
    i = lo - 1
    for j in range(lo, hi):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]
    arr[i + 1], arr[hi] = arr[hi], arr[i + 1]
    return i + 1

# 사용
arr = [3, 1, 4, 1, 5, 9, 2, 6]
quick_sort(arr, 0, len(arr) - 1)
```

#### C++
```cpp
int partition_(vector<int>& arr, int lo, int hi) {   // std::partition 과 이름 충돌 회피
    int rand_idx = lo + rand() % (hi - lo + 1);      // 랜덤 피벗으로 최악 방지
    swap(arr[rand_idx], arr[hi]);

    int pivot = arr[hi];
    int i = lo - 1;
    for (int j = lo; j < hi; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[i + 1], arr[hi]);
    return i + 1;
}

void quickSort(vector<int>& arr, int lo, int hi) {
    if (lo >= hi) return;
    int p = partition_(arr, lo, hi);
    quickSort(arr, lo, p - 1);
    quickSort(arr, p + 1, hi);
}

// 사용
// vector<int> arr = {3, 1, 4, 1, 5, 9, 2, 6};
// quickSort(arr, 0, (int)arr.size() - 1);
```

## 5. 이걸 떠올려야 할 때
- 코테에서 직접 구현할 일은 적음 → Python `arr.sort()` / C++ `sort(arr.begin(), arr.end())`
- 피벗 선택 전략 문제나 분할 방식 이해가 필요한 문제에서 등장
- "K번째 작은 값 O(n)에 찾기" → Quick Select (C++는 `nth_element`)

## 6. 자주 틀리는 포인트
- 피벗을 항상 첫 번째/마지막으로 고르면 이미 정렬된 배열에서 O(n²) → 랜덤 피벗 필수
- Python 재귀 제한 → `sys.setrecursionlimit` 설정하거나 반복문으로 구현
- 기본형(리스트 컴프리헨션/벡터 분리)은 메모리 O(n) 추가 사용
- C++ `rand()`는 품질이 낮음 → 엄밀히는 `<random>`의 `mt19937` 권장, 코테 수준엔 `rand()`로 충분
