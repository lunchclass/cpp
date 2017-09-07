# 오른값 참조에는 std:move를, 보편 참조에는 std::forward를 사용하라

### 보편 참조와 forward를 이용하여 중복적재를 피하자
 아래의 중복적재 코드에서는 임시객체가 생성되어 성능이 악영향을 미칠 수 있다.
```c++

// 중복적재
class Widget {
  private:
    std::string name;
  public:
    void setName(cont std::string& newName)
    { name = newName; }
    void setName(std::string&& newName)
    { name = std::move(newName); }
    std::string& getName()
    { return name; }
};

// 보편 참조와 forward 사용
class Widget {
  private:
    std::string name;
  public:
    template<typename T>
    void setName(T&& newName)
    {
      std::cout<<"call"<<"\n";
      name = std::forward<T>(newName);
    }
    std::string& getName()
    { return name; }
};

int main()
{
  std::string n = std::string("lee");
  Widget w;
  
  w.setName(n);
  std::cout<<w.getName()<<"\n";
  
  w.setName(std::string("changsu"));
  std::cout<<w.getName()<<"\n";
  return 0 ;
}

changsu1.lee@VDBS1383:~/work/test$ ./main 
call
lee
call
changsu
```

### std::move나 std::forward는 마지막에 사용하자.
```c++
template<typename T>
void setSignText(T&& text)
{
  sing.setText(text);
  
  auto now = std::chrono::system_clock::now();
  
  signHistory.add(now, std::forward<T>(text));
}
```
text 변수가 값을 잃을 수 있기 때문에 항상 마지막에 사용하여야 한다.


### 함수 리턴이 값전달 이라면 std::move, std::forward를 사용하자
```c++
Matrix operator+(Matrix&& lhs, const Matrix& rhs)
{
 lhs += rhs;
 return std::move(lhs);   // 이동
 //return lhs;            // 복사
}
```
return에 move를 사용하여 불필요한 복사를 줄일 수 있다.

```c++
template<typename T>
Fraction reduceAndCopy(T&& frac)
{
  frac.reduce();
  return std::forward<T>(frac); // 오른값은 이동, 왼값은 복사
}
```
보편참조를 리턴해야 한다면 forward를 사용해야 한다.

### 반환값 최적화의 대상이 될 수 있는 지역 객체에는 std::move, std::forward를 사용하지 말자.
```c++
Widget makeWidget()
{
 Widget w;
 ~
 ~
 return w;
}

Widget makeWidget()
{
 Widget w;
 ~
 ~
 return std::move(w);
}
```
반환값 최적화가 적용되면 함수의 반환값을 위해 마련한 메모리에 w를 생성기 때문에 이동연산도 없앨 수있다. 

#### 반환값 최적화의 요건
1. 지역 객체의 형식이 함수의 반환 형식과 같아야 한다.
2. 지역 객체가 바로 함수의 반환값이어야 한다.
