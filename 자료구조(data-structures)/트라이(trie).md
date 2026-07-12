# Trie (트라이)

> 한 줄 정리: 문자열을 트리 구조로 저장해 O(L)에 탐색/삽입하는 자료구조 (L = 문자열 길이)

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<unordered_map>`, `<string>`

## 1. 언제 쓰는가
- 문자열 집합에서 특정 문자열 탐색
- 접두사(prefix) 검색
- 자동완성
- 문자열 정렬

## 2. 핵심 아이디어
- 각 노드가 문자 하나를 나타냄
- 루트에서 리프까지의 경로가 하나의 문자열
- 공통 접두사를 공유 → 메모리 효율적
- 탐색/삽입 모두 O(L)

## 3. 시간/공간 복잡도
| 연산 | 복잡도 |
|------|--------|
| 삽입 | O(L) |
| 탐색 | O(L) |
| 공간 | O(알파벳 수 × 노드 수) |

## 4. 기본 코드

### 딕셔너리/해시맵 기반 구현 (간단)

#### Python
```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False  # 단어의 끝 여부

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for c in word:
            if c not in node.children:
                node.children[c] = TrieNode()
            node = node.children[c]
        node.is_end = True

    def search(self, word):
        node = self.root
        for c in word:
            if c not in node.children:
                return False
            node = node.children[c]
        return node.is_end

    def starts_with(self, prefix):
        node = self.root
        for c in prefix:
            if c not in node.children:
                return False
            node = node.children[c]
        return True

# 사용
trie = Trie()
trie.insert("apple")
trie.insert("app")
trie.search("apple")      # True
trie.search("ap")         # False (끝이 아님)
trie.starts_with("app")   # True
```

#### C++
```cpp
struct TrieNode {
    unordered_map<char, TrieNode*> children;
    bool is_end = false;   // 단어의 끝 여부
};

struct Trie {
    TrieNode* root;
    Trie() { root = new TrieNode(); }

    void insert(const string& word) {
        TrieNode* node = root;
        for (char c : word) {
            if (!node->children.count(c))
                node->children[c] = new TrieNode();
            node = node->children[c];
        }
        node->is_end = true;
    }

    bool search(const string& word) {
        TrieNode* node = root;
        for (char c : word) {
            if (!node->children.count(c)) return false;
            node = node->children[c];
        }
        return node->is_end;
    }

    bool startsWith(const string& prefix) {
        TrieNode* node = root;
        for (char c : prefix) {
            if (!node->children.count(c)) return false;
            node = node->children[c];
        }
        return true;
    }
};
```

### 배열 기반 구현 (빠름, 알파벳 소문자 전용)

#### Python
```python
class Trie:
    def __init__(self):
        self.children = [None] * 26
        self.is_end = False

    def insert(self, word):
        node = self
        for c in word:
            idx = ord(c) - ord('a')
            if not node.children[idx]:
                node.children[idx] = Trie()
            node = node.children[idx]
        node.is_end = True

    def search(self, word):
        node = self
        for c in word:
            idx = ord(c) - ord('a')
            if not node.children[idx]:
                return False
            node = node.children[idx]
        return node.is_end
```

#### C++
```cpp
struct Trie {
    Trie* children[26] = { nullptr };
    bool is_end = false;

    void insert(const string& word) {
        Trie* node = this;
        for (char c : word) {
            int idx = c - 'a';
            if (!node->children[idx]) node->children[idx] = new Trie();
            node = node->children[idx];
        }
        node->is_end = true;
    }

    bool search(const string& word) {
        Trie* node = this;
        for (char c : word) {
            int idx = c - 'a';
            if (!node->children[idx]) return false;
            node = node->children[idx];
        }
        return node->is_end;
    }
};
```

## 5. 실전 패턴

### 접두사 카운트

#### Python
```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.count = 0      # 이 노드를 지나는 단어 수
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for c in word:
            if c not in node.children:
                node.children[c] = TrieNode()
            node = node.children[c]
            node.count += 1
        node.is_end = True

    def count_prefix(self, prefix):
        node = self.root
        for c in prefix:
            if c not in node.children:
                return 0
            node = node.children[c]
        return node.count
```

#### C++
```cpp
struct TrieNode {
    unordered_map<char, TrieNode*> children;
    int count = 0;        // 이 노드를 지나는 단어 수
    bool is_end = false;
};

struct Trie {
    TrieNode* root;
    Trie() { root = new TrieNode(); }

    void insert(const string& word) {
        TrieNode* node = root;
        for (char c : word) {
            if (!node->children.count(c))
                node->children[c] = new TrieNode();
            node = node->children[c];
            node->count++;
        }
        node->is_end = true;
    }

    int countPrefix(const string& prefix) {
        TrieNode* node = root;
        for (char c : prefix) {
            if (!node->children.count(c)) return 0;
            node = node->children[c];
        }
        return node->count;
    }
};
```

### XOR 최댓값 (비트 트라이)

#### Python
```python
class BitTrie:
    def __init__(self):
        self.children = [None, None]

    def insert(self, num, bit=29):
        node = self
        for i in range(bit, -1, -1):
            b = (num >> i) & 1
            if not node.children[b]:
                node.children[b] = BitTrie()
            node = node.children[b]

    def max_xor(self, num, bit=29):
        node = self
        result = 0
        for i in range(bit, -1, -1):
            b = (num >> i) & 1
            want = 1 - b  # XOR 최대화를 위해 반대 비트 선택
            if node.children[want]:
                result |= (1 << i)
                node = node.children[want]
            else:
                node = node.children[b]
        return result
```

#### C++
```cpp
struct BitTrie {
    BitTrie* children[2] = { nullptr, nullptr };

    void insert(int num, int bit = 29) {
        BitTrie* node = this;
        for (int i = bit; i >= 0; i--) {
            int b = (num >> i) & 1;
            if (!node->children[b]) node->children[b] = new BitTrie();
            node = node->children[b];
        }
    }

    int maxXor(int num, int bit = 29) {
        BitTrie* node = this;
        int result = 0;
        for (int i = bit; i >= 0; i--) {
            int b = (num >> i) & 1;
            int want = 1 - b;   // XOR 최대화를 위해 반대 비트 선택
            if (node->children[want]) {
                result |= (1 << i);
                node = node->children[want];
            } else {
                node = node->children[b];
            }
        }
        return result;
    }
};
```

## 6. 이걸 떠올려야 할 때
- "문자열 집합에서 접두사 검색" → 트라이
- "자동완성" → 트라이
- "XOR 최댓값" → 비트 트라이
- 단순 문자열 탐색만이면 → set이 더 간단

## 7. 자주 틀리는 포인트
- `is_end` 체크 빠뜨리면 접두사도 단어로 인식
- 배열 기반에서 소문자 외 문자 들어오면 인덱스 오류
- 메모리 많이 씀 → 노드 수 × 알파벳 수만큼 필요
- C++은 `new`로 만든 노드 해제를 신경 써야 하지만, 코테에선 보통 프로그램 종료 시 회수되므로 생략
