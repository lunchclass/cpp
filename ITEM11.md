Prefer deleted functions to private undefined ones
---
(The content is an excerpt and summary from Effective Modern C++ for the purpose of this study.)

* Prevent clients from calling a function
  - Don't declare
  - But sometimes C++ declares functions for you: special member functions

* Prevent use of copy ctor, copy assignment operator
  - C++98
    - Declare them private and not define them
    ```c++
    template <class charT, class traits = char_traits<charT> >
    class basic_ios : public ios_base {
    public:
      ...
    
    private:
      basic_ios(const basic_ios&);
      basic_ios& operator=(const basic_ios&);
    };
    ```
    - If code that still has access to them (member functions or friends) uses them, linking will fail.
  - C++11
    - Better way: use "= delete"
    - Mark copy ctor, copy assignment operator as deleted functions
    ```c++
    template <class charT, class traits = char_traits<charT>>
    class basic_ios : public ios_base {
    public:
      ...
      basic_ios(const basic_ios&) = delete;
      basic_ios& operator=(const basic_ios&) = delete;
      ...
    };
    ```
  - Advantages of C++11 way
    - Deleted functions may not be used in any way (e.g member functions or friends): C++98 behavior wouldn't diagonose until link-time.
    - *Any* function may be deleted (i.e. not only member functions)
    ```c++
    bool isLucky(int number);
    ```
    - Some calls that would compile might not make sense:
    ```c++
    if (isLucky('a')) ...
    if (isLucky(true)) ...
    if (isLucky(3.5)) ...
    ```
    - Create delete overloads for the types we want to filter out:
    ```c++
    bool isLucky(char) = delete;
    bool isLucky(bool) = delete;
    bool isLucky(double) = delete;
    ```
    - Prevent use of template instantiations that should be disabled.
    ```c++
    template <typename T>
    void processPointer(T* ptr);
    ```
    - Assume it shouldn't be able to call processPointer with void* or char*:
    ```c++
    template <>
    void processPointer<void>(void*) = delete;

    template <>
    void processPointer<char>(char*) = delete;

    template <>
    void processPointer<const void>(const void*) = delete;

    template <>
    void processPointer<const char>(const char*) = delete;
    ```
  - Declared public, not private by convention
    - When client code tries to use a member function, C++ checks accessibility before deleted status.
  - When you have a function template inside a class:
  ```c++
  class Widget {
  public:
    ...
    template <typename T>
    void processPointer(T* ptr) { ... }

  private:
    template<> // error!
    void processPointer<void>(vode*);
  };
  ```
  ```c++
  class Widget {
  public:
    ...
    template<typename T>
    void processPointer(T* ptr) { ... }
    ...
  };

  template<>
  void Widget::processPointer<void>(void*) = delete;
  ```
  
The C++98 practice of declaring functions private and not defining them was really an attempt to achieve what C++11's deleted functions actually accomplish.
