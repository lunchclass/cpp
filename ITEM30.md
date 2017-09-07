# 항목 30 : 완벽 전달이 실패하는 경우들을 잘 알아두라.

### 완벽 전달이란?
전달(forwarding)은 말 그대로 한 함수가 자신의 인수들을 다른 함수에 넘겨주는 것을 뜻한다.<br>
이때 전달 받은 함수가 애초에 전달하는 함수가 갖고 있던 것과 동일한 객체를 받게 하는것이다.<br>
범용적인 전달을 위해서는 참조 매개변수들을 사용해야 한다.<br>



```c++
template<typename T>
void fwd(T&& param) // accept any argument 
{
  f(std::forward<T>(param)); // forward it to f 
}
```
