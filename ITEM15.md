일반적인 의미로
const : 상수 constexpr : 컴파일 타임 상수(컴파일 시점에 상수가 보장되지 않으면 에러)
constexpr은 모두 const이지만 반대는 성립 X

```
class conT{
  private:
    const int v; //const int sV; 
    static constexpr int sV=20000; //static이 아니면 에러,초기화 리스트로 초기화 불가
  public: //conT(int x) :v(x),sV(30000) {}; conT(int x) :v(x) {};
    void printV(void);
};
void conT::printV(void)
{
  cout << "This v is " << v << "  !! " << sV << " @@@ " << endl;
}
int main(void)
{
  int y=0; cin  >> y; conT t1(100 * y), t2(200 * y);
  //const V는 명시적으로 runtime에 초기화
  t1.printV();
  t2.printV();
  return 0;
}
```
//어떤 변수의 값을 반드시 컴파일 시점에 상수로 사용해야 한다면 constexpr를 사용하라!

constexpr로 객체를 선언했을때-> 컴파일 타임에 객체값이 정해져야한다.
constexpr로 함수가 선언되었을때

2가지 용도1. constexpr 함수의 인자가 컴파일 시점에 알려졌을때? 컴파일시 계산! 2. 알려지지 않는 하나이상의 값들로 constexpr 함수 호출? 보통함수처럼 작동(런타임에 계산)
결국 컴파일 타임에 계산할수도 있는 함수이다 라는 의미

```
constexpr int conX = 100, conY = 200;
int runX = 1, runY = 2;
constexpr int consAdd(int x, int y)
{
  return x + y;
}
int main(void)
{
  cout << "input_ "; cin  >> runY; //
  cout << consAdd(conX, conY) << endl; //compile time 계산
  cout << consAdd(runX, runY) << endl; //run time 
  cout << consAdd(conX, runY) << endl; //run time 
  return 0;
}
```

constexpr를 사용해서 객체를 선언하고 그객체를 받아서 처리하는 constexpr 함수 예제

```
class Point {
  public: constexpr Point(double xVal = 0, double yVal = 0) noexcept  : x(xVal), y(yVal) {}
    constexpr double xValue() const noexcept { return x; }
    constexpr double xYalue() const noexcept { return y; }
    void setX(double newX) noexcept { x = newX; }
    void setY(double newY) noexcept {y = newY; }
  private: 
    double x, y;
};

constexpr Point p1(9.5, 27.7); 
constexpr Point p2(28.8, 5.3);

constexprPoint midpoint(const Point& p1, const Point& p2) noexcept
{
  return {(p1.xValue() + p2.xValue()) / 2 , (p1.xYalue() + p2.xYalue()) / 2 }; //constexpr 멤버함수들을 호출
}

constexpr auto mid = midpoint(p1, p2);//모두 컴파일 타임에 계산된다.

```
제약사항
- constexpr 함수는 literal type(컴파일 도중에 값을 결정할수 있는형식,빌트인 타입) 만 리턴 받을수 있다.
- c++11에서는 constexpr 함수는 최대 한줄만 허용한다(return 포함해서). 그리고 void가 literal type 이 아님(void 리턴불가,생성자제외)
- c++14에서는 c++11의 제약이 사라짐.(class 멤버변수 set함수(void를 리턴하는)를 constexpr 함수로 만들수 있다)
- constexpr 내에서 입출력식 X

c++14에서는 constexpr 함수 void리턴이 가능하므로 x,y에 대한 set constexpr 함수를 만들수 있다.
이로 인해 constexpr  객체로 다시 constexpr 객체 생성이 가능
```
ex)constexpr auto reflectedMid = reflectedMid(mid);
```
constexpr은 객체나 함수의 인터페이스의 일부이다. 누군가가 constexpr을 제거하면 동작을 보장 못한다.
constexpr를 가능한 항상 사용한다는것은 인터페이스(함수,객체 제약)들를 최대한 오래 유지하겠다는 의미를포함한다고 볼수 있다.(저자주)


