# Sprague-Grundy Theorem (스프라그-그런디 정리)

> 한 줄 정리: 모든 공정 게임은 님(Nim) 게임으로 환원할 수 있고, Grundy 값으로 승패를 판별한다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<map>`, `<set>`, `<algorithm>`, `<cstring>` (`memset`)

## 1. 언제 쓰는가
- 두 플레이어가 번갈아 움직이는 게임의 승패 판별
- 여러 게임이 동시에 진행될 때 전체 승패 판별
- 님(Nim) 게임 변형 문제

## 2. 핵심 개념

### 공정 게임 (Impartial Game)
```
- 두 플레이어가 동일한 행동을 할 수 있는 게임
- 마지막 수를 두는 플레이어가 이기는 규칙 (Normal play)
- 예: 님 게임, 돌 가져가기
```

### Grundy 값 (님버, Nimber)
```
- 각 게임 상태의 Grundy 값 = mex(후속 상태들의 Grundy 값)
- mex(minimum excludant): 집합에 포함되지 않는 최솟값 음이 아닌 정수
- Grundy 값이 0이면 → 현재 플레이어가 짐 (P-position)
- Grundy 값이 0이 아니면 → 현재 플레이어가 이김 (N-position)
```

### mex 예시
```
mex({0, 1, 2}) = 3
mex({0, 2})    = 1
mex({1, 2})    = 0
mex({})        = 0  (빈 집합)
```

### 스프라그-그런디 정리
```
여러 게임이 동시에 진행될 때:
전체 Grundy 값 = 각 게임의 Grundy 값의 XOR

전체 Grundy 값이 0 → 현재 플레이어가 짐
전체 Grundy 값이 0이 아님 → 현재 플레이어가 이김
```

## 3. 기본 코드

### mex 계산

#### Python
```python
def mex(s):
    # 집합 s에 포함되지 않는 최솟값 음이 아닌 정수
    i = 0
    while i in s:
        i += 1
    return i
```

#### C++
```cpp
int mex(const set<int>& s) {
    int i = 0;
    while (s.count(i)) i++;   // 0부터 없는 값 탐색
    return i;
}
```

### Grundy 값 계산 (메모이제이션)

#### Python
```python
from functools import lru_cache

def grundy(state):
    # state: 현재 게임 상태
    # next_states(state): 이동 가능한 다음 상태들
    @lru_cache(maxsize=None)
    def _grundy(state):
        reachable = {_grundy(next_state) for next_state in next_states(state)}
        return mex(reachable)
    return _grundy(state)
```

#### C++
```cpp
map<int, int> memo;   // 상태 → Grundy 값 (상태를 int로 인코딩했다고 가정)

int grundy(int state) {
    if (memo.count(state)) return memo[state];
    set<int> reachable;
    for (int nxt : next_states(state))   // 이동 가능한 다음 상태들
        reachable.insert(grundy(nxt));
    return memo[state] = mex(reachable);
}
```

### 돌 가져가기 게임

#### Python
```python
from functools import lru_cache

# N개의 돌에서 1~K개를 가져갈 수 있을 때 Grundy 값
@lru_cache(maxsize=None)
def grundy_stones(n, k):
    if n == 0:
        return 0  # 가져갈 돌 없음 → 짐 (Grundy = 0)
    reachable = set()
    for take in range(1, min(k, n) + 1):
        reachable.add(grundy_stones(n - take, k))
    return mex(reachable)

# 예시: 10개 돌, 최대 3개 가져가기
for i in range(11):
    print(f"n={i}: grundy={grundy_stones(i, 3)}")
# 패턴: 0,1,2,3,0,1,2,3,... (주기 = k+1)
```

#### C++
```cpp
map<int, int> memo;   // n → Grundy 값 (k는 고정)

// N개의 돌에서 1~K개를 가져갈 수 있을 때 Grundy 값
int grundyStones(int n, int k) {
    if (n == 0) return 0;                    // 가져갈 돌 없음 → 짐 (Grundy = 0)
    if (memo.count(n)) return memo[n];
    set<int> reachable;
    for (int take = 1; take <= min(k, n); take++)
        reachable.insert(grundyStones(n - take, k));
    return memo[n] = mex(reachable);
}
// 패턴: 0,1,2,3,0,1,2,3,... (주기 = k+1)
```

## 4. 실전 패턴

### 님 게임 (Nim)

#### Python
```python
# N개의 더미, 각 더미에서 임의 개수 가져가기
# Grundy 값 = 더미 크기 자체
# 전체 Grundy = 모든 더미 크기의 XOR

def nim_winner(piles):
    xor = 0
    for pile in piles:
        xor ^= pile
    return xor != 0  # True면 현재 플레이어 승리
```

#### C++
```cpp
// Grundy 값 = 더미 크기 자체 → 전체 Grundy = 모든 더미의 XOR
bool nimWinner(const vector<int>& piles) {
    int x = 0;
    for (int pile : piles) x ^= pile;
    return x != 0;   // true면 현재 플레이어 승리
}
```

### 여러 게임 동시 진행

#### Python
```python
# 게임 A와 게임 B를 동시에 진행
# 자기 턴에 둘 중 하나를 선택해서 이동

def combined_winner(state_a, state_b):
    g_a = grundy_a(state_a)
    g_b = grundy_b(state_b)
    return (g_a ^ g_b) != 0  # XOR이 0이 아니면 현재 플레이어 승리
```

#### C++
```cpp
bool combinedWinner(int state_a, int state_b) {
    int g_a = grundyA(state_a);
    int g_b = grundyB(state_b);
    return (g_a ^ g_b) != 0;   // XOR이 0이 아니면 현재 플레이어 승리
}
```

### 보드 게임 Grundy 값 (2차원)

#### Python
```python
@lru_cache(maxsize=None)
def grundy_2d(r, c):
    # (r, c)에서 이동 가능한 상태들의 Grundy 값
    reachable = set()
    for dr, dc in moves:  # 이동 방향
        nr, nc = r + dr, c + dc
        if 0 <= nr < R and 0 <= nc < C:
            reachable.add(grundy_2d(nr, nc))
    return mex(reachable)
```

#### C++
```cpp
int memo2d[R][C];   // -1로 초기화 (memset(memo2d, -1, sizeof(memo2d)))

int grundy2d(int r, int c) {
    if (memo2d[r][c] != -1) return memo2d[r][c];
    set<int> reachable;
    for (auto& mv : moves) {                       // 이동 방향 목록
        int nr = r + mv.first, nc = c + mv.second;
        if (0 <= nr && nr < R && 0 <= nc && nc < C)
            reachable.insert(grundy2d(nr, nc));
    }
    return memo2d[r][c] = mex(reachable);
}
```

## 5. 미제르 게임 (Misère)

### Normal play vs Misère
```
Normal play: 마지막 수를 두는 플레이어가 이김 (일반적)
Misère:      마지막 수를 두는 플레이어가 짐 (반대)
```

### 님 미제르 판별법
```
님 게임에서 Misère 규칙일 때:

모든 더미가 크기 1 이하이면:
    더미 개수가 홀수 → 현재 플레이어 짐
    더미 개수가 짝수 → 현재 플레이어 이김

그 외 (크기 2 이상인 더미가 하나라도 있으면):
    XOR이 0 → 현재 플레이어 짐
    XOR이 0이 아님 → 현재 플레이어 이김

즉, Normal play와 결론이 반대가 되는 경우는
"모든 더미가 크기 1일 때" 뿐
```

### 코드

#### Python
```python
def nim_misere_winner(piles):
    xor = 0
    for pile in piles:
        xor ^= pile

    # 모든 더미가 크기 1 이하인지 확인
    all_one = all(p <= 1 for p in piles)

    if all_one:
        # 더미 개수(= 돌 개수)가 홀수면 현재 플레이어 짐
        return sum(piles) % 2 == 0  # True면 현재 플레이어 승리
    else:
        # Normal play와 동일
        return xor != 0

# 예시
print(nim_misere_winner([1, 1, 1]))  # False → 현재 플레이어 짐 (홀수 개)
print(nim_misere_winner([1, 2, 3]))  # Normal play와 동일하게 XOR = 0 → 짐
print(nim_misere_winner([2, 3]))     # XOR = 1 → 이김
```

#### C++
```cpp
bool nimMisereWinner(const vector<int>& piles) {
    int x = 0;
    long long total = 0;
    bool all_one = true;
    for (int p : piles) {
        x ^= p;
        total += p;
        if (p > 1) all_one = false;   // 크기 2 이상 더미 존재 여부
    }
    if (all_one)
        return total % 2 == 0;   // 돌 개수가 홀수면 현재 플레이어 짐
    else
        return x != 0;           // Normal play와 동일
}
// nimMisereWinner({1,1,1}) → false (홀수 개, 현재 플레이어 짐)
// nimMisereWinner({1,2,3}) → false (XOR = 0)
// nimMisereWinner({2,3})   → true  (XOR = 1, 이김)
```

### Normal play vs Misère 비교
| | Normal play | Misère |
|--|------------|--------|
| 승리 조건 | 마지막 수를 둠 | 마지막 수를 두지 않음 |
| 모든 더미 ≤ 1 | XOR로 판별 | XOR 반대 |
| 크기 2 이상 존재 | XOR로 판별 | XOR로 판별 (동일) |

## 6. 이걸 떠올려야 할 때
- "마지막 수를 두는 사람이 짐" → 미제르 → 님이면 all_one 여부로 분기
- "두 플레이어 게임, 누가 이기나" → 스프라그-그런디
- "여러 게임 동시 진행" → 각 Grundy 값 XOR
- "님 게임 변형" → 더미 크기의 XOR
- Grundy 값에 주기가 있으면 → 큰 입력도 처리 가능

## 7. 자주 틀리는 포인트
- 마지막 수를 두는 사람이 지는 규칙(Misère)이면 별도 처리 필요
- mex는 집합에 없는 **최솟값** → 0부터 순서대로 확인
- 여러 게임 XOR 시 Grundy 값이 0인 게임도 포함해야 함
- 재귀가 깊어지면 Python은 `sys.setrecursionlimit`, C++은 스택 크기/반복 변환 고려
