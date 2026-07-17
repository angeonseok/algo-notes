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

#push
heapq.heappush(heap, 3)
heapq.heappush(heap, 1)
heapq.heappush(heap, 2)

#pop하면 항상 최솟값이 나옴
val = heapq.heappop(heap)  #1

#제거 안 하고 최솟값만 보기
val = heap[0]

#하나씩 push하면 O(n log n). 한꺼번에 만들 땐 heapify가 O(n)이라 이득
arr = [3, 1, 2]
heapq.heapify(arr)

#heapq는 최소 힙뿐. 최대 힙 쓰려면 부호 뒤집어서 넣자
heapq.heappush(heap, -3)
max_val = -heapq.heappop(heap)  #3
```

#### C++
```cpp
//priority_queue는 최대 힙이 기본. greater 껴야 최소 힙 됨
priority_queue<int, vector<int>, greater<int>> heap;

//push
heap.push(3);
heap.push(1);
heap.push(2);

//pop()은 값을 안 돌려주니까 top()으로 먼저 읽고 pop
int val = heap.top();   //1
heap.pop();

//생성자에 반복자 넘기면 O(n). 하나씩 push하는 것보다 이득
vector<int> arr = {3, 1, 2};
priority_queue<int, vector<int>, greater<int>> heap2(arr.begin(), arr.end());

//최대 힙은 걍 기본값이라 비교자 없이 끝
priority_queue<int> maxHeap;
maxHeap.push(3);
int maxVal = maxHeap.top();   //3
```

## 5. 실전 패턴

### K번째 최솟값

#### Python
```python
import heapq

#k번째로 작은 값 뽑기
def kth_smallest(arr, k):
    heapq.heapify(arr)

    #앞의 k-1개는 버리고
    for _ in range(k - 1):
        heapq.heappop(arr)

    #그다음 나오는 게 k번째 최솟값
    return heapq.heappop(arr)
```

#### C++
```cpp
//k번째로 작은 값 뽑기
int kthSmallest(vector<int>& arr, int k) {
    priority_queue<int, vector<int>, greater<int>> heap(arr.begin(), arr.end());

    //앞의 k-1개는 버리고
    for (int i = 0; i < k - 1; i++) heap.pop();

    //그다음 top이 k번째 최솟값
    return heap.top();
}
```

### 튜플로 우선순위 지정

#### Python
```python
import heapq

heap = []

#(우선순위, 값)으로 넣으면 앞칸 기준으로 알아서 정렬됨
heapq.heappush(heap, (1, 'task_a'))
heapq.heappush(heap, (3, 'task_c'))
heapq.heappush(heap, (2, 'task_b'))

priority, task = heapq.heappop(heap)  #(1, 'task_a')
```

#### C++
```cpp
//pair는 first 기준으로 비교되니까 (우선순위, 값)으로 넣자. greater<>는 C++14
priority_queue<pair<int,string>, vector<pair<int,string>>, greater<>> heap;
heap.push({1, "task_a"});
heap.push({3, "task_c"});
heap.push({2, "task_b"});

auto top = heap.top();     //{1, "task_a"}
int priority = top.first;
string task = top.second;
```

### 다익스트라

#### Python
```python
import heapq

#시작점 > 모든 정점까지 최단거리 구하기
def dijkstra(graph, start):
    dist = {node: float('inf') for node in graph}
    dist[start] = 0

    #(거리, 노드) 형태로 힙에 담기
    heap = [(0, start)]

    #다익스트라는 힙을 활용
    while heap:
        d, u = heapq.heappop(heap)

        #dist가 최신 정보. d가 더 크면 구버전이니 스킵
        if d > dist[u]:
            continue

        #거리정보 갱신
        for v, w in graph[u]:
            nd = d + w

            #갱신 가능하면 힙에 다시 넣자
            if nd < dist[v]:
                dist[v] = nd
                heapq.heappush(heap, (nd, v))

    return dist
```

#### C++
```cpp
//시작점 > 모든 정점까지 최단거리 구하기
vector<long long> dijkstra(vector<vector<pair<int,int>>>& graph, int start) {
    int n = graph.size();
    vector<long long> dist(n, LLONG_MAX);
    dist[start] = 0;

    //(거리, 노드) 최소 힙
    priority_queue<pair<long long,int>, vector<pair<long long,int>>, greater<>> heap;
    heap.push({0, start});

    //다익스트라는 힙을 활용
    while (!heap.empty()) {
        long long d = heap.top().first;
        int u = heap.top().second;
        heap.pop();

        //dist가 최신 정보. d가 더 크면 구버전이니 스킵
        if (d > dist[u]) continue;

        //거리정보 갱신
        for (auto& e : graph[u]) {
            int v = e.first, w = e.second;
            long long nd = d + w;

            //갱신 가능하면 힙에 다시 넣자
            if (nd < dist[v]) {
                dist[v] = nd;
                heap.push({nd, v});
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

#왼쪽 절반은 최대 힙(음수 저장), 오른쪽 절반은 최소 힙. 가운데서 만나게 하기
left, right = [], []

def add(num):
    #일단 왼쪽에 넣고
    heapq.heappush(left, -num)

    #왼쪽 최댓값을 오른쪽으로 넘겨서 경계 정리
    heapq.heappush(right, -heapq.heappop(left))

    #왼쪽이 더 작아지면 다시 하나 당겨와서 균형 맞추기
    if len(left) < len(right):
        heapq.heappush(left, -heapq.heappop(right))

#왼쪽 꼭대기가 곧 중앙값
def get_median():
    return -left[0]
```

#### C++
```cpp
//왼쪽 절반은 최대 힙, 오른쪽 절반은 최소 힙. 가운데서 만나게 하기
priority_queue<int> left;                               //최대 힙
priority_queue<int, vector<int>, greater<int>> right;   //최소 힙

void add(int num) {
    //일단 왼쪽에 넣고
    left.push(num);

    //왼쪽 최댓값을 오른쪽으로 넘겨서 경계 정리
    right.push(left.top()); left.pop();

    //왼쪽이 더 작아지면 다시 하나 당겨와서 균형 맞추기
    if (left.size() < right.size()) {
        left.push(right.top()); right.pop();
    }
}

//왼쪽 꼭대기가 곧 중앙값
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
