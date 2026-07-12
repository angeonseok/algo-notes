# Counting Sort (카운팅 정렬)

> 한 줄 정리: 값의 등장 횟수를 세서 정렬, 값의 범위가 작을 때 O(n)으로 가장 빠르다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<algorithm>` (`max_element`, `min_element`)

## 1. 언제 쓰는가
- 값의 범위(max - min)가 작을 때
- 정수 데이터 정렬
- 점수, 나이, 등급 같이 범위가 제한된 데이터
- O(n) 정렬이 필요할 때

## 2. 핵심 아이디어
- 각 값이 몇 번 등장했는지 카운트 배열에 저장
- 카운트 배열을 순서대로 읽어서 정렬 결과 생성
- 비교 기반 정렬이 아니라서 O(n + k) 가능 (k = 값의 범위)
- k가 n보다 훨씬 크면 오히려 비효율적

## 3. 시간/공간 복잡도
| 케이스 | 시간복잡도 |
|--------|-----------|
| 최선 | O(n + k) |
| 평균 | O(n + k) |
| 최악 | O(n + k) |
| 공간복잡도 | O(k) |

- k = 값의 범위 (max - min + 1)
- stable 정렬 (누적 합 방식 사용 시)

## 4. 기본 코드

### 기본형

#### Python
```python
def counting_sort(arr):
    if not arr:
        return arr

    max_val = max(arr)
    min_val = min(arr)
    count = [0] * (max_val - min_val + 1)

    # 등장 횟수 카운트
    for v in arr:
        count[v - min_val] += 1

    # 카운트 순서대로 결과 생성
    result = []
    for i, c in enumerate(count):
        result.extend([i + min_val] * c)

    return result
```

#### C++
```cpp
vector<int> countingSort(vector<int>& arr) {
    if (arr.empty()) return arr;

    int max_val = *max_element(arr.begin(), arr.end());
    int min_val = *min_element(arr.begin(), arr.end());
    vector<int> count(max_val - min_val + 1, 0);

    // 등장 횟수 카운트
    for (int v : arr) count[v - min_val]++;

    // 카운트 순서대로 결과 생성
    vector<int> result;
    for (int i = 0; i < (int)count.size(); i++)
        for (int c = 0; c < count[i]; c++)
            result.push_back(i + min_val);
    return result;
}
```

### 누적 합 방식 (stable 보장)

#### Python
```python
def counting_sort_stable(arr):
    if not arr:
        return arr

    max_val = max(arr)
    min_val = min(arr)
    count = [0] * (max_val - min_val + 1)

    for v in arr:
        count[v - min_val] += 1

    # 누적 합
    for i in range(1, len(count)):
        count[i] += count[i - 1]

    # 뒤에서부터 채워서 stable 유지
    result = [0] * len(arr)
    for v in reversed(arr):
        count[v - min_val] -= 1
        result[count[v - min_val]] = v

    return result
```

#### C++
```cpp
vector<int> countingSortStable(vector<int>& arr) {
    if (arr.empty()) return arr;

    int max_val = *max_element(arr.begin(), arr.end());
    int min_val = *min_element(arr.begin(), arr.end());
    vector<int> count(max_val - min_val + 1, 0);

    for (int v : arr) count[v - min_val]++;

    // 누적 합
    for (int i = 1; i < (int)count.size(); i++)
        count[i] += count[i - 1];

    // 뒤에서부터 채워서 stable 유지
    vector<int> result(arr.size());
    for (int i = (int)arr.size() - 1; i >= 0; i--) {
        int v = arr[i];
        count[v - min_val]--;
        result[count[v - min_val]] = v;
    }
    return result;
}
```

## 5. 이걸 떠올려야 할 때
- "값의 범위가 작다" (0~100만 이하) → 카운팅 정렬
- "O(n)으로 정렬해야 한다" → 카운팅 정렬
- 반대로 값의 범위가 크거나 실수면 → 사용 불가

## 6. 자주 틀리는 포인트
- 음수 값이 있을 때 `min_val`로 오프셋 처리 안 하면 인덱스 에러
- k가 너무 크면 (ex. 값 범위가 10억) 메모리 초과 → 이때는 `arr.sort()` / `sort(...)` 쓰기
- 문자열/실수에는 사용 불가, 정수 전용
