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
#s에 없는 가장 작은 음 아닌 정수 찾기 (mex)
def mex(s):
    #0부터 하나씩 올려보다가 s에 없는 놈 나오면 그게 답
    i = 0
    while i in s:
        i += 1
    return i
```

#### C++
```cpp
//s에 없는 가장 작은 음 아닌 정수 찾기 (mex)
int mex(const set<int>& s) {
    //0부터 하나씩 올려보다가 s에 없는 놈 나오면 그게 답
    int i = 0;
    while (s.count(i)) i++;
    return i;
}
```

### Grundy 값 계산 (메모이제이션)

#### Python
```python
from functools import lru_cache

#현재 상태 state의 Grundy 값 구하기, next_states로 갈 수 있는 다음 상태 받음
def grundy(state):
    #같은 상태 또 계산하기 아까우니까 캐싱
    @lru_cache(maxsize=None)
    def _grundy(state):
        #갈 수 있는 상태들 Grundy 값 모아서
        reachable = {_grundy(next_state) for next_state in next_states(state)}

        #거기 없는 최소값이 내 Grundy 값
        return mex(reachable)
    return _grundy(state)
```

#### C++
```cpp
//상태 → Grundy 값. 상태는 int로 인코딩했다고 치자
map<int, int> memo;

int grundy(int state) {
    //이미 계산한 상태면 걍 꺼내 쓰기
    if (memo.count(state)) return memo[state];

    //갈 수 있는 다음 상태들 Grundy 값 모으기
    set<int> reachable;
    for (int nxt : next_states(state))
        reachable.insert(grundy(nxt));

    //거기 없는 최소값이 내 Grundy 값
    return memo[state] = mex(reachable);
}
```

### 돌 가져가기 게임

#### Python
```python
from functools import lru_cache

#돌 n개에서 1~k개 가져갈 수 있을 때 Grundy 값
@lru_cache(maxsize=None)
def grundy_stones(n, k):
    #가져갈 돌 없으면 지금 둘 차례인 놈이 짐 → Grundy 0
    if n == 0:
        return 0

    #1개부터 k개(남은 것보다 많이는 못 가져감)까지 다 가본 상태 모으기
    reachable = set()
    for take in range(1, min(k, n) + 1):
        reachable.add(grundy_stones(n - take, k))
    return mex(reachable)

#예시: 10개 돌, 최대 3개 가져가기
for i in range(11):
    print(f"n={i}: grundy={grundy_stones(i, 3)}")
#패턴: 0,1,2,3,0,1,2,3,... (주기 = k+1)
```

#### C++
```cpp
//n → Grundy 값. k는 고정
map<int, int> memo;

//돌 n개에서 1~k개 가져갈 수 있을 때 Grundy 값
int grundyStones(int n, int k) {
    //가져갈 돌 없으면 지금 둘 차례인 놈이 짐 → Grundy 0
    if (n == 0) return 0;

    //이미 계산한 n이면 걍 꺼내 쓰기
    if (memo.count(n)) return memo[n];

    //1개부터 k개(남은 것보다 많이는 못 가져감)까지 다 가본 상태 모으기
    set<int> reachable;
    for (int take = 1; take <= min(k, n); take++)
        reachable.insert(grundyStones(n - take, k));
    return memo[n] = mex(reachable);
}
//패턴: 0,1,2,3,0,1,2,3,... (주기 = k+1)
```

## 4. 실전 패턴

### 님 게임 (Nim)

#### Python
```python
#님 게임은 더미 하나의 Grundy가 걍 더미 크기라 전체는 다 XOR 하면 됨
def nim_winner(piles):
    xor = 0
    for pile in piles:
        xor ^= pile

    #XOR이 0 아니면 지금 둘 차례인 놈이 이김
    return xor != 0
```

#### C++
```cpp
//님 게임은 더미 하나의 Grundy가 걍 더미 크기라 전체는 다 XOR 하면 됨
bool nimWinner(const vector<int>& piles) {
    int x = 0;
    for (int pile : piles) x ^= pile;

    //XOR이 0 아니면 지금 둘 차례인 놈이 이김
    return x != 0;
}
```

### 여러 게임 동시 진행

#### Python
```python
#게임 A, B 동시 진행. 자기 턴에 둘 중 하나 골라 움직임
#여러 게임 합칠 땐 각자 Grundy 값을 XOR
def combined_winner(state_a, state_b):
    g_a = grundy_a(state_a)
    g_b = grundy_b(state_b)

    #XOR이 0 아니면 지금 둘 차례인 놈이 이김
    return (g_a ^ g_b) != 0
```

#### C++
```cpp
//여러 게임 합칠 땐 각자 Grundy 값을 XOR
bool combinedWinner(int state_a, int state_b) {
    int g_a = grundyA(state_a);
    int g_b = grundyB(state_b);

    //XOR이 0 아니면 지금 둘 차례인 놈이 이김
    return (g_a ^ g_b) != 0;
}
```

### 보드 게임 Grundy 값 (2차원)

#### Python
```python
#(r, c) 칸의 Grundy 값
@lru_cache(maxsize=None)
def grundy_2d(r, c):
    #갈 수 있는 칸들 Grundy 값 모으기
    reachable = set()

    #정해진 이동 방향으로 뻗어보기
    for dr, dc in moves:
        nr, nc = r + dr, c + dc

        #보드 밖으로 안 나가는 칸만
        if 0 <= nr < R and 0 <= nc < C:
            reachable.add(grundy_2d(nr, nc))
    return mex(reachable)
```

#### C++
```cpp
//-1로 초기화 해두기 (memset(memo2d, -1, sizeof(memo2d)))
int memo2d[R][C];

int grundy2d(int r, int c) {
    //이미 계산한 칸이면 걍 꺼내 쓰기
    if (memo2d[r][c] != -1) return memo2d[r][c];

    //갈 수 있는 칸들 Grundy 값 모으기
    set<int> reachable;
    for (auto& mv : moves) {
        int nr = r + mv.first, nc = c + mv.second;

        //보드 밖으로 안 나가는 칸만
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

    #모든 더미가 크기 1 이하면 결론이 뒤집힘. 그것만 따로 분기
    all_one = all(p <= 1 for p in piles)

    if all_one:
        #이 경우엔 돌 개수 홀수면 지금 둘 차례인 놈이 짐, 짝수면 이김
        return sum(piles) % 2 == 0
    else:
        #크기 2 이상 더미 있으면 걍 Normal play랑 똑같음
        return xor != 0

#예시
print(nim_misere_winner([1, 1, 1]))  #False → 현재 플레이어 짐 (홀수 개)
print(nim_misere_winner([1, 2, 3]))  #Normal play와 동일하게 XOR = 0 → 짐
print(nim_misere_winner([2, 3]))     #XOR = 1 → 이김
```

#### C++
```cpp
bool nimMisereWinner(const vector<int>& piles) {
    int x = 0;
    long long total = 0;
    bool all_one = true;

    //한 번 돌면서 XOR, 총합, 크기 2 이상 있는지 다 챙기기
    for (int p : piles) {
        x ^= p;
        total += p;
        if (p > 1) all_one = false;
    }

    //모든 더미 1 이하면 돌 개수 홀수일 때 지금 둘 차례인 놈이 짐
    if (all_one)
        return total % 2 == 0;

    //크기 2 이상 더미 있으면 걍 Normal play랑 똑같음
    else
        return x != 0;
}
//nimMisereWinner({1,1,1}) → false (홀수 개, 현재 플레이어 짐)
//nimMisereWinner({1,2,3}) → false (XOR = 0)
//nimMisereWinner({2,3})   → true  (XOR = 1, 이김)
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
