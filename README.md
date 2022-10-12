# cpp-tips
Collection of small tips and tricks for C++

- [Initializing std::vector with initializer-list always invokes copy constructor](#tip1)
- [const std::string& and std::string_view can also cause allocation](#tip2)
- [Creating a lambda behaves the same as creating a struct with operator() overloaded](#tip3)
- [Hiding variable names using extra scope](#tip4)
- [Trailing return type](#tip5)
- [Making your variable shared by all translation units](#tip6)
- [Declaring and initializing static member variables at the same time (in a header file)](#tip7)
- [Mimic 'named parameter' for function calls like other languages](#tip8)
- [Fold expressions for variadic template](#tip9)
- [Multi-step (user defined) implicit conversion is not allowed](#tip10)
- [Implicit conversion might cause confusion: consider using 'explicit'](#tip11)
- [Rvalue reference parameter is a lvalue](#tip12)
- [Commas can be used in two ways: seperator and operator](#tip13)

## <a name='tip1'></a>Initializing std::vector with initializer-list always invokes copy constructor
```c++
#include <iostream>
#include <vector>
#include <memory>

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
    
    /* This is thereby impossible because std::unique_ptr doesn't have a copy constructor.
    std::vector<std::unique_ptr<Test>> {
        std::make_unique<Test>(1),
        std::make_unique<Test>(2),
        std::make_unique<Test>(3)
    };
    */
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
Checkout this [stackoverflow question](https://stackoverflow.com/questions/4303513/push-back-vs-emplace-back) for more information on difference between push_back and emplace_back.<br>
On the other hand, this [stackoverflow question](https://stackoverflow.com/questions/9618268/initializing-container-of-unique-ptrs-from-initializer-list-fails-with-gcc-4-7) handles the ```std::vector<std::unique_ptr<T>>``` initialization issue.<br>
## <a name='tip2'></a>const std::string& and std::string_view can also cause allocation
```c++
void foo(const std::string&) {}
void foo2(std::string_view) {}

int main()
{
    // Example 1) use of const std::string& causing an allocation
    {
        foo("asdf"); // temporary std::string instance is created!
        foo2("asdf"); // no allocation happens.
    }
    
    // Example 2) use of std::string_view causing an allocation
    {
        std::string fileContent{"content of a 4GB file"};
        std::string_view contentView{fileContent};

        // Do some stuffs using substrings of fileContent.
        // Note that std::string_view can handle substrings efficiently.

        std::string anotherString(contentView); // Oops! We've just created another 4GB string on our memory
        std::string anotherString2(std::move(fileContent)); // Consider using move semantics if you no longer need the original instance
    }
}
```
Example 2 is quite artificial and probably no one would ever do that intentionally.<br>
So just remember two things:
1. References and views does NOT guarantee that no allocation will happen.<br>
   These might lead to extra allocation or creation of temporary objects when misused.
3. In some cases, std::move might be a better option over references and views.
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

// Return types can be automatically deduced since C++14, so you can omit the -> part.
template<typename A, typename B>
auto multiply2(A a, B b)
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
## <a name='tip8'></a>Mimic 'named parameter' for function calls like other languages
```c++
struct Args
{
    int x;
    int y;
};

auto foo(Args args) -> void
{
    // Do something with args.x and args.y
}

auto main() -> int
{
    foo({.x = 2, .y = -4});
}
```
## <a name='tip9'></a>Fold expressions for variadic template
```c++
#include <iostream>
#include <vector>

using namespace std::string_literals;

template<typename First, typename... Others>
auto sum(First first, Others... others)
{
    // There are four types of fold expression:
    // 1. binary left fold: (init op ... op pack)
    // 2. binary right fold: (pack op ... op init)
    // 3. unary left fold: (... op pack)
    // 4. unary right fold: (pack op ...)
    //
    // left fold groups the leftmost term first, while right fold groups the rightmost one.
    // ex) if parameters are given as E1, E2, E3, E4, and E5,
    // (... + others) turns into (E1 + (E2 + (E3 + (E4 + E5)))), while
    // (others + ...) turns into ((((E1 + E2) + E3) + E4) + E5)
    //
    // The expression below expands to (((first + second) + third) + ...) + last;
    return (first + ... + others);
}

auto main() -> int
{
    // Since sum uses a left fold, std::string + string literal is performed consequently, creating a std::string as a result.
    // If we passed string literal as the first parameter, compiler would complain that (const char* + const char*) is an invalid operation.
    std::cout << sum("Hello, "s, "world", "!") << std::endl; // prints Hello, world!
}
```
Checkout this [cppreference page](https://en.cppreference.com/w/cpp/language/fold) for more information.
## <a name='tip10'></a>Multi-step (user defined) implicit conversion is not allowed
```c++
#include <string>

using namespace std::string_view_literals;

class File
{
public:
    File(std::string_view filename) {}
};

void parseFile(const File& file) {}

int main()
{
    parseFile(File{"file1"}); // OK: const reference allows temporary objects.
    parseFile("file2"sv); // OK: a one-step implicit conversion is allowed.
    parseFile("file3"); // Error: invalid initialization of reference of type const File& from expression of type const char[6]
}
```
Checkout this [stackoverflow question](https://stackoverflow.com/questions/12847272/multiple-implicit-conversions-on-custom-types-not-allowed) for more information on conversion rules.
## <a name='tip11'></a>Implicit conversion might cause confusion: consider using 'explicit'
```c++
#include <vector>
#include <string>

using namespace std::string_view_literals;

class Resource
{
public:
    // Load resource from the file
    /*explicit*/ Resource(std::string_view filename) {}
    
    // Construct with a preloaded data
    /*explicit*/ Resource(int id, std::string&& content) {}
    
    Resource(const Resource& other) = default;
    Resource(Resource&& other) noexcept = default;
    Resource& operator=(const Resource& other) = default;
    Resource& operator=(Resource&& other) noexcept = default;
    ~Resource() = default;
};

class ResourceManager
{
public:
    void addResource(Resource&& resource)
    {
        resources.emplace_back(std::move(resource));
    }
private:
    std::vector<Resource> resources;
};

int main()
{
    auto rm = ResourceManager{};

    // Works as expected.
    auto r = Resource{"file1"};
    rm.addResource(std::move(r)); // case 1) std::move a lvalue
    rm.addResource(Resource{"file2"}); // case 2) temporary object is a rvalue

    // Implicit conversion happens in these cases (not sure if it's intended).
    // You might wonder 'why is this accepting a string?' or 'why are we trying to pass a string?'
    //
    // Given a constructor with single parameter, implicit conversions can happen without any sign (see case 4).
    // If you don't want such behavior, mark the constructor as 'explicit' and these two lines won't compile.
    // 
    // Note) passing a string literal on case 4 would cause a multi-step (user defined) implicit conversion, which won't even compile.
    // To be specific, string literal -> std::string_view -> Resource is required for an implicit Resource construction.
    rm.addResource({"file3"}); // case 3) an implicit convserion using initializer list (somewhat noticeable due to curly braces)
    rm.addResource("file4"sv); // case 4) an implicit conversion quite hard to spot
    
    // Constructors with two or more parameters usually don't cause trouble.
    // Since you can't make implicit conversion happen without using an initializer list (i.e. case 4 cannot happen with multi-parameter constructors)
    rm.addResource({5, "Hello, world!"}); // case 5) an implicit conversion using initializer list
    rm.addResource(6, "you can't do this"); // case 6) Error: no matching function for call to ResourceManager::addResource(int, const char [18])
}
```
Checkout this [stackoverflow question](https://stackoverflow.com/questions/12437241/c-always-use-explicit-constructor) for more opinions on when to use explicit constructors.
## <a name='tip12'></a>Rvalue reference parameter is a lvalue
```c++
#include <iostream>

void foo(const std::string&)
{
    std::cout << "lvalue ";
}

void foo(std::string&&)
{
    std::cout << "rvalue ";
}

void test(std::string&& s)
{
    foo(s); // foo(const std::string&)
    foo(std::move(s)); // foo(std::string&&)
}

int main()
{
    // prints 'lvalue rvalue'
    test("Use std::move if your constructor should 'move' rvalue reference parameters");
}
```
## <a name='tip13'></a>Commas can be used in two ways: seperator and operator
```c++
#include <iostream>

void foo(double d)
{
    std::cout << d << std::endl;
}

void foo(int i, double d)
{
    std::cout << i << " " << d << std::endl;
}

int main()
{
    int i = 1;
    double j = 2.2;
    
    // We usually use commas as seperators because
    // not only do we intend that but also comma operators have one of the lowest priorities.
    foo(i++, j += 1.1); // prints 3.3
    
    // But if we just put enough parenthesis to give a chance for a comma to be interpreted as an operator,
    // the whole expression is turned into a sequential evaluation where the rightmost term is returned.
    // The example below is idential to this:
    //   i++;
    //   j+=1.1;
    //   std::cout << j << std::endl;
    foo((i++, j += 1.1)); // prints 4.4
    
    // Note that the types of each expression doesn't have to be identical unlike ternary operator.
    std::cout << (i++, j += 1.1) << std::endl; // prints 5.5
}
```
