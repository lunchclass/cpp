# reference collapsing 이해하기

### 1. template 타입 추론에서 reference collapsing

```c++
template<typename T>
void func(T&& param);
```
템플릿 인자 T의 타입 추론은 들어온 인자 param이 lvalue인지 rvalue인지에 따라서 결정된다. 
이때 사용되는 결정방법은 매우 간단하다. 
lvalue가 인자로 들어오면 T는 T&로 추론되고, rvalue가 인자로 들어오면 T는 참조 없이 그냥 T로 추론된다.

```c++
Widget widgetFactory();
Widget w;
func(w); //w는 lvalue이므로 T는 Widget&로 추론
func(widgetFactory()); // widgetFactory의 리턴값은 rvalue이므로 T는 Widget으로 추론
```

이 추론 규칙이 universial reference와 std::forward의 토대가 된다. 

C++에서 참조에대한 참조는 허용되지 않는다.

```c++
int x;
...
auto& & rx = x; 
```
//error! 참조에 대한 참조를 선언할 수 없다.

```c++
template<typename T>
void func(T&& param);
func(w); //인스턴싱 되면 아래의 함수로
void func(Widget& && param);  //참조에 대한 참조??
```

하지만 컴파일러는 에러를 내뱉지 않는다. 
그러면 어떻게 컴파일러는 위의 함수 인스턴스를 아래의 함수의 형태로 변경하는 것인가?
```c++
void func(Widget& param)
```

그 답이 바로 reference collapsing이다. 
컴파일러가 참조에 대한 참조를 금지한다고 이야기 했지만, template 인스턴싱과 같은 특정한 상황에서 
컴파일러는 그것을 금지하는 대신 reference collapsing이라는 추가 작업을 수행한다.

### reference collapsing의 Rule
reference의 종류는 2가지 (lvalue, rvalue) 이다. 
따라서 가능한 참조에 대한 참조의 조합은 총 4가지 경우가 있다. 
그 모든 경우에 대해서 컴파일러는 다음의 룰에 따라서 reference collapsing을 진행한다.

```c++
TYPE referenceCollapsing(TYPE A, TYPE B){
    if( A == LVALUE_REF || B == LVALUE_REF )
        return LVALUE_REF;
    else 
        return RVALUE_REF;
}
```

```c++
void func(Widget& && param)
void func(Widget& param)
```

### std::forward 에서 reference collapsing

reference collapsing은 std::forward의 구현에서도 핵심이 된다. (std::forward는 universial reference에 적용)
```c++
template<typename T>
void f(T&& param){
    ...
    someFunc(std::forward<T>(param));
}
```
1. T는 universial reference인 param이 lvalue로 초기화 되느냐, 아니면 rvalue로 초기화 되느냐에 따라서 각각 lvalue reference / lvalue로 추론.

2. T&&인 universial reference의 타입은 T가 lvalue reference인 경우 reference collapsing규칙에 따라서 
각각 lvalue reference가 될 것이고, T가 lvalue인 경우(초기화 인자가 rvalue인 경우) rvalue reference.

3. universial reference가 Widget& 인 경우 T는 Widget& 이고, std::forward<T>는 std::forward<Widget&>로 인스턴싱 된다.
그리고 universial reference가 Widget&&인 경우 T는 Widget이고 std::forward<T>는 std::forward<Widget>로 인스턴싱 된다.

```c++
template<typename T>
T&& forward(typename remove_reference<T>::type& param) {
  return static_cast<T&&>(param);
}
```
이 함수 그대로 universial reference가 Widget&인 경우, std::forward<Widget&>인 상태의 인스턴싱으로 다음과 같은 함수가 된다.

```c++
Widget& && forward(typename remove_reference<Widget&>::type& param)
{ 
  return static_cast<Widget& &&> (param); 
}
```

type_trait인 remove_reference<T>::type은 T의 참조를 제거. remove_reference<T>::type&인 param은 Widget&가 된다. 
Widget& &&는 reference collapsing규칙을 통해 Widget&이 된다. 

```c++
Widget& forward(Widget& param)
{ 
  return static_cast<Widget&> (param);
}
```

universial reference가 Widget&&인 경우 T는 Widget이므로 std::forward<T>는 std::forward<Widget>으로 인스턴싱 될 것이다.

```c++
Widget&& forward(typename remove_reference<Widget>::type& param)
{ 
  return static_cast<Widget&&> (param); 
}
```
여기서는 참조에 대한 참조가 발생하지 않으므로 remove_reference만 적용

```c++
Widget&& forward(Widget& param)
{ 
  return static_cast<Widget&&>(param); 
}
```
결과적으로 universial reference가 rvalue reference인 경우 그대로 rvalue reference로 캐스팅하여 전달.

### 2. auto에서 reference collapsing
```c++
Widget widgetFactory();
Widget w;

auto&& w1 = w; //Widget& && w1 = w; => Widget& w1 = w;
auto&& w2 = widgetFactory();//Widget&& w2 = widgetFactory();
```
universial reference는 새로운 차원의 reference는 아니고, rvalue reference인데, 템플릿으로 추론 + reference collapsing에 의하여 초기화하는 
인자의 값에 따라서 결과적으로 각각 lvalue reference와 rvalue reference로 인스턴싱 되는 것이다.

### 3. typedef & type aliasing(using)
```c++
template<typename T>
class Widget
{
public:
    typedef T&& RvalueRefToT;
};

Widget<int&> w; //int&로 인스턴싱
```

RvalueRefToT 타입은 T&&형태가 되어야 하는데 들어온 T가 int&이므로RvalueRefTOT는 reference collapsing에 의하여 int&이 된다.

### 4. decltype 
decltype은 변수의 타입을 받아올 수 있는데, 이 타입이 참조형일 수 있다. 
그리고 decltype ( variable ) & 의 방식으로 타입을 새로 지정할 수 있으므로, 이 경우 참조에 대한 참조가 발생한다. 
이때 컴파일러는 에러를 내지 않고, reference collapsing을 수행.

이렇듯 reference collapsing은 템플릿의 중요한 기능들을 구현하기위해 반드시 필요한 핵심 요소이다.
reference collapsing을 이해하면서 universial reference와 std::forward등의 구체적인 구현방식들을 잘알 수 있다.

출처 : http://ozt88.tistory.com/46
