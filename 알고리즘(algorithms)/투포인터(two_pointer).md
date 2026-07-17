# Two Pointer (투 포인터)

> 한 줄 정리: 두 개의 포인터를 이용해 배열을 O(N)에 탐색하는 방법

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<algorithm>` (`sort`, `max`), `<string>`, `<unordered_map>`

## 1. 언제 쓰는가
- 정렬된 배열에서 두 값의 합/차 조건
- 연속된 구간의 합이 특정 값인 경우
- 중복 없는 최장 부분 문자열
- 슬라이딩 윈도우와 함께 사용

## 2. 핵심 아이디어
- 두 포인터가 같은 방향 or 반대 방향으로 이동
- 조건에 따라 포인터를 한 칸씩 움직임
- O(N²)을 O(N)으로 줄이는 게 핵심

## 3. 시간/공간 복잡도
| | 복잡도 |
|--|--------|
| 시간 | O(N) ~ O(N log N) (정렬 포함) |
| 공간 | O(1) |

## 4. 기본 코드

### 반대 방향 (정렬된 배열에서 합)

#### Python
```python
#정렬해놓고 양끝에서 좁혀오면서 합이 target인 쌍 찾기
def two_sum(arr, target):
    #정렬돼 있어야 합이 커질지 작아질지 판단 가능
    arr.sort()
    left, right = 0, len(arr) - 1

    while left < right:
        s = arr[left] + arr[right]

        #합이 작으면 왼쪽을 땡겨서 키우고, 크면 오른쪽을 줄이자
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
//정렬해놓고 양끝에서 좁혀오면서 합이 target인 쌍 찾기
pair<int,int> twoSum(vector<int>& arr, int target) {
    //정렬돼 있어야 합이 커질지 작아질지 판단 가능
    sort(arr.begin(), arr.end());
    int left = 0, right = (int)arr.size() - 1;

    while (left < right) {
        int s = arr[left] + arr[right];

        //합이 작으면 왼쪽을 땡겨서 키우고, 크면 오른쪽을 줄이자
        if (s == target)     return {left, right};
        else if (s < target) left++;
        else                 right--;
    }
    return {-1, -1};
}
```

### 같은 방향 (연속 구간 합)

#### Python
```python
#합이 딱 target인 연속 구간이 몇개인지 세기
def count_subarrays(arr, target):
    left = 0
    curr_sum = 0
    cnt = 0

    #right로 창을 늘려나가자
    for right in range(len(arr)):
        curr_sum += arr[right]

        #넘치면 left를 땡겨서 창을 줄이자
        while curr_sum > target and left <= right:
            curr_sum -= arr[left]
            left += 1

        #딱 맞으면 카운트
        if curr_sum == target:
            cnt += 1

    return cnt
```

#### C++
```cpp
//합이 딱 target인 연속 구간이 몇개인지 세기
int countSubarrays(vector<int>& arr, int target) {
    int left = 0, curr_sum = 0, cnt = 0;

    //right로 창을 늘려나가자
    for (int right = 0; right < (int)arr.size(); right++) {
        curr_sum += arr[right];

        //넘치면 left를 땡겨서 창을 줄이자
        while (curr_sum > target && left <= right) {
            curr_sum -= arr[left];
            left++;
        }

        //딱 맞으면 카운트
        if (curr_sum == target) cnt++;
    }
    return cnt;
}
```

### 슬라이딩 윈도우 (고정 크기)

#### Python
```python
#크기 k 구간 중에 합이 제일 큰 놈 찾기
def max_sum_window(arr, k):
    #첫 창은 직접 더해놓고 시작
    window = sum(arr[:k])
    result = window

    #매번 다시 더하지 말고 들어온 놈 더하고 나간 놈 빼기
    for i in range(k, len(arr)):
        window += arr[i] - arr[i - k]
        result = max(result, window)

    return result
```

#### C++
```cpp
//크기 k 구간 중에 합이 제일 큰 놈 찾기
long long maxSumWindow(vector<int>& arr, int k) {
    //첫 창은 직접 더해놓고 시작
    long long window = 0;
    for (int i = 0; i < k; i++) window += arr[i];
    long long result = window;

    //매번 다시 더하지 말고 들어온 놈 더하고 나간 놈 빼기
    for (int i = k; i < (int)arr.size(); i++) {
        window += arr[i] - arr[i - k];
        result = max(result, window);
    }
    return result;
}
```

### 중복 없는 최장 부분 문자열

#### Python
```python
#중복 문자 없는 제일 긴 구간 길이 구하기
def longest_unique(s):
    left = 0

    #문자마다 마지막으로 본 인덱스 기록
    seen = {}
    result = 0

    for right, c in enumerate(s):
        #창 안에서 또 나온 문자면 그 다음칸으로 left 점프
        if c in seen and seen[c] >= left:
            left = seen[c] + 1

        seen[c] = right
        result = max(result, right - left + 1)

    return result
```

#### C++
```cpp
//중복 문자 없는 제일 긴 구간 길이 구하기
int longestUnique(const string& s) {
    int left = 0, result = 0;

    //문자마다 마지막으로 본 인덱스 기록
    unordered_map<char,int> seen;

    for (int right = 0; right < (int)s.size(); right++) {
        char c = s[right];

        //창 안에서 또 나온 문자면 그 다음칸으로 left 점프
        if (seen.count(c) && seen[c] >= left)
            left = seen[c] + 1;

        seen[c] = right;
        result = max(result, right - left + 1);
    }
    return result;
}
```

## 5. 이걸 떠올려야 할 때
- "정렬된 배열 + 두 값의 합/차 조건" → 반대 방향 투 포인터
- "연속 구간의 합" → 같은 방향 투 포인터
- "고정 크기 구간의 최솟값/최댓값" → 슬라이딩 윈도우
- O(N²) 이중 반복문이 시간초과 → 투 포인터 고려

## 6. 자주 틀리는 포인트
- 반대 방향에서 `left < right` 조건 빠뜨리면 같은 원소 중복 사용
- 같은 방향에서 음수가 있으면 투 포인터 불가 → 누적 합 고려
- 슬라이딩 윈도우에서 윈도우 초기값 설정 빠뜨리는 경우
