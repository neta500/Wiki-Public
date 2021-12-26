https://www.youtube.com/watch?v=qNAbGpV1ZkU

https://github.com/hogliux/semimap

## Introduction

```cpp
semi::map<Key, Value>
```

- compile-time 검색(어디에 value가 저장되어있는지 compile-time에 알 수 있음) , run-time 값 저장 및 변경
- 만약 key가 string literal이 아니라면 run-time lookup으로 돌아감
- key를 미리 알 필요 없음

## The basic principle

```cpp
template <int> std::string map;
```

- key : `int`, value : `std::string` 인 map
- compile-time 검색(어디에 value가 저장되어있는지 compile-time에 알 수 있음) , run-time 값 저장 및 변경
- key를 미리 알 필요 없음


일반적으로, 연관 컨테이너는 key로부터 value를 찾는데 runtime overhead를 요구한다.   
하지만, 만약 key를 compile-time에 알 수 있다면(예를 들면 key가 literal인 경우), 이 runtime overhead를 줄일 수 있다.  
이것이 바로 `semi::static_map`, `semi::map`의 목적이다.

사실, C++ literal을 key로 `semi::static_map`을 사용할 때, value 검색은 거의 global variable에 접근하는 것과 같은 효율을 낸다.  
C++ literal을 key로 사용하는 한, 검색 시간은 상수 시간(constant)으로 유지되며, 컨테이너의 key의 수에 따라 증가하지 않는다.  

```cpp
#include <iostream>
#include <string>

#include "semimap.h"

#define ID(x) []() constexpr { return x; }

int main()
{
  semi::map<std::string, std::string> map;

  // 아래처럼 string literal을 key로 쓰면 빠르다.
  // 원소의 개수와 상관없이 상수시간에 찾을 수 있다.
  map.get(ID("food"))  = "pizza";
  map.get(ID("drink")) = "soda";
  std::cout << map.get(ID("drink")) << std::endl;


  // runtime string key를 쓰면 std::unordered_map과 동일하게 작동한다.
  std::string key;

  std::cin >> key;
  std::cout << map.get(key) << std::endl; // for example: outputs "soda" if key is "drink"

  // semi::map의 static 버전도 있다.
  struct Tag {};
  using Map = semi::static_map<std::string, std::string, Tag>;

  // in fact, it is (nearly) as fast as looking up any plain old global variable
  Map::get(ID("food")) = "pizza";
  Map::get(ID("drink")) = "beer";
  
  return 0;
}
```

컨테이너는 매우 간단하고, 함수는 `get`, `erase`, `contains`, `clear` 밖에 없다. `get`은 `std::map`의   
'try_emplace`처럼 생성자에게 전달할 파라미터를 취할 수도 있다.  

```cpp
#define ID(x) []() constexpr { return x; }

semi::map<std::string, std::string> m;

m.get(ID("food"), "pizza");  // key가 map에 없으므로 "pizza" 파라미터로 생성자를 호출한다.
m.get(ID("food"), "spaghetti");  // 이미 "food" key가 map에 있으므로 호출되지 않는다.
```

두 가지 map 타입을 제공한다.

1. `semi::map` 은 연관 컨테이너와 매우 유사하다. 함수는 non-static이기 때문에, 여러 `semi::map` 인스턴스 간에 공유되지 않는다.  
2. `semi::static_map` 은 완벽히 static이다. `semi::map`보다 빠르다. 하지만 모든 함수가 static이기 때문에,   
같은 key value 타입의 `semi::static_map` 들은 원소를 공유한다.  
이를 피하기 위해, "tag" 템플릿 파라미터가 있다. 오직 같은 tag인 `semi::static_map` 끼리만 원소를 공유한다.

```cpp
void foo()
{
  struct Tag {};
  using map = semi::static_map<std::string, int, Tag>;

  map::get(ID("age")) = 18;
}
```

만약 `struct` tag가 다른 블록에서 정의 되더라도, `semi::static_map`은 원소를 공유하지 않게 된다.  

`semi::static_map`의 원소는 프로그램이 종료될 때만 삭제된다. 더 일찍 삭제하고 싶으면 `clear` 함수를 호출하면 된다.  

## Performance

**find value by key 10000000 times**

1. std::unordered_map `operator[]` : O(1) - 68192 msec
2. semi::map `get()` : O(1) - 18936 msec
3. semi::static_map 'get()' : O(1) - 16578 msec
