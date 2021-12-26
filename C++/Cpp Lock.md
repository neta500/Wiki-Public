## std::mutex

**1. 설명**

커널 레벨 동기화 오브젝트. 커널 레벨의 오브젝트이기 때문에 유저 레벨에서 호출할 때 system call이 발생한다.(그만큼 느리다)

하지만 다른 프로세스의 접근도 막을 수 있는 강력한 동기화 객체이다.

**2. 사용**

```cpp

std::mutex mutex;

std::scoped_lock lock(mutex);

```

**std::scoped_lock**

RAII를 고려해서 std::scoped_lock을 사용하는 것이 좋다. 기존의 std::lock_guard는 std::scoped_lock으로 완전히
대체할 수 있다.
std::scoped_lock은 std::lock_guard의 superior 버전으로, 여러개의 lock을 한번에 걸 수 있다.

```cpp
std::scoped_lock lock(mutex1, mutex2);
```

**std::shared_mutex**

std::mutex의 Read/Write 버전으로, std::shared_mutex가 있다. std::shared_mutex를 가지고
shared_lock을 걸면 Read lock, unique_lock을 걸면 R/W lock을 걸 수 있다.

```cpp
// Multiple threads/readers can read the counter's value at the same time.
unsigned int get() const 
{
    std::shared_lock<std::shared_mutex> lock(mutex);
    return value_;
}
// Only one thread/writer can increment/write the counter's value.
void increment() 
{
    std::unique_lock<std::shared_mutex> lock(mutex);
    value_++;
}
```

---

## CriticalSection

**1. 설명**

유저 레벨 동기화 객체. mutex보다 빠른 대신 프로세스 내에서의 동기화만 가능.

CRITICAL_SECTION 구조체는 윈도우에서 <winnt.h> 에 정의되어있다.

```cpp
typedef struct _RTL_CRITICAL_SECTION {
    PRTL_CRITICAL_SECTION_DEBUG DebugInfo;

    //
    //  The following three fields control entering and exiting the critical
    //  section for the resource
    //

    LONG LockCount;
    LONG RecursionCount;
    HANDLE OwningThread;        // from the thread's ClientId->UniqueThread
    HANDLE LockSemaphore;
    ULONG_PTR SpinCount;        // force size on 64-bit systems when packed
} RTL_CRITICAL_SECTION, *PRTL_CRITICAL_SECTION;
```

**2. 사용**

```cpp
CRITICAL_SECTION criticalSection;

InitializeCriticalSection(&criticalSection); // init

EnterCriticalSection(&criticalSection); // Enter

// DoSomething

LeaveCriticalSection(&criticalSection); // Leave

DeleteCriticalSection(&criticalSection); // delete
```

**Spin lock**

InitializeCriticalSectionAndSpinCount(lock, count) 함수를 이용해서 스핀락을 걸 수 있다.

일단 스핀락에 대해서 알아보자.

원래 스레드가 락을 획득하지 못하면 스레드는 Running 상태에서 Blocking(혹은 Waiting, 대기) 상태로 변경될 것이다.
스레드가 Blocking 상태가 된다는 것은 Context Switching이 발생한다는 것이고, 이는 성능 저하를 유발한다.

또 다른 문제는, 스레드가 Blocking 상태가 되면, 원래 락을 획득하고 있던 스레드가 LeaveLock을 해도 블로킹된 스레드가
다시 cpu를 획득하는데 시간이 걸린다는 것이다(cpu의 스케줄링에 따라).

그래서 스레드를 Blocking 상태로 만들지 않고, 락을 획득하지 못해도 일정 시간동안 Running 상태에서 락을 획득하려고
계속 시도를 하도록 만든 방식이 Spin lock이다.(락을 획득하려고 계속 스핀 한다는 뜻)
특히 스레드의 상태 변화가 잦은(임계 영역에서의 cpu 소비시간이 짧은) 경우 스핀락이 굉장히 효율적일 수 있다.

```cpp
CRITICAL_SECTION criticalSection;
InitializeCriticalSectionAndSpinCount(&criticalSection, 4000);
```

---

## SRWLock

**1. 설명**

윈도우 Slim Read/Write Lock. 

**왜 Slim이고, 왜 CriticalSection보다 빠를까?**

SRWLock도 Spin lock을 사용한다. Lock을 얻으려는 스레드는 SRWLock 구조체 내에 Ptr 변수의 내용을 검사한다.
보기엔 비슷해보이는데 다른점은 YieldProcessor() 함수이다.
Yield는 양보하다 라는 뜻이다. 프로세스가 계속 락을 획득하려고 atomic 연산을 수행하면서 커널을 괴롭히지 않고,
스케줄링된 상태에서 잠시 기다리는 것이다.

**2. 사용**

Exclusive, Shared가 나뉘어져 있다.

```cpp
AcquireSRWLockExclusive();

AcquireSRWLockShared();

ReleaseSRWLockExclusive();

ReleaseSRWLockShared();
```

---

## std::atomic

**1. 설명**
**2. 사용**


---

## Lock free

**1. 설명**
**2. 사용**