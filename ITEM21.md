# Item 21 new를 직접 사용하는 것보다 std::make_unique와 std::make_shared를 선호하라.
### std::make_unique
c++14 부터 STL 라이브러리에 추가
Perfect forwarding으로 인자를 전달 받아 object 생성
이 object에 대한 Smart pointer return
아래 구현은 Array와 deleter에 대해 지원이 되지 않음

```c++
template<typename T, typename... Ts>
std::unique_ptr<t> make_unique(Ts&&... params)
{
  return std::unique_ptr<T>(new T(std::forward<Ts>(params)...));
}
```

### make 함수를 이용하는 이유
1) 코드의 중복을 막을 수 있다.

```c++
auto upw1(std::make_unique<Widget>()); // with make func
std::unique_ptr<Widget> upw2(new Widget); // without make func
auto spw1(std::make_shared<Widget>()); // with make func
std::shared_ptr<Widget> spw2(new Widget); // without make func
```
형식을 여러번 되풀이 하는것은 소프트 웨어 공학의 핵심 교의중 하나인 "코드 중복을 피하라"와 충돌,
컴파일 시간이 늘어나고, 목적 코드의 덩치가 커지며, 코드 기반(code base)을 다루기가 어려워짐

2) 예외 안정성
Memory leak을 막을 수 있다.
```c++
// 자원 누수의 위험이 있음
int computerPriority();
processWidget(std::shared_ptr<Widget>(new Widget),
              computePriority);
```

컴파일러가 위 코드를 아래의 순서로 실행 할 수 있다.
1. "new Widget을 실행 한다."
2. computePriority를 실행한다.
3. std::shared_ptr 생성자를 실행한다.

```c++
//자원 누수의 위험이 없음
processWidget (std::make_shared<Widget>(), computePriority);
```

3) 향상된 효율성
std::make_shared를 사용했을 때 약간의 성능 향상이 기대된다.
shared_ptr<T>를 생성하면 object 외에도 control block을 생성하게 된다.
 - new를 이용할 경우 이미 object를 생성하여 shared_ptr에 전달하게 되므로
   control block에 대한 메모리 할당이 한번 더 일어나게 된다.
 - make_share를 활용하면 한번의 메모리 할당으로 object와 control block 생성을 할 수 있게 된다.
```c++
std::shared_ptr<Widget> spw(new Widget);
auto spw = std::make_shared<Widget>();
```
프로그램의 정적 크기가 줄어든다. (한번의 메모리 할당 호출 코드)
실행 크기의 속도가 빨라진다.
프로그램의 전체적인 메모리 사용량이 줄어들 여지가 생긴다.

### make 함수를 선호하라!
make 함수를 사용할 수 없거나 사용하지 않아야 하는 상황이 존재
1) make function은 custom deleter를 지원하지 않는다.
```c++
auto widgetDeleter = [](Widget* pw){...};

std::unique_ptr<Widget, decltype(widgetDeleter)>;
upw(new Widget, widgetDeleter);
std::shared_ptr<Widget> spw(new Widget, widgetDeleter);
```
2) std::initializer_list를 인자로 삼는 constructor로 object를 생성할 수 없는 문제

```c++
auto upv = std::make_unique<std::vector<int>>(10, 20);

//create std::initializer_list
auto initList = { 10, 20 };
//create std::vector using std::initializer_list ctor
auto spv = std::make_shared<std::vector<int>>(initList);
```

3) operator new와 operator delete를 overloading한 class에서 make_shard를 쓰지 않는 것을 권장 
- 이러한 class의 의도는 대부분 sizeof(Widget)만큼의 메모리만 할당하여 쓰고 자함이나
  make_shared 에서는 object의 사이즈에 control block을 추가 할당하므로 의도에 맞지 않음
4) make_shared를 사용하면 메모리를 해제할때 object의 destructor가 불리더라도 control block이
  destruct 될 때까지 release 되지 않는다.
- weak_ptr이 control block을 참조하고 있기 때문에 control block의 life span은 object보다 길게 나타날 수 있음.

```c++
class ReallyBigType {...};
// create very large object via std::make_shared
auto pBigObj = std::make_shared<ReallyBigType>();

// create very large object via new
std::shared_ptr<ReallyBigType> pBigObj(new ReallyBigType);
```

```c++
processWidget(
std::shared_ptr<Widget>(new Widget, cusdel),
computePriority()
); //leak 

std::shared_ptr<Widget> spw(new Widget, cusDel);
processWidget(spw, computePriority()); // correct, but not optimal

processWidget(std::move(spw), computePriority()); // both efficient and exception safe
```

std::shared_ptr의 경우 이동과 복사의 차이가 클 수 있다. (복사하려면 참조 횟수를 원자적으로 증가)
