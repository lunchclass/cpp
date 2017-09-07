# ITEM26: Forwarding Reference에 대하여 Overloading을 피하라!

### Forwarding Reference란?
타입 추론이 발생하는 곳에서(template, auto 등) 사용되는 && Reference. 즉, 다음과 같다.
```c++
template<typename T>
void hello(T&& yaho);
```
또는
```c++
auto&& babo;
```

### Overload Resolution Rule
1. 묵시적 타입 캐스팅(Implicit type casting)이 일어나는 것보다 타입이 정확하게 일치하는 것을 우선한다.
2. 템플릿과 템플릿이 아닌 함수들이 똑같은 타입을 가지면 템플릿이 아닌 함수를 우선한다.

### Perfect Forwarding
내가 받은 대로 돌려주는 것. [참조](http://blog.naver.com/losemarins/221011603702)

### Example1
```c++
template<typename T>
void hello(T&& world)
{
    std::string yaho = world;
    ...
}

void hello(int world)
{
    ...
}

hello(10);          // Ok
hello("sibal");     // Ok
hello((short) 20);  // Fucking
```

### Example2
```c++
class Person {
 public:
  template<typename T>
  explicit Person(T&& n) : name(std::forward<T>(n)) {}
  explicit Person(int idx);
  
  Person(const Person& rhs);  // defined by compiler
  Person(Person&& rhs);       // defined by compiler
};

Person p("Nancy");
auto cloneOfP(p);   // Fucking
```
