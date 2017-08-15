Anyone can agree that using STL containers is a good idea.

But do you want to use more below declaration than once?:
```
    std::unique_ptr<std::unordered_map<std::string, std::string>>
```

To avoid upper case, use typedef:
```
    typedef std::unique_ptr<std::unordered_map<std::string, std::string>> UPtrMapSS;
```

But typedefs are so~~ C++98.
C++11 offers alias declarations:
```
    using UPtrMapSS = std::unique_ptr<std::unordered_map<std::string, std::string>>;
```

the tepedef and the alias declaration do exactly the same thing.
Why should we select one of them???

Let's compare typedef with alias declarations
(This is hardly a compelling reason to choose alias declaration over typedef):
```
    using FP = void (*)(int, const std::string&);
    typedef void (*FP)(int, const std::string&);
```
A compelling reason does exist: templates. : template.
In particular, alias declarations may be templatized, while typedefs cannot.
typedef:
```
    template <typename T>
    struct MyAllocList {
        typedef std::list<T, MyAlloc<T>> type;
    }
  
    MyAllocList<Widget>::type lw;    //client code
```

Alias Template:
```
    template <typename T>
    using MyAllocList = std::list<T, MyAlloc<T>>;

    MyAllocList<Widget> lw;    //client code
```

It get worse.
```
    template <typename T>
    class Widget {
        typename MyAllocList<T>::type list;
    }
```
 ================  Example  ====================== 

c++11 type trait:
```
std::remove_const<T>::type               // yields T from const T
std::remove_reference<T>::type          // yields T from T& and T&&
std::add_lvalue_reference<T>::type      // yields T& from T
```

c++14 type trait:
```
std::remove_const_t<T>               // c++14 (yields T from const T)
std::remove_reference_t<T>          // c++14 (yields T from T& and T&&)
std::add_lvalue_reference_t<T>      // c++14 (yields T& from T)
```
