## Copy Elision

직역하면 '복사 생략' 이다. 여기서는 불필요한 복사를 생략한다는 뜻이고,  

그 방법 중 하나가 Return Value Optimization(RVO) 이다.

---

## Return Value Optimization (RVO) // Named Return Value Optimization (NRVO)  

Copy Elision의 방법 중 하나이다. 

RVO/NRVO는 함수의 반환값이 객체 형식일 때, 불필요한 복사 생성을 막아준다.

```cpp

// Return Value Optimization
Foo RVO_F()
{  
    return Foo();
}
 
// Named Return Value Optimization
Foo NRVO_F()
{
    Foo foo;
    return foo;
}
 
int main()
{
    // 아래 대입문들에서 복사생성 생략
 
    // "Constructed"
    // "Destructed"
    {
        Foo rvo_foo = RVO_F();
    }
 
    // "Constructed"
    // "Destructed"
    {
        Foo nrvo_foo = NRVO_F();
    }
}
```

만약 RVO 최적화가 없다면 복사 생성자가 두번이나 호출된다.

```cpp
// "Constructed" - 로컬에서 객체 생성
// "Move-Constructed" - 함수 반환값 임시객체로의 이동
// "Destructed" - 로컬 파괴
// "Move-Constructed" - main의 rvo_foo로의 이동
// "Destructed" - 반환값 임시객체 파괴
// "Destructed" - rvo_foo 파괴
int main()
{
    Foo rvo_foo = RVO_F();
}
```

NRVO는 최적화 옵션 O1부터 동작한다. (Project Property -> C/C++ -> Optimization)  

## Passing Temporary as Value

```cpp
void f(Foo f)
{
    std::cout << "Fn\n";
}
 
int main()
{  
    // without Copy Elision
    /*
        Constructed
        Move-Construected
        Fn
        Destructed
        Destructed
    */
    f(Foo());
 
    // with Copy Elision
    /*
        Constructed
        Fn
        Destructed
    */
    f(Foo());
}
```
마찬가지로 임시객체를 생성해서 넘기는 경우에도 Copy Elision이 적용되어 불필요한 복사를 막아준다.  

---

## C++17 Guaranteed Copy Ellision

C++17 이전에는 copy, move constructor가 delete된 객체에 대해서는 Copy Elision이 동작하지 않았다.  

```cpp
struct Foo
{
    Foo() { std::cout << "Constructed\n"; }
    // 이동이 불가한 객체
    Foo(const Foo &) = delete;
    Foo(const Foo &&) = delete;
 
    ~Foo() { std::cout << "Destructed\n"; }
};
 
Foo f()
{
    return Foo();
}
 
int main()
{
    Foo foo = f();
}
 
// E1776 : 함수 "Foo::Foo(Const Foo&&)" 을(를) 참조할 수 없습니다. 삭제된 함수입니다.
// E1776 : 함수 "Foo::Foo(Const Foo&&)" 을(를) 참조할 수 없습니다. 삭제된 함수입니다.
```

C++17부터는 non-movable한 객체에 대해서도 Copy Elision이 보장된다.  

하지만, NRVO는 적용이 안된다.

NRVO의 경우 named 값은 prvalue가 아닌, glvalue이기에, C++17의 Guaranteed Copy Elision의 대상에서 제외되었다고 한다.  

출처 : http://egloos.zum.com/sweeper/v/3204454
