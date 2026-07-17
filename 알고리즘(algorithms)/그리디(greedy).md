# Greedy (그리디)

> 한 줄 정리: 매 순간 가장 좋아 보이는 선택을 하는 방법, 전체 최적이 보장될 때만 쓸 수 있다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<algorithm>` (`sort`, `max`), `<utility>` (`pair`)

## 1. 언제 쓰는가
- 매 선택이 이후 선택에 영향을 주지 않을 때
- 부분 최적 = 전체 최적이 성립할 때
- 정렬 후 순서대로 처리하면 풀리는 문제
- 교환 논증(exchange argument)으로 증명 가능한 경우

## 2. 핵심 아이디어
- 그리디가 맞는지 확인하는 게 핵심 → 틀릴 수도 있음
- "지금 당장 최선"이 "전체 최선"이 되는 구조인지 따져야 함
- 보통 정렬이 선행됨
- 증명이 어렵다면 DP로 풀 수 있는지 먼저 고민

## 3. 시간복잡도
- 대부분 O(N log N) → 정렬 비용이 지배
- 정렬 후 한 번 순회: O(N log N + N) = O(N log N)

## 4. 기본 코드

### 거스름돈 문제

#### Python
```python
#큰 단위부터 최대한 털어야 개수가 최소
coins = [500, 100, 50, 10]
change = 1260
cnt = 0

for coin in coins:
    cnt += change // coin
    change %= coin

#답은 6
print(cnt)
```

#### C++
```cpp
//큰 단위부터 최대한 털어야 개수가 최소
int coins[] = {500, 100, 50, 10};
int change = 1260, cnt = 0;

for (int coin : coins) {
    cnt += change / coin;
    change %= coin;
}

//cnt == 6
```

### 회의실 배정 (대표 유형)

#### Python
```python
#일찍 끝나는 놈부터 잡아야 뒤에 자리가 많이 남음
meetings = [(1, 4), (3, 5), (0, 6), (5, 7), (3, 8), (5, 9), (6, 10)]
meetings.sort(key=lambda x: (x[1], x[0]))

cnt = 0
end_time = 0
for start, end in meetings:
    #앞 회의 끝난 뒤에 시작하면 걍 넣기
    if start >= end_time:
        cnt += 1
        end_time = end

#답은 4
print(cnt)
```

#### C++
```cpp
//일찍 끝나는 놈부터 잡아야 뒤에 자리가 많이 남음
vector<pair<int,int>> meetings = {{1,4},{3,5},{0,6},{5,7},{3,8},{5,9},{6,10}};
sort(meetings.begin(), meetings.end(),
     [](const pair<int,int>& a, const pair<int,int>& b){
         if (a.second != b.second) return a.second < b.second;
         return a.first < b.first;
     });

int cnt = 0, end_time = 0;
for (auto& m : meetings) {
    int start = m.first, end = m.second;

    //앞 회의 끝난 뒤에 시작하면 걍 넣기
    if (start >= end_time) {
        cnt++;
        end_time = end;
    }
}

//cnt == 4
```

### 최소 횟수로 구간 커버

#### Python
```python
#0 ~ total을 최소 개수로 덮기
def min_cover(intervals, total):
    intervals.sort()
    cnt = 0
    cur_end = 0
    i = 0

    while cur_end < total:
        next_end = cur_end

        #지금 위치에서 이어붙일 수 있는 놈들 중 제일 멀리 가는 놈 고르기
        while i < len(intervals) and intervals[i][0] <= cur_end:
            next_end = max(next_end, intervals[i][1])
            i += 1

        #한 칸도 못 늘렸으면 구멍 뚫린 거라 커버 불가
        if next_end == cur_end:
            return -1
        cur_end = next_end
        cnt += 1

    return cnt
```

#### C++
```cpp
//0 ~ total을 최소 개수로 덮기
int minCover(vector<pair<int,int>>& intervals, int total) {
    sort(intervals.begin(), intervals.end());
    int cnt = 0, cur_end = 0, i = 0, n = intervals.size();

    while (cur_end < total) {
        int next_end = cur_end;

        //지금 위치에서 이어붙일 수 있는 놈들 중 제일 멀리 가는 놈 고르기
        while (i < n && intervals[i].first <= cur_end) {
            next_end = max(next_end, intervals[i].second);
            i++;
        }

        //한 칸도 못 늘렸으면 구멍 뚫린 거라 커버 불가
        if (next_end == cur_end) return -1;
        cur_end = next_end;
        cnt++;
    }
    return cnt;
}
```

## 5. 그리디가 맞는지 판단하는 법
```
1. "매 선택이 독립적인가?" → 이전 선택이 이후에 영향 없으면 그리디 가능
2. "반례가 있는가?" → 작은 예시로 직접 반례 찾아보기
3. "교환해도 손해 없는가?" → 두 선택의 순서를 바꿔도 결과가 같거나 더 좋으면 그리디 OK
4. 반례가 보이면 → DP 고려
```

## 6. 이걸 떠올려야 할 때
- "최소 횟수", "최소 비용"으로 뭔가를 처리 → 그리디 시도
- 정렬 기준이 명확하게 보일 때 → 그리디
- "~를 최대한 많이" → 그리디
- 반례가 바로 떠오르면 → DP로 전환

## 7. 자주 틀리는 포인트
- 그리디가 항상 맞다고 착각 → 반례 먼저 찾아보는 습관
- 정렬 기준을 잘못 잡으면 틀림 → 회의실 배정에서 시작 시간 기준으로 정렬하면 오답
- 그리디로 풀리는 것 같아도 DP가 맞는 경우 있음 → 확신 없으면 작은 케이스로 검증
- 동률일 때 처리 빠뜨리는 경우 → 정렬 비교자에 두 번째 조건까지 챙기기
