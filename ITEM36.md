#Item 36: 비동기성이 필수일 때는 std::luanch::async를 사용해라.

std::async 호출 ? 비동기적으로 실행하겠다는 의도로 호출이다.
```c++
  auto  fut1 = std::async(f);  //함수 f를 default policy로 호출
  auto  fut2 = std::async(std::launch::async |
                          std::launch::deferred,
                          f);  //함수 f를 비동기 or defered 로 호출
                          
  //defalut policy는 async,deferred두 속성을 지원 , 위 둘은 완전히 같은 의미다. 
  
  //std::launch::async->호출될 F는 반드시 비동기적(다른쓰레드에서)호출
  
  //std::launch::deferred->std : std::future에 의해 get, wait 호출시에만 f 호출  
```

동기, 비동기를 모두 지원하는 유연성은 표준라이브러리의 스레드 관리 구성 요소를 관리하는데 도움이된다.

그러나 default policy를 이용한 호출은 다음과 같은 예측 불가능한 상황을 가지고 있다.

```c++
   auto fut = std::async(f); //default policy로 f 실행  
```
   
- f를 실행할 스레드 t가 저 코드와 동시에 실행될지 알 수 없다.  
- f가 fut에 대해 get이나 wait호출하는 스레드 t와는 다른 스레드에서 실행될지 알수 없다.  
- 모든 코드 path에서 get,wait이 호출된다는 보장이 없을수 있으니 f가 반드시 실행되는지 알수 없다.

 default policy의 스케쥴링(동기, 비동기) 유연성은 thread_local 변수(TLS)를 사용하는 work와는 궁합이 맞지 않는 경우가 많다.
 
 f에서 사용하는 TLS가 어느 쓰레드의것인지 알 수 없다.
 
 wait 기반처리에 영향.
```c++
      using namespace std::literals;
      void f(){
        std::this_thread::sleep_for(1s);       //1초후 리턴
      }
      
      auto fut = std::async(f);
      
      while (fut.wait_for(100ms) != std:future_status::ready)   //f실행이 끝날때까지 기다린다.
      {         ...      }
      
      //비정상적인 상황 heavy load로 인한 스케쥴링이 늦어 f 호출이 지연되었을 경우
      //wait_for 리턴은 항상 std::future::deferred를 리턴
      //단일 개발환경에서는 거의 나오지 않지만 위험한코드
      //따라서 예상치 못한 예외상황인 deferred 에대한 처리를 따로 해줘야한다.
```
   
   이러한 불확실성 때문에 default policy로 async를 사용하는것은 아래 조건이 모두 성립할때만 적합하다.
   
- f가 get,wait를 호출하는 스레드와 반드시 동시적으로 되지 않아도 될때
  - 예측불허 임으로 동시 시작을 가정하면 안된다.
  
- TLS(Thread Local Storage)에 종속적이지 않을때                     
  - TLS가 중요해지면 어느 쓰레드에서 f가 실행되는 지 알수 없으므로 결과 예측불가능
  
- get,wait이 반드시 호출되거나 f가 실행되지 않아도 될때  
  - 호출이 안될수도 있다. 실행이 반드시 되어야 하는 작업이면 망   
  
- f가 지연된 예외상황이 처리될수 있을때.                              
  - 지연 처리가 안되면 위 예제 처럼 무한루프
  
  
### 하나라도 성립이 안되면 비동기로 강제 해야한다.
```c++
  auto fut = std::async(std::launch::async,f)
  //c++11 & 14
  auto fut = reallyAsync(); //단순 asnyc 속성만 주고 호출할뿐이다.

```

