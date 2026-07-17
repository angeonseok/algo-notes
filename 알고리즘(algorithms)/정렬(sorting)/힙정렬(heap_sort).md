# Heap Sort (힙 정렬)

> 한 줄 정리: 힙 자료구조를 이용한 정렬, 항상 O(n log n)이고 추가 메모리가 필요 없다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<utility>` (`swap`), `<queue>` (`priority_queue`), `<functional>` (`greater`)

## 1. 언제 쓰는가
- 추가 메모리 없이 O(n log n) 보장이 필요할 때
- 코테에서 직접 쓸 일은 거의 없지만 힙 개념과 연결해서 이해
- "K개의 최솟값/최댓값만 필요" → heapq / priority_queue로 대체

## 2. 핵심 아이디어
- 전체를 최대 힙으로 만들기 (heapify)
- 루트(최댓값)를 맨 뒤로 보내고 힙 크기 줄이기
- 반복하면 오름차순 정렬 완성
- Python `heapq`, C++ `priority_queue`(기본 최대 힙)의 방향 차이에 주의

## 3. 시간/공간 복잡도
| 케이스 | 시간복잡도 |
|--------|-----------|
| 최선 | O(n log n) |
| 평균 | O(n log n) |
| 최악 | O(n log n) |
| 공간복잡도 | O(1) |

- unstable 정렬 (같은 값의 순서가 바뀔 수 있음)

## 4. 기본 코드

### 기본형

#### Python
```python
#최대 힙 만들어놓고 루트를 하나씩 빼면서 정렬
def heap_sort(arr):
    n = len(arr)

    #리프는 자식이 없어서 heapify 필요 없음. 마지막 내부 노드부터 거꾸로 올라가자
    for i in range(n // 2 - 1, -1, -1):
        heapify(arr, n, i)

    #루트가 항상 최댓값이니까 맨 뒤로 보내고 힙 크기를 줄이기
    for i in range(n - 1, 0, -1):
        arr[0], arr[i] = arr[i], arr[0]

        #루트가 딴 놈으로 바뀌었으니까 확정된 뒷부분 뺀 i개만 다시 정리
        heapify(arr, i, 0)

    return arr

#i번 노드를 자식들과 비교해서 최대 힙 조건 맞추기
def heapify(arr, n, i):
    #일단 자기 자신이 제일 크다고 치고 시작
    largest = i
    left  = 2 * i + 1
    right = 2 * i + 2

    #범위 안에 있는 자식 중에 더 큰 놈 있나 찾기
    if left < n and arr[left] > arr[largest]:
        largest = left
    if right < n and arr[right] > arr[largest]:
        largest = right

    #자식이 더 크면 바꿔주고, 내려간 놈 밑도 깨졌을 수 있으니까 재귀
    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest)
```

#### C++
```cpp
//i번 노드를 자식들과 비교해서 최대 힙 조건 맞추기
void heapify(vector<int>& arr, int n, int i) {
    //일단 자기 자신이 제일 크다고 치고 시작
    int largest = i;
    int left  = 2 * i + 1;
    int right = 2 * i + 2;

    //범위 안에 있는 자식 중에 더 큰 놈 있나 찾기
    if (left  < n && arr[left]  > arr[largest]) largest = left;
    if (right < n && arr[right] > arr[largest]) largest = right;

    //자식이 더 크면 바꿔주고, 내려간 놈 밑도 깨졌을 수 있으니까 재귀
    if (largest != i) {
        swap(arr[i], arr[largest]);
        heapify(arr, n, largest);
    }
}

//최대 힙 만들어놓고 루트를 하나씩 빼면서 정렬
void heapSort(vector<int>& arr) {
    int n = arr.size();

    //리프는 자식이 없어서 heapify 필요 없음. 마지막 내부 노드부터 거꾸로 올라가자
    for (int i = n / 2 - 1; i >= 0; i--)
        heapify(arr, n, i);

    //루트가 항상 최댓값이니까 맨 뒤로 보내고 힙 크기를 줄이기
    for (int i = n - 1; i > 0; i--) {
        swap(arr[0], arr[i]);

        //루트가 딴 놈으로 바뀌었으니까 확정된 뒷부분 뺀 i개만 다시 정리
        heapify(arr, i, 0);
    }
}
```

### 코테에서 실제로 쓰는 방식 (heapq / priority_queue)

#### Python
```python
import heapq

def heap_sort(arr):
    heapq.heapify(arr)
    return [heapq.heappop(arr) for _ in range(len(arr))]
```

#### C++
```cpp
//최소 힙으로 하나씩 꺼내면 걍 오름차순
vector<int> heapSortSTL(vector<int> arr) {
    //priority_queue는 기본이 최대 힙이라 greater 박아서 최소 힙으로 뒤집기
    priority_queue<int, vector<int>, greater<int>> pq(arr.begin(), arr.end());

    //작은 놈부터 나오니까 꺼내는 순서가 곧 정답
    vector<int> result;
    while (!pq.empty()) {
        result.push_back(pq.top());
        pq.pop();
    }
    return result;
}

//또는 <algorithm>의 make_heap + sort_heap 쓰면 제자리 정렬
//최대 힙 구성
//make_heap(arr.begin(), arr.end());

//오름차순 정렬
//sort_heap(arr.begin(), arr.end());
```

## 5. 이걸 떠올려야 할 때
- 코테에서 직접 구현할 일 없음 → Python `arr.sort()` / C++ `sort(...)`
- 힙 정렬보다 heapq / priority_queue로 K개 최솟값 구하는 패턴이 더 자주 등장
- 병합/퀵 정렬과 비교: 추가 메모리 없이 O(n log n) 보장이 필요할 때

## 6. 자주 틀리는 포인트
- `heapify` 시작을 `n // 2 - 1`부터 하는 이유 → 리프 노드는 자식이 없어서 heapify 불필요
- Python `heapq`는 최소 힙 → 최대 힙은 값에 `-` 붙이기 / C++ `priority_queue`는 기본 최대 힙 → 최소 힙은 `greater<int>`
- unstable 정렬이라 같은 값의 순서 보장 안 됨
