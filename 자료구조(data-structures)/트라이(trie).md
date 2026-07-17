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
        #자식은 {문자: 노드} 로 들고 있자
        self.children = {}

        #여기서 단어가 끝나는지 표시
        self.is_end = False

class Trie:
    def __init__(self):
        #루트는 빈 노드. 아무 문자도 안 가짐
        self.root = TrieNode()

    def insert(self, word):
        node = self.root

        #한 글자씩 타고 내려가면서 없으면 새로 파기
        for c in word:
            if c not in node.children:
                node.children[c] = TrieNode()
            node = node.children[c]

        #끝까지 내려왔으니 단어 끝 표시
        node.is_end = True

    def search(self, word):
        node = self.root

        #중간에 길 끊기면 그런 단어 없음
        for c in word:
            if c not in node.children:
                return False
            node = node.children[c]

        #길은 있어도 단어 끝이 아니면 접두사일 뿐
        return node.is_end

    def starts_with(self, prefix):
        node = self.root

        #끝까지 길만 이어지면 됨. is_end는 안 봄
        for c in prefix:
            if c not in node.children:
                return False
            node = node.children[c]
        return True

#사용
trie = Trie()
trie.insert("apple")
trie.insert("app")
trie.search("apple")      #True
trie.search("ap")         #False (끝이 아님)
trie.starts_with("app")   #True
```

#### C++
```cpp
struct TrieNode {
    //자식은 {문자: 노드} 로 들고 있자
    unordered_map<char, TrieNode*> children;

    //여기서 단어가 끝나는지 표시
    bool is_end = false;
};

struct Trie {
    //루트는 빈 노드. 아무 문자도 안 가짐
    TrieNode* root;
    Trie() { root = new TrieNode(); }

    void insert(const string& word) {
        TrieNode* node = root;

        //한 글자씩 타고 내려가면서 없으면 새로 파기
        for (char c : word) {
            if (!node->children.count(c))
                node->children[c] = new TrieNode();
            node = node->children[c];
        }

        //끝까지 내려왔으니 단어 끝 표시
        node->is_end = true;
    }

    bool search(const string& word) {
        TrieNode* node = root;

        //중간에 길 끊기면 그런 단어 없음
        for (char c : word) {
            if (!node->children.count(c)) return false;
            node = node->children[c];
        }

        //길은 있어도 단어 끝이 아니면 접두사일 뿐
        return node->is_end;
    }

    bool startsWith(const string& prefix) {
        TrieNode* node = root;

        //끝까지 길만 이어지면 됨. is_end는 안 봄
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
        #a~z 26칸 미리 잡아두면 딕셔너리보다 빠름
        self.children = [None] * 26

        #여기서 단어가 끝나는지 표시
        self.is_end = False

    def insert(self, word):
        node = self

        #문자를 0~25 인덱스로 바꿔서 타고 내려가기
        for c in word:
            idx = ord(c) - ord('a')
            if not node.children[idx]:
                node.children[idx] = Trie()
            node = node.children[idx]

        #끝까지 내려왔으니 단어 끝 표시
        node.is_end = True

    def search(self, word):
        node = self

        #중간에 길 끊기면 그런 단어 없음
        for c in word:
            idx = ord(c) - ord('a')
            if not node.children[idx]:
                return False
            node = node.children[idx]

        #길은 있어도 단어 끝이 아니면 접두사일 뿐
        return node.is_end
```

#### C++
```cpp
struct Trie {
    //a~z 26칸 미리 잡아두면 해시맵보다 빠름
    Trie* children[26] = { nullptr };

    //여기서 단어가 끝나는지 표시
    bool is_end = false;

    void insert(const string& word) {
        Trie* node = this;

        //문자를 0~25 인덱스로 바꿔서 타고 내려가기
        for (char c : word) {
            int idx = c - 'a';
            if (!node->children[idx]) node->children[idx] = new Trie();
            node = node->children[idx];
        }

        //끝까지 내려왔으니 단어 끝 표시
        node->is_end = true;
    }

    bool search(const string& word) {
        Trie* node = this;

        //중간에 길 끊기면 그런 단어 없음
        for (char c : word) {
            int idx = c - 'a';
            if (!node->children[idx]) return false;
            node = node->children[idx];
        }

        //길은 있어도 단어 끝이 아니면 접두사일 뿐
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

        #이 노드를 지나간 단어 수. 이게 곧 접두사 개수
        self.count = 0

        #여기서 단어가 끝나는지 표시
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root

        #내려가면서 지나는 노드마다 카운트 올려두자
        for c in word:
            if c not in node.children:
                node.children[c] = TrieNode()
            node = node.children[c]
            node.count += 1

        #끝까지 내려왔으니 단어 끝 표시
        node.is_end = True

    def count_prefix(self, prefix):
        node = self.root

        #길 끊기면 이 접두사로 시작하는 단어 없음
        for c in prefix:
            if c not in node.children:
                return 0
            node = node.children[c]

        #insert 때 세둔 값 그대로 꺼내 쓰기
        return node.count
```

#### C++
```cpp
struct TrieNode {
    unordered_map<char, TrieNode*> children;

    //이 노드를 지나간 단어 수. 이게 곧 접두사 개수
    int count = 0;

    //여기서 단어가 끝나는지 표시
    bool is_end = false;
};

struct Trie {
    TrieNode* root;
    Trie() { root = new TrieNode(); }

    void insert(const string& word) {
        TrieNode* node = root;

        //내려가면서 지나는 노드마다 카운트 올려두자
        for (char c : word) {
            if (!node->children.count(c))
                node->children[c] = new TrieNode();
            node = node->children[c];
            node->count++;
        }

        //끝까지 내려왔으니 단어 끝 표시
        node->is_end = true;
    }

    int countPrefix(const string& prefix) {
        TrieNode* node = root;

        //길 끊기면 이 접두사로 시작하는 단어 없음
        for (char c : prefix) {
            if (!node->children.count(c)) return 0;
            node = node->children[c];
        }

        //insert 때 세둔 값 그대로 꺼내 쓰기
        return node->count;
    }
};
```

### XOR 최댓값 (비트 트라이)

#### Python
```python
class BitTrie:
    def __init__(self):
        #문자 대신 비트 0/1 이니까 자식은 두 개면 끝
        self.children = [None, None]

    def insert(self, num, bit=29):
        node = self

        #높은 비트부터 박아야 나중에 큰 자리부터 고를 수 있음
        for i in range(bit, -1, -1):
            b = (num >> i) & 1
            if not node.children[b]:
                node.children[b] = BitTrie()
            node = node.children[b]

    def max_xor(self, num, bit=29):
        node = self
        result = 0

        #높은 자리부터 욕심내는 그리디
        for i in range(bit, -1, -1):
            b = (num >> i) & 1

            #XOR은 다른 비트끼리 만나야 1이니까 반대쪽을 원함
            want = 1 - b

            #반대쪽 길 있으면 이 자리 1 챙기고, 없으면 걍 같은 쪽으로
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
    //문자 대신 비트 0/1 이니까 자식은 두 개면 끝
    BitTrie* children[2] = { nullptr, nullptr };

    void insert(int num, int bit = 29) {
        BitTrie* node = this;

        //높은 비트부터 박아야 나중에 큰 자리부터 고를 수 있음
        for (int i = bit; i >= 0; i--) {
            int b = (num >> i) & 1;
            if (!node->children[b]) node->children[b] = new BitTrie();
            node = node->children[b];
        }
    }

    int maxXor(int num, int bit = 29) {
        BitTrie* node = this;
        int result = 0;

        //높은 자리부터 욕심내는 그리디
        for (int i = bit; i >= 0; i--) {
            int b = (num >> i) & 1;

            //XOR은 다른 비트끼리 만나야 1이니까 반대쪽을 원함
            int want = 1 - b;

            //반대쪽 길 있으면 이 자리 1 챙기고, 없으면 걍 같은 쪽으로
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
