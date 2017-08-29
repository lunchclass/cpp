# 22. Pimpl 관용구를 사용할때는 특수 멤버함수들을 구현파일에서 정의 하라

## 뭔소리임?
클래스의 자료멤버들을 구현클래스(or 구조체)를 가리키는 포인터로 대체하자.

## 왜?
클래스 해더간의 의존성이 클수록 컴파일이 늦어지고 컴파일 범위도 커진다. 그러니 의존성을 줄여보자.

```
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

```
//widget.h
class Widget_98
{
public:
  Widget_98();
  ~Widget_98();
private:
  struct Impl;  //단순 선언만 
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

 /** 구현부에서 필요한 부분만 include .*/
```



