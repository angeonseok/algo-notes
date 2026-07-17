# Hash (해시 - set / dict)

> 한 줄 정리: 키를 해시 함수로 변환해 O(1)에 저장/탐색/삭제하는 구조, Python은 set/dict, C++은 unordered_set/unordered_map

> C++ 코드는 **C++14** 기준, `using namespace std;` 전제. 필요한 헤더: `<unordered_set>`, `<unordered_map>`, `<vector>`, `<string>`

## 1. 언제 쓰는가
- 빠른 탐색/삽입/삭제가 필요할 때 (O(1))
- 중복 제거
- 방문 여부 체크 (visited)
- 값의 등장 횟수 카운트
- 두 집합의 교집합/합집합/차집합

## 2. 핵심 아이디어
- 해시 함수: 키 → 인덱스로 변환
- 같은 키는 항상 같은 인덱스 → O(1) 접근
- 충돌(collision) 발생 가능 → 최악의 경우 O(n)이지만 평균 O(1)
- `set`/`unordered_set` = 키만 저장, `dict`/`unordered_map` = 키-값 쌍 저장
- **정렬이 필요하면** Python `sorted()` / C++ `set`·`map`(균형 트리, O(log n))

## 3. 시간복잡도
| 연산 | 평균 | 최악 |
|------|------|------|
| 삽입 | O(1) | O(n) |
| 탐색 | O(1) | O(n) |
| 삭제 | O(1) | O(n) |

- 최악은 해시 충돌이 심할 때, 실제로는 거의 O(1)
- 순서 있는 `set`/`map`(트리)은 항상 O(log n)

## 4. 기본 코드

### set

#### Python
```python
s = set()

#삽입
s.add(1)
s.add(2)

#중복은 걍 무시됨
s.add(2)

#탐색은 O(1)
if 1 in s:
    print("있음")

#remove는 없으면 KeyError
s.remove(1)

#discard는 없어도 에러 안 남
s.discard(1)

#집합 연산
a = {1, 2, 3}
b = {2, 3, 4}
a | b   #합집합 {1, 2, 3, 4}
a & b   #교집합 {2, 3}
a - b   #차집합 {1}
a ^ b   #대칭 차집합 {1, 4}

#초기화
s = set([1, 2, 3])
s = {1, 2, 3}
```

#### C++
```cpp
unordered_set<int> s;

//삽입
s.insert(1);
s.insert(2);

//중복은 걍 무시됨
s.insert(2);

//탐색은 평균 O(1)
if (s.count(1)) { /* 있음 */ }

//erase는 없어도 에러 안 남
s.erase(1);

//집합 연산은 내장 연산자가 없으니까 순회하며 구현
//set_intersection 같은 거 쓰려면 정렬된 set<int>나 정렬된 vector 필요

//초기화
unordered_set<int> s2 = {1, 2, 3};
```

### dict

#### Python
```python
d = {}

#삽입 / 수정
d['a'] = 1
d['b'] = 2

#탐색은 O(1)
if 'a' in d:
    print(d['a'])

#get 쓰면 없는 키여도 KeyError 안 터짐
d.get('a')          #없으면 None
d.get('z', 0)       #없으면 기본값 0

#del은 없으면 KeyError
del d['a']

#pop에 기본값 주면 없어도 에러 안 남
d.pop('a', None)

#키만 순회
for k in d:
    print(k, d[k])

#키-값 같이 순회
for k, v in d.items():
    print(k, v)

#등장 횟수 카운트
from collections import Counter
arr = [1, 2, 2, 3, 3, 3]
cnt = Counter(arr)  #Counter({3: 3, 2: 2, 1: 1})
cnt[3]              #3
cnt.most_common(2)  #[(3, 3), (2, 2)]

#없는 키 기본값 미리 정해두면 KeyError 신경 안 써도 됨
from collections import defaultdict
d = defaultdict(int)    #없는 키 접근 시 0
d = defaultdict(list)   #없는 키 접근 시 []
d['a'] += 1
```

#### C++
```cpp
unordered_map<string,int> d;

//삽입 / 수정
d["a"] = 1;
d["b"] = 2;

//탐색
if (d.count("a")) cout << d["a"];

//operator[]는 없는 키를 0으로 만들어버림. 안 만들려면 find 쓰자
auto it = d.find("z");
if (it == d.end()) { /* 없음 */ }

//erase는 없어도 에러 안 남
d.erase("a");

//C++14라 구조적 바인딩 없음. .first/.second로 까기
for (auto& kv : d)
    cout << kv.first << " " << kv.second << "\n";

//Counter 대용. operator[]가 없는 키를 0에서 시작해주니까 걍 ++ 박기
vector<int> arr = {1, 2, 2, 3, 3, 3};
unordered_map<int,int> cnt;
for (int x : arr) cnt[x]++;
cout << cnt[3];   //3
```

## 5. 실전 패턴

### 중복 제거

#### Python
```python
arr = [1, 2, 2, 3, 3, 3]

#set 거치면 중복 날아감. 대신 순서 보장 안 됨
unique = list(set(arr))
```

#### C++
```cpp
vector<int> arr = {1, 2, 2, 3, 3, 3};

//set 거치면 중복 날아감. 대신 순서 보장 안 됨
unordered_set<int> st(arr.begin(), arr.end());
vector<int> unique(st.begin(), st.end());
```

### 방문 체크 (visited)

#### Python
```python
visited = set()
if node not in visited:
    visited.add(node)
```

#### C++
```cpp
unordered_set<int> visited;
if (!visited.count(node)) visited.insert(node);
```

### 두 리스트 공통 원소

#### Python
```python
a = [1, 2, 3, 4]
b = [3, 4, 5, 6]
common = set(a) & set(b)  #{3, 4}
```

#### C++
```cpp
vector<int> a = {1, 2, 3, 4}, b = {3, 4, 5, 6};

//한쪽만 set으로 만들어두면 다른 쪽 훑으면서 O(1)로 확인 가능
unordered_set<int> sa(a.begin(), a.end());
vector<int> common;
for (int x : b) if (sa.count(x)) common.push_back(x);   //{3, 4}
```

### 등장 횟수 카운트

#### Python
```python
from collections import Counter
s = "abracadabra"
cnt = Counter(s)
cnt['a']  #5
```

#### C++
```cpp
string s = "abracadabra";
unordered_map<char,int> cnt;
for (char c : s) cnt[c]++;
cout << cnt['a'];   //5
```

### anagram 판별

#### Python
```python
#순서 상관없이 글자 개수만 똑같으면 애너그램
def is_anagram(s, t):
    return Counter(s) == Counter(t)
```

#### C++
```cpp
//순서 상관없이 글자 개수만 똑같으면 애너그램
bool isAnagram(const string& s, const string& t) {
    //길이부터 다르면 볼 것도 없음
    if (s.size() != t.size()) return false;

    //s 글자 수 먼저 세두고
    unordered_map<char,int> cnt;
    for (char c : s) cnt[c]++;

    //t로 빼보다가 음수 뜨면 t에만 있는 글자니까 탈락
    for (char c : t) if (--cnt[c] < 0) return false;
    return true;
}
```

## 6. 이걸 떠올려야 할 때
- "탐색을 자주 해야 한다" → list 대신 set/dict (O(n) → O(1))
- "중복을 없애야 한다" → set
- "등장 횟수가 필요하다" → Counter / unordered_map 카운팅
- "그래프/BFS에서 방문 체크" → set이 list보다 빠름
- "없는 키도 안전하게 접근" → defaultdict/get() / unordered_map operator[]

## 7. 자주 틀리는 포인트
- `set`/`unordered_set`은 순서 보장 안 됨 → 순서 필요하면 정렬 or ordered `set`(트리)
- Python `dict`는 3.7부터 삽입 순서 보장 (C++ unordered_map은 순서 없음)
- `remove()`는 없으면 KeyError → 불확실하면 `discard()` / C++ `erase`는 없어도 안전
- **C++ `unordered_map::operator[]`는 없는 키를 자동 생성** → 탐색만 할 땐 `count`/`find` 사용
- 선형 탐색(`in` / 순회)은 O(n) → 자주 탐색하면 해시로 변환
