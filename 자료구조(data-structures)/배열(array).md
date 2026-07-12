# Array (배열)

> 한 줄 정리: 연속된 메모리에 데이터를 순서대로 저장하는 구조, Python은 list / C++은 vector가 배열 역할

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<algorithm>` (`sort`, `reverse`, `find`), `<utility>` (`pair`)

## 1. 언제 쓰는가
- 인덱스로 빠르게 접근해야 할 때
- 데이터를 순서대로 저장할 때
- 슬라이싱, 정렬이 필요할 때
- 2차원 배열로 그리드/행렬 표현할 때

## 2. 핵심 아이디어
- Python `list` / C++ `vector`는 dynamic array → 크기 자동 조절
- 인덱스로 O(1) 접근 가능
- 중간 삽입/삭제는 O(n) → 느림
- Python은 음수 인덱스(`arr[-1]`) 지원, C++은 `arr.back()`

## 3. 시간복잡도
| 연산 | 복잡도 |
|------|--------|
| 접근 (arr[i]) | O(1) |
| 탐색 | O(n) |
| 맨 뒤 삽입 (append / push_back) | O(1) |
| 중간 삽입 (insert) | O(n) |
| 맨 뒤 삭제 (pop / pop_back) | O(1) |
| 중간 삭제 | O(n) |

## 4. 기본 코드

#### Python
```python
arr = [1, 2, 3, 4, 5]

# 접근
arr[0]   # 1
arr[-1]  # 5

# 슬라이싱
arr[1:3]   # [2, 3]
arr[::-1]  # [5, 4, 3, 2, 1] 뒤집기

# 삽입 / 삭제
arr.append(6)       # 맨 뒤 추가 O(1)
arr.insert(2, 99)   # 중간 삽입 O(n)
arr.pop()           # 맨 뒤 삭제 O(1)
arr.pop(2)          # 중간 삭제 O(n)

# 탐색
if 3 in arr:        # O(n)
    print("있음")
arr.index(3)        # 인덱스 반환, 없으면 ValueError

# 정렬
arr.sort()              # 제자리 정렬
sorted_arr = sorted(arr)  # 새 리스트 반환
arr.sort(reverse=True)  # 내림차순
```

#### C++
```cpp
vector<int> arr = {1, 2, 3, 4, 5};

// 접근
arr[0];       // 1
arr.back();   // 5  (Python arr[-1] 대응)

// "슬라이싱"은 없음 → 반복자로 부분 벡터 생성
vector<int> sub(arr.begin() + 1, arr.begin() + 3);   // {2, 3}
reverse(arr.begin(), arr.end());                     // 뒤집기

// 삽입 / 삭제
arr.push_back(6);                  // 맨 뒤 추가 O(1)
arr.insert(arr.begin() + 2, 99);   // 중간 삽입 O(n)
arr.pop_back();                    // 맨 뒤 삭제 O(1)
arr.erase(arr.begin() + 2);        // 중간 삭제 O(n)

// 탐색 O(n)
if (find(arr.begin(), arr.end(), 3) != arr.end()) { /* 있음 */ }

// 정렬
sort(arr.begin(), arr.end());     // 오름차순
sort(arr.rbegin(), arr.rend());   // 내림차순 (역방향 반복자)
```

### 2차원 배열

#### Python
```python
# 초기화
N, M = 3, 4
grid = [[0] * M for _ in range(N)]  # 3x4 배열

# 주의: 아래처럼 하면 안 됨 (같은 리스트를 참조)
grid = [[0] * M] * N  # 절대 쓰지 말 것

# 접근
grid[1][2] = 5
print(grid[1][2])

# 순회
for i in range(N):
    for j in range(M):
        print(grid[i][j])
```

#### C++
```cpp
// 초기화 (참조 공유 문제 없음)
int N = 3, M = 4;
vector<vector<int>> grid(N, vector<int>(M, 0));   // 3x4, 0으로 초기화

// 접근
grid[1][2] = 5;

// 순회
for (int i = 0; i < N; i++)
    for (int j = 0; j < M; j++)
        cout << grid[i][j] << " ";
```

## 5. 실전 패턴

### 투 포인터

#### Python
```python
# 정렬된 배열에서 합이 target인 쌍 찾기
def two_sum(arr, target):
    left, right = 0, len(arr) - 1
    while left < right:
        s = arr[left] + arr[right]
        if s == target:
            return left, right
        elif s < target:
            left += 1
        else:
            right -= 1
    return -1, -1
```

#### C++
```cpp
// 정렬된 배열에서 합이 target인 쌍 찾기
pair<int,int> twoSum(vector<int>& arr, int target) {
    int left = 0, right = (int)arr.size() - 1;
    while (left < right) {
        int s = arr[left] + arr[right];
        if (s == target)      return {left, right};
        else if (s < target)  left++;
        else                  right--;
    }
    return {-1, -1};
}
```

### 슬라이딩 윈도우

#### Python
```python
# 길이 k인 구간의 최대 합
def max_sum(arr, k):
    window = sum(arr[:k])
    result = window
    for i in range(k, len(arr)):
        window += arr[i] - arr[i - k]
        result = max(result, window)
    return result
```

#### C++
```cpp
// 길이 k인 구간의 최대 합
long long maxSum(vector<int>& arr, int k) {
    long long window = 0;
    for (int i = 0; i < k; i++) window += arr[i];
    long long result = window;
    for (int i = k; i < (int)arr.size(); i++) {
        window += arr[i] - arr[i - k];
        result = max(result, window);
    }
    return result;
}
```

### 누적 합 (Prefix Sum)

#### Python
```python
# 구간 합을 O(1)에 구하기
arr = [1, 2, 3, 4, 5]
prefix = [0] * (len(arr) + 1)
for i in range(len(arr)):
    prefix[i + 1] = prefix[i] + arr[i]

# arr[l:r+1] 구간 합
def range_sum(l, r):
    return prefix[r + 1] - prefix[l]
```

#### C++
```cpp
// 구간 합을 O(1)에 구하기
vector<int> arr = {1, 2, 3, 4, 5};
vector<long long> prefix(arr.size() + 1, 0);
for (int i = 0; i < (int)arr.size(); i++)
    prefix[i + 1] = prefix[i] + arr[i];

// arr[l..r] 구간 합
long long rangeSum(int l, int r) {   // prefix가 전역이라고 가정
    return prefix[r + 1] - prefix[l];
}
```

## 6. 이걸 떠올려야 할 때
- "연속된 구간의 합/최솟값/최댓값" → 슬라이딩 윈도우 or 누적 합
- "정렬된 배열에서 두 값의 조건" → 투 포인터
- "2차원 격자/지도" → 2차원 배열
- "인덱스로 빠르게 접근" → 배열

## 7. 자주 틀리는 포인트
- Python `[[0]*M]*N`은 행이 같은 객체 참조 → 리스트 컴프리헨션 사용 / C++ `vector<vector<int>>(N, vector<int>(M))`는 안전
- Python `arr.pop(0)`·C++ `vector` 앞 삽입/삭제는 O(n) → 앞에서 자주 꺼내면 `deque`
- 슬라이싱/부분 벡터는 새 배열 반환 → 원본 안 바뀜
- 선형 탐색(`in` / `find`)은 O(n) → 자주 탐색하면 `set`/`unordered_set`
