# Heap (힙)

> 한 줄 정리: 항상 최솟값(또는 최댓값)을 O(1)에 꺼낼 수 있는 구조

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<queue>` (`priority_queue`), `<vector>`, `<functional>` (`greater`), `<climits>` (`LLONG_MAX`)

## 1. 언제 쓰는가
- 가장 작은/큰 값을 반복적으로 꺼내야 할 때
- K번째 최솟값/최댓값을 구할 때
- 우선순위가 있는 작업을 처리할 때 (우선순위 큐)
- 다익스트라 알고리즘
- 실시간으로 중앙값을 구해야 할 때

## 2. 핵심 아이디어
- Python `heapq`는 기본이 **최소 힙**, C++ `priority_queue`는 기본이 **최대 힙**
- Python 최대 힙은 값에 `-`, C++ 최소 힙은 `greater<int>` 비교자
- push → `heappush` / `push`, pop → `heappop` / `pop`
- 내부적으로 완전 이진 트리 구조

## 3. 시간복잡도
- push: O(log n)
- pop: O(log n)
- 최상단 값 확인: O(1) → `heap[0]` / `top()`
- heapify (리스트 → 힙): O(n)

## 4. 기본 코드

#### Python
```python
import heapq

heap = []

# push
heapq.heappush(heap, 3)
heapq.heappush(heap, 1)
heapq.heappush(heap, 2)

# pop (항상 최솟값)
val = heapq.heappop(heap)  # 1

# 최솟값 확인 (제거 안 함)
val = heap[0]

# 리스트를 힙으로 변환
arr = [3, 1, 2]
heapq.heapify(arr)

# 최대 힙 (음수로 저장)
heapq.heappush(heap, -3)
max_val = -heapq.heappop(heap)  # 3
```

#### C++
```cpp
priority_queue<int, vector<int>, greater<int>> heap;   // 최소 힙

// push
heap.push(3);
heap.push(1);
heap.push(2);

// top 확인 + pop (항상 최솟값)
int val = heap.top();   // 1
heap.pop();

// 리스트를 힙으로 변환 (생성자에 반복자 → O(n))
vector<int> arr = {3, 1, 2};
priority_queue<int, vector<int>, greater<int>> heap2(arr.begin(), arr.end());

// 최대 힙 (기본)
priority_queue<int> maxHeap;
maxHeap.push(3);
int maxVal = maxHeap.top();   // 3
```

## 5. 실전 패턴

### K번째 최솟값

#### Python
```python
import heapq

def kth_smallest(arr, k):
    heapq.heapify(arr)
    for _ in range(k - 1):
        heapq.heappop(arr)
    return heapq.heappop(arr)
```

#### C++
```cpp
int kthSmallest(vector<int>& arr, int k) {
    priority_queue<int, vector<int>, greater<int>> heap(arr.begin(), arr.end());
    for (int i = 0; i < k - 1; i++) heap.pop();
    return heap.top();
}
```

### 튜플로 우선순위 지정

#### Python
```python
import heapq

heap = []
# (우선순위, 값) 형태로 넣으면 우선순위 기준으로 정렬
heapq.heappush(heap, (1, 'task_a'))
heapq.heappush(heap, (3, 'task_c'))
heapq.heappush(heap, (2, 'task_b'))

priority, task = heapq.heappop(heap)  # (1, 'task_a')
```

#### C++
```cpp
// (우선순위, 값) — pair는 first 기준으로 비교됨
priority_queue<pair<int,string>, vector<pair<int,string>>, greater<>> heap;  // greater<>는 C++14
heap.push({1, "task_a"});
heap.push({3, "task_c"});
heap.push({2, "task_b"});

auto top = heap.top();     // {1, "task_a"}
int priority = top.first;
string task = top.second;
```

### 다익스트라

#### Python
```python
import heapq

def dijkstra(graph, start):
    dist = {node: float('inf') for node in graph}
    dist[start] = 0
    heap = [(0, start)]  # (거리, 노드)

    while heap:
        cost, node = heapq.heappop(heap)
        if cost > dist[node]:  # 이미 처리된 노드 스킵
            continue
        for neighbor, weight in graph[node]:
            new_cost = cost + weight
            if new_cost < dist[neighbor]:
                dist[neighbor] = new_cost
                heapq.heappush(heap, (new_cost, neighbor))

    return dist
```

#### C++
```cpp
vector<long long> dijkstra(vector<vector<pair<int,int>>>& graph, int start) {
    int n = graph.size();
    vector<long long> dist(n, LLONG_MAX);
    dist[start] = 0;
    // 최소 힙: (거리, 노드)
    priority_queue<pair<long long,int>, vector<pair<long long,int>>, greater<>> heap;
    heap.push({0, start});

    while (!heap.empty()) {
        long long cost = heap.top().first;
        int node = heap.top().second;
        heap.pop();
        if (cost > dist[node]) continue;   // 이미 처리된 노드 스킵
        for (auto& e : graph[node]) {
            int next = e.first, weight = e.second;
            long long nc = cost + weight;
            if (nc < dist[next]) {
                dist[next] = nc;
                heap.push({nc, next});
            }
        }
    }
    return dist;
}
```

### 중앙값 유지 (최대 힙 + 최소 힙)

#### Python
```python
import heapq

# 왼쪽 절반: 최대 힙 (음수 저장), 오른쪽 절반: 최소 힙
left, right = [], []

def add(num):
    heapq.heappush(left, -num)
    heapq.heappush(right, -heapq.heappop(left))
    if len(left) < len(right):
        heapq.heappush(left, -heapq.heappop(right))

def get_median():
    return -left[0]
```

#### C++
```cpp
// 왼쪽 절반: 최대 힙, 오른쪽 절반: 최소 힙
priority_queue<int> left;                               // 최대 힙
priority_queue<int, vector<int>, greater<int>> right;   // 최소 힙

void add(int num) {
    left.push(num);
    right.push(left.top()); left.pop();
    if (left.size() < right.size()) {
        left.push(right.top()); right.pop();
    }
}

int getMedian() {
    return left.top();
}
```

## 6. 이걸 떠올려야 할 때
- "매번 최솟값/최댓값을 꺼내야 한다" → 힙
- "K개만 유지하면서 최솟값/최댓값" → 힙
- "최단 경로" → 다익스트라 + 힙
- "우선순위가 있다" → 힙
- 정렬하면 O(n log n)인데 매번 꺼내는 게 반복된다면 → 힙이 더 적합

## 7. 자주 틀리는 포인트
- Python heapq는 최소 힙만 → 최대 힙은 `-` / C++ priority_queue는 최대 힙 기본 → 최소 힙은 `greater<int>`
- 다익스트라에서 `if cost > dist[node]: continue` 빠뜨리면 시간초과
- 최상단 값 확인은 되지만 중간 인덱스 접근은 의미 없음 (정렬된 배열 아님)
- heapify는 O(n)이지만 매번 push하면 O(n log n) → 한꺼번에 만들 땐 생성자/heapify가 유리
- **C++ `priority_queue::pop()`은 void** → `top()`으로 먼저 읽기
