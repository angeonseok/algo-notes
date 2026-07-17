# Insertion Sort (삽입 정렬)

> 한 줄 정리: 카드 정렬하듯 하나씩 적절한 위치에 끼워넣는 정렬, 거의 정렬된 배열에서 강하다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<algorithm>` (`lower_bound`)

## 1. 언제 쓰는가
- 거의 정렬된 배열에서 매우 빠름 (최선 O(n))
- 소규모 데이터 정렬
- Tim Sort(Python 기본 정렬)의 내부에서 부분적으로 사용됨

## 2. 핵심 아이디어
- 두 번째 원소부터 시작해 앞쪽 정렬된 구간에 끼워넣기
- 현재 원소보다 큰 값들을 한 칸씩 오른쪽으로 밀고 빈 자리에 삽입
- 앞쪽은 항상 정렬된 상태 유지

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
#카드 정렬하듯 앞쪽 정렬된 구간에 하나씩 끼워넣기
def insertion_sort(arr):
    #0번은 이미 정렬된 셈이니까 1번부터 시작
    for i in range(1, len(arr)):
        key = arr[i]
        j = i - 1

        #key보다 큰 놈들은 한 칸씩 오른쪽으로 밀기
        while j >= 0 and arr[j] > key:
            arr[j + 1] = arr[j]
            j -= 1

        #while이 멈춘 자리가 key 들어갈 자리
        arr[j + 1] = key
    return arr
```

#### C++
```cpp
//카드 정렬하듯 앞쪽 정렬된 구간에 하나씩 끼워넣기
void insertionSort(vector<int>& arr) {
    //0번은 이미 정렬된 셈이니까 1번부터 시작
    for (int i = 1; i < (int)arr.size(); i++) {
        int key = arr[i];
        int j = i - 1;

        //key보다 큰 놈들은 한 칸씩 오른쪽으로 밀기
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }

        //while이 멈춘 자리가 key 들어갈 자리
        arr[j + 1] = key;
    }
}
```

### 이분 탐색 활용 (탐색만 O(log n), 전체는 여전히 O(n²))

#### Python
```python
import bisect

#끼워넣을 자리를 이분 탐색으로 찾기
def insertion_sort_bisect(arr):
    result = []

    #result는 항상 정렬된 상태니까 bisect 쓸 수 있음
    for v in arr:
        idx = bisect.bisect_left(result, v)

        #자리는 O(log n)에 찾아도 insert 자체가 O(n)이라 전체는 그대로 O(n²)
        result.insert(idx, v)
    return result
```

#### C++
```cpp
//끼워넣을 자리를 이분 탐색으로 찾기
vector<int> insertionSortBisect(vector<int>& arr) {
    vector<int> result;

    //result는 항상 정렬된 상태니까 lower_bound 쓸 수 있음
    for (int v : arr) {
        auto it = lower_bound(result.begin(), result.end(), v);

        //자리는 O(log n)에 찾아도 insert 자체가 O(n)이라 전체는 그대로 O(n²)
        result.insert(it, v);
    }
    return result;
}
```

## 5. 이걸 떠올려야 할 때
- "거의 정렬된 배열" → 삽입 정렬이 실질적으로 빠름
- 실시간으로 데이터가 들어오면서 정렬 상태 유지할 때
- 코테에서 직접 쓸 일은 적지만 이분 탐색(bisect / lower_bound)과 연계해서 알아두면 유용

## 6. 자주 틀리는 포인트
- `arr[j + 1] = key` 위치 헷갈리는 경우 → while 끝난 후 j+1이 삽입 위치
- 위치를 이분 탐색으로 찾아도 삽입(`list.insert` / `vector::insert`)이 O(n)이라 전체 복잡도는 그대로
