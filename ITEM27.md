Universal reference 오버로딩에 대한 대안
--

Universal reference를 오버로딩하는 것은 많은 문제를 일으킨다. 하지만 오버로딩이 유용한 경우에 대한 예도 보여

주었다.

이 장에서는 universal reference를 오버로딩하지 않도록 디자인 하거나, 하더라도 인자 타입에 제약을 걸어서 원하

는 방향으로 활용하는 방법을 알아본다.

**오버로딩 포기하기**

예를들어 logAndAdd 함수를 오버로딩하지 않고 logAndAddName과 logAndAddNameIdx로 나누어 구현하는 방

법이다. 하지만 constructor와 같이 이름이 정해져 있는 경우에는 쓸 수 없고, 또 누가 오버로딩을 포기하고 싶겠는

가?

** Pass by const T& **

Pass-by-universal-reference를 사용하지 않고 pass-by-lvalue-reference-to-const를 사용하는 방법. C++98로 돌

아간 이 방법은 잘 동작하지만 성능 개선은 포기한 방법이다: Move나 emplace 장점을 활용할 수 없다.

** Pass by value **

성능 향상 이유는 항목 41에 설명합니다. 이 방법은 universal reference를 사용하지 않고 그냥 pass-by-value로 넘

어온 parameter를 std::move로 전환해서 사용하는 방법입니다.

```c++
class Person {
public:
  explicit Person(std::string n)
    : name(std::move(n)) {}
  explicit Person(int idx)
    : name(nameFromIdx(idx)) {}
…
private:
  std::string name;
};
```

여기서 std::size_t, short, long 타입의 인자가 넘어오면 int를 받는 함수가 call되게 됩니다.

** Tag dispatch 활용 **

위에서 살펴본 pass by lvalue reference to const와 pass by value 방법은 perfect forwarding을 지원하지 않습니

다. Universal reference를 사용하려는 목적이 perfect forwarding을 활용하는 것이었다면 universal reference를 

사용하는 것 말고는 대안이 없습니다. 하지만 오버로딩도 포기하고 싶지 않다면 어떤 방법이 있을까요?

방법은 오버로딩 된 함수 호출 시 param/arg 조합의 최고 매칭을 선택하는 점을 이용하는 것입니다.

```c++
std::multiset<std::string> names;

template<typename T>
void logAndAdd(T&& name)
{
  auto now = std::chrono::system_clock::now();
  log(now, "logAndAdd");
  names.emplace(std::forward<T>(name));
}
```

이 함수는 잘 동작하겠자만 int 인자를 받는 함수를 오버로딩하면 문제가 생겼었지요? 이를 회피하기 위해서, 오버

로딩을 하지 말고 각각 integral 값과 다른 모든 값을 다루는 두개의 함수에 delegate하는 logAndAdd를 재 구현하

는 방법을 사용해 봅시다. 실제 일을 하는 함수는 logAndAddImpl 하나로 이름을 짓고요, 여기서 오버로딩을 할 것

입니다. 오버로딩된 하나의 함수는 Universal reference를 사용할 것이니까 결국 오버로딩과 universal reference를 

모두 활용하게 되겠네요. 중요한 부분은 두개의 오버로딩 함수의 두번째 param 값으로 인자가 integral 값인지 아닌지를 받을 것입니다.

```c++
template<typename T>
void logAndAdd(T&& name)
{
  logAndAddImpl(std::forward<T>(name),
                           std::is_integral<T>()); // not quite correct
}
```

```c++
template<typename T>
void logAndAdd(T&& name)
{
  logAndAddImpl(std::forward<T>(name),
                           std::is_integral<typename std::remove_reference<T>::type>());
}

template<typename T>
void logAndAddImpl(T&& name, std::false_type)
{
  auto now = std::chrono::system_clock::now();
  log(now, "logAndAdd");
  names.emplace(std::forward<T>(name));
}
```
