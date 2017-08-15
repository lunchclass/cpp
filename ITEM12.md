# ITEM12: 재정의 함수들을 override로 선언하라!

## 재정의가 되려면 다음과 같은 조건을 만족해야 한다.
1. 부모 클래스의 함수가 virtual일 것.
2. 함수 이름이 반드시 같아야 할 것. (소멸자 제외)
3. 함수 인자의 타입이 반드시 같아야 할 것.
4. 함수의 상수성이 일치해야 할 것.
5. 리턴타입(return type)과 예외지정(exception specification)이 호환가능해야 할 것.
6. 멤버함수들의 참조한정사(reference qualifier)가 동일해야 한다. (C++ 11부터 적용)

#### 조건을 만족하지 않으면 함수가 재정의 되지 않는다! 그럼에도 불구하고 아무 문제도 발생하지 않는 것이 더 문제!
```c++
class Base {
 public:
  virtual void mf1() const;
  virtual void mf2(int x);
  virtual void mf3() &;
  void mf4() const;
};

class Derived : public Base {
 public:
  virtual void mf1();                // 4. 상수성이 다름
  virtual void mf2(unsigned int x);  // 3. 인자 타입 다름
  virtual void mf3() &&;             // 6. 참조한정사 다름
  void mf4() const;                  // 1. virtual 없음
};
```

## 그러므로 우리는 override 키워드를 쓴다!
```c++
class Base {
 public:
  virtual void mf1() const;
  virtual void mf2(int x);
  virtual void mf3() &;
  void mf4() const;
};

class Derived : public Base {
 public:
  void mf1() override;                // 4. 상수성이 다름, 그래서 에러
  void mf2(unsigned int x) override;  // 3. 인자 타입 다름, 그래서 에러
  void mf3() && override;             // 6. 참조한정사 다름, 그래서 에러
  void mf4() const override;          // 1. virtual 없음, 그래서 에러
};
```

### 참조한정사(reference qualifier)란 무엇인가?
