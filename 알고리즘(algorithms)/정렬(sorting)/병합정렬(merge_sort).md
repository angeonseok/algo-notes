# Merge Sort (병합 정렬)

> 한 줄 정리: 반으로 나누고 정렬하면서 합치는 정렬, 항상 O(n log n)이 보장된다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`

## 1. 언제 쓰는가
- 안정적인 O(n log n)이 필요할 때
- 연결 리스트 정렬 (퀵 정렬보다 유리)
- 외부 정렬 (데이터가 메모리에 다 안 들어올 때)
- 역전 쌍(inversion count) 문제

## 2. 핵심 아이디어
- 분할(Divide): 배열을 반으로 계속 나눔
- 정복(Conquer): 각 부분을 재귀적으로 정렬
- 병합(Merge): 정렬된 두 배열을 합침
- 항상 O(n log n) 보장, 단 추가 메모리 O(n) 필요

## 3. 시간/공간 복잡도
| 케이스 | 시간복잡도 |
|--------|-----------|
| 최선 | O(n log n) |
| 평균 | O(n log n) |
| 최악 | O(n log n) |
| 공간복잡도 | O(n) |

- stable 정렬 (같은 값의 순서 유지)

## 4. 기본 코드

### 기본형

#### Python
```python
#반으로 쪼개서 각각 정렬하고 다시 합치기
def merge_sort(arr):
    #하나 남으면 더 쪼갤 게 없으니까 그대로 반환
    if len(arr) <= 1:
        return arr

    #반 갈라서 각각 재귀로 정렬해두자
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])

    #둘 다 정렬됐으니까 합치기만 하면 끝
    return merge(left, right)

#정렬된 두 놈을 하나로 합치기
def merge(left, right):
    result = []
    i = j = 0

    #양쪽 앞에서부터 작은 놈 골라 담기. 같으면 왼쪽 먼저라 stable 유지
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1

    #한쪽 먼저 비면 남은 놈들 걍 붙이기
    result.extend(left[i:])
    result.extend(right[j:])
    return result
```

#### C++
```cpp
//std::merge랑 이름 충돌 피하려고 mergeHalves로 명명. 전방 선언 박기
vector<int> mergeHalves(vector<int>& left, vector<int>& right);

//반으로 쪼개서 각각 정렬하고 다시 합치기
vector<int> mergeSort(vector<int>& arr) {
    //하나 남으면 더 쪼갤 게 없으니까 그대로 반환
    if (arr.size() <= 1) return arr;

    //반 갈라서 각각 재귀로 정렬해두자
    int mid = arr.size() / 2;
    vector<int> left(arr.begin(), arr.begin() + mid);
    vector<int> right(arr.begin() + mid, arr.end());
    left = mergeSort(left);
    right = mergeSort(right);

    //둘 다 정렬됐으니까 합치기만 하면 끝
    return mergeHalves(left, right);
}

//정렬된 두 놈을 하나로 합치기
vector<int> mergeHalves(vector<int>& left, vector<int>& right) {
    vector<int> result;
    int i = 0, j = 0;

    //양쪽 앞에서부터 작은 놈 골라 담기. 같으면 왼쪽 먼저라 stable 유지
    while (i < (int)left.size() && j < (int)right.size()) {
        if (left[i] <= right[j]) result.push_back(left[i++]);
        else                     result.push_back(right[j++]);
    }

    //한쪽 먼저 비면 남은 놈들 걍 붙이기
    while (i < (int)left.size())  result.push_back(left[i++]);
    while (j < (int)right.size()) result.push_back(right[j++]);
    return result;
}
```

### 역전 쌍 카운트 (inversion count)

#### Python
```python
count = 0

#합치면서 역전 쌍까지 같이 세기
def merge_count(arr):
    global count

    #하나 남으면 더 쪼갤 게 없으니까 그대로 반환
    if len(arr) <= 1:
        return arr

    #반 갈라서 각각 재귀로 정렬해두자
    mid = len(arr) // 2
    left = merge_count(arr[:mid])
    right = merge_count(arr[mid:])

    result = []
    i = j = 0

    #양쪽 앞에서부터 작은 놈 골라 담기
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1

            #오른쪽 놈이 먼저 나왔으니까 left에 남은 놈들은 전부 역전 쌍
            count += len(left) - i

    #한쪽 먼저 비면 남은 놈들 걍 붙이기
    result.extend(left[i:])
    result.extend(right[j:])
    return result
```

#### C++
```cpp
//역전 쌍 개수는 최대 n(n-1)/2라 int로는 터짐
long long invCount = 0;

//합치면서 역전 쌍까지 같이 세기
vector<int> mergeCount(vector<int>& arr) {
    //하나 남으면 더 쪼갤 게 없으니까 그대로 반환
    if (arr.size() <= 1) return arr;

    //반 갈라서 각각 재귀로 정렬해두자
    int mid = arr.size() / 2;
    vector<int> left(arr.begin(), arr.begin() + mid);
    vector<int> right(arr.begin() + mid, arr.end());
    left = mergeCount(left);
    right = mergeCount(right);

    vector<int> result;
    int i = 0, j = 0;

    //양쪽 앞에서부터 작은 놈 골라 담기
    while (i < (int)left.size() && j < (int)right.size()) {
        if (left[i] <= right[j]) {
            result.push_back(left[i++]);
        } else {
            result.push_back(right[j++]);

            //오른쪽 놈이 먼저 나왔으니까 left에 남은 놈들은 전부 역전 쌍
            invCount += (long long)left.size() - i;
        }
    }

    //한쪽 먼저 비면 남은 놈들 걍 붙이기
    while (i < (int)left.size())  result.push_back(left[i++]);
    while (j < (int)right.size()) result.push_back(right[j++]);
    return result;
}
```

## 5. 이걸 떠올려야 할 때
- "최악도 O(n log n) 보장" → 병합 정렬 (C++ `stable_sort`도 동일 특성)
- "역전 쌍 개수 세기" → 병합 정렬 응용
- "stable 정렬 + O(n log n)" → 병합 정렬 (퀵 정렬은 unstable)
- 코테에서 단순 정렬은 `arr.sort()` / `sort(...)` 쓰되, 역전 쌍 문제에서 직접 구현

## 6. 자주 틀리는 포인트
- 슬라이싱(`arr[:mid]`)·부분 벡터 복사가 새 배열을 만들어 메모리 O(n) 추가 사용
- 재귀 깊이가 깊어지면 Python 스택 오버플로 가능 → `sys.setrecursionlimit` 설정
- 역전 쌍 카운트 시 `count += len(left) - i` 위치 헷갈리지 않기
- C++ 역전 쌍은 개수가 커서 `int` 오버플로 → `long long` 사용
