# 특수 멤버 함수들의 자동 작성 조건을 숙지하라
### 특수 멤버함수(special member function)
From C++ 98
-------
### - 기본생성자
### - 소멸자
### - 복사생성자(copy constructor)
### - 복사 배정 생성자(copy assignment)


From C++11
-------
### - 이동 생성자(move constructor)
### - 이동 배정 생성자(move assignment constructor)
#### 1. 둘을 합쳐서 이동 연산자라고 부르자. non-static member에 std::move를 적용하는것
#### 2. 멤버별 이동을 **요청** 한다고 생각하는편이 좋다(item 23)

________________________________
###  이동 연산자의 자동 생성 규칙
#### 1. C++98의 복사 연산과 유사하게 명시적으로 추가한 연산은 자동으로 생성 되지 않는다
#### 2. 이동 연산자는 하나를 만들면 다른 하나가 자동으로 생성되지 않는다
####    ex) 이동 생성자를 추가하면, 이동 배정은 자동으로 생성되지 않음. 반면 복사연산은 그렇지 않다
#### 3. 복사 생성자를 하나만 만들어도 이동 연산자들은 자동 생성되지 않는다.
#### 4. 반대로 이동 연산자를 하나만 만들어도 복사 연산자가 생성되지 않는다.

### Rule of Three(3의 법칙)
#### 복사 생성자, 복사 배정 연산자, 소멸자 중 하나라도 선언했다면 나머지 둘도 선언해야 한다.
#### => 자원(멤버변수)과 연관된 연산들이기 때문. 특히 소멸자는 힙등 자원을 위해 해제하는 경우가 많음
#### 


### 정리
#### 1. 클래스에 어떤 복사 연산도 선언되어 있지 않다
#### 2. 클래스에 어떤 이동 연산도 선언되어 있지 않다.
#### 3. 클래스에 소멸자가 선언되어 있지 않다.(3의 법칙)
#### 
#### 왜? => 기본 제공 멤버별 복사나 이동 등이 적합하지 않아서 개발자가 직접 추가했을테니 기본 이동 연산자가 적합하지 않을 가능성이 크다
#### (다들 그렇게 생각하시나요?)
-------------------------------

### 기본 연산자들을 살려두기
```c++
class Base { 
  public:
  virtual ~Base() = default;    // make dtor virtual
  Base(Base&&) = default;       // support moving  Base& operator=(Base&&) = default;
  Base(const Base&) = default;  // support copying  Base& operator=(const Base&) = default;
  …
  Base(Base&&) = default;       // support moving  Base& operator=(Base&&) = default;
  Base(const Base&) = default;  // support copying  Base& operator=(const Base&) = default
};
```

----------------------------------------------
### 생각해 볼 점
```c++
class StringTable {
 public:
  StringTable()  { makeLogEntry("Creating StringTable object"); }  // added
  ~StringTable()                                                   // also
  { makeLogEntry("Destroying StringTable object"); }               // added
  …                                                               // other funcs as before

  private:
  std::map<int, std::string> values;                               // as before
 }; 
```
#### - 최초에 소멸자를 정의하지 않았다면 move/copy 등이 기본 제공되었을것(STL 기본제공)
#### - 로깅등을 위해 소멸자를 추가하면, 의도치 않게 모든 연산들이 move 에서 copy 로 변경되어 성능하락이 될 수 있음
#### - 명시적으로 default 를 사용하는것이 좋다. 

-------------------------------------------------
### 결론
#### 1. 특수 멤버함수인 생성자, 소멸자, 복사연산자, 이동 연산자의 생성조건들을 알아두자
#### 2. C++98 과는 다르게 복사생성자는 이동 연산자를 추가하면 자동 생성되지 않는다.
#### 3. 소멸자를 추가하면 이동 연산자는 자동 생성되지 않는다.


