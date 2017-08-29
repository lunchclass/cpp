# Item 24 : Distinguish universal references from rvalue references
```c++
void f(Widget&& param);             // rvalue reference
Widget&& var1 = Widget();           // rvalue reference
auto&& var2 = var1;                 // not rvalue reference
template<typename T> void f(std::vector<T>&& param);     // rvalue reference
template<typename T> void f(T&& param);                  // not rvalue reference
```
## T&& 에 담긴 두 가지 의미
1. RValue reference
2. RValue reference or LValue reference => 둘다 할 수 있으니 *보편참조* 라고 부르자
#### item25 에 따라 보편참조는 대부분 std::forwared 를 호출해야 해서 forwarding reference 라고 정해질 가능성이 높다 함
#### http://en.cppreference.com/w/cpp/language/reference#Forwarding_references

## Universal references(or Forwarding references) 가 나타날 수 있는 2가지 context
1. 템플릿 매개변수
```c++
template<typename T>
void f(T&& param);             // param is a universal reference
```
2. auto 선언
```c++
auto&& var2 = var1;            // var2 is a universal reference
```

### 결론 : 보편참조는 - Type deduction + T&& 형태

#### 예제
```c++
template<typename T>
void f(T&& param);     // param is a universal reference
Widget w;
f(w);                  // lvalue passed to f; param's type is Widget& (i.e., an lvalue reference)
f(std::move(w));       // rvalue passed to f; param's type is Widget&& (i.e., an rvalue reference) 
```

--------------------------------------------------------
### type deduction 이 없는 이런 형태는 rvalue reference
```c++
void f(Widget&& param);        // no type deduction param is an rvalue reference
Widget&& var1 = Widget();      // no type deduction var1 is an rvalue reference 
```
--------------------------------------------------------

## 몇 가지 상기할만한 예

### 다음은 보편참조가 아니다.
```c++
template<typename T>
void f(std::vector<T>&& param);  // param is an rvalue reference
```
### const 가 붙은 다음 예제도 보편참조가 아니다
```c++
template<typename T>
void f(const T&& param);         // param is an rvalue referenc
```
### 보편참조처럼 보이지만 rvalue reference. vector 는 이미 template intance화 되어있어서 type deduction 없음
```c++
template<class T, class Allocator = allocator<T>>  // from C++ class vector Standards 
{                                     
  public:
  void push_back(T&& x);
  …
};
```
### 반면에 아래의 예는 type deduction 이 일어나기 때문에 보편참조임
```c++
template<class T, class Allocator = allocator<T>>  // still from 
class vector {                                     // C++ 
public:                                            // Standards  
  template <class... Args>  
  void emplace_back(Args&&... args);  
… 
};
```

http://en.cppreference.com/w/cpp/container/vector/emplace_back

### auto가 사용된 보편참조의 예
```c++
auto timeFuncInvocation =
  [](auto&& func, auto&&... params)               // C++14
    {
          //start timer;
          std::forward<decltype(func)>(func)(           // invoke func
                std::forward<decltype(params)>(params)...   // on params
          );
          stop timer and record elapsed time;  
    };
```
-------------------------------------------
## 왜 universal 인지 rvalue reference 인지 알아야 되나
1. 코드를 정확하게 보는데 도움이 되고
2. communication 에 도움이 된다 (이름이 있는 추상적 단어를 사용하므로)
3. item 25/26 을 이해하는데 필수임(std::move / std::forward 의 사용)
