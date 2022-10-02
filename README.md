# cpp-tips
Collection of small tips and tricks for C++

## Trailing return type
```c++
#include <iostream>
#include <type_traits>

// Trailing return type was introduced in c++11
// This not only looks cool, but also helps writing
// template functions with return type dependent on the template type. 
template<typename A, typename B>
auto multiply(A a, B b) -> decltype(a*b)
{
    return a * b;
}

// Note that expressions passed to decltype() are not evaluated.
// If you compile and execute this program, you'll see nothing happens.
// You can also know that it's true because static_assert works.
int foo()
{
    std::cout << "foo() was executed" << std::endl;
    return 0;
}

auto main() -> int
{
    static_assert(std::is_same_v<decltype(foo()), int>); // decltype(foo()) is same as int.
}
```
Checkout this [link](https://www.danielsieger.com/blog/2022/01/28/cpp-trailing-return-types.html) for more information.

## Making your variable shared by all translation units
Counter.h
```c++
// 'inline' keyword allows multiple identical definitions!
// If we declared it without 'inline', the compiler would complain about ODR violation.
inline int counter = 0;
```
Foo.h
```c++
#include "Counter.h"
inline auto foo() -> int
```
Foo.cpp
```c++
#include "Foo.h"

auto foo() -> int
{
    return ++counter;
}
```
main.cpp
```c++
#include <iostream>
#include "Foo.h"

auto main() -> int
{
    std::cout << foo() << foo() << counter << std::end; // prints 122
}
```
