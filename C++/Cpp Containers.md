## std::vector

**1. 설명**

- 메모리에 연속적으로 할당 -> Random Access 가능
- 원소를 집어넣다가 메모리 공간이 부족하면 1.5배, 2배 만큼 새로 메모리를 할당 후 복사.
- reserve : capacity를 확보함, resize : size만큼 공간을 할당하고 초기화함.(실제로 원소가 들어감. -1인가 0인가)
- Random Access가 가능하기 때문에 특정 위치의 원소에 접근하는게 빠름.
- Random Access가 가능하기 때문에 binary search 가능. (물론 정렬이 되어있어야 함.) -> O(1) ~ O(logn)에 탐색 가능
- 연속적인 메모리 공간을 사용하기 때문에 중간에 insert, delete 연산이 느림. insert는 넣고 뒤로 밀어야되고, delete는 빼고 당겨야함.

**2. 성능**

- access : O(1)
- insert : push_back O(1), insert(iter, value) O(n)
- delete : O(n)
- search : binary search - O(1) ~ O(logn), 정렬이 안되있을 경우 O(n)
- sort   : quick sort - O(nlogn) 사용 가능

---

## std::list

**1. 설명**

- 메모리 공간에 불연속적으로 할당. 기존 linked-list와 같이 다음 node의 포인터는 그 전 node가 가지고 있음.
- 그래서 Random Access 안됨 -> 원소 접근 느림, search 느림(sequencial search만 가능)
- insert, delete가 빠름.

**2. 성능**
- access : O(n)
- insert : O(1)
- delete : O(1)
- search : sequencial search - O(n)
- sort   : std::sort(quick sort) 사용 안됨, list::sort(merge sort) 사용, O(nlogn)

---

## std::map

**1. 설명**
- key-value 자료구조. 
- key가 정렬되어 있다. 정렬은 Red-black tree의 정렬 알고리즘을 사용. 물론 구조도 tree 구조임.
- 모든 작업의 성능은 tree의 depth에 비례해서 O(logn) ~ O(n) (worst : 모든 노드를 방문)

**2. 성능**
- access : O(logn)
- insert : O(logn)
- delete : O(logn)
- search : O(logn)

**왜 tree의 depth는 시간복잡도 O(logn)으로 계산되는가?**

tree의 성능은 결국 얼마나 많은 노드를 방문하느냐 이다.
binary tree의 노드 수는 tree의 높이가 증가할 때마다 두배로 늘어난다. 
그렇다면 총 노드 수(n)는 다음과 같다.

n = 1 + 2^1 + 2^2 + 2^3 + ... + 2^h
얘를 간단히 정리한 후 로그를 취하면 시간복잡도 O(logn)이 된다.

reference : http://www.cs.gettysburg.edu/~ilinkin/courses/Fall-2012/cs216/notes/bintree.pdf

---

## std::unordered_map
**1. 설명**
- key-value 자료구조
- 이름 그대로 정렬되있지 않은 map이고, hash table로 구현되어 있다.
- hast table의 특성대로, 성능은 O(1) ~ O(n)이다. 보통 O(1)이다.

**2. 성능**
- access : O(1) ~ O(n)
- insert : O(1) ~ O(n)
- delete : O(1) ~ O(n)
- search : O(1) ~ O(n)

**Hash Collision Handling?**

all practical implementations of std::unordered_set (or unordered_map) undoubtedly use collision chaining. While it might be (just barely) possible to meet the requirements using linear probing or double hashing, such an implementation seems to lose a great deal and gain nearly nothing in return.

요약 : collision chaining을 사용한다.

체이닝은 오버플로우 문제를 해시테이블의 구조변경을 통해 연결리스트로 해결하는 방법이다. 즉, 각 버킷에 
고정된 슬롯을 할당하는 것이 아니라 각 버킷에, 삽입과 삭제가 용이한 연결리스트를 할당한다. 버킷 내에서는 
원하는 항목을 찾을때 연결리스트를 순차 탐색하면 된다. 

---

## std::set
**1. 설명**
- 정렬된 자료구조인데 value를 key처럼 쓴다.
- 내부적으로 std::_Tree를 상속받는다, std::_Tree는 map/multimap/set/multiset을 위한 red-black tree이다.
- 성능은 red-black tree와 동일하다.
- map과 다른점은 set은 <key, value>를 pair로 저장하고, map은 [key, value]를 매핑시켜 저장한다.
  그래서 map은 연관 컨테이너고 set은 정렬된 원소의 컬렉션일 뿐이다. 좀 비슷하긴 하다

**2. 성능**
- access : O(logn)
- insert : O(logn)
- delete : O(logn)
- search : O(logn)

---

## std::multi_map
**1. 설명**
- map인데 중복 key를 허용하는 map이다.
- 그래서 map[key] = value 이게 안된다. multimap.emplace(1, 100) 이렇게 해야된다.
- 정확히 말하면 std::pair<key, value>를 원소로 가지는 map이다. (set과 비슷)
- find(key) 했는데 key에 해당하는 value가 여러개면 제일 앞에 있는 원소의 iter가 나온다.

**2. 성능**
- access : O(logn)
- insert : O(logn)
- delete : O(logn)
- search : O(logn)

---

## std::multi_unordered_map
**1. 설명**
- unordered_map의 multi버전이다. 위 설명과 동일

**2. 성능**
- access : O(1) ~ O(n)
- insert : O(1) ~ O(n)
- delete : O(1) ~ O(n)
- search : O(1) ~ O(n)

---

## std::multi_set
**1. 설명**
- set의 multi버전이다. 

**2. 성능**
- access : O(logn)
- insert : O(logn)
- delete : O(logn)
- search : O(logn)

---

## boost::flat_map
**1. 설명**
- map인데 boost 설명에는 fast retrieval of values of another type T based on the keys
  결국 find가 더 빠른 map이다. 

**왜 find가 빠른가?**

boost::flat_map은 원소를 저장할 때 sorted vector(정렬된 vector) 형식으로 저장하기 때문이다.
sorted vector는 원소를 find를 하는데 binary search를 사용할 수 있다.(sorted + random access 가능이니까)
binary search는 시간복잡도가 O(1) ~ O(logn)이다.
기존의 map은 red black tree 구조이고, red black tree는 원소를 find하는데 tree의 depth에 비례해서 O(logn)이다.
이 때문에 boost::flat_map의 find 속도가 빠른 것이다.

그리고 메모리 사용 공간복잡도 면에서도 vector가 map보다 우수하다. 메모리에 연속적으로 할당되니까 캐시 지역성이 확보된다.
(서치 속도보다 이게 클듯?)

**단점**

find를 빠르게 한 대신, insert, delete 속도를 잃었다. sorted vector는 결국 vector. vector는 insert, delete 비용이
O(n)이다. 반면 map의 insert, delete 비용은 O(logn). 삽입/삭제를 버리고 검색을 취하겠다는 마인드.
boost 홈페이지에는 어디에 삽입되는가에 따라 complexity가 다르다고 나와있다.
access는 왜 O(logn)이지.. sorted vector라면 O(n)이 맞는데

**2. 성능**
- access : O(logn)
- insert : O(logn) ~ O(n)
- delete : O(logn) ~ O(n)
- search : O(1) ~ O(logn)

---

## boost::multi_index
**1. 설명**
- key 중복여부를 선택할 수 있고, ordered/hash를 선택할 수 있는 선택적 자료구조이다.
- 원소의 종류를 어떻게 하냐에 따라 성능이 천차만별이 될 수 있다.

**2. 성능**
- 원소의 속성(if key unique, ordered or hash)에 따라 다름