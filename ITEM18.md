# 소유권 독점 자원의 관리에는 std::unique_ptr를 사용하라

#### 일반 포인터의 단점을 먼저 나열해보면 아래와 같다.

 1. 선언만 봐서는 배열을 가리키는지 객체 하나를 가리키는지 구분할 수 없다.
    그렇기 때문에 delete를 사용해야 하는지 delete[]를 사용해야 하는지 알 수 없다.
 2. 선언만 봐서는 포인터가 객체를 가리키고 있는지 아닌지 알 수 없다.
 3. 선언만 봐서는 메모리를 해제 할 때 delete를 사용하면 되는지 특정 해제 메커니즘을 사용해야 하는지 알 수 없다.
 4. 모든 경로에서 메모리 해제가 정확히 한번 일어남을 보장하기 어렵다.

위 단점을 예를 들어 확인해보자.

```c++
char* foo;
char* bar;
 
foo = factory(...);
bar = factory(...);
```
위  foo와 bar의 해제하기 위해서는 factory 함수의 정의를 확인해서 어떤 포인터를 리턴하는지 확인해야하는 불편함이 있다.

```c++
char* foo;

foo = new char;

if(...) {
  delete foo;
  return;
}

if(...) {
  delete foo;
  return;
}

//using foo
```
위와 같이 함수에서 foo를 할당한 후 발생하는 모든 예외에 대해서 해제함수를 고려해야한다.

#### std::unique_ptr의 특징을 알아보자.

1. 선언에 어떤 해제 함수를 사용해야 하는지 명확히 한다.
2. 선언에 배열의 포인터인지 객체 하나의 포인터인지 명확히 한다.
3. 소멸자를 이용하여 메모리 해제를 하기 때문에 사용자는 scope 만 신경쓰면 된다.
4. 복사가 불가능하고 이동만 가능하기 때문에 객체의 포인터가 공유되는 것을 근본적으로 막는다.

#### std::unique_ptr의 메소드에 대해서 알아보자.

```c++
std::unique_ptr<Foo> up;  // up is empty
std::unique_ptr<Foo> up(nullptr);  // up is empty
std::unique_ptr<Foo> up(new Foo); //up now owns a Foo
std::unique_ptr<Foo, D> up3(new Foo, d); // up owns a Foo, set deleter 
```
생성자는 대표적으로 일반 포인터 하나를 받는 것과 해제 함수를 같이 받는 것이 있다.

```c++
struct Foo {
    Foo() { std::cout << "Foo\n"; }
    ~Foo() { std::cout << "~Foo\n"; }
};
 
int main()
{
    std::cout << "Creating new Foo...\n";
    std::unique_ptr<Foo> up(new Foo());
 
    std::cout << "About to release Foo...\n";
    Foo* fp = up.release();
 
    assert (up.get() == nullptr);
    std::cout << "Foo is no longer owned by unique_ptr...\n";
 
    delete fp;
}
```
release 메소드를 호출하면 일반 포인터를 반환하고 소유권을 놓는다.

```c++
int main()
{
    std::unique_ptr<std::string> s_p(new std::string("Hello, world!"));
    std::string *s = s_p.get();
    std::cout << *s << '\n';
}
```
get method는 일반 포인터를 반환한다.

```c++
int main()
{
    std::cout << "Creating new Foo...\n";
    std::unique_ptr<Foo, D> up(new Foo(), D());  // up owns the Foo pointer (deleter D)
 
    std::cout << "Replace owned Foo with a new Foo...\n";
    up.reset(new Foo());  // calls deleter for the old one
 
    std::cout << "Release and delete the owned Foo...\n";
    up.reset(nullptr);      
}
```
reset method 는 가지고 있던 자원을 해제하고 매개변수로 전달된 자원의 소유권을 갖는다.
