## Item 29: 이동 연산이 존재하지 않고, 저렴하지 않고 ,적용되지 않는다고 가정하라

C++ 11에서 가장 주된 기능은 이동 시멘틱이다. 

일반적으로 비싼 복사 세멘틱을 이동 시멘틱으로 대체하는 경우 c++98코드 기반을 c++11을 준수하는 컴파일러와
표준라이브러리로컴파일 하면 소프트웨어거 저절로 더 빠르게 향상되길 기대한다. 

하지만 현실은 X, 안되는 경우가 많다. 이동 시맨틱을 사용했을때 성능이 증가 되는 경우를 명확히 알아야 할 필요가 있다.

C++11은 C++98 표준 라이브러리를 전체적으로 개선되었으나 (이동이 복사보다 빠른경우 이동 연산이 추가됨) 

이동 시멘틱의 장점을 활용하도록 되어 있지 않는 기존 코드들은 단순 컴파일로 성능향상을 볼수 없다.
이동 시멘틱을 사용 한 방식으로 코드를 결국 수정해야함. 

ex)

### 1. 이동 연산이 없다. 
이동할 객체가 이동연산들을 제공X, 이동 연산 요청은 복사 연산으로 대체됨.
일반적으로 컴파일러가 이동연산을 만들어 주게 되어 있으나, 이동연산 코드를 만들어주지 않는경우가 존재

 - 복사 연산 , 이동 연산 , 소멸자가 정의된 경우 (item 17)
 - 이동연산자가 delete 된경우 (item 11)
 - 이동연산을 구현해야함... 안하면 어차피 빌드가 안되..
 
 ```c++
#include<iostream>
#include<string>

using namespace std;

class pm {
public:
    int key;
    pm(int a):key(a) { cout << "pm() : key " << key << endl;  }
    ~pm() { cout << "~pm() : key " << key <<  endl; }

    pm(pm& np) :key(np.key) {};

    pm& operator = (pm&& rvalue) {
        cout << "pm move " << endl;
        key = rvalue.key;
        return *this;
    }
pm& operator = (pm& lvalue) {
        cout << "pm copy " << endl;
        key = lvalue.key;
        return *this;
    }
};

class ttt {
public:
    pm _p;
    string tstr;
    ttt(string &p, int b) :_p(b), tstr(p){};

};

int main()
{
    ttt a(string("A객체"), 10);
    ttt b(string("복사연산"), 2000);
    ttt v(ttt(string("0"), 0));
    
    //이동연산 정의가 없는경우, 컴파일러가 만들어주는 함수에 의해 v의 멤버인 pm이 어떤 결과를 리턴하는지 확인
    //pm이 이동이 정의되어 있어 이동연산이 일어남. ttt는 이동연산이 없음 . 

    v = std::move(b); //이동이 있는경우 이동
    v = b;              //이동이 존재할때 복사가 있어야만 컴파일됨, 
}
//std:move는 pm move, v=b는 pm copy가 출력된다
////자동 이동연산은 멤머별 std::move로 만들어지는것 같다.

 ```
 
 ### 2. 이동이 더 빠르지 않다 
 이동할 객체의 이동 연산이 해당 복사 연산보다 빠르지 않다.
 
 ```c++
 std:vector<weget> vw1;
 auto vw2 = std:move(vw1);
 //동작 : vw1이 가리키는 객체를 vw2가 가리키게 하고 vw1 연결을 끊는다. 
 
 std:array<weget> vw1;
 auto vw2 = std:move(vw1);
 //동작 : 모든 원소를 vw2로 이동시 선형시간만큼 실행된다.
 
 ```
 std:array는 복사 연산시 상수시간과 선형시간 두개가 존재 (작은문자열 최적화 SSO)
 작은 문자열의 경우 std:array안의 객체에 데이터를 저장(heap x)
 
 ### 3. 이동을 사용할 수 없다.
 이동이 일어나려면 이동 연산이 예외를 방출하지 않아야 하는 코드(noexcept 함수내부 에서 이동이 적합할 때 인듯)에서
 해당연산이 noexcept로 선언이 되어있지 않다. 컴파일러가 복사로 대체 할 수 있음.
 
 ### 4. 원본객체가 왼값이다.
 아주 드문 경우 경우(item 25)에 따라 오직 오른값만 이동연산의 원본이 될 수 있는 경우도 있다. 왼값은 복사가 되는듯?
 
 ## 정리 
 c++11을 쓴다고 이동 시멘틱 성능 향상을 무조건 얻을수 있는게 아니다. 결국 위 조건이 만족할 경우에만(효과적인 이동연산이 제공될 경우에만)
 이동 시멘틱이 효과를 볼 수 있다. 
  
 
 
