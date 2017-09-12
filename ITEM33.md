# ITEM33: std::forward를 통해서 전달할 auto&& 매개변수에는 decltype을 사용하라.

### Lambda의 구현
다음과 같은 람다식은
```c++
auto f = [](auto x) { return func(normalize(x)); };
```

내부적으로 이렇게 구현되어질 수 있다.
```c++
class GeneratedCode {
 public:
  template<typename T>
  auto operator()(T x) const {
    return func(normalize(x));
  }
};
```

### normalize가 lvalue와 rvalue를 각각 다르게 처리한다면?
위의 람다식에서 x는 ```auto```이므로 언제나 lvalue 함수가 호출될 것이다.
문제 해결을 위해 우리가 지난 시간들에서 배운 Forwarding Reference와 std::forward()를 활용하여 Perfect Forwarding을 시도해볼 수 있다.

즉, 다음과 같은 코드를
```c++
class GeneratedCode {
 public:
  template<typename T>
  auto operator()(T&& x) const {
    return func(normalize(std::forward<T>(x)));
  }
};
```

람다로 작성해보면.. WTF
```c++
auto f = [](auto&& x) { return func(normalize(std::forward<???>(x))); };
```

template을 사용했을 때와는 달리 auto의 경우 type parameter ```T```가 없기 때문에 std::forward에 전달해줄 type이 없다.
그래서 우리는 이럴 때 decltype을 사용할 수 있다.
```c++
auto f = [](auto&& x) { return func(normalize(std::forward<decltype(x)>(x))); };
```

# decltype(x)의 문제점
x의 타입을 여과없이 추론함. 즉, x가 rvalue reference였다면, rvalue reference가 나옴. [참조: decltype](https://github.com/jwvg0425/ModernCppStudy/wiki/item-3)
x가 lvalue reference인 경우는 decltype(x)는 lvalue reference이고, 이것에 대한 것은 성대현 선생님이 [Item28](https://github.com/lunchclass/cpp/blob/master/ITEM28.md)에서 다루었음.
x가 rvalue reference인 경우는 decltype(x)는 rvalue reference이고, 이것은 아무런 문제가 없는가?

std::forward의 구현이 다음과 같을 때,
```c++
template<typename T>
T&& forward<remove_reference_t<T>& param)
{
    return static_cast<T&&>(param);
}
```

때려넣으면,
```c++
Widget&& && forward(Widget& param)
{
    return static_cast<Widget&& &&>(param);
}
```

하지만 성대현 선생의 [Item28](https://github.com/lunchclass/cpp/blob/master/ITEM28.md)에 따라 reference collapsing되어 결국에는,
```c++
Widget&& forward(Widget& param)
{
    return static_cast<Widget&&>(param);
}
```
