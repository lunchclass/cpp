# 객체를 클로저 안으로 이동하려면 초기화 갈무리를 사용하라

클로저 안으로 객체를 이동 시켜야 할때가 있다. 이동 전용 객체 혹은 복사보다 이동 성능이 더 좋은 객체를 
클로저 안에서 사용해야 하는 경우이다. 하지만 C++11 에서는 이를 지원하지 않으며 C++14에서는 초기화 캡쳐를 통해 가능하다.
C++11에서는 편법을 사용해서 흉내 낼 수 있다.

#### c++14 초기화 캡쳐 
람다로 부터 생성되는 클로저  클래스에 속한 자료 멤버의 이름을 지정하고 표현식을 통해 초기화 할 수 있다.

```c++
class Widget {
  public:
    void test() {
      std::cout << "test call\n";
    }
};

int main() {

   auto pw = std::make_unique<Widget>();

   auto func = [pw = std::move(pw)]{
    return pw->test();
   };

   func();

  return 0;
}
```
위 코드에서 = 의 좌변은 클로저 클래스 안의 자료멤버 이름이고, 우변은 그것을 초기화 하는 표현식이다.

#### c++11 에서 이동 캡쳐를 흉내내는 방법
바인드의 역활은 인자로 넘어온 함수와 매개변수로 함수객체를 생성하여 반환하는 기능을 가지고 있다.

바인드는 함수객체 내에 매개변수로 전달된 객체를 포함할 때 왼값 매개변수는 복사되고 오른값 매개변수는 이동된다.
이 성질을 이용하여 c++11 클로저의 한계를 우회한다.

바인드 함수 객체가 호출되면 바인드에 저장된 매개변수들이 첫 매개변수로 지정된 객체에 전달된다.
```c++
class Widget {
  public:
    void test() {
      std::cout << "test call\n";
    }
};

int main() {

   auto func = std::bind([](std::unique_ptr<Widget>& pw){
        return pw->test();
       }, std::unique_ptr<Widget>(new Widget()));

   func();

  return 0;
}
```
