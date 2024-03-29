# cpp-tips
Collection of small tips and tricks for C++<br>
* [한국어 버전](READMEkr.md)

## Introduction
As a C++ user, I had a hard time learning the rules and best practices of this awesome language.<br>
There are so many things that we can do with it, but it doesn't come without costs: the notorious complexity and steep learning curve.

I also had to struggle with many compiler errors which don't always point to the problem directly,<br>
and it took me so much time to figure out the details which were the core reasons for my error.<br>
Although there are numerous tutorials suited for each level of proficiency,<br>
I've discovered that the situation and requirements we face are simply uncountable that even these nice guides can't cover them all.<br>

These kinds of practical information or know-hows we earn by countless trial and error are hardly known to C++ beginners.<br>
It also takes quite a long time to do so, therefore I couldn't just recommend others to learn it through experiences just like I did.<br>

That's exactly the reason why I chose to make this repository:<br>
to share my experience with others so that they can solve different problems<br>
they might face in the future with minimal effort and advance their skills.<br>

I still highly appreciate learning through trial and error.<br>
Making toy projects and surfing Stackoverflow to solve each problem you face are precious experiences<br>
that will let you stand on your own and face challenges you can overcome with some effort.<br>

By the way, I highly recommend reading [c++ language reference](https://learn.microsoft.com/en-us/cpp/cpp/cpp-language-reference?view=msvc-170) created by Microsoft!<br>
Beginners might not get much out of it but I believe intermediate users would find out<br>
the features they've never knew or concepts which were misunderstood, while reading this document from A to Z.

## Not C++ specific but useful documents
- [How should I reuse codes if some of the concrete classes don't share the same behavior?](https://softwareengineering.stackexchange.com/questions/246273/code-re-use-in-c-via-multiple-inheritance-or-composition-or)
- [How should I order the members of a class?](https://stackoverflow.com/questions/308581/how-should-i-order-the-members-of-a-c-class)

## List of Contents
### Language Features
- [Trailing return type](#tip5)
- [Structured binding](#tip20)
- [Virtual destructor](#tip27)
- [Fold expressions for variadic template](#tip9)
- [Designated initializer (ft. named parameter)](#tip8)
- [Commas can be used in two ways: separator and operator](#tip13)
- [Dynamic and static cast for smart pointers](#tip15)
- [Three ways of overloading binary operators](#tip25)
- [The meaning of 'qualified name' and 'unqualified access'](#tip18)
- [Name mangling and extern "C"](#tip35)
- [Using 'auto' as parameter type](#tip36)
- [```std::declval<T>()```](#tip37)
- [Member function/variable pointer](#tip39)
- [Making return values more noticeable with ```[[nodiscard]]```](#tip42)
### Initialization and Construction
- [Initializing std::vector with initializer-list always invokes copy constructor](#tip1)
- [const std::string& and std::string_view can also cause allocation](#tip2)
- [Declaring and initializing static member variables at the same time (in a header file)](#tip7)
- [How to initialize a reference member (ft. member initializer list)](#tip21)
### Function Parameters and Perfect Forwarding
- [Rvalue reference parameter is an lvalue](#tip12)
- [Template argument deduction (ft. std::forward and universal reference)](#tip23)
- [Why do we need std::forward in addition to universal reference? (ft. perfect forwarding)](#tip31)
- [Perfect forwarding in a lambda](#tip24)
### Linkage and Scope
- [Hiding variable names using extra scope](#tip4)
- [Making your variable shared by all translation units](#tip6)
- [External linkage vs internal linkage (with examples)](#tip19)
### Tricky Behaviors
- [Multi-step (user-defined) implicit conversion is not allowed](#tip10)
- [Implicit conversion might cause confusion: consider using 'explicit'](#tip11)
- [The reason why using static_cast for downcast is unsafe](#tip16)
- [Mutability of captured variables in a lambda](#tip28)
- [std::vector<bool> doesn't store booleans](#tip30)
- [A malicious bypass: passing int as an enum class parameter](#tip41)
### Concurrency
- [Manually locking and unlocking a mutex can be dangerous](#tip29)
- [How to wait for a signal or create a synchronization point for multiple threads](#tip34)
- [Printing to std::cout without race condition](#tip43)
### Other Tips
- [Using nested symbol of a template type as a type name](#tip22)
- [Declaring variables inside a switch statement](#tip17)
- [const keyword applies to the left token, unless it comes at the start](#tip14)
- [Polymorphism without runtime overhead (ft. CRTP)](#tip26)
- [Creating a lambda behaves the same as creating a struct with operator() overloaded](#tip3)
- [A simple yes/no guideline for deciding member variable type](#tip32)
- [Make sure that ```set(CMAKE_CXX_STANDARD ??)``` comes after ```project()```](#tip33)
- [How to make a basic command-line argument parser with ranges library](#tip38)
- [Do not use 'using namespace' in headers](#tip40)

## Language Features
### <a name='tip5'></a>Trailing return type
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
Check out this [link](https://www.danielsieger.com/blog/2022/01/28/cpp-trailing-return-types.html) for more information.

### <a name='tip20'></a>Structured binding
```c++
#include <iostream>
#include <string>
#include <tuple>
#include <map>

using namespace std::string_literals;

struct CustomType
{
    int i;
    double d;
};

// Warning: this example is solely dedicated to demonstrating the structured binding feature.
// Try to use a struct instead of a tuple if returning multiple values is needed.
// Tuple elements do not have a name, which makes the values hard to understand without looking at the description.
// Structs, on the other hand, have named member variables which are quite self-explanatory.
auto getStudentInfo()
{
    auto age = 17;
    auto height = 175.3;
    auto name = "Jack"s;

    return std::tuple{age, height, name};
}

int main()
{
    // Structured binding enhances readability by giving names
    // to elements of aggregate objects like key-value pair.
    {
        auto m = std::map<int, double>{{1, 1.11}, {2, 2.22}};

        // Double the values of m.
        std::cout << "Basic for loop using iterator:" << std::endl;
        for (auto it = m.begin(); it != m.end(); ++it)
        {
            std::cout << "changing m[" << it->first << "] to " << it->second * 2 << std::endl;
            it->second *= 2;
        }

        // Do the same thing using structured binding.
        std::cout << "Range-based for loop using structured binding:" << std::endl;
        for (auto& [key, value] : m)
        {
            std::cout << "changing m[" << key << "] to " << value * 2 << std::endl;
            value *= 2;
        }
    }

    // Structured binding can be used to give names to tuple elements
    {
        auto [age, height, name] = getStudentInfo();
        std::cout << "name: " << name << ", age: " << age << ", height: " << height << std::endl;
    }

    // It also works on custom types!
    {
        auto [numItems, totalWeight] = CustomType{33, 99.0};
        std::cout << "number of items: " << numItems << ", avg weight: " << totalWeight / numItems << std::endl;
    }
}
```
Expected output:
```
Basic for loop using iterator:
changing m[1] to 2.22
changing m[2] to 4.44
Range-based for loop using structured binding:
changing m[1] to 4.44
changing m[2] to 8.88
name: Jack, age: 17, height: 175.3
number of items: 33, avg weight: 3
```

### <a name='tip27'></a>Virtual destructor
```c++
#include <iostream>
#include <memory>

class Base
{
public:
    // Making parent class have virtual destructor lets the compiler
    // check if the instance that Base* is pointing at is a derived class
    // such that it requires additional destructor calls such as ~Derived()
    /*virtual*/ ~Base()
    {
        std::cout << "~Base" << std::endl;
    }
};

class Derived : public Base
{
public:
    ~Derived()
    {
        std::cout << "~Derived" << std::endl;
    }
};

int main()
{
    // OK: base and derived are both released
    Derived* d = new Derived();
    delete d; // ~Derived ~Base

    // Bad: only the base region is released (memory leak can happen!)
    Base* b = new Derived();
    delete b; // ~Base

    // Ok: smart pointers store the destructor they should call, so it works without virtual destructors
    std::shared_ptr<Base> sb = std::make_shared<Derived>();
    sb.reset(); // ~Derived ~Base
}
```
There's a short [article](https://blog.the-pans.com/why-you-dont-need-virtual-destructor-with-smart-pointers/) about smart pointers working well without a virtual destructor

### <a name='tip9'></a>Fold expressions for variadic template
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
    // (... + others) turns into ((((E1 + E2) + E3) + E4) + E5), while
    // (others + ...) turns into (E1 + (E2 + (E3 + (E4 + E5))))
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
Check out this [cppreference page](https://en.cppreference.com/w/cpp/language/fold) for more information.

### <a name='tip8'></a>Designated initializer (ft. named parameter)
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
    // From c++20, you can initialize structs with member names.
    // By packing function arguments into one struct,
    // you can mimic the 'named parameter' feature of other languages like Python!
    foo({.x = 2, .y = -4});
}
```

### <a name='tip13'></a>Commas can be used in two ways: separator and operator
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
    
    // We usually use commas as separators because
    // not only do we intend that but also comma operators have one of the lowest priorities.
    foo(i++, j += 1.1); // prints 1 3.3
    
    // But if we just put enough parenthesis to give a chance for a comma to be interpreted as an operator,
    // the whole expression is turned into a sequential evaluation where the rightmost term is returned.
    // The example below is identical to this:
    //   i++;
    //   j+=1.1;
    //   std::cout << j << std::endl;
    foo((i++, j += 1.1)); // prints 4.4
    
    // Note that the types of each expression don't have to be identical, unlike ternary operator.
    std::cout << (i++, j += 1.1) << std::endl; // prints 5.5
}
```

### <a name='tip15'></a>Dynamic and static cast for smart pointers
```c++
#include <iostream>
#include <memory>

// Base1 and Derived1 are polymorphic.
// Use std::dynamic_pointer_cast for such classes.
class Base1
{
public:
    virtual ~Base1() = default; // We need at least one virtual function for dynamic_cast to work
};
class Derived1 : public Base1 {};

// Base2 and Derived2 are not polymorphic.
// Use std::static_pointer_cast for such classes.
class Base2 {};
class Derived2 : public Base2 {};

int main()
{
    // Case 1) polymorphic class
    {
        std::shared_ptr<Base1> pBase = std::make_shared<Derived1>(); // Ok: usual upcasting
        
        // Cast to a relevant type
        std::dynamic_pointer_cast<Derived1>(pBase); // Ok: pBase is pointing at a complete object of Derived1
        std::static_pointer_cast<Derived1>(pBase); // Uhhh: this works because we know pBase poitns to a Derived1 instance

        // Cast to an irrelevant type
        std::dynamic_pointer_cast<Derived2>(pBase); // Bad: the cast fails at runtime and nullptr is returned
        std::static_pointer_cast<Derived2>(pBase); // Error: invalid 'static_cast' from type Base1* to type Derived2*

        // Wait, do we even need a downcast like dynamic_cast while we use polymorphism?
        // In principle, your code should not depend on concrete class (i.e. derived class) but interface (i.e. base class).
        // That means using dynamic_cast or std::dynamic_pointer_cast is also considered as a code smell!
        // If you ever find yourself in such situation, try to redesign your class hierachy so that everything can be done without downcasting.
    }

    // Case 2) nonpolymorphic class
    {
        std::shared_ptr<Base2> pBase = std::make_shared<Derived2>(); // Ok: usual upcasting

        // Cast to a relevant type
        std::dynamic_pointer_cast<Derived2>(pBase); // Error: source type is not polymorphic
        std::static_pointer_cast<Derived2>(pBase); // Ok: pBase is pointing at a complete object of Derived2

        // Cast to an irrelevant type
        std::dynamic_pointer_cast<Derived1>(pBase); // Error: source type is not polymorphic
        std::static_pointer_cast<Derived1>(pBase); // Error: invalid 'static_cast' from type Base2* to type Derived1*
    }
}
```

### <a name='tip25'></a>Three ways of overloading binary operators
```c++
struct Int
{
    int val;

    // Case 1) member function
    constexpr Int operator+(const Int& other) const
    {
        return { val + other.val };
    }

    // Case 2) global function with access to private members
    friend constexpr Int operator-(const Int& lhs, const Int& rhs)
    {
        return { lhs.val - rhs.val };
    }
};

// Case 3) global function with access to public members only
constexpr bool operator==(const Int& lhs, const Int& rhs)
{
    return lhs.val == rhs.val;
}

int main()
{
    static_assert(Int{2} + Int{3} == Int{5});
    static_assert(Int{2} - Int{3} == Int{-1});
}
```

### <a name='tip18'></a>The meaning of 'qualified name' and 'unqualified access'
```c++
// Qualified name is a full name that includes an identifier's namespace or class name.
// Qualified access means we specify the identifier by its full name (e.g. std::vector).
// Unqualified access, on the other hand, means that we omit some parts of the qualified name while we specify an identifier.
// This is possible in some places like nested namespace or a region after using namespace ~ statement is used.
namespace Toolbox
{
    // The qualified name (i.e. the full name) for this function is Toolbox::three.
    int three() { return 3; }

    namespace Math
    {
        // The qualified name for this function is Toolbox::Math::triple.
        // Since Math is a nested namespace, everything declared in the parent namespace 'Toolbox'
        // can be accessed without qualifiers (i.e. unqualified access).
        // This means that instead of Toolbox::three(), we can simply write three().
        int triple(int val) { return val * three(); }
    }
};
```

### <a name='tip35'></a>Name mangling and extern "C"
```c++
// C++ allows function overloading.
// This means that there could be multiple functions with the same name but different parameters.
// To let the compiler distinguish these symbols, C++ gives each function a new unique name.
//
// For example, appending the parameter type as a prefix should suffice to distinguish the two foo() below.
// ex) foo(int) -> foo_i, foo(double) -> foo_d
void foo(int);
void foo(double);

// By specifying extern "C", you can turn off name mangling.
// This will make function overloading impossible, but the function will be usable in C projects.
extern "C" void goo();

// You can also use a scope to avoid writing extern "C" one by one.
extern "C"
{

void haha();
void hoho();

}
```

### <a name='tip36'></a>Using 'auto' as parameter type
```c++
// Valid from C++20
void foo(auto arg)
{
    std::cout << arg;
}

// This allows us to use concepts with minimal effort
// Note: std::integral is a standard concept defined in header <concepts>
auto square(std::integral auto value)
{
    return value * value;
}

// Traditional template usage and 'auto' can coexist
template<typename T>
void goo(T first, auto... others)
{
    std::cout << first;
}

// You can also use variadic template and perfect forwarding
void print(auto&&... args)
{
    // A fold expression using the comma operator that expands to
    // (((std::cout << arg1 << " "), (std::cout << arg2 << " ")), ...)
    ((std::cout << std::forward<decltype(args)>(args) << " "), ...);
}

int main()
{
    print("haha", 1234, 5.6); // Prints "haha 1234 5.6 "
}
```


### <a name='tip37'></a>```std::declval<T>()```
```c++
#include <iostream>
#include <concepts>

// Suppose that we want to make a concept that checks
// the existence of member function "Ret T::operator(Arg arg)".
//
// The concept proposed below does work
// when T and Arg both have default constructors.
// But what if they don't...?
//
// Although decltype() expression doesn't get evaluated,
// invalid expressions will cause substitution failure!
template<typename T, typename Ret, typename Arg>
concept Function = std::is_same_v<decltype(T{}(Arg{})), Ret>;

// std::declval<T>() comes in handy in such cases.
// It returns an lvalue reference type without calling a constructor,
// allowing us to assume that an instance of T exists!
template<typename T, typename Ret, typename Arg>
concept Function2 = std::is_same_v<decltype(std::declval<T>()(std::declval<Arg>())), Ret>;

// We can also use the 'requires clause' to implement the same feature.
// To make things more interesting,
// let's make a concept for variadic functions.
template<typename T, typename Ret, typename... Arg>
concept Function3 = requires(T func) {
    { func(std::declval<Arg>()...) } -> std::same_as<Ret>;
};

// A test function that only accepts instances with
// member function "void operator()(std::string_view)"
template<typename T>
    requires Function3<T, void, std::string_view>
void test(T func) {
    func("wow this works!");
}

// A test functor that doesn't have a default constructor
class PrintMultipleTimes
{
public:
    PrintMultipleTimes(int repeat) : repeat(repeat) {}

    void operator()(std::string_view message)
    {
        for (int i = 0; i < repeat; ++i)
        {
            std::cout << message << std::endl;
        }
    }

private:
    int repeat;
};

int main()
{
    // Works with lambda
    test([](std::string_view s) {
        std::cout << s << std::endl;
    });

    // Also works with objects that cannot be default-constructed
    test(PrintMultipleTimes(3));
}
```

### <a name='tip39'></a>Member function/variable pointer
```c++
struct Test
{
    int mem_var;

    int mem_func(double, bool) {}
};

int main()
{
    auto test = Test{};

    // How to understand the type step by step:
    // 1. *mem_func_ptr ==> mem_func_ptr is a pointer
    // 2. Test::*mem_func_ptr ==> ... to a member of Test
    // 3. (Test::*mem_func_ptr)(double, bool) ==> ... which is a function with double and bool arguments
    // 4. int (Test::*mem_func_ptr)(double, bool) ==> ... that returns int
    int (Test::*mem_func_ptr)(double, bool) = &Test::mem_func;

    // Similar to the example above, the type of mem_var_ptr implies that
    // mem_var_ptr is a pointer to a member of Test which is an integer.
    int Test::*mem_var_ptr = &Test::mem_var;

    // Using the pointers to access member function/variable.
    (test.*mem_func_ptr)(1.0, true); // test.mem_func(1.0, true);
    test.*mem_var_ptr = 4567; // test.mem_var = 4567;
}
```

### <a name='tip42'></a>Making return values more noticeable with ```[[nodiscard]]```
```c++
// [[nodiscard]] is an attribute that tells the compiler
// to warn you whenever a return value is not used.
//
// This could be useful when ignoring the return value makes
// the function call useless or cause problems like memory leak.
[[nodiscard]] bool is_empty();

void foo()
{
    is_empty(); // Warning: return value ignored
}
```
- [stackoverflow - why clang tidy suggests to add nodiscard everywhere](https://stackoverflow.com/questions/67059884/why-clang-tidy-suggests-to-add-nodiscard-everywhere)
<details>
    <summary>Example scenario where ignoring a return value leads to a dangling pointer problem</summary>

```c++
#include <functional>
#include <memory>

template<typename T>
class Event
{
public:
    // Add an event handler and return its ID.
    // The ID should be used to unregister the callback
    // when it no longer needs to subscribe.
    /*[[nodiscard]]*/ int register_callback(std::function<void(T)>&& callback)
    {
        /* ... */
        return callback_id;
    }

    // Remove an event handler with specified callback ID.
    void unregister_callback(int callback_id)
    {
        /* ... */
    }

    // Invoke registered callbacks.
    void signal(T arg)
    {
        /* ... */
    }
};

class Player
{
public:
    Player(Event<int>& on_enemy_attack)
    {
        // We forgot to use the return value!
        // Things will get messy when the event outlives this instance,
        // but currently there is no way we can notice the danger
        // before we actually run into a bug.
        on_enemy_attack.register_callback([this](int damage){
            hp -= damage;
        });
    }

private:
    int hp = 100;
};

// Suppose that we are making an action game.
int main()
{
    // This will be invoked whenever an enemy tries to attack the player.
    auto on_enemy_attack = Event<int>();

    // Player gets spawned on the map and start fighting.
    auto player = std::make_shared<Player>(on_enemy_attack);

    // During combat, the player dies and the object gets destroyed.
    player.reset();

    // Right after the player's death,
    // a delayed attack from an enemy triggers on_enemy_attack.
    // Since the callback registered by player still remains,
    // now we face a dangling pointer/reference problem.
    on_enemy_attack.signal(10);
}
```
</details>

## Initialization and Construction
### <a name='tip1'></a>Initializing std::vector with initializer-list always invokes copy constructor
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
Check out this [stackoverflow question](https://stackoverflow.com/questions/4303513/push-back-vs-emplace-back) for more information on difference between push_back and emplace_back.<br>
On the other hand, this [stackoverflow question](https://stackoverflow.com/questions/9618268/initializing-container-of-unique-ptrs-from-initializer-list-fails-with-gcc-4-7) handles the ```std::vector<std::unique_ptr<T>>``` initialization issue.<br>

### <a name='tip2'></a>const std::string& and std::string_view can also cause allocation
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
2. In some cases, std::move might be a better option over references and views.

### <a name='tip7'></a>Declaring and initializing static member variables at the same time (in a header file)
IDGenerator.h
```c++
class IDGenerator
{
private:
    // Valid from C++17.
    //
    // Specifying 'inline' can be thought as telling the compiler that
    // we are going to violate ODR but in a gentle way (same definition everywhere),
    // so you should pick one and use it as if it were declared only once.
    //
    // Though 'inline' keyword doesn't always make things have external linkage,
    // it is usually involved with symbols that are external (e.g. static inline member variables).
    // I think that's because symbols with internal linkage have no problem dealing with ODR.
    //
    // Note that 'inline static' and 'static inline' have the same effect,
    // but the latter is preferred because 'static' is a storage class specifier
    // and C standard says that such keywords should come first.
    //
    // Static member variables have external linkage if a class has external linkage.
    // Classes are external by default, but anonymous namespace can make things internal.
    // That means static data members can have internal linkage
    // if a class is declared inside an anonymous namespace (i.e. has internal linkage).
    static inline int nextID = 0;
};

// An anonymous namespace
namespace
{
    class Test // internal linkage
    {
    public:
        static inline int test = 1234; // internal linkage
    };
}
```
Check out this [stackoverflow question](https://stackoverflow.com/questions/61714110/static-inline-vs-inline-static) for more information on 'inline static' vs 'static inline'.<br>
This [stackoverflow question](https://stackoverflow.com/questions/16386256/inline-functions-and-external-linkage), on the other hand, will help you distinguish  'inline' and 'external linkage'.

### <a name='tip21'></a>How to initialize a reference member (ft. member initializer list)
```c++
struct Device {};

struct Renderer
{
    // Note: using reference type for member variables has several downsides.
    // 1. We can't perform assignment without copying the referenced object.
    //    - 'ref = otherRef' does NOT change what 'ref' points to (it invokes copy assignment)
    // 2. We can't use a default constructor.
    //    - 'int& i;' is invalid (alias to nothing doesn't make sense)
    //
    // Since this topic is too heavy for a single comment block,
    // I'll leave a link below for those who want further information.
    Device& device;

    // Member variables are initialized before we reach the first line of a constructor.
    // This means that '=' inside a constructor implies an assignment.
    //
    // Since default initialization is not available for reference type,
    // we need another place to perform initialization prior to the function body.
    //
    // That's where 'member initializer list' comes into play.
    // Things written here are executed before the constructor body.
    Renderer(Device& device)
        : device(device) // This section is the member initializer list.
    {}
    
    /*
    Renderer(Device& device) // Error: uninitialized reference member ('Device& Renderer::device' should be initialized)
    {
        // The line below isn't an initialization; it copies 'device' into 'this->device'!
        // Considering that reference types are used to avoid copying, this behavior is hardly intended.
        this->device = device; 
    }
    */
};

int main()
{
    Device d;
    Renderer r(d);
}
```
['References, simply' by Herb Sutter](https://herbsutter.com/2020/02/23/references-simply/) for deeper analysis and guidelines about reference type.

## Function Parameters and Perfect Forwarding
### <a name='tip12'></a>Rvalue reference parameter is an lvalue
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

### <a name='tip23'></a>Template argument deduction (ft. std::forward and universal reference)
```c++
// Still cleaning up the mess...

#include <type_traits>

template<typename T>
void test(T param) {}

template<typename T>
void testR(T& param) {}

template<typename T>
void testCR(const T& param) {}

template<typename T>
void testUR(T&& param) {}

// Add & at the type
template<typename T>
using R = T&;

// Add && at the type
template<typename T>
using RR = T&&;

// Get the return type of std::forward on instance of type T
template<typename T>
using forwardType = decltype(std::forward<T>(std::declval<T>()));

// Get the return type of std::move on instance of type T
template<typename T>
using moveType = decltype(std::move<T>(std::declval<T>()));

int main()
{
    // Reference collapsing
    {
        static_assert(std::is_same_v<R<int&>, int&>);    // T& &   -> T&
        static_assert(std::is_same_v<R<int&&>, int&>);   // T&& &  -> T&
        static_assert(std::is_same_v<RR<int&>, int&>);   // T& &&  -> T&
        static_assert(std::is_same_v<RR<int&&>, int&&>); // T&& && -> T&&
    }

    // std::forward performs static_cast<T&&>, using the last two reference collapsing rules.
    {
        static_assert(std::is_same_v<forwardType<int&>, int&>);
        static_assert(std::is_same_v<forwardType<int&&>, int&&>);
        static_assert(std::is_same_v<forwardType<const int&>, const int&>);
        static_assert(std::is_same_v<forwardType<const int&&>, const int&&>);
    }

    // std::move makes everything && without touching const qualifier
    {
        static_assert(std::is_same_v<moveType<int&>, int&&>);
        static_assert(std::is_same_v<moveType<int&&>, int&&>);
        static_assert(std::is_same_v<moveType<const int&>, const int&&>);
        static_assert(std::is_same_v<moveType<const int&&>, const int&&>);
    }

    // Let's see how template functions declared with parameter type as T, T&, const T&, and T&&
    // deduce the type of parameter i when variables with various const reference qualifiers are passed.
    int i = 1;
    const int ci = 1;
    int& ri = i;
    const int& cri = ci;

    // test(T param): strips all consts and references
    {
        test(1); // int
        test(i); // int
        test(ri); // int
        test(ci); // int
        test(cri); // int
        test(std::move(i)); // int
        test(std::move(ri)); // int
        test(std::move(ci)); // int
        test(std::move(cri)); // int
    }

    // testR(T& param): accepts everything as reference without changing const qualifier, but rejects rvalue
    {
        //testR(1); // Error: passing rvalue of type int as lvalue reference of type int&
        testR(i); // int&
        testR(ri); // int&
        testR(ci); // const int&
        testR(cri); // const int&
        //testR(std::move(i)); // Error: passing rvalue of type int as lvalue reference of type int&
        //testR(std::move(ri)); // Error: passing rvalue of type int as lvalue reference of type int&
        testR(std::move(ci)); // const int&
        testR(std::move(cri)); // const int&
    }

    // testCR(const T& param): makes everything const reference
    {
        testCR(1); // const int&
        testCR(i); // const int&
        testCR(ri); // const int&
        testCR(ci); // const int&
        testCR(cri); // const int&
        testCR(std::move(i)); // const int&
        testCR(std::move(ri)); // const int&
        testCR(std::move(ci)); // const int&
        testCR(std::move(cri)); // const int&
    }
    
    // testUR(T&& param): what we pass is what we get! (T&& is called the 'universal reference')
    {
        testUR(1); // int&&
        testUR(i); // int&
        testUR(ri); // int&
        testUR(ci); // const int&
        testUR(cri); // const int&
        testUR(std::move(i)); // int&&
        testUR(std::move(ri)); // int&&
        testUR(std::move(ci)); // const int&&
        testUR(std::move(cri)); // const int&&
    }
}
```

### <a name='tip31'></a>Why do we need std::forward in addition to universal reference? (ft. perfect forwarding)
```c++
#include <utility>

struct Test{};

constexpr int foo(Test&)
{
    return 0;
}

constexpr int foo(Test&&)
{
    return 1;
}

// A template function using universal reference only.
template<typename T>
constexpr int goo1(T&& arg)
{
    return foo(arg);
}

// A template function that also uses std::forward.
template<typename T>
constexpr int goo2(T&& arg)
{
    return foo(std::forward<T>(arg));
}

int main()
{
    Test t;

    // Since universal reference should give T the exact type we pass,
    // it seems like goo1 should also be able to distinguish lvalue and rvalue parameters.
    // However, goo1 always calls foo(Test&) because the parameter 'arg' itself is an lvalue!
    static_assert(goo1(Test{}) == goo1(t));

    // On the other hand, goo2 succeeds calling foo(Test&&) for an rvalue parameter.
    // That's because std::forward behaves like std::move() on rvalue parameters while having no effect on lvalue parameters due to reference collapsing.
    // Note: given that T is Test&& in this case, std::forward<T> performs static_cast<Test&& &&> which becomes Test&&.
    //
    // Forwarding refers to the act of passing arguments to other functions just as we did with foo and goo.
    // Since universal reference and std::forward does it perfectly,
    // we call this kind of implementation 'perfect forwarding'.
    static_assert(goo2(Test{}) != goo2(t));
}
```

### <a name='tip24'></a>Perfect forwarding in a lambda
```c++
#include <string>

using namespace std::string_literals;

constexpr int LValRef = 1;
constexpr int RValRef = 2;

constexpr int foo(const std::string&)
{
    return LValRef;
}

constexpr int foo(std::string&&)
{
    return RValRef;
}

int main()
{
    // Method 1) generic lambda (c++14)
    auto lambda1 = [](auto&& str) {
        // return foo(std::forward(str)); // Error: template argument deduction/substitution failed
        return foo(std::forward<decltype(str)>(str));
    };

    // Method 2) template lambda (c++20)
    auto lambda2 = []<typename T>(T&& str) {
        // return foo(std::forward(str)); // Error: template argument deduction/substitution failed
        return foo(std::forward<T>(str));
    };

    auto str = "Hello, world!"s;
    static_assert(lambda1(str) == LValRef);
    static_assert(lambda2(str) == LValRef);
    static_assert(lambda1(std::move(str)) == RValRef);
    static_assert(lambda2(std::move(str)) == RValRef);
}
```
Although generic and template lambda serve similar purpose, template lambda was introduced for [several reasons](https://stackoverflow.com/questions/54126204/what-is-the-need-of-template-lambda-introduced-in-c20-when-c14-already-has-g)

## Linkage and Scope
### <a name='tip4'></a>Hiding variable names using extra scope
```c++
int main()
{
    // If you need, you can nest an extra scope to reuse a variable name.
    // Use this in case you have to create several variables of similar intent
    // such as desc1, desc2, result1, result2, ...
    // But don't forget that avoiding this situation by giving them
    // distinguishable and meaningful names are the best option.
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

### <a name='tip6'></a>Making your variable shared by all translation units
Counter.h
```c++
// 'inline' keyword allows multiple identical definitions!
// If we declared it without 'inline', the compiler would complain about ODR violation.
inline int counter = 0;

// 'static' keyword is a little bit different.
// Since global variables marked as 'static' have internal linkage, compiler doesn't say anything about ODR
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

### <a name='tip19'></a>External linkage vs internal linkage (with examples)
Test.h
```c++
// External linkage:
//     A symbol declared in one translation unit is visible to other translation units.
//     An external variable has the same value in every translation unit.
//     Think of it as having one instance per program.
//     Symbols with external linkage should obey ODR (One Definition Rule).
//
// Internal linkage:
//     A symbol is visible only to the translation unit where it was declared.
//     An internal variable might have different values in different translation units.
//     Think of it as having one instance per translation unit.


// Global variables are external by default.
// ODR violation if multiple translation units include this header file using #include.
// If you ever have to, just declare the variable with 'extern' keyword
// and define it once in a single translation unit (without 'extern' keyword).
/*extern*/ int case1 = 1234; // external linkage

// 'static' keyword makes variables on global scope internal.
// Declaring a variable inside an anonymous namespace does the same job.
static int case2 = 1234; // internal linkage

// static global variables already have internal linkage, so 'inline' doesn't have any effect here.
static inline int case3 = 1234; // internal linkage

// Global functions are external by default.
// ODR violation if multiple translation units include this header file using #include.
// Do not define the body of a global function inside a header file.
// If you ever need to, consider using static function or moving it into an anonymouse namespace.
int case4() // external linkage
{
    return 1234;
}

// Again, 'static' keyword makes it have internal linkage, just like the global variable did.
static int case5()
{
    return 1234;
}

// Classes are external unless declared inside an anonymous namespace.
// Though they have external linkage, defining classes inside a header file doesn't seem to violate ODR.
class External // external linkage
{
public:
    // Member functions have external linkage by default.
    int case6(); // external linkage
    
    // Member functions defined inside a class definition (i.e. inline member functions) are inline by default.
    // This makes definition of a function body in a header file possible.
    /*inline*/ int case7() // external linkage
    {
        return 1234;
    }

    // Static member variables of a class with external linkage is also external.
    static inline int case8 = 1234; // external linkage

private:
    // It doesn't make much sense to talk about linkage of non-static member variables.
    // They are discussed on instance level, not translation unit level.
    //
    // Think of these two declarations in a header file:
    //     extern External e1;
    //     static External e2;
    // Does case9 have internal linkage or external linkage?
    // Well, e1 is external and e2 is internal, so neither can be the answer!
    int case9 = 1234;
};

// ODR violation if multiple translation units include this header file using #include.
// Unlike External::case7, External::case6 is not an inline member function.
// These kinds of definitions should come at the corresponding implementation file (e.g. External.cpp).
int External::case6()
{
    return 1234;
}

// Anonymous namespace makes symbols declared inside have internal linkage.
namespace
{
    // Same as writing 'static int case10 = 1234' on a global scope.
    int case10 = 1234; // internal linkage
    
    // Same as writing 'static int case11() { ... }' on a global scope.
    int case11() // internal linkage
    {
        return 1234;
    }

    // Note: 'inline' keyword cannot be applied to a class definition (i.e. 'inline class' is not valid)
    class Internal // internal linkage
    {
    public:
        // Static member variables of a class with internal linkage are also internal.
        static inline int case12 = 1234; // internal linkage
    };
}
```
Linkage of member variables is discussed in this [stackoverflow question](https://stackoverflow.com/questions/46103512/do-member-variables-have-external-linkage).<br>
The 'static data member' section of this [cppreference page](https://en.cppreference.com/w/cpp/language/static) explains the linkage of static member variables.<br>
This [stackoverflow question](https://stackoverflow.com/questions/154469/unnamed-anonymous-namespaces-vs-static-functions) handles discussion about static function vs functions inside anonymous namespace.

## Tricky Behaviors
### <a name='tip10'></a>Multi-step (user-defined) implicit conversion is not allowed
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
Check out this [stackoverflow question](https://stackoverflow.com/questions/12847272/multiple-implicit-conversions-on-custom-types-not-allowed) for more information on conversion rules.

### <a name='tip11'></a>Implicit conversion might cause confusion: consider using 'explicit'
```c++
#include <vector>
#include <string>

using namespace std::string_view_literals;

class Resource
{
public:
    // Load resource from the file
    /*explicit*/ Resource(std::string_view filename) {}
    
    // Construct with preloaded data
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
    // Note) passing a string literal on case 4 would cause a multi-step (user-defined) implicit conversion, which won't even compile.
    // To be specific, string literal -> std::string_view -> Resource is required for an implicit Resource construction.
    rm.addResource({"file3"}); // case 3) an implicit conversion using initializer list (somewhat noticeable due to curly braces)
    rm.addResource("file4"sv); // case 4) an implicit conversion quite hard to spot
    
    // Constructors with two or more parameters usually don't cause trouble.
    // Since you can't make implicit conversion happen without using an initializer list (i.e. case 4 cannot happen with multi-parameter constructors)
    rm.addResource({5, "Hello, world!"}); // case 5) an implicit conversion using initializer list
    rm.addResource(6, "you can't do this"); // case 6) Error: no matching function for call to ResourceManager::addResource(int, const char [18])
}
```
Check out this [stackoverflow question](https://stackoverflow.com/questions/12437241/c-always-use-explicit-constructor) for more opinions on when to use explicit constructors.

### <a name='tip16'></a>The reason why using static_cast for downcast is unsafe
```c++
#include <iostream>

class Base
{
public:
    Base(int baseMember)
        : baseMember(baseMember)
    {}

    void foo()
    {
        std::cout << baseMember << std::endl;
    }

protected:
    int baseMember;
};

class Derived : public Base
{
public:
    Derived(int baseMember, int derivedMember)
        : Base(baseMember), derivedMember(derivedMember)
    {}

    void goo()
    {
        std::cout << baseMember << ", " << derivedMember << std::endl;
    }

private:
    int derivedMember;
};

void test(Base* pb)
{
    // This compiles without error because Base and Derived are in inheritance relation,
    // however, we cannot be sure that this is always a valid conversion.
    // In case the object which the base pointer contains doesn't have a member or method we try to access,
    // memory regions allocated to other objects could be 'invaded'.
    Derived* pd = static_cast<Derived*>(pb);
    pd->goo();
}

int main()
{
    // x86-64 gcc 12.2 was used to analyze the result, thanks to godbolt.org!
    //
    // Memory layout:
    // ---- higher address ----
    // d.derivedMember 12345 <- [rbp - 4]
    // d.baseMember    5555  <- [rbp - 8]  (&d)
    // b.baseMember    3333  <- [rbp - 12] (&b)
    //                       <- [rsp] (i.e. the top of the stack)
    // ---- lower address ----
    //
    // Note that baseMember has an offset of 0, while derivedMember has +4.
    Derived d(5555, 12345);
    Base b(3333);

    // Called with address [rbp - 8]:
    // 1. goo() assumes that pd->baseMember is at [rbp - 8] (valid: accessing d.baseMember)
    // 2. goo() assumes that pd->derivedMember is at [rbp - 4] (valid: accessing d.derivedMember)
    test(&d); // prints 5555, 12345

    // Called with address [rbp - 12]:
    // 1. goo() assumes that pd->baseMember is at [rbp - 12] (valid: accessing b.baseMember)
    // 2. goo() assumes that pd->derivedMember is at [rbp - 8] (INVALID: thats the address for d.baseMember!)
    test(&b); // prints 3333, 5555
}
```

### <a name='tip28'></a>Mutability of captured variables in a lambda
```c++
#include <utility>

int main()
{
    auto i = 12345;
    const auto ci = 12345;

    // capture by value (const by default)
    [x = i]{};             // const int x
    [x = ci]{};            // const int x
    [x = ci]() mutable {}; // int x ('mutable' keyword allows captured variables to be modified!)

    // capture by reference (inherits const qualifier)
    [&x = i]{};                // int& x
    [&x = ci]{};               // const int& x
    [&x = std::as_const(i)]{}; // const int& x (std::as_const() adds const qualifier)

    // example) a sequential integer generator using stateful lambda
    auto generator = [count = 0]() mutable { return ++count; };
    generator(); // 1
    generator(); // 2
    generator(); // 3
}
```

### <a name='tip30'></a>std::vector<bool> doesn't store booleans
```c++
#include <vector>
#include <array>

int main()
{
    // std::vector<bool> stores bit instead of bool for efficiency.
    // This makes getting a bool& quite complicated,
    // because bool is 1 byte but the actual data is 1 bit!
    // What std::vector<bool>::operator[] returns is actually a proxy instance std::vector<bool>::reference.
    //
    // The code below implicitly converts std::vector<bool>::reference to an rvalue bool,
    // so it is basically same as binding a temporary bool to a reference, which is not allowed.
    std::vector<bool> v{true, false};
    bool& b1 = v[0]; // Error: cannot bind non-const lvalue reference of type bool& to an rvalue of type bool
    bool& b2 = true; // Same error!

    std::array<bool, 2> a{true, false};
    bool& b3 = a[0]; // std::array actually stores bool, so this is valid.
}
```
This is also stated in [Microsoft's C++ documentation](https://learn.microsoft.com/en-us/cpp/standard-library/vector-bool-class?view=msvc-170).

### <a name='tip41'></a>A malicious bypass: passing int as an enum class parameter
```c++
enum class Animal { Dog, Cat };

void foo(Animal animal) {}

int main()
{
    // We know that enum class doesn't allow implicit conversion from int
    foo(123); // Error: cannot convert from int to Animal

    // But for some reason, this also compiles.
    // Actually, I wonder why this is valid...
    // Just keep in mind that strange things can happen
    // even if you use enum class instead of classic enum.
    foo(Animal{123});
}
```

## Concurrency
### <a name='tip29'></a>Manually locking and unlocking a mutex can be dangerous
```c++
#include <mutex>

// Some asynchronous job that might throw exceptions.
void critical_section()
{
    throw std::exception();
}

int main()
{
    // Suppose we have a resource protected with this mutex.
    std::mutex m;

    try
    {
        // Case 1) manual lock/unlock
        {
            m.lock();

            // You're not sure if this will throw exception.
            // In this case, it DOES throw an exception.
            critical_section();

            // Since the exception will bring us to the catch clause,
            // this line doesn't get executed and m stays locked.
            m.unlock();
        }

        // Case 2) RAII style lock/unlock
        {
            // When an exception happens and lg goes out of scope,
            // the destructor of std::lock_guard will call m.unlock automatically.
            auto lg = std::lock_guard(m);
            critical_section();
        }
    }
    catch(const std::exception& e)
    {
        // Handle exception
    }
}
```
Use std::lock_guard or std::unique_lock for exception safety.

### <a name='tip34'></a>How to wait for a signal or create a synchronization point for multiple threads
```C++
#include <thread>
#include <mutex>
#include <barrier>
#include <condition_variable>
#include <queue>
#include <atomic>
#include <format>
#include <iostream>

// Desired behavior:
// - a thread should wait for an event triggered by another thread
//
// Tools used:
// - std::mutex
// - std::condition_variable
// - std::unique_lock
//
// Required standard:
// - C++11 or higher
// 
// Example use case:
// - producer thread accumulates request packets sent by clients into a queue
// - consumer thread pops packet from the queue and handle the requst
void wait_for_signal()
{
    // Used to nofity consumer thread that producer thread is terminated
    auto producer_done = std::atomic_bool{ false };

    // Shared instance used to pass information from producer to consumer
    auto items = std::queue<int>{};

    // Synchornization primitives
    auto m = std::mutex{};
    auto cv = std::condition_variable{};

    // Producer thread pushes an integer to items vector every 500ms
    auto producer = std::thread([&] {
        for (int i = 0; i < 3; ++i)
        {
            std::this_thread::sleep_for(std::chrono::milliseconds(500));

            // Critical section
            {
                auto lg = std::lock_guard(m);
                items.push(i);
            }

            cv.notify_one();
        }

        producer_done = true;
    });

    // Consumer thread waits for items vector to fill up and print the content
    auto consumer = std::thread([&] {
        auto exit = false;
        while (!exit)
        {
            // 1. lock m.
            auto ul = std::unique_lock(m);

            // 2. evaluate predicate (the lambda passed as the second parameter)
            //    if the result is true, proceed to the next line (m stays locked)
            //    if the result is false, unlock m and start waiting.
            // 3. when a signal arrives, lock m and check predicate to see if it was a 'spurious wakeup'.
            //    if the result is true, proceed to the next line (m stays locked)
            //    if the result is false, unlock m and repeat step 3.
            // 
            // 'cv.wait(ul, pred)' is equivalent to 'while (!pred()) cv.wait(ul)'
            // and this prevents two problems that come with condition variables.
            // 
            // 1) spurious wakeup
            // - cv.wait(ul) can be unblocked even if we do not send any signal (not sure why...)
            // - by using the conditional loop, we can go back to the waiting state on fake signals.
            // 
            // 2) lost wakeup: 
            // - if no thread is waiting on cv.wait(), signals from notify_one() or notify_all() are lost.
            // - if we use wait() without predicate, the thread is always blocked and will never wake up until new signal or spurious wakeup happens.
            // - since conditional loop checks predicate before calling cv.wait(),
            //   we can avoid blocking a thread forever even if some of the signals are lost.
            cv.wait(ul, [&] { return items.size() > 0; });

            auto item = items.front();
            items.pop();

            // Finish the loop if producer thread termiated and all the items are handled
            if (producer_done && items.empty())
            {
                exit = true;
            }

            ul.unlock();

            // Handle the item
            std::cout << item << std::endl;
        }
    });

    producer.join();
    consumer.join();
}

// Desired behavior:
// - multiple threads should wait on a synchronization point
//
// Tools used:
// - std::barrier
// - std::latch (single-use version of std::barrier)
//
// Required standard:
// - C++20 or higher
// 
// Example use case:
// - you are making a game where multiple objects should update() and render() on each frame
// - update() can be parallelized, so you are going to use thread pool
// - render() should be called only after every update() is done
void wait_for_tasks()
{
    constexpr auto num_workers = 3;

    // Create a barrier with initial count and a completion callback.
    // Each arrive_and_wait() decreases the count by one
    // and the callback gets called when the count reaches zero.
    // Note: callback function should be 'noexcept'
    auto sync_point = std::barrier(num_workers, []() noexcept {
        std::cout << "everyone reached sync_point\n";
        });

    // Create worker threads
    auto workers = std::vector<std::thread>();
    for (int i = 0; i < num_workers; ++i)
    {
        workers.push_back(std::thread([&, id = i] {
            std::cout << std::format("worker thread #{} started\n", id);
            std::this_thread::sleep_for(std::chrono::milliseconds(100 * id));
            std::cout << std::format("worker thread #{} reached sync_point\n", id);
            sync_point.arrive_and_wait();
            std::cout << std::format("worker thread #{} ended\n", id);
            }));
    }

    for (auto& worker : workers)
    {
        worker.join();
    }
}

int main()
{
    std::cout << "Example 1: waiting for a signal" << std::endl;
    wait_for_signal();

    std::cout << "Example 2: waiting on a synchronization point" << std::endl;
    wait_for_tasks();
}
```
Expected output:
```
Example 1: waiting for a signal
0
1
2
Example 2: waiting on a synchronization point
worker thread #0 started
worker thread #2 started
worker thread #0 reached sync_point
worker thread #1 started
worker thread #1 reached sync_point
worker thread #2 reached sync_point
everyone reached sync_point
worker thread #1 ended
worker thread #2 ended
worker thread #0 ended
```

### <a name='tip43'></a>Printing to std::cout without race condition
```c++
// Valid from C++20
#include <format>
#include <vector>
#include <thread>
#include <iostream>

int main()
{
    auto threads = std::vector<std::jthread>{};
    for (int i = 0; i < 3; ++i)
    {
        threads.push_back(std::jthread([i]{
            // The order of "hello, thread #", i, and "\n" from multiple threads is undefined,
            // because we cannot guarantee that the three operator<< will get executed
            // sequentially before other threads try the same job.
            // ex) hello, thread #hello, thread #hello, thread #0\n2\n1\n
            std::cout << "hello, thread #" << i << "\n";

            // However, each operator<< behaves like an atomic operation!
            // Simply using std::format to pass the whole message at once solves the problem.
            std::cout << std::format("hello, thread #{}\n", i);
        }));
    }
}
```

## Other Tips
### <a name='tip22'></a>Using nested symbol of a template type as a type name
```c++
struct Int
{
    using Something = int;
};

struct Double
{
    using Something = double;
};

struct Haha
{
    static constexpr int Something = 1234;
};

// T::Something can be either a type or a non-type (e.g. static member variable):
//   Int::Something  -> int
//   Haha::Something -> compile-time constant integer of value 1234.
//
// To distinguish type and non-type usage:
// 1. Specify 'typename' if a scoped name of a template type like T::Something should be interpreted as a type.
// 2. Do not specify 'typename' if a scoped name of a template type is non-type.
template<typename T>
constexpr auto asType()
{
    return typename T::Something{123};
}

template<typename T>
constexpr auto asNonType()
{
    return T::Something;
}

int main()
{
    /*
    asNonType<Int>(); // Error: dependent-name T::Something is parsed as a non-type
    asNonType<Double>(); // Error: dependent-name T::Something is parsed as a non-type
    asType<Haha>(); // Error: no type named 'Something' in 'struct Haha'
`   */

    constexpr auto i = asType<Int>();
    constexpr auto d = asType<Double>();
    constexpr auto haha = asNonType<Haha>();

    static_assert(i == 123);
    static_assert(d == 123.0);
    static_assert(haha == 1234);
}
```

### <a name='tip17'></a>Declaring variables inside a switch statement
```c++
#include <iostream>

int main()
{
    int val = 2;

    // Switch-case version
    switch(val)
    {
    case 1:
    {
        std::string msg = "Hello, world!"; // invalid without scoping the 'case' clause
        std::cout << msg << std::endl;
        break;
    }
    case 2:
        std::cout << "Goodbye, world!" << std::endl;
        break;
    }


    // Goto version
    if (val == 1)
    {
        goto case1;
    }
    else if (val == 2)
    {
        goto case2;
    }
    else
    {
        goto exit;
    }
case1:
    {
        std::string msg = "Hello, world!";
        std::cout << msg << std::endl;
        goto exit;
    }
case2:
    std::cout << "Goodbye, world!" << std::endl;
    goto exit;
    // Imagine what would have happened if the compiler allowed declaration of 'msg' without our extra scope
    // and we tried to execute codes like 'std::cout << msg.size()' right here.
    // That would have caused skipping initialization of the variable 'msg'!
    // For the same reason, goto statement also prohibits variable declaration between labels.
exit:
    return 0;
}
```
Further details can be found in this [stackoverflow question](https://stackoverflow.com/questions/92396/why-cant-variables-be-declared-in-a-switch-statement)

### <a name='tip14'></a>const keyword applies to the left token, unless it comes at the start
```c++
// Reading a type name from right to left helps to understand what it means
const int* i1 = nullptr; // A pointer to an integer that is constant
int const* i2 = nullptr; // A pointer to a constant integer (same as i1)
const int* const i3 = nullptr; // A constant pointer to an integer that is constant
int const* const i4 = nullptr; // A constant pointer to a constant integer (same as i3)

// Only the leftmost const applies to the right token,
// so both 'const's are decorating 'int'.
// The example below DOES NOT mean "a pointer that is constant which points to an integer that is constant"
// Rather, it is "a pointer to a constant integer that is constant".
const int const* i5 = nullptr; // Error: duplicate 'const'

// Unlike pointers, we cannot change what a reference variable is pointing at.
// That means decorating a reference with const ('T& const') is unnecessary.
// Actually, compilers prohibit declaring types interpreted as "a constant reference to ...".
int& const i6 = 0; // Error: 'const' qualifiers cannot be applied to 'int&'
const int& const i7 = 0; // Error: 'const' qualifiers cannot be applied to 'const int&'

// By the way, pointer to a reference is illegal just like '& const' stuff.
int&* i8 = nullptr; // Error: cannot declare a pointer to 'int&'
```
This [stackoverflow question](https://stackoverflow.com/questions/54359088/const-qualifiers-cannot-be-applied-to-stdvectorlong-unsigned-int) handles '& const' issue.<br>
This [stackoverflow question](https://stackoverflow.com/questions/1898524/difference-between-pointer-to-a-reference-and-reference-to-a-pointer) explains more about '&*'

### <a name='tip26'></a>Polymorphism without runtime overhead (ft. CRTP)
```c++
#include <iostream>
#include <memory>

// Polymorphism through  function
class Base
{
public:
    virtual void foo() const = 0;
};

class Derived : public Base
{
public:
    virtual void foo() const override
    {
        std::cout << "runtime polymorphism" << std::endl;
    }
};

// Polymorphism-ish behavior by passing derived class as template parameter
// This kind of inheritance pattern is called the CRTP (Curiously Recurring Template Pattern)
template<typename T>
class CRPTBase
{
public:
    void foo() const
    {
        static_cast<const T*>(this)->foo();
    }
};

class CRPTDerived : public CRPTBase<CRPTDerived>
{
public:
    void foo() const
    {
        std::cout << "compile time polymorphism" << std::endl;
    }
};

// The underlying instance is determined at runtime.
//  function table is used to determine which foo to use.
void test(const Base& b)
{
    b.foo();
}

// Since T is given at compile time, there is no need for vtable lookup or any kind of runtime check.
// The downside is that we should use templates everywhere in order to utilize this trick.
template<typename T>
void test(const CRPTBase<T>& b)
{
    b.foo();
}

int main()
{
    Derived d1;
    CRPTDerived d2;

    test(d1);
    test(d2);
}
```

### <a name='tip3'></a>Creating a lambda behaves the same as creating a struct with operator() overloaded
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

### <a name='tip32'></a>A simple yes/no guideline for deciding member variable type
```c++
#include <memory>

// Given that T is not a fundamental type such as int,
// let us decide what variant of T should we use as a member variable.
//
// These are the five most frequently used types for member variables:
// - T
// - T*
// - T&
// - std::shared_ptr<T>
// - std::unique_ptr<T>
//
// Keep answering yes or no to the questions below
// to find out which of the five best fits you.
//
// For people who are not sure what ownership means,
// let's assume that you should own an instance when the instance's lifetime should depend on you.
//
// ex) A car should own an engine because you don't need the engine alone.
//
// On the other hand, you don't necessarily require ownership
// when the instance is guaranteed to be valid while you reference it.
//
// ex) A function that calls Player::name() doesn't need to own Player instance,
//     you know that the instance will be alive at least inside that function.
//
// Although there are cases you don't need to own something, it doesn't mean that you shouldn't.
// Having ownership does come with cost but it also has benefits.
// For example, you won't suffer from dangling references or pointers!
//
// If you are sure that such problems (e.g., null pointer exception) don't exist
// and you need to take care of performance, consider non-owning references.
//
// WARNING: this is a personal guideline, so the suggestion might not fit all situations.
template<typename T>
void member_type_guideline()
{
    // Should you own an instance of T?
    if (true)
    {
        // Is there any other object that also requires ownership to it?
        if (true)
        {
            // WARNING: circular reference might cause memory leak (use std::weak_ptr in such cases)
            std::shared_ptr<T> member_variable;
        }
        else
        {
            // Is T a polymorphic class?
            // Note: T is polymorphic if it has virtual functions.
            if (true)
            {
                // Reason: polymorphism only works with references or pointers.
                std::unique_ptr<T> member_variable;
            }
            else
            {
                T member_variable;
            }
        }
    }
    else
    {
        // Do you need to provide copy constructor and copy assignment operator for your class?
        if (true)
        {
            // Reason: you can't copy an object when you use reference type for a member variable.
            T* member_variable;
        }
        else
        {
            // Do you need to reassign a new value to the member variable?
            if (true)
            {
                // Reason: you can't reassign a reference variable.
                T* member_variable;
            }
            else
            {
                // Since you have to initialize reference variables on construction,
                // you'll have to provide an instance of T as a constructor parameter.
                T t;
                T& member_variable = t;
            }
        }
    }
}
```

### <a name='tip33'></a>Make sure that `set(CMAKE_CXX_STANDARD ??)` comes after `project()`
Bad
```cmake
cmake_minimum_required(VERSION 3.10)

# These settings are ignored!
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

project(test)

add_executable(test main.cpp)
```
Good
```cmake
cmake_minimum_required(VERSION 3.10)

project(test)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

add_executable(test main.cpp)
```
- Note: ```target_compile_features(test PUBLIC cxx_std_17)``` is preferred over ```set(CMAKE_CXX_STANDARD 17)```

### <a name='tip38'></a>How to make a basic command-line argument parser with ranges library
```c++
#include <iostream>
#include <optional>
#include <ranges>
#include <vector>
#include <format>

struct Argument
{
    std::string name;
    std::optional<std::string> value = {};
};

// Returns an iterable range of std::string_view
// that contains argument tokens of a command line input.
// 
// Consecutive blanks and the initial token are ignored.
// ex) "asdf.exe a    b c  d" -> "a" "b" "c" "d"
auto split_tokens(std::string_view prompt_input)
{
    auto is_not_empty = [](auto subrange) {
        return subrange.size() > 0;
    };
    auto to_string_view = [](auto subrange) {
        return std::string_view(subrange);
    };

    return prompt_input
        | std::views::split(' ')
        | std::views::filter(is_not_empty) // Prevent consecutive ' ' from generating empty subranges.
        | std::views::drop(1) // We should ignore the first token, which is the name of an executable.
        | std::views::transform(to_string_view);
}

// Parse the command line input into a list of name-value pairs.
// 
// We assume that argument names are prefixed with '-'
// and the value for an argument comes right after the name.
// 
// Note: exception handling was intentionally skipped
//       in order to focus solely on library feature demonstration.
std::vector<Argument> parse_arguments(std::string_view prompt_input)
{
    auto arguments = std::vector<Argument>();
    for (auto token : split_tokens(prompt_input))
    {
        // New argument name
        if (token.starts_with('-'))
        {
            token.remove_prefix(1);
            arguments.emplace_back(std::string(token)); 
        }
        // Argument value
        else
        {
            arguments.back().value = token;
        }
    }

    return arguments;
}

int main()
{
    for (const auto& [name, value] : parse_arguments("cmake -S . -B build -Wno-dev"))
    {
        std::cout << std::format("name: {:<8} value: {}\n", name, value.value_or("none"));
    }
}
```
Expected output:
```
name: S        value: .
name: B        value: build
name: Wno-dev  value: none
```

### <a name='tip40'></a>Do not use 'using namespace' in headers
- Namespace is very useful for preventing name collisions, but this makes qualified names quite long and verbose.
- ```using namespace ...``` allows unqualified access to the namespace, but this leads to potential name collision.
- The real problem happens when you use ```using namespace``` in a header.
Every source file that uses the header also has access to symbols inside that namespace whether they want it or not.
- On the other hand, ```using namespace``` in a cpp file isn't a big problem.  
  It only affects a single file and is easy to fix.
#### Example scenario
- Suppose that you have two "vector" classes: std::vector and math::vector.
```c++
// vector.h
namespace math
{

template<typename T>
class vector
{
public:
    vector(T x, T y, T z);
};

} // namespace math
```
- One developer starts to use ```using namespace math``` in a header file because prefix ```math::``` was all over the place.
```c++
// rigidbody.h
using namespace math;

namespace physics
{

class rigidbody
{
public:
    rigidbody(double mass, const vector<double>& position, const vector<double>& rotation);
    void add_force(const vector<double>& force);

private:
    double mass;
    vector<double> position;
    vector<double> rotation;
    vector<double> velocity;
};

} // namespace physics
```
- You, on the other hand, were working on another source file that uses rigidbody.h  
  and discovered that your 'vector' suddenly became ambiguous!
```c++
// main.cpp
#include "rigidbody.h"
#include <vector>

using namespace std;

int main()
{
    vector<rigidbody> objects; // error: 'vector' is ambiguous
}
```
