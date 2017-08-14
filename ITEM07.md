항목 7 : 객체 생성시 괄호와 중괄호를 구분하라.


1) 기본 객체 생성
객체를 생성 할 때 아래와 같이 할 수 있다.

int x(0) ; // 초기치를 괄호로 감싼 예
int y = 0; // 초기치를 "=" 다음에 지정한 예 
int z { 0 }; // 초기치를 중괄호로 감싼 예
int z = { 0 } // "="와 중괄호로 초기치를 지정한 예, 대체로 중괄호만 사용한 구문과 동일

Widget w1; // 기본 생성자를 호출
Widget w2 = w1; // 배정이 아님; 복사 생성자를 호출
w1 = w2; 배정; 복사 배정 연산자를 호출

2) 균일 초기화의 장점 (중괄호 초기화)
C++98도 여러가지 초기화 구문을 지원했지만, 원하는 초기화를 명시적으로 표한할수 없는 상황이 있다.
(ex. 서로 다른 임의의 값들을 담는 STL 컨테이너를 직접 생성하는 것)

C++11은 균일 초기화를 도입! 이것은 어디서나 사용할 수 있고 모든 것을 표현할 수 있는 단 한 종류의 초기화 구문.
std::vector<int> v{ 1, 3, 5 }; // v의 초기 내용은 1, 3, 5

class Widget
{
private:
    int x{0}; // OK
    int y = 0; // OK
    int z(0); // NOT OK
    // not-static value에 대한 default값을 초기화 하는데 ()는 안된다.
}

복사할 수 없는 객체(이를테면 std::atomic )는 중괄호나 괄호로는 초기화할 수 있지만 "="로는 초기화 할 수 없다.

std::automic<int> ai1{ 0 } // OK
std::automic<int> ai2(0) // OK
std::automic<int> ai3 = 0 // NOT OK
// Copy가 안되는 Object에 대해서는 ()는 되는데 =는 안된다.

위 내용을 보면 어디서나 사용할 수 있는 것은 {} 뿐이다. (중괄호 초기화를 '균일' 초기화라고 부르는 이유!)

중괄호 초기화의 혁신적인 기능 하나는 암묵적 좁히기 변환을 방지해 준다!

double x, y, z;
int sum1{ x + y + z }; //오류 double들의 합을 int로 표현하지 못할 수 있음

괄호나 "="를 이용한 초기화는 이러한 좁히기 변환을 점검하지 않는다.
int sum2(x + y + z); // OK
int sum3 = x + y + z; // OK

중괄호 초기화의 또 다른 주목할 만한 특징! C++에서 가장 성가신 구문 해석(most vexing parse)에 자유롭다!

C++에서 가장 성가신 구문 해석은 "선언으로 해석할 수 있는 것은 항상 선언으로 해석해야 한다."는 C++의 규칙에서 비롯된 하나의 부작용

Widget w1(10) // 인수 10으로 Widget의 생성자를 호출
Widget w2(); // 가장 성가신 구문 해석! Wideget을 돌려주는, w2라는 이르의 함수를 선언

중괄호를 이용해서 객체를 기본 생성할 때에는 이러한 문제를 겪지 않는다.

Widget w3{}; // 인수 없이 Widget의 생성자를 호출

중괄호 초기화에는 칭찬할점이 많다.
가장 다양한 문맥에서 사용할 수 있는 구문! 암묵적인 좁히기 변환을 방지! C++의 가장 성가신 구문 해석으로부터 자유롭다.

3) 중괄호 초기화의 단점 및 std::initializer_list
하지만 중괄호 초기화의 단점은 종종 예상치 못한 행동을 보인다.
중괄호 초기화, std::initializer_list, 생성자 중복적재 해소 사이의 뒤얽힌 관계에서 비롯된다.

다른 방식으로(auto를 사용하지 않고) 선언된 변수에서는 중괄호 초기치가 좀 더 직관적인 형식으로 연역 되었지만
auto로 선언된 변수에 대해서는 std::initializer_list 형식으로 연역되는 경우가 많다.

class Widget {
public:
  Widget(int i, bool b);
  Widget(int i, double d);
};

Widget w1(10, true); // 첫 생성자를 호출
Widget w2{10, true); // 첫 생성자를 호출
Widget w3(10, 5.0); // 둘째 생성자를 호출
Widget w4{10, 5.0} // 둘째 생성자를 호출

But! 생성자 중 하나 이상이 std::initializer_list 형식의 매개 변수를 선언시
중괄호 초기화 구문은 이상하게도 std::initializer_list를 받는 중복 적재 버전을 강하게 선호한다.

class Widget {
public:
  Widget(int i, bool b);
  Widget(int i, double d);
  Widget(std::initializer_list<long diouble> il); // 추가
};

// 중괄호를 사용한 경우; std::initializer_list 생성자 호출 (10, true가 long double로 변환)
Widget w2{10, true}; 
Widget w4{10, 5.0};

보통은 복사 생성이나 이동 생성이 일어났을 상황에서도 std::initializer_list 생성자가 끼어들어서 기회를 가로챈다.

class Widget {
public:
  Widget(int i, bool b);
  Widget(int i, double d);

  Widget(std::initializer_list<bool> il); // 요소의 형식이 bool
  // 암묵적 변환 함수는 없음

};
Widger w{10, 5.0} // 오류 좁히기 변환이 필요함

컴파일러가 자신의 결심을 포기하고 보통의 중복적재 해소로 물러나는 경우는 중괄호 초기치의 인수 형식들을
std::initializer_list 안의 형식으로 변환하는 방법이 아예 없을 때!

std::initializer_list<std::string>

class Widget {
public:
  Widget(int i, bool b);
  Widget(int i, double d);
  Widget(std::initializer_list<std::string> il); // 추가
};

Widget w2{10, true}; // 중괄호 사용, 첫 생성자 호출
Widger w4{10, 5.0}; // 중괄호 사용, 둘때 생성자 호출

// 빈 중괄호 쌍
class Widget {
public:
  Widget(); // 기본 생성자
  Widget(std::initializer_list<int> il); // std::initializer_list 생성자
  // 암묵적 변환 함수 없음
};

Widget w1; // 기본 생성자를 호출
Widget w2{}; // 기본 생성자를 호출
Widget w3(); // 가장 성가신 구문 해석! 함수 선언!

// 만약 std::initializer_list 생성자를 호출하고 싶다면 -> {{}} , ({})
Widget w4({});
Widget w5{{}};

// std::vector
std::vector에는 컨테이너의 초기 크기와 컨테이너의 모든 초기 요소의 값을 지정할 수 있는 
비 std::initializer_list , std::initializer_list 생성자가 있다.

std::vector<int> v1(10, 20); // 모든 요소의 값이 20인, 요소 10개짜리 std::vector가 생성됨
std::vector<int> v2{10, 20}; // 값이 각각 10, 20인 두 요소를 담은 std::vectore가 생성됨

4) 정리
첫째. 
클래스를 작성할 때에는, 만일 일단의 중복적재된 생성자 중에 std::initializer_list를 받는 함수가 하나 이상
존재한다면, 중괄호 초기화 구문을 이용하는 클라이언트 코드에는 std::initializer_list 중복적재들만 적용 될 수 있음을 주의

둘째.
클래스 사용자로서 객체를 생성할 때 괄호와 중괄호를 세심하게 선택해야 한다는 것이다.
대부분의 개발자는 둘 중 하나를 기본으로 삼아서 항상 사용하고, 다른 하나는 꼭 필요할때만 사용한다.
중괄호를 기본으로 사용하는 경우
1. 다양한 문맥에 적용할 수 있다.
2. 좁히기 변환을 방지해 준다.
3. 가장 성가신 구문 해석에서 자유롭다.
- 이때 괄호가 필요한 경우가 있음을 이해 (ex. std::vector)

괄호를 기본으로 사용하는 경우
1. C++ 구문적 전통과의 일관성
2. auto가 std::initializer_list를 연역하는 문제가 없다는 점
3. 객체 생성시 의도치 않게 std::initializer_list가 생성자가 호출되는 일이 없다는 점
- 중괄호로만 가능한 경우가 있음을 시인 (구체적인 값들로 컨테이너를 생성)

조언은 둘중 하나를 선택해서 일관되게 적용하라는 것!

템플릿 작성자에게는 객체 생성시 괄호와 중괄호의 긴장 관계가 특히나 짜증스러울 수 있는데 둘 중 어느것을
사용해야하는지를 판단하는 것이 불가능하다.
ex.
임의의 개수의 인수들을 지정해서 임의의 형식의 객체를 생성하는 경우
template<typename T, typename... Ts>
void doSimeWork(Ts&&... params) {
  params... 으로부터 지역 T 객체를 생성한다.
}

T localObject(std::forward<Ts>(params)...); // 괄호를 사용
T localObject{std::forward<Ts>(params)...}; // 중괄호를 사용

std::vector<int> v;
...
doSomeWork<std::vector<int>>(10, 20);
만약 localObject가 괄호를 사용했다면 요소가 10개인 std::vector
중괄호를 사용했다면 요소가 2개인 std::vector이다.

내부적으로 정책을 정하고 인터페이스의 일부에 문서화 함으로써 문제 해결(std::make_unique, std::make_shared)

5) 참고
https://www.slideshare.net/seokjoonyun9/c-korea-effective-modern-c-study-item-7-distinguish-between-and-when-creating-objects
