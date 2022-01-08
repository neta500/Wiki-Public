https://github.com/CppCon/CppCon2020/blob/main/Presentations/retiring_the_singleton_pattern/retiring_the_singleton_pattern__peter_muldoon__cppcon_2020.pdf

Stack Overflow : 싱글톤은 왜 나쁜걸까?

"이 주제의 최악인 점은, 싱글톤을 싫어하는 사람들이 싱글톤 대신 사용할 방법에 대한 구체적이고, 실용적인 제안을 거의 하지 않는다는 것이다."

```cpp
class Singleton
{
public:
    static Singleton& GetInstance()
    {
        static Singleton instance;
        return instance;
    }

private:
    Singleton();
}

// usage
Singleton::GetInstance()->func()
```

보통 싱글톤을 이렇게 작성한다. (Meyer's Singleton)

- 어떤 타입에 대한 객체는 전역으로 '한 개만' 존재한다.
- 전역에서 접근 가능하다.
- 변경 가능하며, 프로그램의 생명 주기에 묶여있다.
- 초기화를 원하는대로 할 수 없다. (private constructor)

### 싱글톤의 약점 (drawback)
- 초기화를 원하는 대로 할 수 없다.
- 멀티스레드에서 안전하지 않다. (이부분은 Meyer's Singleton에서 해결됨)
- 싱글톤이 여러개이고, 어떤 다른 싱글톤과 초기화가 연관되어있는 경우 따로 초기화 과정을 고려해줘야 한다.

잘 알려져있다 싶이, 싱글톤의 최대 단점은 초기화라고들 생각한다.

### 그래도 싱글톤을 쓰면 좋은 이유
- 편하니까

ppt를 쭉 봤는데 별 얘기가 없다.. 