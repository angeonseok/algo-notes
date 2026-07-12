# Selection Sort (선택 정렬)

> 한 줄 정리: 매 패스마다 최솟값을 찾아 앞으로 가져오는 정렬, 스왑 횟수가 적다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<utility>` (`swap`)

## 1. 언제 쓰는가
- 실무/코테에서 직접 쓸 일은 거의 없음
- 스왑 비용이 매우 클 때 이론적으로 유리 (스왑이 최대 N-1번)
- 정렬 알고리즘 개념 이해용

## 2. 핵심 아이디어
- 전체에서 최솟값을 찾아 맨 앞과 교환
- 그 다음 위치부터 반복
- 매 패스마다 앞쪽이 하나씩 확정됨
- 스왑 횟수가 O(n)으로 적지만 비교 횟수는 항상 O(n²)

## 3. 시간/공간 복잡도
| 케이스 | 시간복잡도 |
|--------|-----------|
| 최선 | O(n²) |
| 평균 | O(n²) |
| 최악 | O(n²) |
| 공간복잡도 | O(1) |

- unstable 정렬 (같은 값의 순서가 바뀔 수 있음)

## 4. 기본 코드

### 기본형

#### Python
```python
def selection_sort(arr):
    n = len(arr)
    for i in range(n - 1):
        min_idx = i
        for j in range(i + 1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j
        arr[i], arr[min_idx] = arr[min_idx], arr[i]
    return arr
```

#### C++
```cpp
void selectionSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {
        int min_idx = i;
        for (int j = i + 1; j < n; j++)
            if (arr[j] < arr[min_idx]) min_idx = j;
        swap(arr[i], arr[min_idx]);
    }
}
```

## 5. 이걸 떠올려야 할 때
- 코테에서 직접 구현할 일 없음 → Python `arr.sort()` / C++ `sort(arr.begin(), arr.end())`
- 버블 정렬과 비교: 둘 다 O(n²)이지만 선택 정렬이 스왑 횟수가 적음
- unstable이라는 특성 기억해두기

## 6. 자주 틀리는 포인트
- 조기 종료가 없어서 이미 정렬된 배열도 항상 O(n²)
- unstable 정렬이라 같은 값이 있을 때 순서가 보장 안 됨 → 순서 보존이 필요하면 `stable_sort`
