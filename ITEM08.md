# 0과 Null보다 nullptr을 선호하라 & 중복 적제를 피하라
- 0 -> int
- NULL -> 컴파일러에 따라 int이외의 것도 될수 있지만 중요한것은 포인터 형식이 아니다. (구체적인 형식이 컴파일러의 재량에 맡겨짐)

```

void f(int );

void f(bool);

void f(void *);

f(0);     // f(void*)  f(int) 호출

f(NULL);  // 컴파일이 안될수도 있지만 보통은 f(int)이고 f(void *)가 아니다.

```

- f(NULL)의 외관상 의미는 널 포인터로 f를 호출한다 -> 실제 동작은 정수로 f를 호출한다.

- nullptr 장점은 정수 형식이 아님 : std:nullptr_t(실제 형식)이며 모든 포인터 형식으로 암묵적 변환이 된다.
```
f(nullptr); //f(void*) 호출
```

- nullptr 는 코드의 명확성을 높여준다.

```
auto result = findRecord(/* parameter list */)

if (result == 0) { ////코드의 중의성이 생김. auto 추론이 어려울경우 포인터인지 0인지 명확히 설명하기 어렵다.
 ...
}

if (result == nullptr){
...
}
```


# nullptr template에서 유용하다.

```
//
int f1(std::shared_ptr<Widget> spw);
double f2(std::unique_ptr<Widget> upw);
bool f3(std::Widget* pw);

std:mutex f1m, f2m, f3m;

using MuxGuard = std::lock_guard<std::mutex>; //c++11 typedef 
...
{
  MuxGard g(f1m);
  auto result = f1(0);
}
...
{
  MuxGard g(f2m);
auto result = f2(NULL);
}
...
{
  MuxGard g(f3m); 
  auto result = f3(nullptr);
}
```

- 실행구문을 템플릿 함수로 묶었을 경우

```
template<typename Functype,
         typename MuxType,
         typename PtrType>
auto lockAndCall(FuncType func, MuxType mutex, Ptrypte ptr) -> decltype(func(ptr))
{
  using MuxGard = std::lock_guard<MuxType>;
  return func(ptr);
}

...
auto result1 = lockAndCall(f1, f1m, 0);       //PtrType -> int deduction , compile error 
        
auto result1 = lockAndCall(f2, f2m, NULL);    //PtrType -> integer deduction, compile error 

auto result1 = lockAndCall(f3, f3m, nullptr); //ok

```














