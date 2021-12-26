**Exception Safety in cpp**

https://msdn.microsoft.com/en-us/library/hh279653.aspx?f=255&MSPPError=-2147217396

**Errors and Exception Handling (Modern C++)**

https://msdn.microsoft.com/en-us/library/hh279678.aspx?f=255&MSPPError=-2147217396

**Effective C++ item 29: Strive For Exception-safe Code**

---

## Exception Safety in cpp
예외 안전성을 확보하는 작업은 프로그램의 죽기살기가 달렸기 때문에 매우 중요하다. 

예외 안전성을 가졌다는 것은, 예외가 발생했을 때 다음과 같이 작동해야 한다.
* 자원이 새도록 만들지 않는다. (feat. RAII)
* 자료구조가 더럽혀지지 않는다.

예외 안전을 보장하는 방법은 단계에 따라 나눌 수 있는데 아래와 같다.
예외에 안전한 함수는 아래 세 가지 방법 중 하나를 제공해야 한다.

## No-fail(No-throw) Guarantee

예외를 던지지 않겠다는 보장이다. c++의 기본 제공 타입(int, 포인터 연산 등)에 대한 연산은 예외를 던지지 않게 되어있다.
아래의 Strong, Basic Exception Guarantee는 객체의 소멸자가 noexcept라는 가정을 하고 보장하는 것이다.

## Strong Exception Guarantee

강력한 보장은 예외가 발생해도 프로그램에 전혀 영향을 주지 않는다는 보장이다. db transaction의 commit과 rollback과 같이,
함수 호출이 실패해서 예외가 발생해도, 함수 호출이 없었던 것처럼 프로그램이 동작해야 한다는 것이다.
간단하게, 강력한 보장을 제공하는 함수는 프로그램의 상태를 두가지로 제한할 수 있다.
함수가 성공적으로 실행을 마친 후의 상태(commit), 혹은 함수가 호출될 때의 상태(rollback).

강력한 보장을 제공하는 함수를 만드는 방법 중 하나는 copy-and-swap 기법이다. 원래 객체의 사본을 만들고, 그 사본을
수정하는 것이다. 수정하다가 예외가 발생하면 원본으로 갈아끼우고, 수정이 성공하면 원본을 사본으로 교체하면 된다.

하지만 강력한 보장은 강력한 대신 높은 비용을 요구한다.(시간, 공간에 대한)
상황에 따라 어떻게 예외 안전성을 제공할 지는 프로그래머의 몫이다.

## Basic Exception Guarantee

기본 보장은 예외가 발생했을 때, 프로그램에 관련된 모든 것들(객체, 자료구조 등등)을 '유효한' 상태로 유지하겠다는 보장이다.
유효한 상태란, '쓸 수 있는' 상태일 뿐이고, 객체의 내부 구조는 사용자 입장에서 모른다는 것이다. 그래서
변한 객체를 쓸 수는 있지만 제대로 동작한다는 보장은 없다. (알아내려면 객체 상태를 알려주는 함수를 호출하던가)
요약하면, 기본 보장은 예외가 발생해도 프로그램은 동작하지만 제대로 동작한다는 보장은 아니다.

## RAII (Resource Acquisition is Initialization)

직역하면 리소스의 획득은 초기화이다. 인데 무슨 말이냐하면 자원 관리는 객체의 생명과 연결되어있다는 것이다.
자원의 획득은 객체가 생성될 때(특히 초기화 과정에서), 자원의 회수는 객체가 소멸될 때(특히 스코프를 벗어날 때)
한다는 c++의 idiom이다.

예를 들면,

```cpp
void UnsafeFunction()
{
    Resource* resource = new Resource();

    /* Do something with resource */

    thisFunctionCanThrowException();

    /* Do something else with resource */

    delete resource;
}
```

위 함수는 exception safe 하지 않다. heap memory(자원)를 할당해 놓았는데, 해제하는 코드가 실행되기 전에
예외가 발생하면, 자원이 제대로 해제되지 않는다. 즉 이것은 memory leak의 원인이 된다.

```cpp
void UnMaintainableFunction() 
{
    Resource* resource = nullptr;
    try 
    {
        resource = new Resource();
  
        /* Do something with resource */
  
        thisFunctionCanThrowException();

        /* Do something else with resource */

        delete resource;
    }
    catch(std::exception& e) 
    {
        delete resource;
        throw e;
    }
}
```

위 코드는 예외가 발생하면 예외를 잡아서 자원의 해제를 해준다. 하지만 보기에 좋지 않고, 유지보수도 힘들어 보인다.
그래서...

```cpp
void SafeFunction() 
{
    auto resource = std::make_unique<Resource>();

    /* Do something with resource */

    thisFunctionCanThrowException();

    /* Do something else with resource */
}
```

스마트 포인터를 이용해서 객체를 생성했다. 스마트 포인터는 포인터의 생명주기가 다하면 자동으로 할당된
객체를 해제해주기 때문에 memory leak을 유발하지 않고, 코드도 깔끔하다.

## noexcept

이 함수는 예외를 던지지 않는다. 를 표시.
std::move_if_noexcept()처럼 함수가 예외를 던지지 않을 경우에만 move를 하는 함수도 있다.

## stack unwinding