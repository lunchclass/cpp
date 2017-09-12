std::bind 보다 lamda를 선호하라
--

std::bind는 TR1 재정 이후로 오랫동안 존재해 와서 많은 개발자들이 익숙할 수도 있는데 C++11과 C++14부터는 

lamda가 거의 언제나 더 강력하다. 이번 항목에서는 lamda를 선호하는 이유를 알아본다.

**더 나은 가독성**

```c++
using Time = std::chrono::steady_clock::time_point;

enum class Sound { Beep, Siren, Whistle };

using Duration = std::chrono::steady_clock::duration;

void setAlarm(Time t, Sound s, Duration d);
```

한시간 후에 30초 동안 울리는 알람을 세팅하려고 한다고 가정하자. 여기서 알람 시간과 울리는 기간은 이미 정해져 

있다. 사운드의 종류만 미리 결정되어 있지 않다.

이에 Sound 값만 받아서 알람을 설정하는 wrapper를 만들려고 합니다. 먼저 lamda는:

```c++
auto setSoundL = [](Sound s) {
  using namespace std::chrono;

  setAlarm(steady_clock::now() + hours(1),
           s,
           seconds(30));
};
```

다음으로 이것을 std::bind 버전으로 구현하려면 아래와 같은 모양이 나온다:

```c++
using namespace std::chrono;
using namespace std::literals;

using namespace std::placeholders;

auto setSoundB =
  std::bind(setAlarm,
            steady_clock::now() + 1h,
            _1,
            30s);
```

- 첫번째로 setSoundL은 param 타입을 명시적으로 알 수 있는데 반해서 setSoundB를 호출할 때는 placeholder 

_1의 타입이 무언지 알기 위해서 setAlarm의 시그너처를 확인해 보아야 한다.

- 두번째로 lamda에서는 표현식 “steady_clock::now() + 1h”가 setAlarm으로 직접 넘어가는 반면에 std::bind의 

경우 “steady_clock::now() + 1h”가 std::bind의 인자로 넘어간다. 즉, 전달된 표현식이 std::bind가 호출될 때 해석

된다. 따라서 이 경우 알람이 setAlarm 실행 후 한시간이 아닌 std::bind call 후 한시간 후에 울리게 된다. 이에 대한 

해결책이 있는데 표현식의 해석을 연기하는 것이다:

```c++
using namespace std::chrono;
using namespace std:: placeholders;

auto setSoundB =
  std::bind(setAlarm,
            std::bind(std::plus<steady_clock::time_point>(),
                      steady_clock::now(),
                      hours(1)),
            _1,
            seconds(30));
```

다음으로 setAlarm이 오버로딩 되는 경우를 더 생각해 보자. 여기서 추가적인 문제가 발생한다.

```c++
enum class Volume { Normal, Loud, LoudPlusPlus };

void setAlarm(Time t, Sound s, Duration d, Volume v);
```

Lamda의 경우 오버로딩 resolution이 인자 3개짜리 setAlarm을 선택하기 때문에 그대로 잘 동작한다. 반면에 

std::bind의 경우 주어진 정보가 setAlarm 함수 이름밖에 없기 때문에 컴파일 에러가 발생한다. 이를 해결하기 위해

서는 아래와 같이 원하는 함수 포인터 형식으로 캐스팅을 해야한다:

```c++
using SetAlarm3ParamType = void(*)(Time t, Sound s, Duration d);

auto setSoundB =
  std::bind(static_cast<SetAlarm3ParamType>(setAlarm),
            std::bind(std::plus<>,
                      steady_clock::now(),
                      h1),
            _1,
            30s);
```

- 여기서 한가지 차이점이 또 발생한다. Lamda의 경우 내부에서 호출하는 setAlarm이 일반 함수이기 때문에 컴파

일러가 inline 함수로 생성할 수 있다. 반면에 std::bind의 경우 함수 포인터를 통해서 호출이 되는데 이 경우 컴파일

러가 inline으로 생성할 확률은 많이 낮아진다. 즉, Lamda로 작성했을 때 컴파일러가 더 효율적인 코드를 생성할 확

률이 높다.

단순한 함수 호출이 아닌 좀 더 복잡한 상황을 구현하려면 std::bind는 더 복잡해 진다.

```c++
auto betweenL =
  [lowVal, highVal]
  (const auto& val)
  { return lowVal <= val && val <= highVal; };
```

```c++
using namespace std::placeholders;

auto betweenB =
  std::bind(std::logical_and<bool>(),
            std::bind(std::less_euqal<int>(), lowVal, _1),
            std::bind(std::less_equal<int>(), _1, highVal));
```

Lamda 버전이 더 짧을뿐 아니라 더 이해하기 쉽고 유지보수하기 쉽다는데 동의하기 바란다. :)

**std::bind의 인자 전달 방식**

```c++
enum class CompLevel { Low, Normal, High };

Widget compress(const Widget& w,
                CompLevel lev);

Widget w;

using namespace std::placeholders;

auto compressRateB = std::bind(compress, w, _1);
```

여기서 w는 compressRateB가 만든 bind object에 저장되는데 값은 by value로 넘어간다. std::bind의 경우 이렇

게 동작한다는 것을 숙지하고 있을 수 밖에 없다. 하지만 lamda의 경우 capture 방식이 명시적이다.

```c++
auto compressRateL =
  [w](CompLevel lev)
  { return compress(w, lev); };
```

파라미터를 넘기는 것도 마찬가지이다. Lamda의 경우:

```c++
compressRateL(CompLevel::Hight);
```

이 예에서는 by value로 넘어가는 것이 명확하고, 구현자의 의도를 명시적으로 반영한다. 반면 std::bind의 경우:

```c++
compressRateB(CompLevel::High);
```

명시적으로 표현되지 않는다. std::bind를 통해 bind object에 넘어가는 모든 parameter는 by reference로 넘어간

다는 것을 알고 있는 방법밖에 없다.

Lamda에 비해서 std::bind는 읽기 어렵고, 표현하기 어렵고 덜 효율적이다. std::bind는 C++14에서는 합리적인 

use case가 없다. 다만 C++11에서는 두가지 제약적인 상황에서는 필요할 때가 있다:

 - Move capture: C++11 lamda는 move capture를 지원하지 않는다.
 - Polymorphic function objects

```C++
class PolyWidget {
public:
  template<typename T>
  void operator()(const T& param);
  ...
};

PolyWidget pw;

auto boundPW = std::bind(pw, _1);

boundPW(1930);

boundPW(nullptr);

boundPW("Rosebud");
```

C++11 lamda에서는 이렇게 할 수 있는 방법이 없었다. 하지만 C++14 부터는 auto paramter를 사용해서 이를 간

단히 해결할 수 있다:

```c++
auto boundPW = [pw](const auto& param)
               { pw(param); };
```
