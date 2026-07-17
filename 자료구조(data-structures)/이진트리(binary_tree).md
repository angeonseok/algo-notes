# Binary Tree / BST (이진트리 / 이진 탐색 트리)

> 한 줄 정리: 자식이 최대 2개인 트리, BST는 왼쪽 < 루트 < 오른쪽 규칙으로 탐색을 O(log n)으로 만든다

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<set>`, `<algorithm>` (`max`), `<climits>`

## 1. 언제 쓰는가
- 정렬된 데이터를 빠르게 탐색/삽입/삭제할 때
- 범위 내 값을 찾아야 할 때
- 중위 순회로 정렬된 순서가 필요할 때

## 2. 핵심 아이디어
- **이진트리**: 모든 노드의 자식이 최대 2개
- **BST 규칙**: 왼쪽 자식 < 부모 < 오른쪽 자식
- BST에서 중위 순회 = 오름차순 정렬
- Python은 BST 내장 없음 → `sortedcontainers.SortedList` / **C++은 `std::set`/`std::map`이 곧 균형 BST**

## 3. 시간복잡도
| 연산 | 평균 | 최악 (편향 트리) |
|------|------|-----------------|
| 탐색 | O(log n) | O(n) |
| 삽입 | O(log n) | O(n) |
| 삭제 | O(log n) | O(n) |

- 최악은 입력이 이미 정렬된 경우 → 한쪽으로 편향됨 (C++ `set`은 레드블랙트리라 항상 O(log n))

## 4. 기본 코드

### 노드 정의

#### Python
```python
class TreeNode:
    def __init__(self, val):
        self.val = val
        self.left = None
        self.right = None
```

#### C++
```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int v) : val(v), left(nullptr), right(nullptr) {}
};
```

### BST 삽입

#### Python
```python
def insert(root, val):
    #빈 자리 만나면 거기다 새 노드 박기
    if not root:
        return TreeNode(val)

    #작으면 왼쪽, 크거나 같으면 오른쪽으로 내려보내자
    if val < root.val:
        root.left = insert(root.left, val)
    else:
        root.right = insert(root.right, val)
    return root
```

#### C++
```cpp
TreeNode* insert(TreeNode* root, int val) {
    //빈 자리 만나면 거기다 새 노드 박기
    if (!root) return new TreeNode(val);

    //작으면 왼쪽, 크거나 같으면 오른쪽으로 내려보내자
    if (val < root->val) root->left = insert(root->left, val);
    else                 root->right = insert(root->right, val);
    return root;
}
```

### BST 탐색

#### Python
```python
def search(root, val):
    #끝까지 갔는데 못 찾으면 없는 거
    if not root:
        return False

    #찾으면 바로 True
    if root.val == val:
        return True

    #작으면 왼쪽, 크면 오른쪽만 보면 되니까 반씩 쳐냄
    if val < root.val:
        return search(root.left, val)
    return search(root.right, val)
```

#### C++
```cpp
bool search(TreeNode* root, int val) {
    //끝까지 갔는데 못 찾으면 없는 거
    if (!root) return false;

    //찾으면 바로 true
    if (root->val == val) return true;

    //작으면 왼쪽, 크면 오른쪽만 보면 되니까 반씩 쳐냄
    if (val < root->val) return search(root->left, val);
    return search(root->right, val);
}
```

### BST 삭제

#### Python
```python
def delete(root, val):
    #없는 값이면 걍 스킵
    if not root:
        return None

    #지울 놈 찾을 때까지 작으면 왼쪽, 크면 오른쪽으로 내려가자
    if val < root.val:
        root.left = delete(root.left, val)
    elif val > root.val:
        root.right = delete(root.right, val)
    else:
        #자식이 하나 이하면 있는 자식을 걍 끌어올리면 끝
        if not root.left:
            return root.right
        if not root.right:
            return root.left

        #자식이 둘이면 오른쪽 서브트리 최솟값으로 값만 갈아끼우고, 그 최솟값을 아래서 지우자
        min_node = root.right
        while min_node.left:
            min_node = min_node.left
        root.val = min_node.val
        root.right = delete(root.right, min_node.val)
    return root
```

#### C++
```cpp
//delete는 예약어라 deleteNode로
TreeNode* deleteNode(TreeNode* root, int val) {
    //없는 값이면 걍 스킵
    if (!root) return nullptr;

    //지울 놈 찾을 때까지 작으면 왼쪽, 크면 오른쪽으로 내려가자
    if (val < root->val)      root->left = deleteNode(root->left, val);
    else if (val > root->val) root->right = deleteNode(root->right, val);
    else {
        //자식이 하나 이하면 있는 자식을 걍 끌어올리면 끝
        if (!root->left)  return root->right;
        if (!root->right) return root->left;

        //자식이 둘이면 오른쪽 서브트리 최솟값으로 값만 갈아끼우고, 그 최솟값을 아래서 지우자
        TreeNode* minNode = root->right;
        while (minNode->left) minNode = minNode->left;
        root->val = minNode->val;
        root->right = deleteNode(root->right, minNode->val);
    }
    return root;
}
```

## 5. 실전 패턴

### 코테에서 BST 대신 쓰는 것들

#### Python
```python
#정렬된 리스트만 있으면 될 때는 bisect로 충분
import bisect
arr = []

#삽입은 밀어야 되니까 O(n)이지만 위치 탐색 자체는 O(log n)
bisect.insort(arr, val)
idx = bisect.bisect_left(arr, val)

#삽입/삭제가 많으면 SortedList가 나음
from sortedcontainers import SortedList
sl = SortedList()
sl.add(val)
sl.remove(val)

#앞이 최솟값, 뒤가 최댓값
sl[0]
sl[-1]
```

#### C++
```cpp
//C++은 set/map이 곧 균형 BST(레드블랙트리)라 삽입/삭제/탐색 다 O(log n)
set<int> s;
s.insert(val);
s.erase(val);

//앞이 최솟값, 뒤가 최댓값
*s.begin();
*s.rbegin();

//val 이상인 첫 원소 반복자
s.lower_bound(val);

//중복 허용이 필요하면 multiset 쓰자
```

### 트리 높이 구하기

#### Python
```python
#자식 둘 중 더 깊은 쪽에 자기 자신 한 층 더하기
def height(root):
    if not root:
        return 0
    return 1 + max(height(root.left), height(root.right))
```

#### C++
```cpp
//자식 둘 중 더 깊은 쪽에 자기 자신 한 층 더하기
int height(TreeNode* root) {
    if (!root) return 0;
    return 1 + max(height(root->left), height(root->right));
}
```

### 트리가 BST인지 확인

#### Python
```python
#내려가면서 각 노드가 들어갈 수 있는 (min, max) 범위를 좁혀가며 검사하자
def is_bst(node, min_val=float('-inf'), max_val=float('inf')):
    if not node:
        return True

    #허용 범위 벗어나면 BST 아님
    if not (min_val < node.val < max_val):
        return False

    #왼쪽은 상한을 현재 값으로, 오른쪽은 하한을 현재 값으로 조여줌
    return (is_bst(node.left, min_val, node.val) and
            is_bst(node.right, node.val, max_val))
```

#### C++
```cpp
//내려가면서 각 노드가 들어갈 수 있는 (min, max) 범위를 좁혀가며 검사하자
bool isBST(TreeNode* node, long long minVal = LLONG_MIN, long long maxVal = LLONG_MAX) {
    if (!node) return true;

    //허용 범위 벗어나면 BST 아님
    if (!(minVal < node->val && node->val < maxVal)) return false;

    //왼쪽은 상한을 현재 값으로, 오른쪽은 하한을 현재 값으로 조여줌
    return isBST(node->left,  minVal, node->val)
        && isBST(node->right, node->val, maxVal);
}
```

## 6. 이걸 떠올려야 할 때
- "정렬된 상태로 삽입/삭제/탐색" → BST / `SortedList` / C++ `set`
- "중위 순회 결과가 정렬" → BST 성질 활용
- "트리에서 최솟값/최댓값" → BST면 가장 왼쪽/오른쪽 노드

## 7. 자주 틀리는 포인트
- 편향 트리 주의 → 정렬된 입력을 직접 구현 BST에 그냥 삽입하면 O(n)
- 삭제할 때 자식이 둘인 경우 처리가 제일 까다로움 → 오른쪽 서브트리 최솟값으로 대체
- Python에 BST 내장 없음 → `bisect` 또는 `SortedList` / C++은 `set`·`map`으로 바로 해결
- `bisect.insort`는 삽입이 O(n)이라 대량 삽입엔 `SortedList` / C++ `set`이 나음
