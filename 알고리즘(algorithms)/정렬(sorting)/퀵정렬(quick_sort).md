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
#피벗 기준으로 작은 놈/큰 놈 갈라서 재귀
def quick_sort(arr):
    #하나 남으면 더 쪼갤 게 없으니까 그대로 반환
    if len(arr) <= 1:
        return arr

    #중앙값을 피벗으로 잡기
    pivot = arr[len(arr) // 2]

    #피벗 기준으로 작은 놈/같은 놈/큰 놈 세 덩이로 분리
    left   = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right  = [x for x in arr if x > pivot]

    #피벗과 같은 놈들은 이미 제자리니까 양쪽만 정렬해서 이어붙이기
    return quick_sort(left) + middle + quick_sort(right)
```

#### C++
```cpp
//피벗 기준으로 작은 놈/큰 놈 갈라서 재귀
vector<int> quickSort(vector<int>& arr) {
    //하나 남으면 더 쪼갤 게 없으니까 그대로 반환
    if (arr.size() <= 1) return arr;

    //중앙값을 피벗으로 잡기
    int pivot = arr[arr.size() / 2];

    //피벗 기준으로 작은 놈/같은 놈/큰 놈 세 덩이로 분리
    vector<int> left, mid, right;
    for (int x : arr) {
        if (x < pivot)      left.push_back(x);
        else if (x == pivot) mid.push_back(x);
        else                right.push_back(x);
    }

    //피벗과 같은 놈들은 이미 제자리니까 양쪽만 정렬
    left = quickSort(left);
    right = quickSort(right);

    //left 뒤에 mid, right 순서로 이어붙이기
    left.insert(left.end(), mid.begin(), mid.end());
    left.insert(left.end(), right.begin(), right.end());
    return left;
}
```

### 제자리 정렬 (in-place, 추가 메모리 최소화)

#### Python
```python
import random

#새 리스트 안 만들고 arr 안에서 직접 자리 바꾸기
def quick_sort(arr, l, h):
    #구간에 원소가 하나 이하면 볼 것도 없음
    if l >= h:
        return

    #피벗 자리는 확정됐으니까 걔 빼고 양옆만 재귀
    pivot_idx = partition(arr, l, h)
    quick_sort(arr, l, pivot_idx - 1)
    quick_sort(arr, pivot_idx + 1, h)

#피벗보다 작은 놈은 왼쪽에 몰고 피벗의 최종 자리 반환
def partition(arr, l, h):
    #피벗 고정으로 잡으면 이미 정렬된 입력에서 O(n²) 터짐. 랜덤으로 뽑아서 맨 뒤로 보내자
    rand_idx = random.randint(l, h)
    arr[rand_idx], arr[h] = arr[h], arr[rand_idx]

    #i는 작은 놈들이 쌓인 구간의 끝
    pivot = arr[h]
    i = l - 1

    #피벗 이하인 놈 만나면 i를 한 칸 늘려서 그 자리로 스왑
    for j in range(l, h):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]

    #작은 놈들 바로 뒤가 피벗 자리
    arr[i + 1], arr[h] = arr[h], arr[i + 1]
    return i + 1

#사용
arr = [3, 1, 4, 1, 5, 9, 2, 6]
quick_sort(arr, 0, len(arr) - 1)
```

#### C++
```cpp
//피벗보다 작은 놈은 왼쪽에 몰고 피벗의 최종 자리 반환
//std::partition이랑 이름 충돌 피하려고 partition_로 명명
int partition_(vector<int>& arr, int l, int h) {
    //피벗 고정으로 잡으면 이미 정렬된 입력에서 O(n²) 터짐. 랜덤으로 뽑아서 맨 뒤로 보내자
    int rand_idx = l + rand() % (h - l + 1);
    swap(arr[rand_idx], arr[h]);

    //i는 작은 놈들이 쌓인 구간의 끝
    int pivot = arr[h];
    int i = l - 1;

    //피벗 이하인 놈 만나면 i를 한 칸 늘려서 그 자리로 스왑
    for (int j = l; j < h; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }

    //작은 놈들 바로 뒤가 피벗 자리
    swap(arr[i + 1], arr[h]);
    return i + 1;
}

//새 벡터 안 만들고 arr 안에서 직접 자리 바꾸기
void quickSort(vector<int>& arr, int l, int h) {
    //구간에 원소가 하나 이하면 볼 것도 없음
    if (l >= h) return;

    //피벗 자리는 확정됐으니까 걔 빼고 양옆만 재귀
    int p = partition_(arr, l, h);
    quickSort(arr, l, p - 1);
    quickSort(arr, p + 1, h);
}

//사용
//vector<int> arr = {3, 1, 4, 1, 5, 9, 2, 6};
//quickSort(arr, 0, (int)arr.size() - 1);
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
