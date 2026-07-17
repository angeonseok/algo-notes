# Deque (덱)

> 한 줄 정리: 앞뒤 양쪽에서 삽입/삭제가 모두 가능한 구조

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<deque>`, `<vector>`, `<string>`, `<algorithm>` (`rotate`)

## 1. 언제 쓰는가
- 앞뒤 양쪽에서 삽입/삭제가 필요할 때
- 슬라이딩 윈도우 최솟값/최댓값 문제
- 팰린드롬 검사
- 스택과 큐를 동시에 써야 할 때
- 앞에서도 빠르게 꺼내야 할 때 (list 대체)

## 2. 핵심 아이디어
- 양방향으로 열려있는 큐
- appendleft / popleft → 앞쪽 O(1) (C++ `push_front` / `pop_front`)
- append / pop → 뒤쪽 O(1) (C++ `push_back` / `pop_back`)
- Python `collections.deque`, C++ `deque<T>`

## 3. 시간복잡도
- append / appendleft: O(1)
- pop / popleft: O(1)
- 인덱스 접근 (deque[i]): O(1) in C++ / O(n) in Python (양끝이 아닐 때)

## 4. 기본 코드

#### Python
```python
from collections import deque

dq = deque()

#뒤에 삽입 / 앞에 삽입
dq.append(1)
dq.appendleft(0)

#뒤에서 제거 / 앞에서 제거
dq.pop()
dq.popleft()

#앞은 dq[0], 뒤는 dq[-1]
dq[0]
dq[-1]

#초기값으로 생성
dq = deque([1, 2, 3])

#maxlen 걸어두면 꽉 찼을 때 반대쪽이 자동으로 빠짐
dq = deque(maxlen=3)
```

#### C++
```cpp
deque<int> dq;

//뒤에 삽입 / 앞에 삽입
dq.push_back(1);
dq.push_front(0);

//뒤에서 제거 / 앞에서 제거. pop_back/pop_front도 값 반환 안 함
dq.pop_back();
dq.pop_front();

//앞 / 뒤 확인
dq.front();
dq.back();

//초기값으로 생성
deque<int> dq2 = {1, 2, 3};

//C++ deque엔 maxlen 자동 유지 기능 없으니까 크기를 직접 관리하자
```

## 5. 실전 패턴

### 슬라이딩 윈도우 최솟값 (단조 덱)
- 덱에 인덱스를 저장, 앞쪽은 항상 현재 윈도우의 최솟값

#### Python
```python
from collections import deque

#길이 k 윈도우마다 최솟값 뽑기
def sliding_window_min(arr, k):
    #값 말고 인덱스를 담아야 윈도우 범위 체크가 됨
    dq = deque()
    result = []

    for i, v in enumerate(arr):
        #윈도우 벗어난 놈은 앞에서 빼자
        while dq and dq[0] < i - k + 1:
            dq.popleft()

        #현재 값보다 큰 놈은 최솟값 후보가 아니니까 뒤에서 빼기(단조 증가 유지)
        while dq and arr[dq[-1]] > v:
            dq.pop()

        dq.append(i)

        #윈도우가 꽉 차야 정답 하나 나옴. 앞쪽이 곧 최솟값
        if i >= k - 1:
            result.append(arr[dq[0]])

    return result
```

#### C++
```cpp
//길이 k 윈도우마다 최솟값 뽑기
vector<int> slidingWindowMin(vector<int>& arr, int k) {
    //값 말고 인덱스를 담아야 윈도우 범위 체크가 됨
    deque<int> dq;
    vector<int> result;

    for (int i = 0; i < (int)arr.size(); i++) {
        //윈도우 벗어난 놈은 앞에서 빼자
        while (!dq.empty() && dq.front() < i - k + 1)
            dq.pop_front();

        //현재 값보다 큰 놈은 최솟값 후보가 아니니까 뒤에서 빼기(단조 증가 유지)
        while (!dq.empty() && arr[dq.back()] > arr[i])
            dq.pop_back();

        dq.push_back(i);

        //윈도우가 꽉 차야 정답 하나 나옴. 앞쪽이 곧 최솟값
        if (i >= k - 1)
            result.push_back(arr[dq.front()]);
    }
    return result;
}
```

### 팰린드롬 검사

#### Python
```python
from collections import deque

#앞뒤에서 하나씩 꺼내가며 맞춰보자
def is_palindrome(s):
    dq = deque(s)

    #양쪽 끝이 하나라도 다르면 바로 아웃
    while len(dq) > 1:
        if dq.popleft() != dq.pop():
            return False
    return True
```

#### C++
```cpp
//앞뒤에서 하나씩 꺼내가며 맞춰보자
bool isPalindrome(const string& s) {
    deque<char> dq(s.begin(), s.end());

    //양쪽 끝이 하나라도 다르면 바로 아웃
    while (dq.size() > 1) {
        if (dq.front() != dq.back()) return false;
        dq.pop_front();
        dq.pop_back();
    }
    return true;
}
```

### 회전 (rotate)

#### Python
```python
dq = deque([1, 2, 3, 4, 5])

#양수면 오른쪽으로 2칸 → [4, 5, 1, 2, 3]
dq.rotate(2)

#음수면 왼쪽으로 2칸 → [1, 2, 3, 4, 5]
dq.rotate(-2)
```

#### C++
```cpp
//C++ deque엔 rotate 메서드가 없으니까 <algorithm>의 rotate 쓰자
deque<int> dq = {1, 2, 3, 4, 5};

//오른쪽 2칸 → {4, 5, 1, 2, 3}
rotate(dq.begin(), dq.end() - 2, dq.end());

//왼쪽 2칸은 이렇게
//rotate(dq.begin(), dq.begin() + 2, dq.end());
```

## 6. 이걸 떠올려야 할 때
- "구간 최솟값/최댓값을 빠르게 구하라" → 단조 덱
- "앞뒤로 뭔가 비교해야 한다" → 덱
- "큐인데 앞에서도 넣어야 한다" → 덱
- "회전하는 배열/원형 구조" → rotate

## 7. 자주 틀리는 포인트
- Python 덱은 중간 인덱스 접근이 O(n) → 중간 원소 자주 접근하면 list (C++ deque는 O(1))
- 슬라이딩 윈도우에서 값이 아닌 인덱스를 저장해야 윈도우 범위 체크 가능
- `rotate()`는 Python이 양수→오른쪽, C++ `std::rotate`는 middle 반복자를 앞으로 당김 → 방향 헷갈리면 직접 테스트
- Python maxlen 설정 시 반대쪽이 자동으로 밀려남 → 의도치 않게 데이터 날릴 수 있음 (C++엔 없음)
