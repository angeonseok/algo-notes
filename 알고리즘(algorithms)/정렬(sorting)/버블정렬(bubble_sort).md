# Bubble Sort (버블 정렬)

> 한 줄 정리: 인접한 두 원소를 비교해 큰 값을 뒤로 밀어내는 정렬, 구현은 쉽지만 느리다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<utility>` (`swap`)

## 1. 언제 쓰는가
- 실무/코테에서 직접 쓸 일은 거의 없음
- 정렬 알고리즘의 기초 개념 이해용
- 거의 정렬된 배열에서 조기 종료 최적화 적용 시 나쁘지 않음

## 2. 핵심 아이디어
- 매 패스마다 인접한 두 원소를 비교해 큰 값을 오른쪽으로 이동
- 1패스가 끝나면 가장 큰 값이 맨 뒤로 확정
- N-1번 패스 반복하면 정렬 완료
- 스왑이 한 번도 없으면 조기 종료 가능 → 최선 O(n)

## 3. 시간/공간 복잡도
| 케이스 | 시간복잡도 |
|--------|-----------|
| 최선 (이미 정렬됨) | O(n) |
| 평균 | O(n²) |
| 최악 (역순 정렬) | O(n²) |
| 공간복잡도 | O(1) |

- stable 정렬 (같은 값의 순서 유지)

## 4. 기본 코드

### 기본형

#### Python
```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n - 1):
        for j in range(n - 1 - i):  # 뒤쪽은 이미 정렬됨
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
    return arr
```

#### C++
```cpp
void bubbleSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++)
        for (int j = 0; j < n - 1 - i; j++)   // 뒤쪽은 이미 정렬됨
            if (arr[j] > arr[j + 1])
                swap(arr[j], arr[j + 1]);
}
```

### 최적화 (조기 종료)

#### Python
```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n - 1):
        swapped = False
        for j in range(n - 1 - i):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True
        if not swapped:  # 스왑 없으면 이미 정렬됨
            break
    return arr
```

#### C++
```cpp
void bubbleSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {
        bool swapped = false;
        for (int j = 0; j < n - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr[j], arr[j + 1]);
                swapped = true;
            }
        }
        if (!swapped) break;   // 스왑 없으면 이미 정렬됨
    }
}
```

## 5. 이걸 떠올려야 할 때
- 코테에서 직접 구현할 일 없음 → Python `arr.sort()` / C++ `sort(arr.begin(), arr.end())` (`<algorithm>`)
- 정렬 알고리즘 비교 문제에서 특성 파악용으로 알아두기
- "stable 정렬이 필요하다" → Python 기본 정렬(Tim Sort) stable / C++ `stable_sort`

## 6. 자주 틀리는 포인트
- 안쪽 루프 범위를 `n - 1 - i`로 줄이지 않으면 불필요한 비교 발생
- 조기 종료 없이 구현하면 이미 정렬된 배열도 O(n²)
- 실제로 쓸 거라면 Python `arr.sort()`(Tim Sort) / C++ `sort`(Introsort) 가 압도적으로 빠름 (O(n log n))
