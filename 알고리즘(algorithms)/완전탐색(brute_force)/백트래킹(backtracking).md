# Backtracking (백트래킹)

> 한 줄 정리: 재귀로 모든 경우를 탐색하되, 조건에 맞지 않으면 즉시 가지치기해서 탐색 범위를 줄인다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<set>`

## 1. 언제 쓰는가
- 순열/조합/부분집합을 직접 구현할 때
- 조건을 만족하는 경우만 탐색해야 할 때
- N-Queen, 스도쿠 같은 제약 조건 탐색 문제
- 완전탐색인데 가지치기로 최적화가 가능할 때

## 2. 핵심 아이디어
- 재귀로 선택지를 하나씩 시도
- 현재 선택이 조건을 만족하지 않으면 즉시 되돌아감 (가지치기)
- 가지치기가 얼마나 효과적이냐에 따라 성능이 결정됨
- 완전탐색 + 가지치기 = 백트래킹

## 3. 시간복잡도
- 가지치기 없으면 완전탐색과 동일
- 순열: O(N!), 조합: O(2^N)
- 가지치기 효과에 따라 실제로는 훨씬 빠름

## 4. 기본 코드

### 백트래킹 기본 구조
```
void backtrack(상태) {
    if (종료 조건) { 결과 저장; return; }

    for (선택지 : 가능한_선택지) {
        if (유망하지_않으면) continue;   // 가지치기
        상태 변경;                       // 선택
        backtrack(다음 상태);
        상태 복원;                       // 되돌리기 (undo)
    }
}
```

### 순열 직접 구현

#### Python
```python
#순서 있는 뽑기. used로 이미 쓴 놈 걸러가면서
def permutation(arr, path, used):
    #다 채웠으면 하나 완성
    if len(path) == len(arr):
        print(path)
        return

    for i in range(len(arr)):
        #이미 쓴 놈이니까 스킵
        if used[i]:
            continue

        #일단 놓고 들어가보기
        used[i] = True
        path.append(arr[i])
        permutation(arr, path, used)

        #돌아왔으면 놨던 거 도로 빼기(짝 맞추기)
        path.pop()
        used[i] = False

arr = [1, 2, 3]
permutation(arr, [], [False] * len(arr))
```

#### C++
```cpp
vector<int> arr = {1, 2, 3};
vector<int> path;
vector<bool> used;

//순서 있는 뽑기. used로 이미 쓴 놈 걸러가면서
void permutation() {
    //다 채웠으면 하나 완성
    if (path.size() == arr.size()) {
        // path 출력/저장
        return;
    }
    for (int i = 0; i < (int)arr.size(); i++) {
        //이미 쓴 놈이니까 스킵
        if (used[i]) continue;

        //일단 놓고 들어가보기
        used[i] = true;
        path.push_back(arr[i]);
        permutation();

        //돌아왔으면 놨던 거 도로 빼기(짝 맞추기)
        path.pop_back();
        used[i] = false;
    }
}
// used.assign(arr.size(), false); permutation();
```

### 조합 직접 구현

#### Python
```python
#순서 상관없는 뽑기. start로 뒤쪽만 봐서 중복 방지
def combination(arr, start, path, r):
    #r개 채웠으면 하나 완성
    if len(path) == r:
        print(path)
        return

    #앞엣 놈 다시 안 보게 start부터
    for i in range(start, len(arr)):
        path.append(arr[i])
        combination(arr, i + 1, path, r)

        #돌아왔으면 놨던 거 빼기
        path.pop()

arr = [1, 2, 3, 4]
combination(arr, 0, [], 2)
```

#### C++
```cpp
vector<int> arr = {1, 2, 3, 4};
vector<int> path;

//순서 상관없는 뽑기. start로 뒤쪽만 봐서 중복 방지
void combination(int start, int r) {
    //r개 채웠으면 하나 완성
    if ((int)path.size() == r) {
        // path 출력/저장
        return;
    }

    //앞엣 놈 다시 안 보게 start부터
    for (int i = start; i < (int)arr.size(); i++) {
        path.push_back(arr[i]);
        combination(i + 1, r);

        //돌아왔으면 놨던 거 빼기
        path.pop_back();
    }
}
// combination(0, 2);
```

### 부분집합 직접 구현

#### Python
```python
#부분집합은 종료조건 없이 매 노드가 다 답
def subset(arr, idx, path):
    #지금까지 고른 게 그 자체로 부분집합 하나
    print(path)

    for i in range(idx, len(arr)):
        path.append(arr[i])
        subset(arr, i + 1, path)

        #돌아왔으면 놨던 거 빼기
        path.pop()

arr = [1, 2, 3]
subset(arr, 0, [])
```

#### C++
```cpp
//부분집합은 종료조건 없이 매 노드가 다 답
void subset(int idx) {
    // path = 지금까지 고른 게 그 자체로 부분집합 하나

    for (int i = idx; i < (int)arr.size(); i++) {
        path.push_back(arr[i]);
        subset(i + 1);

        //돌아왔으면 놨던 거 빼기
        path.pop_back();
    }
}
// subset(0);
```

## 5. 실전 패턴

### N-Queen

#### Python
```python
#한 행에 하나씩, 충돌 안 나게 퀸 놓기
def n_queen(row, cols, diag1, diag2):
    #마지막 행까지 다 놨으면 성공한 배치 하나
    if row == n:
        result.append(cols[:])
        return

    #이 행에서 놓을 열 다 시도
    for col in range(n):
        #같은 열이거나 두 대각선 중 하나라도 겹치면 컷
        if col in cols or (row - col) in diag1 or (row + col) in diag2:
            continue

        #일단 놓고 대각선까지 점유 표시
        cols.append(col)
        diag1.add(row - col)
        diag2.add(row + col)
        n_queen(row + 1, cols, diag1, diag2)

        #돌아왔으면 놨던 거 싹 다 원복
        cols.pop()
        diag1.remove(row - col)
        diag2.remove(row + col)
```

#### C++
```cpp
int n;
vector<vector<int>> result;
vector<int> cols;

//열, ↘대각, ↗대각 점유 여부
set<int> usedCol, diag1, diag2;

//한 행에 하나씩, 충돌 안 나게 퀸 놓기
void nQueen(int row) {
    //마지막 행까지 다 놨으면 성공한 배치 하나
    if (row == n) {
        result.push_back(cols);
        return;
    }
    for (int col = 0; col < n; col++) {
        //같은 열이거나 두 대각선 중 하나라도 겹치면 컷
        if (usedCol.count(col) || diag1.count(row - col) || diag2.count(row + col))
            continue;

        //일단 놓고 대각선까지 점유 표시
        cols.push_back(col);
        usedCol.insert(col);
        diag1.insert(row - col);
        diag2.insert(row + col);
        nQueen(row + 1);

        //돌아왔으면 놨던 거 싹 다 원복
        cols.pop_back();
        usedCol.erase(col);
        diag1.erase(row - col);
        diag2.erase(row + col);
    }
}
```

### 중복 없는 순열 (같은 값 있을 때)

#### Python
```python
#같은 값이 섞여있을 때 결과 중복 안 나게 하는 순열
def permutation_unique(arr, path, used):
    #다 채웠으면 하나 완성
    if len(path) == len(arr):
        print(path)
        return

    #이 depth에서 이미 써본 값은 다시 안 쓰려고 seen 둠
    seen = set()
    for i in range(len(arr)):
        #이미 쓴 인덱스거나, 같은 값 이번 depth에서 또 나오면 스킵
        if used[i] or arr[i] in seen:
            continue

        #이번 depth에서 이 값 써봤다고 기록
        seen.add(arr[i])

        #일단 놓고 들어가보기
        used[i] = True
        path.append(arr[i])
        permutation_unique(arr, path, used)

        #돌아왔으면 놨던 거 빼기
        path.pop()
        used[i] = False
```

#### C++
```cpp
//같은 값이 섞여있을 때 결과 중복 안 나게 하는 순열
void permutationUnique() {
    //다 채웠으면 하나 완성
    if (path.size() == arr.size()) {
        // path 출력/저장
        return;
    }

    //이 depth에서 이미 써본 값은 다시 안 쓰려고 seen 둠
    set<int> seen;
    for (int i = 0; i < (int)arr.size(); i++) {
        //이미 쓴 인덱스거나, 같은 값 이번 depth에서 또 나오면 스킵
        if (used[i] || seen.count(arr[i])) continue;

        //이번 depth에서 이 값 써봤다고 기록
        seen.insert(arr[i]);

        //일단 놓고 들어가보기
        used[i] = true;
        path.push_back(arr[i]);
        permutationUnique();

        //돌아왔으면 놨던 거 빼기
        path.pop_back();
        used[i] = false;
    }
}
```

## 6. 이걸 떠올려야 할 때
- "조건을 만족하는 모든 경우" → 백트래킹
- 라이브러리(itertools / next_permutation)로 못 쓰는 상황 (조건 복잡, 중간 가지치기) → 직접 구현
- "N-Queen", "스도쿠", "암호 만들기" 유형 → 백트래킹

## 7. 자주 틀리는 포인트
- 되돌리기(undo) 빠뜨리면 상태가 오염 → `pop`, `used[i] = false` 짝 맞추기
- 가지치기 조건을 너무 늦게 체크하면 시간초과
- 중복 값이 있는 순열에서 `seen` 처리 안 하면 중복 결과 나옴
- Python은 `result.append(path[:])`로 복사 저장 / C++ `result.push_back(cols)`는 값 복사라 안전
