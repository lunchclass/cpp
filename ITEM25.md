# 오른값 참조에는 std:move를, 보편 참조에는 std::forward를 사용하라

#### 보편 참조와 forward를 이용하여 중복적재를 피하자

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



