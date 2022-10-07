# cpp-tips
Collection of small tips and tricks for C++

- [Initializing std::vector with initializer-list always invokes copy constructor](#tip1)
- [const std::string& can also cause allocation](#tip2)
- [Creating a lambda behaves the same as creating a struct with operator() overloaded](#tip3)
- [Hiding variable names using extra scope](#tip4)
- [Trailing return type](#tip5)
- [Making your variable shared by all translation units](#tip6)
- [Declaring and initializing static member variables at the same time (in a header file)](#tip7)

## <a name='tip1'></a>Initializing std::vector with initializer-list always invokes copy constructor
```c++
#include <iostream>
#include <vector>

struct Test
{
    int id;

    Test(int id)
        : id(id)
    {
        std::cout << "default constructor " << id << std::endl;
    }
    Test(const Test& other)
        : id(other.id)
    {
        std::cout << "copy constructor " << id << std::endl;
    }
    Test(Test&& other)
        : id(other.id)
    {
        std::cout << "move constructor " << id << std::endl;
    }
    Test& operator=(const Test& other)
    {
        id = other.id;
        std::cout << "copy assignment " << id << std::endl;
        return *this;
    }
    Test& operator=(Test&& other)
    {
        id = other.id;
        std::cout << "move assignment " << id << std::endl;
        return *this;
    }
    ~Test() = default;
};

int main()
{
    // Two Test instances are created and copied.
    auto v = std::vector<Test>{{1}, {2}};

    std::cout << "======== reserve ========" << std::endl;

    // While the vector is resizing, copy constructors are called.
    v.reserve(100);

    std::cout << "======== push_back ========" << std::endl;

    // Given that v has enough space, push_back also invokes move constructor!
    // One default constructor and one move constructor are called.
    v.push_back({3});

    std::cout << "======== emplace_back ========" << std::endl;
    
    // Default constructor is called.
    // Although push_back(T&&) exists, emplace_back still has advantage on not creating temporary objects.
    // Note that only emplace_back is capable of perfect forwarding!
    v.emplace_back(4);
}
```
Expected output:
```
default constructor 1
default constructor 2
copy constructor 1
copy constructor 2
======== reserve ========
copy constructor 1
copy constructor 2
======== push_back ========
default constructor 3
move constructor 3
======== emplace_back ========
default constructor 4
```
Checkout this [stackoverflow question](https://stackoverflow.com/questions/4303513/push-back-vs-emplace-back) for more information on difference between push_back and emplace_back
## <a name='tip2'></a>const std::string& can also cause allocation
```
void foo(const std::string&) {}
void foo2(std::string_view) {}

int main()
{
    foo("asdf"); // temporary std::string instance is created!
    foo2("asdf"); // no allocation happens.
}
```
## <a name='tip3'></a>Creating a lambda behaves the same as creating a struct with operator() overloaded
```c++
#include <iostream>

struct Lambda
{
    int captureThis = 234; // Captured variable stored as a member variable
    int operator()(int val)
    {
        return val + captureThis;
    }
};

int main()
{
    int captureThis = 234;
    
    int result1 = Lambda()(1000);
    int result2 = [=](int val){ return val + captureThis; }(1000);
    
    std::cout << result1 << " " << result2 << std::endl; // prints 1234 1234
}
```
## <a name='tip4'></a>Hiding variable names using extra scope
```c++
int main()
{
    // If you need, you can nest an extra scope to reuse a variable name.
    // Use this in case you have to create several variables of similar intent
    // such as desc1, desc2, result1, result2, ...
    // But don't forget that avoiding this situation by giving them
    // distinguishable and meaningful names is the best option.
    {
        int result = someComplexFunction();
        // Handle result...
    }
    {
        std::vector<int> result = anotherComplexFunction();
        // Handle result
    }
{
```

## <a name='tip5'></a>Trailing return type
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

## <a name='tip6'></a>Making your variable shared by all translation units
Counter.h
```c++
// 'inline' keyword allows multiple identical definitions!
// If we declared it without 'inline', the compiler would complain about ODR violation.
inline int counter = 0;

// 'static' keyword is a little bit different.
// Since global variables marked as 'static' have internal linkage, compilder doesn't say anything about ODR
// but the value of counter2 could be different for each translation unit.
static int counter2 = 0;
```
Foo.h
```c++
#include "Counter.h"

auto foo() -> int;
auto foo2() -> int;
```
Foo.cpp
```c++
#include "Foo.h"

auto foo() -> int
{
    return ++counter;
}

auto foo2() -> int
{
    return ++counter2;
}
```
main.cpp
```c++
#include <iostream>
#include "Foo.h"

auto main() -> int
{
    std::cout << foo() << foo() << counter << std::end; // prints 122
    std::cout << foo2() << foo2() << counter2 << std::end; // prints 120
}
```
## <a name='tip7'></a>Declaring and initializing static member variables at the same time (in a header file)
IDGenerator.h
```c++
class IDGenerator
{
private:
    // Valid from C++17.
    // Note that 'inline static' and 'static inline' have same effect,
    // but the latter is prefered because 'static' is a storage class specifier
    // and C standard says that such keywords should come first.
    static inline int nextID = 0; 
};
```
Checkout this [stackoverflow question](https://stackoverflow.com/questions/61714110/static-inline-vs-inline-static) for more information on 'inline static' vs 'static inline'.
