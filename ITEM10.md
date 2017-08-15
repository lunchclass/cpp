# Item 10: Prefer scoped enums to unscoped enums.
## Why? 
## 3 reason

## 1. Name leakage
### Unscoped enums (default in C++98)
#### - Default enum is global scope (leaking names)
```
enum Color { black, white, red };    // black, white, red are
                                     // in same scope as Color
auto white = false;                  // error! white already
                                     // declared in this scope
```
### Scoped enum (= enum class, from C++11)
#### - Scoped, need namespace
```
enum class Color { black, white, red }; // black, white, red are scoped to Color
auto white = false;    // fine, no other "white" in scope
Color c = white;       // error! no enumerator named "white" is in this scope
Color c = Color::white;    // fine
auto c = Color::white;     // also fine (and in accord with Item 5's advice)
```

## 2. Implicit conversions
### In C++98
#### - Allowed implicit type conversion
```
enum class Color { black, white, red };    // enum is now scoped
Color c = Color::red;     // as before, but with scope qualifier
if (c < 14.5) {     // error! can't compare Color and double
 auto factors =     // error! can't pass Color to
   primeFactors(c); // function expecting std::size_t
```
### In C++11 (enum class)
#### - Need explicit type conversion
```
if (static_cast<double>(c) < 14.5) {    // odd code, but it's valid
 auto factors =    // suspect, but
 primeFactors(static_cast<std::size_t>(c));    // it compiles
 …
}
```

## 3. Forward declaration
### In C++98
#### - Can not use forward declared enum
### In C++11 (it can)
#### - enum class
```
enum class Status;    // forward declaration
void continueProcessing(Status s);    // use of fwd-declared enum
```
#### - unscoped enum(C++98 style)
#### Import 'underlying type specification'. default enum type is int
#### In C++98, there is no underlying type, compiler decide it for optimization
```
enum class Status: std::uint32_t; // underlying type for Status is std::uint32_t (from <cstdint>)
```
#### - enum class can also use underlying type
```
enum class Status: std::uint32_t { good = 0,
 failed = 1,
 incomplete = 100,
 corrupt = 200,
 audited = 500,
 indeterminate = 0xFFFFFFFF
 };
```

Appendix
## Most in cases scoped enum is better, but sometimes need unscoped 
### for example, std::tuple with std::get (accept std::size_t)
```
using UserInfo =    // type alias; see Item 9
 std::tuple<std::string,    // name
            std::string,    // email
            std::size_t> ;  // reputation
 
UserInfo uInfo;    // object of tuple type
…
auto val = std::get<1>(uInfo); // get value of field 1
```
#### using unscoped enum
```
enum UserInfoFields { uiName, uiEmail, uiReputation };
UserInfo uInfo; // as before
…
auto val = std::get<uiEmail>(uInfo); // ah, get value of email field, implicit type conversion
```

#### using scoped enum
```
enum class UserInfoFields { uiName, uiEmail, uiReputation };
UserInfo uInfo;    // as before
…
auto val =
 std::get<static_cast<std::size_t>(UserInfoFields::uiEmail)>    // ????
 (uInfo);
```
#### Need getting constant value using underlying type helper template function
```
template<typename E> // C++14
constexpr auto
 toUType(E enumerator) noexcept
{
 return static_cast<std::underlying_type_t<E>>(enumerator);
}

auto val = std::get<toUType(UserInfoFields::uiEmail)>(uInfo);
```
