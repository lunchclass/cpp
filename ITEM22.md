# 22. Pimpl 관용구를 사용할때는 특수 멤버함수들을 구현파일에서 정의 하라

## 뭔소리임?
클래스의 자료멤버들을 구현클래스(or 구조체)를 가리키는 포인터로 대체하자. (이런짓을 pimpl 관용구라고 한다.)

## 왜?
클래스 해더간의 의존성이 클수록 컴파일이 늦어지고 컴파일 범위도 커진다. 그러니 의존성을 줄여보자. 
해더에 오타나면 빌드에러 대박날듯

```c++
//widget.h
#include "gadget.h" 
#include <string>
#include <vector>

class Widget_98
{
public:
  Widget_98() {};
  ~Widget_98() {};
private:
  std::string name;
  std::vector<double> data;
  Gadget g1, g2, g3;  //Gadget 사용자 정의 Class
 };
 
 /**
 1. widget은 반드시 멤버변수 타입을 알수 있는 해더를 include 해야한다.
 2. gadget.h가 바뀌면 widget.h를 include하고 있는 모든 파일이 컴파일시 영향을 받는다->의존성 생김
 */
```

## 지금까지는 어떻게 해왔나?

```c++
//widget.h
class Widget_98
{
public:
  Widget_98();
  ~Widget_98();
private:
  struct Impl;  //단순 선언만 , 이렇게 선언만 하고 정의는 하지 않는 형식을 불완전 형식(imcomplete type)이라고 한다.
  Impl *pImpl;
 };
 
 /** widget.h 에서 여러 해더를 포함할 필요가 없다.*/
 
 //widget.cpp
 #include "widget.h"
 #include "gadget.h"
 #include <string>
 #include <vector>

struct Widget_98::Impl { //정의
  std::string name;
  std::vector<double> data;
  Gadget g1, g2, g3;
  };
  
Widget_98::Widget_98() : pImpl(new Impl){}
Widget_98::~Widget_98() {delete pImpl;}

 /** 구현부에서 필요한 부분만 include,  의존성이 .h에서 .cpp로 옮겨졌다.*/
 ```

## 개선할점?
raw 포인터 사용 하는것
new & delete 호출이 원시 시적이다.
'Widget 생성자에서 Widget::Impl객체를 할당하고 Widget이 파괴될 때 그 객체를 해제 해줘야한다'는 관점에서 std::unique_ptr(클래스 독점적으로 사용하는 경우)을 써야한다. 라고 저자가 주장.. 와닿지는 않는다...

```c++
// widget.h
#include <memory>  
class Widget
{
public:
  Widget();
  //~Widget();  //소멸자 제거
private:
  struct Impl;
  std::unique_ptr<Impl> pImpl;  //스마트 포인터를 사용하자
};

// widget.cpp
...
//항목 21에 따라 make_unique 사용
Widget::Widget() : pImpl(std::make_unique<Impl>()){}
....

int main()
{
Widget w; //여기서 에러가 나야하는데 visual c++ 2017에서 안나네?
return 0;
}

```
자동생성되는 소멸자코드에서 unique_ptr이 delete를 호출할때 컴파일러에서 불완전 형식을 체크 (static_asser)한다.

```c++
//std::unique_ptr <Widget::Impl>를 파괴하는 코드가 만들어지는 시점에 Widget::Impl이 완전한 형식이되게 하면 문제가 해결

//위 코드에서 widget.h에
class Widget{
public:
  Widget();
  ~Widget();  //선언만
  ...
};


//widget.cpp에 
Widget::~Widget(){} //소멸자 정의

//이렇게 하면 Widget::Impl이 정의되어 있어 완전한 형식이 된후 Widget이 선언되었다.
```
구현파일에 소멸자를 선언한 의미를 강조하는 용도로
```c++
Widget::~Widget() = default;
```


