# Sweeping (스위핑)

> 한 줄 정리: 좌표를 한 방향으로 훑으면서 이벤트를 처리하는 방법, 정렬 후 순서대로 처리한다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<vector>`, `<algorithm>` (`sort`, `max`, `unique`, `lower_bound`), `<utility>`, `<array>`

## 1. 언제 쓰는가
- 겹치는 구간의 최대 개수를 구할 때
- 구간들의 합집합 길이를 구할 때
- 좌표 기반 문제에서 불필요한 탐색을 줄일 때
- 회의실 배정, 주차장 문제 등 시간/구간 관련 문제

## 2. 핵심 아이디어
- 이벤트(시작/끝)를 좌표 기준으로 정렬
- 왼쪽에서 오른쪽으로 한 번만 훑으며 처리
- 시작점 +1, 끝점 -1로 구간을 이벤트로 변환
- 정렬이 선행되어야 함

## 3. 시간복잡도
| | 복잡도 |
|--|--------|
| 시간 | O(N log N) (정렬이 지배) |
| 공간 | O(N) |

## 4. 기본 코드

### 구간 겹침 최대 개수

#### Python
```python
def max_overlap(intervals):
    events = []
    for start, end in intervals:
        events.append((start, 1))   # 시작: +1
        events.append((end, -1))    # 끝: -1

    # 같은 좌표면 끝(-1)을 먼저 처리
    events.sort(key=lambda x: (x[0], x[1]))

    max_count = 0
    count = 0
    for _, delta in events:
        count += delta
        max_count = max(max_count, count)

    return max_count
```

#### C++
```cpp
int maxOverlap(vector<pair<int,int>>& intervals) {
    vector<pair<int,int>> events;   // (좌표, +1/-1)
    for (auto& iv : intervals) {
        events.push_back({iv.first, 1});    // 시작: +1
        events.push_back({iv.second, -1});  // 끝: -1
    }
    // pair 정렬: 같은 좌표면 -1(끝)이 1(시작)보다 먼저
    sort(events.begin(), events.end());

    int max_count = 0, count = 0;
    for (auto& e : events) {
        count += e.second;
        max_count = max(max_count, count);
    }
    return max_count;
}
```

### 구간 합집합 길이

#### Python
```python
def union_length(intervals):
    if not intervals:
        return 0

    intervals.sort()
    total = 0
    cur_start, cur_end = intervals[0]

    for start, end in intervals[1:]:
        if start <= cur_end:            # 겹치면 합치기
            cur_end = max(cur_end, end)
        else:                           # 안 겹치면 확정
            total += cur_end - cur_start
            cur_start, cur_end = start, end

    total += cur_end - cur_start
    return total
```

#### C++
```cpp
long long unionLength(vector<pair<int,int>>& intervals) {
    if (intervals.empty()) return 0;

    sort(intervals.begin(), intervals.end());
    long long total = 0;
    int cur_start = intervals[0].first, cur_end = intervals[0].second;

    for (int i = 1; i < (int)intervals.size(); i++) {
        int start = intervals[i].first, end = intervals[i].second;
        if (start <= cur_end) {            // 겹치면 합치기
            cur_end = max(cur_end, end);
        } else {                           // 안 겹치면 확정
            total += cur_end - cur_start;
            cur_start = start;
            cur_end = end;
        }
    }
    total += cur_end - cur_start;
    return total;
}
```

### 차이 배열 스위핑 (구간 업데이트)

#### Python
```python
def sweeping_diff(n, updates):
    # updates: [(l, r, val), ...]
    diff = [0] * (n + 2)

    for l, r, val in updates:
        diff[l] += val
        diff[r + 1] -= val

    result = []
    cur = 0
    for i in range(1, n + 1):
        cur += diff[i]
        result.append(cur)

    return result
```

#### C++
```cpp
vector<long long> sweepingDiff(int n, vector<array<int,3>>& updates) {
    // updates: {l, r, val}
    vector<long long> diff(n + 2, 0);
    for (auto& u : updates) {
        diff[u[0]] += u[2];
        diff[u[1] + 1] -= u[2];
    }
    vector<long long> result;
    long long cur = 0;
    for (int i = 1; i <= n; i++) {
        cur += diff[i];
        result.push_back(cur);
    }
    return result;
}
```

### 좌표 압축 + 스위핑

#### Python
```python
def sweeping_with_compression(intervals):
    coords = sorted(set(x for s, e in intervals for x in (s, e)))
    rank = {v: i for i, v in enumerate(coords)}

    events = []
    for start, end in intervals:
        events.append((rank[start], 1))
        events.append((rank[end], -1))

    events.sort(key=lambda x: (x[0], x[1]))

    count = 0
    max_count = 0
    for _, delta in events:
        count += delta
        max_count = max(max_count, count)

    return max_count
```

#### C++
```cpp
int sweepingWithCompression(vector<pair<int,int>>& intervals) {
    // 좌표 모으기 + 압축
    vector<int> coords;
    for (auto& iv : intervals) {
        coords.push_back(iv.first);
        coords.push_back(iv.second);
    }
    sort(coords.begin(), coords.end());
    coords.erase(unique(coords.begin(), coords.end()), coords.end());

    vector<pair<int,int>> events;
    for (auto& iv : intervals) {
        int s = lower_bound(coords.begin(), coords.end(), iv.first)  - coords.begin();
        int e = lower_bound(coords.begin(), coords.end(), iv.second) - coords.begin();
        events.push_back({s, 1});
        events.push_back({e, -1});
    }
    sort(events.begin(), events.end());

    int count = 0, max_count = 0;
    for (auto& ev : events) {
        count += ev.second;
        max_count = max(max_count, count);
    }
    return max_count;
}
```

## 5. 이걸 떠올려야 할 때
- "겹치는 구간의 최대 개수" → 시작/끝 이벤트 스위핑
- "구간들의 합집합 길이" → 정렬 후 병합 스위핑
- "구간에 값을 더하는 연산이 많다" → 차이 배열 스위핑
- "좌표가 너무 크다" → 좌표 압축 후 스위핑

## 6. 자주 틀리는 포인트
- 같은 좌표에서 시작과 끝이 만날 때 정렬 기준 → 끝(-1)을 먼저/나중 처리는 문제에 따라 다름
- 구간 합집합에서 `cur_end = max(cur_end, end)` 빠뜨리면 오답 (완전히 포함된 구간 처리)
- 차이 배열에서 `diff[r+1]` 범위 초과 → 크기를 n+2로 선언
- C++ pair 정렬은 first→second 순 → 이벤트 순서 트릭에 활용 가능
