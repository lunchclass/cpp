# std::shared_ptr 처럼 작동하되 대상을 잃을 수도 있는 포인터가 필요하면 std::weak_ptr를 사용하라

#### weak_ptr의 특징
1. std::shared_ptr로 부터 생성된다. 이때 weak_ptr은 shared_ptr과 동일한 객체를 가리키지만 참조 횟수에는 영향을 주지 않는다.
2. weak_ptr이 가리키는 객체가 해제되었는지 확인할 수 있다.

#### weak_ptr 생성
```c++
int main()
{
  auto sp = std::make_shared<int>(42);
  std::weak_ptr<int> wp1(sp);
  std::weak_ptr<int> wp2(wp1);
  std::weak_ptr<int> wp3;
  std::weak_ptr<int> wp4;
  wp3 = sp;
  wp4 = wp3;

  std::cout << wp1.use_count() << "\n";
  std::cout << wp2.use_count() << "\n";
  std::cout << wp3.use_count() << "\n";
  std::cout << wp4.use_count() << "\n";
}

cs-lee@ubuntu:~/work/test$ ./main 
1
1
1
1
```
shared_ptr를 매개변수로한 생성할 수 있고 같은 weak_ptr로도 생성할 수 있다.
또한 shared_ptr를 대입해서 생성할 수있다.

#### weak_ptr 을 이용한 객체 해제 여부 확인
```c++
std::weak_ptr<int> wp;

void f()
{
  if (!wp.expired()) {
    std::cout << "wp is valid\n";
  }
  else {
    std::cout << "wp is expired\n";
  }
}

int main()
{
  {
    auto sp = std::make_shared<int>(42);
    wp = sp;

    f();
  }

  f();
}

cs-lee@ubuntu:~/work/test$ ./main 
wp is valid
wp is expired
```
expired method를 통해 객채 해제 여부를 확인하는 방법이다.
expired() 호출이 끝난 순간에 다른 thread에서 객체의 자원 해제가 일어날 수 있기 때문에다.

```c++
std::weak_ptr<int> wp;

void f()
{
  std::cout << "use_count == " << wp.use_count() << ": ";
  if (auto sp = wp.lock()) {
    std::cout << *sp << "\n";
    std::cout << sp.use_count() << "\n";
  }
  else {
    std::cout << "wp is expired\n";
  }
}

int main()
{
  {
    auto sp = std::make_shared<int>(42);
    wp = sp;

    f();
  }
  f();
}

cs-lee@ubuntu:~/work/test$ ./main 
use_count == 1: 42
2
use_count == 0: wp is expired
```
lock method를 통해 객체 해제 여부를 확인하는 방법이다.
초기화에 사용된 sp가 해제되자 wp.lock()이 null을 리턴한 것을 볼 수 있다.
또한 shared_ptr이 리턴되는 순간 참조 카운트가 증가하기 때문에
자원이 해제되지 않는다.

#### weak_ptr의 사용
```c++
std::shared_ptr<const Widget> fastLoadWidget(WidgetID id)
{
  static std::unordered_map<WidgetID, std::weak_ptr<const Widget>> caceh;
  
  auto objPtr = cache[id].lock();
  
  if (!objPtr) {
    objPtr = loadWidget();
    cache[id] = objPtr;
  }
  return objPtr;
}
```
Widget 팩토리 함수를 구현한 예이다.
성능을 위해 cache를 사용하고 있고 cache내에 있는 widget 객체들의 자원 해제 여부는
shared_ptr을 리턴하기 때문에 함수를 호출한 사용자가 결정할 수 있다. 또한 사용자가
더 이상 사용하지 않는 widget을 cache로 가지고 있지 않으며 해제 여부 확인을 위해 lock 함수를 사용하고 있다.

```c++
A ---std::shared_ptr---> B <---std::shared_ptr--- C


A ---std::shared_ptr---> B <---std::shared_ptr--- C
  <----     ?      ----
```
일반 포인터로 A를 가리키면 A가 해제되었을때 B가 알지못하기 때문에 역참조가가 일어날 수 있다.
shared_ptr을 사용하게되면 A와 B가 순환 참조가되기 때문에 자원이 해제되지 않는 문제가 발생한다.
weak_ptr을 사용하면 A가 해제되었을때 B가 알 수있기 때문에 역참조를 방지 할 수 있으며 참조횟수에 영향을 주지 않기 때문에 자원해제에도 문제가 없다.
