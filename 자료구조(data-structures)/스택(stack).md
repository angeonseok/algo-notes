# Stack (스택)

> 한 줄 정리: 가장 최근에 넣은 것을 먼저 꺼내는 구조 (LIFO)

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<stack>`, `<vector>`, `<string>`

## 1. 언제 쓰는가
- LIFO(Last In First Out) 구조가 필요할 때
- 가장 최근에 넣은 값을 먼저 처리해야 할 때
- 괄호 검사 문제
- 문자열 처리 (뒤집기, 폭발 문자열)
- DFS를 재귀 대신 구현할 때
- 상태를 저장했다가 되돌리는 경우 (undo)

## 2. 핵심 아이디어
- 가장 마지막에 넣은 데이터를 먼저 꺼낸다.
- push → 삽입 (`append` / `push`)
- pop → 제거 (`pop`)
- top → 마지막 원소 (`stack[-1]` / `top()`)
- Python은 list, C++은 `stack<T>`로 구현

## 3. 시간복잡도
- push: O(1)
- pop: O(1)
- top: O(1)

## 4. 기본 코드

#### Python
```python
stack = []

#push
stack.append(1)
stack.append(2)

#pop
top = stack.pop()

#top은 맨 뒤 원소
top = stack[-1]

#empty 체크
if not stack:
    print("empty")
```

#### C++
```cpp
stack<int> st;

//push
st.push(1);
st.push(2);

//C++ pop()은 값을 반환 안 하니까 top()으로 먼저 읽고 pop 하자
int top = st.top();
st.pop();

//empty 체크
if (st.empty()) {
    //empty
}
```

## 5. 실전 패턴

### 괄호 검사

#### Python
```python
stack = []
for c in s:
    #여는 괄호는 일단 쌓기
    if c == '(':
        stack.append(c)

    #닫는 괄호인데 짝지을 게 없으면 아웃, 있으면 하나 빼기
    elif c == ')':
        if not stack:
            return False
        stack.pop()

#끝까지 봤는데 남아있으면 짝 안 맞는 거
return not stack
```

#### C++
```cpp
bool isValid(const string& s) {
    stack<char> st;
    for (char c : s) {
        //여는 괄호는 일단 쌓기
        if (c == '(') {
            st.push(c);

        //닫는 괄호인데 짝지을 게 없으면 아웃, 있으면 하나 빼기
        } else if (c == ')') {
            if (st.empty()) return false;
            st.pop();
        }
    }

    //끝까지 봤는데 남아있으면 짝 안 맞는 거
    return st.empty();
}
```

### 단조 스택 (Monotonic Stack)
- 현재 원소보다 크거나 작은 이전 원소를 빠르게 찾을 때
- "오큰수", "히스토그램" 유형에서 자주 등장

#### Python
```python
stack = []
for i, v in enumerate(arr):
    #스택 top보다 현재 값이 크면, 걔의 오큰수가 지금 확정됨
    while stack and arr[stack[-1]] < v:
        idx = stack.pop()
        #arr[idx]의 오른쪽 첫 번째 큰 수 = v

    stack.append(i)
```

#### C++
```cpp
//값 말고 인덱스를 담자
stack<int> st;
for (int i = 0; i < (int)arr.size(); i++) {
    //스택 top보다 현재 값이 크면, 걔의 오큰수가 지금 확정됨
    while (!st.empty() && arr[st.top()] < arr[i]) {
        int idx = st.top(); st.pop();
        //arr[idx]의 오른쪽 첫 번째 큰 수 = arr[i]
    }
    st.push(i);
}
```

### DFS (재귀 대신 스택)

#### Python
```python
stack = [start]
visited = set()
while stack:
    node = stack.pop()

    #이미 방문한 놈이면 스킵
    if node in visited:
        continue
    visited.add(node)

    #이웃들 다 쌓아두면 나중에 하나씩 꺼내지면서 파고 들어감
    for v in graph[node]:
        stack.append(v)
```

#### C++
```cpp
stack<int> st;
st.push(start);

//노드 수 n
vector<bool> visited(n, false);
while (!st.empty()) {
    int node = st.top(); st.pop();

    //이미 방문한 놈이면 스킵
    if (visited[node]) continue;
    visited[node] = true;

    //이웃들 다 쌓아두면 나중에 하나씩 꺼내지면서 파고 들어감
    for (int v : graph[node])
        st.push(v);
}
```

## 6. 이걸 떠올려야 할 때
- "이전 값과 비교해야 한다" → 스택에 이전 값 쌓아두기
- "뒤에서부터 처리해야 한다" → 스택으로 순서 뒤집기
- "열고 닫는 쌍이 있다" → 괄호, 태그 검사
- "가장 가까운 ~를 찾아라" → 단조 스택
- "중간에 뭔가 터지거나 사라진다" → 폭발 문자열 유형

## 7. 자주 틀리는 포인트
- pop() 전에 empty 체크 안 해서 IndexError / 미정의 동작 나는 경우
- **C++ `stack::pop()`은 void** → 값이 필요하면 `top()`으로 먼저 읽고 `pop()`
- top을 확인만 하고 싶은데 실수로 pop() 해버리는 경우
- 단조 스택에서 인덱스를 넣을지 값을 넣을지 헷갈리는 경우 (보통 인덱스를 넣는 게 유연함)
