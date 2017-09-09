# Item31 : Avoid default capture modes

### Lamda expression : 표현식, 소스코드의 일부로 아래가 람다 표현식이다
```c++
std::find_if(container.begin(), container.end(),
             [](int val) { return 0 < val && val < 10; }); 
```
### Closure : 람다에 의해 만들어진 실행시점 *객체*, capture mode 에 따라 클로저는 capture된 data의 복사본이나 참조를 가질 수 있다.
### (위 예제에서 세번째 paramter 로 전달되는 객체)

### Closure class : 클로저를 만드는데 사용된 클래스, 컴파일러는 각 람다들에 대해 고유한 closure class 를 만들어낸다.
------------------------------------

## Default capture mode
### 1. By Reference "\[&\]\(\) {...}"
### * 참조가 대상을 잃을(dangling) 위험이 있다.
### 2. By Value "\[=\]\(\) {...}"
### * by value 는 capture 된 객체가 대상을 잃지 않을것 같지만 실제로 그렇지 않고, 
### self-contained 처럼 보이지만 그렇지 않은 경우도 있다.
----------------------------------------

## 1. By Reference 로 발생할 수 있는 문제
```c++
using FilterContainer =                    
  std::vector<std::function<bool(int)>>;    // 필터링 하는 "함수 객체"를 받는 컨테이너
FilterContainer filters;                    // 필터링 함수들

void addDivisorFilter() {
  auto calc1 = computeSomeValue1();
  auto calc2 = computeSomeValue2();

  auto divisor = computeDivisor(calc1, calc2);
  filters.emplace_back(                              // danger! ref to divisor will dangle
    [&](int value) { return value % divisor == 0; } 
  );
 }                                                  
```
#### filter 가 추가되고 이후에 다른 context 에서 filters 에 포함된 함수객체들이 호출이 될 수 있으나 해당 시점에는 지역 객체인 divisor 가 존재하지 않음
```c++
filters.emplace_back(
  [&divisor](int value)             // default caputre 모드를 사용하지 않아도 여전히 같은 문제 발생
  { return value % divisor == 0; }                
 );                                              
```

#### closure 가 바로 사용된다면?
```c++
template<typename C>
 void workWithContainer(const C& container)
 {
  auto calc1 = computeSomeValue1();             
  auto calc2 = computeSomeValue2();             
  auto divisor = computeDivisor(calc1, calc2);  

  using ContElemT = typename C::value_type;     
                                                
                                                
  using std::begin;                             
  using std::end;                               
                                                
  if (std::all_of(                              
        begin(container), end(container),       // 컨테이터의 모든 값이 divisor의 배수인가?
        [&](const ContElemT& value)             
        { return value % divisor == 0; })       
      ) {
    …                                           // 그런 경우
  } else {
    …                                           // 아닌 경우
  }                                              
```
#### 위 예제는 안전함, 하지만 해당 람다를 copy&paste 해서 가져다가 쓰는 경우에는 분명히 문제가 발생함
#### (일반 함수라면 문제가 발생하지 않았을 것)

### => 람다가 의존하는 지역 변수들과 매개변수를 명시적으로 표기하는것이 소프트웨어 공학적인 관점에서 더 좋다.
------------------------------------

## 2. By Value 로 발생할 수 있는 문제
### - Dangling 이 여전히 발생 할 수 있음
### - Self-contained(자기 완결성?) 인것처럼 보이지만 그렇지 않음
```c++
filters.emplace_back(                              
  [=](int value) { return value % divisor == 0; }  // 더 이상 dangling 문제가 발생하지 않음   
);                                                    
```

#### 하지만 by value 로 pointer 가 capture 된다면?
```c++
// Widget 클래스가 필터들의 컨테이너에 필터 함수를 추가하는 method 를 가지고 있다고 가정
class Widget { public:
  …                                 // 생성자 등등
  void addFilter() const;            // filter를 filters 에 추가

private:
  int divisor;                       // Widget의 필터에 사용됨
 }
```
#### divisor 를 lamda 에서 사용하고 싶다면 어떻게 해야 할까?

#### capture 는 static 이 아닌 지역변수(함수의 paramter)에만 적용되어서 아래처럼 사용은 불가함
```c++
void Widget::addFilter() const
 {
  filters.emplace_back(                             
    [](int value) { return value % divisor == 0; }  // 멤버 변수 Widget::divisor를 사용할 수 없음
 );
```
```c++
void Widget::addFilter() const
{
  filters.emplace_back(
    [divisor](int value)                
    { return value % divisor == 0; }  // 지역 변수 divisor 가 없기 때문에 컴파일 되지 않음
  );
}                                                 
```

```c++
void Widget::addFilter() const
{
  filters.emplace_back(
    [=](int value) { return value % divisor == 0; }
  );
}
```
#### 위 코드는 안전할것 같아 보이지만 실제로는 addFilter 는 멤버 함수이기 때문에 this가 사용되어 아래와 동일한 코드가 됨
```c++
void Widget::addFilter() const
{
  auto currentObjectPtr = this;
  filters.emplace_back(
    [currentObjectPtr](int value)
    { return value % currentObjectPtr->divisor == 0; }
  );
}
// 이 람다로 만들어진 closure의 유효성이 해당 closure가 만들어진 시점에 capture된 Widget 객체의 수명에 의해 제한됨(this 때문)
```
#### 문제가 발생하는 예제
```c++
using FilterContainer =
  std::vector<std::function<bool(int)>>;
FilterContainer filters;

void doSomeWork() 
{
  auto pw =                       
    std::make_unique<Widget>();   
                                  

  pw->addFilter();                // Widget::divisor를 사용하는 필터를 추가
  …
 }                                // Widget 이 제거 되고 filters 에 dangling pointer 가 발생
```

#### 멤버변수 divisor 를 사용하고 싶다면 아래와 같이 하면 된다.
```c++
void Widget::addFilter() const {
  auto divisorCopy = divisor;                // copy data member

  filters.emplace_back(
    [divisorCopy](int value)                 // capture the copy
    { return value % divisorCopy == 0; }     // use the copy
  ); 
```

#### C++ 14 에서는 멤버 변수를 capture 하는 편한 방법이 있다.
```c++
void Widget::addFilter() const
{  filters.emplace_back(              // C++14:
    [divisor = divisor](int value)    // copy divisor to closure
    { return value % divisor == 0; }  // use the copy
  );
}
```


### Self-contained 가 아닌 예
람다는 static 으로 선언된 객체를 "사용" 할수는 있지만 "capture" 할수는 없다.
```c++
// 아래예는 모든 내용을 값으로 "capture" 하고 람다 내에서 사용한다고 생각할 수 있지만 실제로는 전혀 그렇지 않다.
void addDivisorFilter()
{
  static auto calc1 = computeSomeValue1();      // now static
  static auto calc2 = computeSomeValue2();      // now static

  static auto divisor =                         // now static
    computeDivisor(calc1, calc2);

  filters.emplace_back(
    [=](int value)                     // 아무것도 capture 되지 않음
    { return value % divisor == 0; }   // 그냥 위의 static divisor 를 가리킨다
  );
  ++divisor;                           // divisor 를 수정
}
```
