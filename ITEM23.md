std::move와 std::forward를 숙지하라

std::move - 주어진 인수를 무조건 오른값으로 캐스팅
std::forward - 특정 조건이 만족될 때에만 오른값으로 캐스팅

c++11에서 std::move를 구현한 예는 하기와 같다

```
template<typename T> // in namespace std 
typename remove_reference<T>::type&&
move(T&& param)
{
     using ReturnType =                       // alias declaration;
       typename remove_reference<T>::type&&;  // see Item 9
return static_cast<ReturnType>(param); 
}
```

위에서 문제점은 형식 T가 왼값 참조이면 T&&는 왼값 참조가 된다는것이다(보편참조)
따라서 리턴에서 강제 캐스팅을 해주고 있다.

조금더 간단하게는 아래와 같이 변경가능하다.

```
template<typename T> 
decltype(auto) move(T&& param) 
{
  using ReturnType = remove_reference_t<T>&&;
  return static_cast<ReturnType>(param);
}
```

std::move라는 이름은 이동할 수 있는 객체를 좀 더 쉽게 지정하기 위한 함수라는 
점에서 붙은것이다.

```
class Annotation {
public:
  explicit Annotation(const std::string text)
  : value(std::move(text)) // "move" text into value; this code 
  { ... }   // doesn't do what it seems to!
  ...
private:
    std::string value;
};
```

이 코드는 자료 멤버 value를 text의 내용으로 설정한다.
이 코드가 독자의 의도를 완벽하게 실현하지 못하는 유일한 결함은, 
text가 value로 이동하는 것이 아니라 복사된다는 것이다.

```
class string {            // std::string is actually a
   public:                   // typedef for std::basic_string<char>
  ...
  string(const string& rhs);  // copy constructor
  string(string&& rhs);       // move constructor
  ...
};
```

std::move(text)의 결과는 const std::string 형식의 오른값이다.
이동 생성자는 const를 받지 않고 있기에, 이동생성자 호출이 불가능하고,
복사 생성자만 호출이 가능하다.

이런 행동은 c++ 언어가 아래의 문제를 미연에 방지하기 때문에 일어나지 않는다.
"한 객체의 어떤 값을 바깥으로 이동하면 그 객체는 수정되며,
따라서 전달된 객체를 수정할 수도 있는 함수에 const 개체를 전달하는 행위"

여기서 배울 2가지는 아래와 같다.
1. 이동을 지원한 객체는 const로 선언하지 말자.(이동 요청은 아무런 warning없이 복사된다)
2. std::move는 아무것도 실제로 이동하지 않은 뿐더러,
  캐스팅 되는 객체가 이동 자격을 갖추게 된다는 보장도 제공하지 않는다.
  
