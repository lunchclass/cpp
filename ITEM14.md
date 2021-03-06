# Item 14 : 예외를 방출하지 않을 함수는 noexcept로 선언하라

### throw & noexcept
c++11에서 함수 선언시 그 함수가 예외를방출하지 않을것임을명시할 때 noexcept라는 키워드를 사용
noexcept를 사용하는 것은 인터페이스 설계에 중요하며,함수의 호출자는 함수의 noexcept여부를 통해 코드의 예외 안전성을 체크할 수 있다.

```c++
//c++98방식
int f( int x ) throw();
virtual void open() throw(FileNotFound, SocketNotReady, InterprocessObjectNotImplemented) // 예외 한정 확장의 경우
struct<typename T> // 예외 한정 불가
{
    void CreateOtherClass() { T t{}; }
};

//c++11방식
int f( int x ) noexcept;
```

### 최적화 유연성
c++98의 경우 예외 명세가 위반되면 호출 스택이 f를 호출한 지점에 도달할 때까지 풀린다. (unwind)
c++11의 경우프로그램 실행이 종료되기 전에 호출 스택이 풀릴수도 있고 풀리지 않을 수도 있다.

호출 스택이 풀리는 것과 풀릴 수도 있는 차이는 컴파일러의 코드 작성에 큰 영향을 끼친다.
예외 명세가 throw()인 함수의 경우 최적화 유연성이 없으며, 예외 명세가 아예 없는 함수 역시 마찬가지이다.

반환형식 함수이름( 매개변수 목록 ) noexcept; //최적화 여지가 가장 크다
반환형식 함수이름( 매개변수 목록 ) throw(); //최적화 여지가 적다.
반환형식 함수이름( 매개변수 목록); //최적화 여지가 적다.

### noexcept인 경우 동작하는 기능
std::vector에 새 요소를 추가할 경우, 충분한 공간이 부족할 경우 std::vector은 더 큰 메모리 공간을 할당하고 사용한다. 
c++98에서는 기존 메모리에서 새 메모리로 일일이 복사후 기존 메모리에 객체를 파괴하여 강력한 예외 안전성을 복사하였다.

c++11에서는 std::vector 요소들의 복사를 이동으로 대체함으로써 요소 옮기기를 최적화하는 것이 자연스러운 방식이다. 
하지만 push_back의 예외 안전성 보장이 위반될 수 있다.(특정 객체를 이동하는 도중 오류가 발생할 경우, 원래대로 복원하는 것이 불가능 하기 때문) 
이러한 문제 때문에 c++11컴파일러는 예외 방출이 확실하지 않을 경우에는 복사 연산들을 이동연산으로 변경하지 않는다.

std::vector::push_back, 표준 라이브러리의 여러 함수는 "가능하면 이동하되 필요하면 복사한다" 전략을 활용한다. 
( std::vector::reserve, std::deque::insert 등 ) 이러한 함수들은 예외를 방출하지 않음이 알려진 경우에만 복사 연산에서 이동 연산으로 대체한다. 
이러한 예외를 방출하지 않는 기준은 주어진 연산이 noexcept로 선언되어 있는지 확인한다.

예) swap, swap는 noexcept여부를 사용자 정의 swap들의 noexcept 여부에 의존한다.
```c++
template< class T, size_t N >
void swap( T (&a) [N], T(&b) [N] ) noexcept ( noexcept ( swap(*a, *b)));

template< class T1, class T2 >
struct pair{
   ...
   void swap( pair& p) noexcept( noexcept( swap(first, p.first )) && noexcept( swap ( second, p.second)));
}
```
이 함수들은 조건부 noexcept이다. 즉, noexcept인지의 여부는 noexcept 절 안의 표현식들의 noexcept 인지에 의존한다. 
noexcept인지의 여부에 따라 swap함수를 작성할 때에는 가능한 한 항상 noexcept를 지정하는 것이 바람직하다.

```c++
template <typename T>
typename std::conditional<
    !std::is_nothrow_move_constructible<T>::value && std::is_copy_constructible<T>::value,
    const T&,
    T&&
>::type move_if_noexcept(T& x);
```
이동(move) 또는 복사(copy)될 수 있는 "X" 를 인자로 받아, 
이동 생성자가 noexcept이면 std::move(X)를 반환, 그렇지 않으면 X를 반환한다.

### noexcept를 잘 알고 사용하자
- noexcept 였다가 후에 후에 바뀌는 경우
- 예외 중립적인 함수
- noexcept를  쓰기 위해 함수 내부에서 예외 처리 등을 많이 해서 성능의 이익을 못보는 경우
- 메모리 해제 함수와 모든 소멸자는 암묵적으로 noexcept로 선언된다.
- 넓은 계약을 가진 함수들에 대해서만 noexcept 키워드를 사용
넓은 계약 : 함수를 호출하기 위한 사전 조건이 존재하지 않는다.
```c++
void f(const std::string& s) noexcept;
// 전제조건 : s.length() <= 32
```

위 경우 미 정의 행동에 대해 원인을 추적하기 쉽지 않다. 기본적으로 오류 검출을 위한 직접적인 접근 방식은 "전제 조건이 위반 되었음"
을 나타내는 예외를 던지는 것이지만 , f는 noexcept로 선언되어 있으므로 불가능

그 결과 라이브러리 설계자들은 넓은 계약을 가진 함수들에 대해서만 noexcept를 사용하려는 경향이 있다.

### Reference
http://egloos.zum.com/sweeper/v/3148916
