# 항목 30 : 완벽 전달이 실패하는 경우들을 잘 알아두라.

### 완벽 전달이란?
전달(forwarding)은 말 그대로 한 함수가 자신의 인수들을 다른 함수에 넘겨주는 것을 뜻한다.<br>
이때 전달 받은 함수가 애초에 전달하는 함수가 갖고 있던 것과 동일한 객체를 받게 하는것이다.<br>
완벽 전달이라는 말은 객체뿐만 아니라, 타입, L-Value R-Value여부, const or volatile의 유무까지 포워딩 하는것 을 의미한다.<br>
범용적인 전달을 위해서는 참조 매개변수들을 사용해야 한다.<br>

```c++
template<typename... Ts>
void fwd(Ts&&... params) // 임의의 인수를 받는다
{
 f(std::forward<Ts>(params)...); // f에 그것들을 전달한다
}
```

위와 같이 대상 함수 f와 전달 함수 fwd가 있다고 할 때, 어떤 인수로 f를 호출했을 때 일어나는 일과,<br>
같은 인수로 fwd를 호출했을 때 일어나는 일이 다르다면 완벽 전달은 실패한 것이다.<br>

## 중괄호 초기치

f가 다음과 같이 선언 되었을 때, f는 컴파일 되지만, fwd는 컴파일 되지 않는다.
```c++
void f(const std::vector<int>& v);

f({ 1, 2, 3 });  //"{1, 2, 3}은 암묵적으로 std::vector<int>로 변환됨"
fwd({ 1, 2, 3 }); // 오류! 컴파일 되지 않음.
```

### 실패 원인
f 호출시 컴파일러는 호출 지점에서 함수에 전달되 인수들의 형식들과 f의 선언된 매개 변수들의 형식들을 비교
해서 호환 여부를 파악하고, 필요하다면 적절한 암묵적 변환이 수행되어 호출을 성사 시킨다.

그러나 fwd의 호출 지점에서 전달되 인수들과 f의 선언된 매개 변수를 직접 비교할 수 없다.<br>
대신 컴파일러는 fwd에 전달되는 인수들의 형식을 연역하고, 연역되 형식들을 f의 매개변수 선언들과 비교한다.

### 완벽전달이 실패하는 2가지 조건
1. fwd의 매개변수들 중 하나 이상에 대해 컴파일러가 형식을 연역하지 못한다. 
2. fwd의 매개변수들 중 하나 이상에 대해 컴파일러가 형식을 잘못 연역한다.

### 초기 문제 원인
"fwd({ 1, 2, 3 })" 호출에서 문제는 std::initializer_list가 될 수 없는 형태로 선언되어 있어서<br>
fwd 호출에 쓰인 표현식 {1, 2, 3}의 형식을 컴파일러가 연역하는 것이 금지 되어있다.


## 널 포인터를 뜻하는 0 or NULL

0 또는 NULL 대신 nullptr을 사용하면 된다.

## 선언만 된 정수 static const 및 constexpr 자료 멤버

일반적으로 정수형의 static const 데이터 멤버를 클래스에서 정의할 필요가 없다.

```c++
class Widget {
public:
 static const std::size_t MinVals = 28; // MinVals 선언
 …
};
…                                       // MinVals 정의는 없다
std::vector<int> widgetData;
widgetData.reserve(Widget::MinVals); 

```

### 문제점
MinVals가 언급된 모든 곳에 28로 배치된다.
그러나 MinVals의 주소를 취한다면, 컴파일을 되지만 링킹이 안된다.

위와 같이 정의 되었을 때, 하기 f를 MinVals로 호출하는 것을 문제가 없다.
그러나 fwd를 거쳐서 f를 호출하려 하면 상황이 어려워진다.

```c+
f(Widget::MinVals); // "f(28)" 호출 됨
fwd(Widget::MinVals); // 링크 에러!
```

### 해결책
컴파일에 따라서 잘되는 것도 있다. 그러나 이식성은?
구현부에 정의를 통해서 해결 가능하다.

```c+
const std::size_t Widget::MinVals; // Widget.cpp 파일
```

## 중복적재된 함수 이름과 템플릿 이름

아래 두 f함수의 차이점은? - 없다.

```c++
void f(int (*pf)(int));
void f(int pf(int));

int processVal(int value);
int processVal(int value, int priority);

f(processVal);    // 성공
fwd(processVal);  // 실패
```

### 문제점
f는 함수를 가리키는 포인터를 기대하지만, processVal은 함수 포인터가 아니다.<br>
사실 함수도 아니며, 서로 다른 두 함수가 공유하는 하나의 이름이다.<br>
하지만 컴파일러는 f가 원하는 매개변수를 알기에, 그 주소를 넘겨준다. <br>
그러나 fwd는 호출에 필요한 형식에 관한 정보가 없다.<br>

### 해결책

전달하고자 하는 중복적재나 템플릿 인스턴스를 명시적으로 지정하면 된다.

```c++
template<typename T>
T workOnVal(T param) 
{ … }
fwd(workOnVal); // 에러
```

f의 매개변수와 같은 형식의 함수 포인터를 만들어서 processVal이나 workOnVal로 초기화하고<br>
그 포인터를 fwd에 넘겨주면 된다.

```c++
using ProcessFuncType = int (*)(int);
ProcessFuncType processValPtr = processVal; // specify needed
fwd(processValPtr); // 성공
fwd(static_cast<ProcessFuncType>(workOnVal)); //성공
```

## 비트 필드

아래와 같이 비트 필드가 함수의 인수로 쓰일 때 어떻게 동작하는지 보자.

```c++
struct IPv4Header {
 std::uint32_t version:4,
 IHL:4,
 DSCP:6,
 ECN:2,
 totalLength:16;
 …
};

void f(std::size_t sz); 

IPv4Header h;
…
f(h.totalLength);  // 성공
fwd(h.totalLength);  // 실패
```

### 문제점
C++ 표준은 "비const 참조는 절대로 비트필드에 묶이지 않아야 한다"라고 명확하게 선고한다. <br>
이러한 금지 이유는 워드의 일부분인 비트를 직접적으로 지칭하는 방법이 없기 때문이다. <br>
포인터는 가장작은 단위가 char이고, 참조 또한 불가능하다. <br>

### 해결책
비트필드를 인수로 받는 임의의 함수는 그 비트필드의 값의 복사본을 받게된다.
비트필드를 매개변수에 전달하는 2가지 방법
 1. 값으로 전달
 2. const에 대한 참조로 전달
 
```c++
auto length = static_cast<std::uint16_t>(h.totalLength);
fwd(length); // 
```
