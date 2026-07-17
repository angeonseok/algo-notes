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

#접근은 O(1). arr[0]은 1, arr[-1]은 5(파이썬은 음수 인덱스 됨)
arr[0]
arr[-1]

#슬라이싱. arr[1:3]은 [2, 3], arr[::-1]은 [5, 4, 3, 2, 1]로 뒤집기
arr[1:3]
arr[::-1]

#맨 뒤는 O(1). 중간은 뒤를 다 밀어야 되니까 O(n)
arr.append(6)
arr.insert(2, 99)
arr.pop()
arr.pop(2)

#탐색은 O(n). index는 없으면 ValueError 뱉으니까 주의
if 3 in arr:
    print("있음")
arr.index(3)

#sort는 제자리, sorted는 새 리스트 반환. reverse=True면 내림차순
arr.sort()
sorted_arr = sorted(arr)
arr.sort(reverse=True)
```

#### C++
```cpp
vector<int> arr = {1, 2, 3, 4, 5};

//접근은 O(1). arr[0]은 1, arr.back()은 5(Python arr[-1] 대응)
arr[0];
arr.back();

//"슬라이싱"은 없으니까 반복자로 부분 벡터 뜨기 → {2, 3}
vector<int> sub(arr.begin() + 1, arr.begin() + 3);

//뒤집기
reverse(arr.begin(), arr.end());

//맨 뒤는 O(1). 중간은 뒤를 다 밀어야 되니까 O(n)
arr.push_back(6);
arr.insert(arr.begin() + 2, 99);
arr.pop_back();
arr.erase(arr.begin() + 2);

//탐색은 O(n). 못 찾으면 end() 나옴
if (find(arr.begin(), arr.end(), 3) != arr.end()) { /* 있음 */ }

//오름차순. 내림차순은 역방향 반복자로 걍 뒤집어서 넣기
sort(arr.begin(), arr.end());
sort(arr.rbegin(), arr.rend());
```

### 2차원 배열

#### Python
```python
#3x4 배열 만들기
N, M = 3, 4
mat = [[0] * M for _ in range(N)]

#이렇게 하면 행들이 죄다 같은 리스트를 참조함. 절대 쓰지 말자
mat = [[0] * M] * N

#접근
mat[1][2] = 5
print(mat[1][2])

#순회
for i in range(N):
    for j in range(M):
        print(mat[i][j])
```

#### C++
```cpp
//3x4, 0으로 초기화. C++은 참조 공유 문제 없음
int N = 3, M = 4;
vector<vector<int>> mat(N, vector<int>(M, 0));

//접근
mat[1][2] = 5;

//순회
for (int i = 0; i < N; i++)
    for (int j = 0; j < M; j++)
        cout << mat[i][j] << " ";
```

## 5. 실전 패턴

### 투 포인터

#### Python
```python
#정렬된 배열에서 합이 target인 쌍 찾기
def two_sum(arr, target):
    #양 끝에서 시작해서 좁혀오자
    left, right = 0, len(arr) - 1

    while left < right:
        s = arr[left] + arr[right]

        #딱 맞으면 끝. 합이 작으면 왼쪽을 당기고, 크면 오른쪽을 당기자
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
//정렬된 배열에서 합이 target인 쌍 찾기
pair<int,int> twoSum(vector<int>& arr, int target) {
    //양 끝에서 시작해서 좁혀오자
    int left = 0, right = (int)arr.size() - 1;

    while (left < right) {
        int s = arr[left] + arr[right];

        //딱 맞으면 끝. 합이 작으면 왼쪽을 당기고, 크면 오른쪽을 당기자
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
#길이 k인 구간의 최대 합
def max_sum(arr, k):
    #첫 윈도우는 걍 다 더해서 시작
    window = sum(arr[:k])
    result = window

    #한 칸 밀 때마다 들어온 놈 더하고 빠진 놈 빼면 O(1)
    for i in range(k, len(arr)):
        window += arr[i] - arr[i - k]
        result = max(result, window)
    return result
```

#### C++
```cpp
//길이 k인 구간의 최대 합
long long maxSum(vector<int>& arr, int k) {
    //첫 윈도우는 걍 다 더해서 시작
    long long window = 0;
    for (int i = 0; i < k; i++) window += arr[i];
    long long result = window;

    //한 칸 밀 때마다 들어온 놈 더하고 빠진 놈 빼면 O(1)
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
#구간 합을 O(1)에 구하기
arr = [1, 2, 3, 4, 5]

#prefix[i]는 arr[0 ~ i-1]까지의 합. 한 칸 밀어서 잡으면 경계 처리가 편함
prefix = [0] * (len(arr) + 1)
for i in range(len(arr)):
    prefix[i + 1] = prefix[i] + arr[i]

#arr[l:r+1] 구간 합은 걍 빼기 한 번
def range_sum(l, r):
    return prefix[r + 1] - prefix[l]
```

#### C++
```cpp
//구간 합을 O(1)에 구하기
vector<int> arr = {1, 2, 3, 4, 5};

//prefix[i]는 arr[0 ~ i-1]까지의 합. 한 칸 밀어서 잡으면 경계 처리가 편함
vector<long long> prefix(arr.size() + 1, 0);
for (int i = 0; i < (int)arr.size(); i++)
    prefix[i + 1] = prefix[i] + arr[i];

//arr[l..r] 구간 합은 걍 빼기 한 번. prefix는 전역이라고 가정
long long rangeSum(int l, int r) {
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
