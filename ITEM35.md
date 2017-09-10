# 스레드 기반 프로그래밍보다 과제 기반 프로그래밍을 선호하라

### Asynchronous task

- Thread-based 
- Task-based

```c++
int doAsyncWork(); // this is considered as a task

std::thread t(doAsyncWork); // thread-based
auto fut = std::async(doAsyncWork); // task-based, fut is future object for this task
```

### return value 접근에 편리한 get() 함수 제공
- thread-based approach는 return value에 접근할 수 있는 직관적인 방법을 제공하지 않음
- task-based approach는 function template std::async가 return하는 class template
  std::future<T>를 통해 return 값에 접근할 수 있음. std::future<T>.get()

### Task 실행 중 예외 발생 처리 가능
- thread-based approach로 task doAsyncWork() 실행 시 예외 발생하면 프로그램이 죽게 된다. (std::terminate)

- task-based approach에서는 앞서 언급한 std::future<t>.get() 함수를 통해 예외를 전달 받을 수 있어서
  이에 대한 처리가 가능하다

### Fundamental basis for threading
- Hardware threads 
  실제 연산을 수행하는 하드웨어 thread로 core마다 1개 이상을 가질 수 있다.
- Software threads
  OS Level에서 동작하는 thread로서 보통 하드웨어 thread보다 개수가 많아 OS가 제공하는 scheduler의 제어를 통해 동작한다.
- std::threads
  c++에서 제공하는 thread로서 software thread를 효과적으로 사용하기 위한 handle로서 동작한다.
  실제 thread와 binding 되어 있을수도 있고 그렇지 않을 수도 있다.

### Problems with thread-based approach
- Exception with limited software thread
  리소스 부족으로 thread생성이 불가능할 때, C++ 라이브러리는 다음의 예외를 발생시키다. (std::system_error)
  주목할 만한 점은 task에 아래와 같이 noexcept를 선언을 해도 예외가 발생한다는 것이다.

```c++
int doAsyncWork() noexcept;
std::thread t(doAsyncWork); // throws if no more
// threads are available
```

Possible Alternatives
- 별도의 thread를 생성하지 않고 현재 thread에서 task를 실행한다.

1) thread 간 load balance가 무너질 수 있다.
2) 어플리케이션의 반응성이 저하될 수 있다. (GUI Thread)

- 다른 thread의 작업이 끝날 때까지 기다렸다 실행하고자 하는 task의 thread를 생성한다.
  thread 간의 dead lock이 발생할 수 있다.

Oversubscription
- hardware thread보다 software thread의 개수가 많을 때 발생
- scheduler에 의해 각 software thread가 hardware thread에 할당 될 때 해당 thread간 context switching으로 인해 cost가 발생
- 그러므로 context switching이 잦아지면 thread management의 overhead가 증가

Oversubscription으로 인한 문제는 피할 방법이 없음

### Use std::async

C++ STL의 thread management로 thread 관리의 부담 감소
(eg. Resource 부족으로 인한 thread 할당이 불가능할 때 STL의 동작)
- thread-based 사례와 다르게 exception을 발생시키지 않는다.
- STL 내부의 scheduler에서 task를 실행할 때 thread 생성 여부를 결정
  (thread를 생성하지 않으면 task를 요청한 thread에서 해당 task를 실행)
- thread 생성을 줄이므로 oversubscription 문제도 완화

### Issue in std::async
- Load Balancing
  C++ STL runtime scheduler가 알아서 처리하므로 믿고 쓰자.
- Responsiveness in GUI thread
  std::async를 사용하더라도 이 문제는 여전히 남아 있을수 있다.
  명시적으로 std::launch::async 옵션을 사용하면 해결 할 수 있다.
  이 방법은 무조건 thread를 생성하는 방식

### when std::thread shoud be used
- pthread나 windows thread와 같은 low level API에 접근할 때 
  std::thread는 native_handle을 제공하여 이 API에 접근할 수 있도록 한다.
  std::async와 std::future<T>와 같은 template class나 function은 이와 같은 handle을 제공하지 않는다.

- 성능을 끌어올리기 위해 thread management를 최적화 할 때
  고성능의 서버 프로그램
- C++ concurrency API를 능가하는 thread management를 구현하고 싶을 때

Summary
- std::thread는 return value를 받을 방법과 exception handling 대안이 없다.
- std::thread는 thread할당 실패, oversubscription, load balancing, portability 면에서 관리가 힘들다.
- default launch policy를 적용한 std::async를 사용하면 이러한 문제에서 해방 될 수 있다.
