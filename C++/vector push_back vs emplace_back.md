# vector push_back vs emplace_back

1줄 요약 : emplace_back을 쓰되, 암시적 호출이 될 지 한 번 더 생각하자.

```c++
std::vector<Cat> catVec;

// 참고 : Cat은 int를 생성자로 받을 수 있다.
Cat cat1{11};
catVec.push_back(cat1); // copy 1
catVec.push_back(std::move(cat1)); // move 1
catVec.push_back(Cat{}); // move 1
catVec.push_back(5); // move 1

/atVec.emplace_back(cat1); // copy 1
catVec.emplace_back(std::move(cat1)); // move 1
catVec.emplace_back(Cat{}); // move 1
catVec.emplace_back(10); // zero overhead

// 하지만 emplace_back은 가능한 모든 생성자에 대해 호출하기 때문에 의도하지 않은 작업이 일어날 수 있다.
std::vector<std::unique_ptr<Cat>> catPtrVec;		

catPtrVec.push_back(std::addressof(cat1));
catPtrVec.emplace_back(std::addressof(cat1)); // cat1은 스택 변수이기 때문에 unique_ptr가 소멸되면서 삭제 요청 시 heap runtime error가 발생한다.

auto catPtr = std::make_unique<Cat>(13);
```