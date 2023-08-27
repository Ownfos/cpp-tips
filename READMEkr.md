# cpp-tips 한국어 번역
혹시 필요하신 분들 계실까봐 만들어봤습니다!

## List of Contents
### Language Features
- [Trailing return type](#tip5)
- [Structured binding](#tip20)
- [Virtual destructor](#tip27)
- [가변 길이 템플릿을 위한 fold expression](#tip9)
- [Designated initializer (ft. named parameter)](#tip8)
- [쉼표는 separator와 operator 두 가지로 사용 가능합니다](#tip13)
- [스마트 포인터를 위한 dynamic/static cast](#tip15)
- [이항 연산자를 정의하는 세 가지 방법](#tip25)
- ['qualified name'과 'unqualified access'의 의미](#tip18)
- [name mangling과 extern "C"](#tip35)
- [파라미터 타입으로 'auto' 사용하기](#tip36)
- [```std::declval<T>()```](#tip37)
### Initialization and Construction
- [initializer-list를 사용한 std::vector초기화는 항상 복사 생성자를 호출합니다](#tip1)
- [const std::string&와 std::string_view도 메모리 할당을 일으킬 수 있습니다](#tip2)
- [헤더 파일에서 static 멤버 변수를 선언하는 동시에 정의하는 방법](#tip7)
- [레퍼런스 타입의 멤버 변수를 초기화하는 방법 (ft. member initializer list)](#tip21)
### Function Parameters and Perfect Forwarding
- [Rvalue reference 파라미터는 lvalue입니다](#tip12)
- [Template 타입 추론 (ft. std::forward와 universal reference)](#tip23)
- [왜 universal reference에 std::forward까지 필요한가요? (ft. perfect forwarding)](#tip31)
- [람다에서 perfect forwarding 구현하는 방법](#tip24)
### Linkage and Scope
- [추가적인 스코프로 변수 이름 가리기](#tip4)
- [모든 translation unit에서 변수 공유되게 만들기](#tip6)
- [External linkage vs internal linkage (with examples)](#tip19)
### Tricky Behaviors
- [다단계 암시적 변환은 허용되지 않습니다](#tip10)
- [암시적 변환은 혼란을 야기할 수 있습니다: 'explicit'을 사용해보세요](#tip11)
- [static_cast로 하는 downcasting이 위험한 이유](#tip16)
- [람다에서 캡처한 변수의 mutability](#tip28)
- [std::vector<bool>은 bool을 저장하지 않습니다](#tip30)
### Concurrency
- [뮤텍스를 수동으로 잠그고 푸는 것은 위험합니다](#tip29)
- [멀티스레딩을 할 때 특정 신호를 기다리거나 동기화 지점을 만드는 방법](#tip34)
### Other Tips
- [템플릿 타입의 중첩된 symbol을 타입 이름으로 사용하기](#tip22)
- [switch문 안에서 변수 선언하기](#tip17)
- [const 키워드는 제일 앞에 오지 않는 이상 왼쪽 토큰에 적용됩니다](#tip14)
- [런타임 오버헤드 없는 다형성 (ft. CRTP)](#tip26)
- [람다를 만드는 것은 operator()를 정의한 구조체를 만드는 것과 비슷합니다](#tip3)
- [멤버 변수의 타입을 결정하는 법](#tip32)
- [```set(CMAKE_CXX_STANDARD ??)```는 꼭 ```project()``` 뒤에 나와야합니다](#tip33)

## Language Features
### <a name='tip5'></a>Trailing return type
```c++
#include <iostream>
#include <type_traits>

// 함수 뒤에 ->로 리턴 타입을 명시하는 기능은 C++11부터 사용 가능합니다.
// 보기에도 좋지만 아래 예시처럼 리턴 타입이 템플릿 타입에 종속적인 경우에 실질적인 도움을 줍니다.
template<typename A, typename B>
auto multiply(A a, B b) -> decltype(a*b)
{
    return a * b;
}

// C++14부터는 리턴 타입 추론이 가능해서 -> 부분을 생략해도 됩니다.
template<typename A, typename B>
auto multiply2(A a, B b)
{
    return a * b;
}

// 참고로 decltype()안에 들어가는 식은 실행되지 않습니다.
// 실행해도 아무 일도 일어나지 않죠!
int foo()
{
    std::cout << "foo() was executed" << std::endl;
    return 0;
}

auto main() -> int
{
    static_assert(std::is_same_v<decltype(foo()), int>); // decltype(foo())는 int와 동일
}
```
더 많은 내용은 [이곳](https://www.danielsieger.com/blog/2022/01/28/cpp-trailing-return-types.html)에서 확인하실 수 있습니다.

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

// 주의사항: 이 예시는 structured binding 기능을 선보이기 위한 목적으로 만들어졌습니다.
// 실제 상황에서 여러 값을 리턴해야한다면 구조체를 사용하는 것이 더 좋을 수 있습니다.
// std::tuple은 값에 이름이 없어서 직관적이지 않지만 struct의 멤버는 이름이 있다보니 훨씬 알아보기 쉽거든요.
auto getStudentInfo()
{
    auto age = 17;
    auto height = 175.3;
    auto name = "Jack"s;

    return std::tuple{age, height, name};
}

int main()
{
    // Structured binding을 사용하면 key-value pair같은 객체를 사용할 때 가독성을 높일 수 있습니다.
    {
        auto m = std::map<int, double>{{1, 1.11}, {2, 2.22}};

        // m의 값을 두 배로 올리기
        std::cout << "Basic for loop using iterator:" << std::endl;
        for (auto it = m.begin(); it != m.end(); ++it)
        {
            std::cout << "changing m[" << it->first << "] to " << it->second * 2 << std::endl;
            it->second *= 2;
        }

        // Structured binding을 사용한 동일한 작업
        std::cout << "Range-based for loop using structured binding:" << std::endl;
        for (auto& [key, value] : m)
        {
            std::cout << "changing m[" << key << "] to " << value * 2 << std::endl;
            value *= 2;
        }
    }

    // std::tuple을 사용할 때 각 원소에 이름을 부여하기 위해 사용할 수도 있습니다
    {
        auto [age, height, name] = getStudentInfo();
        std::cout << "name: " << name << ", age: " << age << ", height: " << height << std::endl;
    }

    // 직접 정의한 구조체에도 사용 가능합니다!
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
    // 여기에 virtual destructor를 넣어주면 컴파일러가 Base*를 삭제할 때
    // 이 포인터가 자식 클래스를 가리켜서 ~Derived()같은 소멸자를 추가로 실행해야 하는지 확인합니다.
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
    // OK: Base와 Derived의 소멸자가 모두 호출됩니다
    Derived* d = new Derived();
    delete d; // ~Derived ~Base

    // Bad: Base의 소멸자만 호출됩니다! Derived에 해당하는 메모리 영역이 해제되지 않아 메모리 누수를 일으킬 수 있습니다./
    Base* b = new Derived();
    delete b; // ~Base

    // Ok: 스마트 포인터는 virtual destructor 없이도 잘 작동합니다.
    std::shared_ptr<Base> sb = std::make_shared<Derived>();
    sb.reset(); // ~Derived ~Base
}
```
virtual destructor가 없어도 스마트 포인터가 소멸자를 제대로 호출할 수 있는 이유는 [여기서](https://blog.the-pans.com/why-you-dont-need-virtual-destructor-with-smart-pointers/) 확인하실 수 있습니다.

### <a name='tip9'></a>가변 길이 템플릿을 위한 fold expression
```c++
#include <iostream>
#include <vector>

using namespace std::string_literals;

template<typename First, typename... Others>
auto sum(First first, Others... others)
{
    // Fold expression의 네 가지 종류:
    // 1. binary left fold: (init op ... op pack)
    // 2. binary right fold: (pack op ... op init)
    // 3. unary left fold: (... op pack)
    // 4. unary right fold: (pack op ...)
    //
    // left fold는 왼쪽 항을 먼저 묶고 right fold는 오른쪽 항을 먼저 묶습니다.
    // ex) 템플릿으로 E1, E2, E3, E4, E5가 넘어왔다면
    // (... + others)는 ((((E1 + E2) + E3) + E4) + E5)이 되고
    // (others + ...)는 (E1 + (E2 + (E3 + (E4 + E5))))가 됩니다.
    //
    // 아래의 식은 (((first + second) + third) + ...) + last;와 동일합니다.
    return (first + ... + others);
}

auto main() -> int
{
    // 우리가 left fold를 사용했기 때문에 왼쪽에서부터 차례대로 std::string + const char* 연산이 일어나 결과적으로 std::string 객체를 리턴하게 됩니다.
    // 만약 첫 번째 인자로 문자열 리터럴을 넘겨줬다면 컴파일러가 const char* + const char*는 불가능하다고 에러를 출력했을겁니다.
    std::cout << sum("Hello, "s, "world", "!") << std::endl; // prints Hello, world!
}
```
정확한 내용은 [cppreference page](https://en.cppreference.com/w/cpp/language/fold)를 참고해주세요.

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
    // C++20부터는 구조체를 초기화할 때 멤버 이름으로 값을 지정할 수 있습니다.
    // 만약 함수 파라미터를 모아서 구조체를 만들면 이렇게 named parameter기능을 흉내낼 수 있습니다.
    foo({.x = 2, .y = -4});
}
```

### <a name='tip13'></a>쉼표는 separator와 operator 두 가지로 사용 가능합니다
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
    
    // 사실 쉼표가 연산자로 사용될 수도 있는데 우선순위가 워낙 낮아서 평소에는 separator로 사용됩니다.
    foo(i++, j += 1.1); // prints 1 3.3
    
    // 하지만 충분한 괄호로 감싸주면 쉼표를 왼쪽부터 차례대로 실행하고 제일 오른쪽 항을 리턴하는 연산자로 만들어버릴 수도 있습니다/
    // 아래의 식은 다음과 같은 효과를 냅니다:
    //   i++;
    //   j+=1.1;
    //   std::cout << j << std::endl;
    foo((i++, j += 1.1)); // prints 4.4
    
    // 참고로 삼항 연산자와 다르게 쉼표로 구분된 각각의 식이 다른 타입을 가져도 됩니다.
    // 이 친구는 항상 제일 오른쪽 항만 고르니까요!
    std::cout << (i++, j += 1.1) << std::endl; // prints 5.5
}
```

### <a name='tip15'></a>스마트 포인터를 위한 dynamic/static cast
```c++
#include <iostream>
#include <memory>

// Base1과 Derived1은 polymorphic합니다 (다형성이 있다...?)
// 이런 경우에는 dynamic_cast또는 std::dynamic_pointer_cast를 사용해야 합니다.
// raw pointer라면 전자, 스마트 포인터라면 후자입니다.
class Base1
{
public:
    virtual ~Base1() = default; // We need at least one virtual function for dynamic_cast to work
};
class Derived1 : public Base1 {};

// Base2와 Derived2는 polymorphic하지 않습니다.
// 이런 경우에는 static_cast나 std::static_pointer_cast를 사용해야 합니다.
class Base2 {};
class Derived2 : public Base2 {};

int main()
{
    // Case 1) polymorphic class
    {
        std::shared_ptr<Base1> pBase = std::make_shared<Derived1>(); // Ok: 흔한 upcasting
        
        // Cast to a relevant type
        std::dynamic_pointer_cast<Derived1>(pBase); // Ok: pBase가 Derived1 객체를 가리키고 있어서 괜찮습니다
        std::static_pointer_cast<Derived1>(pBase); // Uhhh: 되긴 하는데, 이건 pBase를 Derived1*로 바꿔도 괜찮은 상황이라 그렇습니다.

        // Cast to an irrelevant type
        std::dynamic_pointer_cast<Derived2>(pBase); // Bad: pBase를 Derived2*로 변환할 수 없기 때문에 캐스팅이 실패하고 nullptr를 리턴합니다.
        std::static_pointer_cast<Derived2>(pBase); // Error: invalid 'static_cast' from type Base1* to type Derived2*

        // 그런데 우리가 정말 downcasting을 필요로 할 상황이 있을까요?
        // 원칙적으로는 우리 코드가 concrete class (i.e. derived class)가 아니라 인터페이스에만 (i.e. base class) 의존하게 만들어야 합니다.
        // 그러니까 dynamic_cast나 std::dynamic_pointer_cast를 사용하는 것 자체가 code smell이라고 여길 수도 있는거죠!
        // 가능하다면 downcasting 없이 다형성만으로 문제를 해결할 수 있도록 재설계를 해보고, 최후의 수단으로 dynamic_cast를 사용해보는건 어떨까요?
    }

    // Case 2) nonpolymorphic class
    {
        std::shared_ptr<Base2> pBase = std::make_shared<Derived2>(); // Ok: 흔한 upcasting

        // Cast to a relevant type
        std::dynamic_pointer_cast<Derived2>(pBase); // Error: Base2가 polymorphic하지 않기 때문에 오류가 발생합니다
        std::static_pointer_cast<Derived2>(pBase); // Ok: pBase가 Derived2 객체를 가리키고 있어서 괜찮습니다

        // Cast to an irrelevant type
        std::dynamic_pointer_cast<Derived1>(pBase); // Error: Base2가 polymorphic하지 않기 때문에 오류가 발생합니다
        std::static_pointer_cast<Derived1>(pBase); // Error: invalid 'static_cast' from type Base2* to type Derived1*
    }
}
```

### <a name='tip25'></a>이항 연산자를 정의하는 세 가지 방법
```c++
struct Int
{
    int val;

    // Case 1) 멤버 함수로 정의하기
    constexpr Int operator+(const Int& other) const
    {
        return { val + other.val };
    }

    // Case 2) 전역 함수에 friend 붙여서 private 멤버 접근 권한 주기
    friend constexpr Int operator-(const Int& lhs, const Int& rhs)
    {
        return { lhs.val - rhs.val };
    }
};

// Case 3) 전역 함수에서 public 멤버만 사용해 정의하기
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

### <a name='tip18'></a>'qualified name'과 'unqualified access'의 의미
```c++
// Qualified name이란 식별자의 namespace나 class name을 포함한 full name을 말합니다.
// Qualified access는 식별자를 full name으로 부르는 것을 말합니다 (e.g. std::vector).
// Unqualified access는 반대로 qualified name의 일부분을 생략하고 부르는 것을 의미합니다.
// 중첩된 namespace 안이나 using namespace ~ 뒤에 오는 코드에서 흔히 볼 수 있죠.
namespace Toolbox
{
    // 이 친구의 qualified name은 Toolbox::three입니다.
    int three() { return 3; }

    namespace Math
    {
        // 이 친구의 qualified name은 Toolbox::Math::triple입니다.
        // Math가 Toolbox안에 중첩된 namespace라서 Toolbox에 선언된 모든 것들을 qualifier없이 접근할 수 있습니다 (i.e. unqualified access)
        // 그러니까 Toolbox::three()대신 three()라고만 적어도 됩니다.
        int triple(int val) { return val * three(); }
    }
};
```

### <a name='tip35'></a>name mangling과 extern "C"
```c++
// C++는 함수 오버로딩을 허용하므로 이름이 같고 파라미터는 다른 함수가 여럿 존재할 수 있습니다.
// 컴파일러는 이런 함수들을 식별할 방법이 필요하기 때문에 C++는 각각의 함수에 고유한 이름을 부여합니다.
//
// 예를 들어, 파라미터 타입의 첫 글자를 뒤에 붙인다면 아래에 있는 두 개의 foo()를 구분할 수 있을겁니다.
// ex) foo(int) -> foo_i, foo(double) -> foo_d
void foo(int);
void foo(double);

// 앞에 extern "C"를 붙여주면 name mangling을 사용하지 않을 수 있습니다.
// 함수 오버로딩은 사용할 수 없게 되지만 C 프로젝트에서도 함수를 사용할 수 있게 됩니다.
extern "C" void goo();

// extern "C"를 일일이 붙여주는 대신 스코프를 지정할 수도 있습니다.
extern "C"
{

void haha();
void hoho();

}
```

### <a name='tip36'></a>파라미터 타입으로 'auto' 사용하기
```c++
// C++20부터 가능
void foo(auto arg)
{
    std::cout << arg;
}

// 이 기능을 사용하면 concept를 간편하게 사용할 수 있습니다
// 참고: std::integral은 <concepts> 헤더에 정의된 표준 concept입니다
auto square(std::integral auto value)
{
    return value * value;
}

// 일반적인 템플릿과 함께 사용할 수 있습니다
template<typename T>
void goo(T first, auto... others)
{
    std::cout << first;
}

// 가변 인자와 perfect forwarding도 똑같이 쓸 수 있습니다
void print(auto&&... args)
{
    // 쉼표를 연산자로 사용하는 fold expression이며 다음과 같이 확장됨
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

// "Ret T::operator(Arg arg)"이라는 멤버 함수가
// 존재하는지 확인하는 concept를 만든다고 가정합니다.
//
// 아래에 나와있는 concept는 T와 Arg에 기본 생성자가 있다면 잘 작동합니다.
// 하지만 그렇지 않다면 어떻게 될까요?
//
// decltype()안에 들어간 식은 계산되지 않지만,
// 애초에 올바르지 않은 식이 되어버리므로 치환에 실패하게됩니다!
template<typename T, typename Ret, typename Arg>
concept Function = std::is_same_v<decltype(T{}(Arg{})), Ret>;

// 이런 경우에는 std::declval<T>()이 매우 유용합니다.
// 생성자를 호출하지 않고 바로 lvalue reference를 반환해주기때문에
// 저희는 그냥 T의 인스턴스가 존재한다고 가정하고 식을 작성할 수 있습니다.
template<typename T, typename Ret, typename Arg>
concept Function2 = std::is_same_v<decltype(std::declval<T>()(std::declval<Arg>())), Ret>;

// 'requires clause'를 사용해도 같은 기능을 구현할 수 있습니다.
// 예시를 조금 더 흥미롭게 만들기 위해 이번에는 가변인자 함수를 위한 concept를 만들어보겠습니다.
template<typename T, typename Ret, typename... Arg>
concept Function3 = requires(T func) {
    { func(std::declval<Arg>()...) } -> std::same_as<Ret>;
};

// 멤버 함수 "void operator()(std::string_view)"를 갖는 객체만 받는 테스트 함수
template<typename T>
    requires Function3<T, void, std::string_view>
void test(T func) {
    func("wow this works!");
}

// 기본 생성자가 없는 테스트용 functor
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
    // 람다 사용 가능
    test([](std::string_view s) {
        std::cout << s << std::endl;
    });

    // 기본 생성자가 없는 경우에도 사용 가능
    test(PrintMultipleTimes(3));
}
```

## Initialization and Construction
### <a name='tip1'></a>initializer-list를 사용한 std::vector초기화는 항상 복사 생성자를 호출합니다
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
    // Test 객체가 두 개 생성된 후 각각 복사됩니다.
    auto v = std::vector<Test>{{1}, {2}};

    std::cout << "======== reserve ========" << std::endl;

    // std::vector의 크기가 바뀔 때에도 복사가 일어납니다.
    v.reserve(100);

    std::cout << "======== push_back ========" << std::endl;

    // 만약 v에 공간이 충분하다면 push_back으로도 이동 생성자를 호출할 수 있습니다 (복사x)
    // 기본 생성자와 이동 생성자가 호출됩니다.
    v.push_back({3});

    std::cout << "======== emplace_back ========" << std::endl;
    
    // 기본 생성자 하나만 호출됩니다.
    // push_back(T&&)가 있긴 하지만, emplace_back은 임시 객체를 만들지 않는다는 점에서 이점이 있습니다.
    // 참고로 emplace_back만 perfect forwarding이 가능합니다!
    v.emplace_back(4);
    
    /* std::unique_ptr는 복사를 금지하기 때문에 아래와 같은 사용은 불가능합니다...
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
push_back과 emplace_back의 차이는 이 [stackoverflow 질문](https://stackoverflow.com/questions/4303513/push-back-vs-emplace-back)에서 더욱 자세히 다룹니다.<br>
이 [stackoverflow 질문](https://stackoverflow.com/questions/9618268/initializing-container-of-unique-ptrs-from-initializer-list-fails-with-gcc-4-7)에서는 ```std::vector<std::unique_ptr<T>>```의 초기화에 대한 내용을 읽어보실 수 있습니다.<br>

### <a name='tip2'></a>const std::string&와 std::string_view도 메모리 할당을 일으킬 수 있습니다
```c++
void foo(const std::string&) {}
void foo2(std::string_view) {}

int main()
{
    // Example 1) const std::string&을 사용했는데 메모리 할당이 일어나는 경우
    {
        foo("asdf"); // 임시 std::string 객체가 생성됩니다!
        foo2("asdf"); // 여기는 할당 없이 실행됩니다.
    }
    
    // Example 2) std::string_view를 사용했는데 메모리 할당이 일어나는 경우
    {
        std::string fileContent{"4GB짜리 파일"};
        std::string_view contentView{fileContent};

        // fileContent의 내용을 처리합니다.
        // 참고로 std::string_view를 사용하면 substring을 효율적으로 다룰 수 있습니다.

        std::string anotherString(contentView); // 4GB짜리 문자열이 하나 더 생겨버렸습니다...!
        std::string anotherString2(std::move(fileContent)); // 만약 원본이 필요하지 않다면 move semantics을 사용해보세요
    }
}
```
예시 2는 꽤 인위적이라 실제로 이런 일이 일어나지는 않을겁니다.<br>
그러니까 두 가지만 기억해주세요:
1. 레퍼런스와 view를 사용한다고 메모리 할당이 일어나지 않는 것은 아닙니다.<br>
   잘못 사용되면 임시 객체가 생기거나 추가적인 할당이 일어날 수도 있어요.
2. 가끔은 레퍼런스나 view보다 std::move가 더 좋은 선택지일 수 있습니다.

### <a name='tip7'></a>헤더 파일에서 static 멤버 변수를 선언하는 동시에 정의하는 방법
IDGenerator.h
```c++
class IDGenerator
{
private:
    // C++17부터 사용 가능합니다.
    //
    // 'inline'을 붙이는 것은 컴파일러에게 우리가 ODR을 어길건데
    // 모든 정의가 동일하니까 그 중에 하나만 골라서 마치 한 번만 정의한 것처럼 해달라고 부탁하는 것과 같습니다.
    //
    // 'inline'을 붙인다고 external linkage를 갖는 것은 아니지만
    // static inline 멤버 변수처럼 external symbol에 종종 사용됩니다.
    // internal linkage를 갖는 symbol은 보통 ODR을 신경쓸 일이 없어서 그런 것 아닌가 싶습니다.
    //
    // 참고로 'inline static'과 'static inline'은 기능적으로 동일하지만,
    // C 표준에서 static같은 storage specifier가 제일 앞에 나와야 한다고 명시해서 후자가 선호된다고 하네요.
    //
    // static 멤버 변수는 클래스의 linkage를 따라가는데 클래스는 기본적으로 external linkage를 가져서 대부분의 경우에 external입니다.
    // 하지만 익명 namespace 안에서는 class도 internal linkage를 가지기 때문에 internal인 경우도 가능하긴 하죠.
    static inline int nextID = 0;
};

// 익명 namespace (내부에 선언된 친구들이 internal linkage를 가지게 만듭니다)
namespace
{
    class Test // internal linkage
    {
    public:
        static inline int test = 1234; // internal linkage
    };
}
```
'inline static'과 'static inline'에 대한 설명은 이 [stackoverflow 질문](https://stackoverflow.com/questions/61714110/static-inline-vs-inline-static)에서 다룹니다.<br>
이 [stackoverflow 질문](https://stackoverflow.com/questions/16386256/inline-functions-and-external-linkage)은 'inline'과 'external linkage'를 구분하는 데에 도움을 줍니다.

### <a name='tip21'></a>레퍼런스 타입의 멤버 변수를 초기화하는 방법 (ft. member initializer list)
```c++
struct Device {};

struct Renderer
{
    // 참고: 멤버 변수를 레퍼런스 타입으로 하면 몇 가지 부작용이 있습니다.
    // 1. 참조하는 객체를 복사하지 않고는 대입 연산을 수행할 수 없습니다.
    //    - 'ref = otherRef'는 'ref'가 가리키는 객체를 바꾸는게 아니라 복사 대입 연산자를 호출합니다
    // 2. 기본 생성자를 사용할 수 없습니다.
    //    - 'int& i;'는 불가능합니다 (존재하지 않는 것에 대한 별칭이란 개념은 이상하기 때문이죠)
    //
    // 주석 한 블럭으로 끝내기엔 너무 무거운 주제라 밑에 링크를 달아놓겠습니다.
    Device& device;

    // 멤버 변수는 생성자의 첫 번째 라인에 들어가기 전에 모두 초기화됩니다.
    // 그러니까 생성자 안에서 사용된 '='는 모두 대입 연산자를 의미합니다.
    //
    // 레퍼런스 타입은 무조건 초기화할 때 값을 줘야 하기 때문에 생성자 본문 말고 초기화를 해줄 다른 곳이 필요합니다.
    //
    // 여기서 'member initializer list'가 빛을 발합니다.
    // 생성자 이름 뒤에 :로 시작하는 부분은 생성자 본문보다 앞서 실행되어 멤버 변수 초기화를 수행합니다.
    Renderer(Device& device)
        : device(device) // 이 부분이 'member initializer list'입니다
    {}
    
    /*
    Renderer(Device& device) // Error: uninitialized reference member ('Device& Renderer::device' should be initialized)
    {
        // 아래의 식은 this->device를 초기화하는게 아니라 'device'를 'this->device'로 복사합니다.
        // 레퍼런스 타입이 복사를 피하기 위해 주로 사용된다는걸 고려하면 아마 의도치 않은 결과겠죠?
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
['References, simply' by Herb Sutter](https://herbsutter.com/2020/02/23/references-simply/)에서 레퍼런스 타입에 대한 깊은 분석과 조언을 제공합니다.

## Function Parameters and Perfect Forwarding
### <a name='tip12'></a>Rvalue reference 파라미터는 lvalue입니다
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
    // 여기서 s는 rvalue를 받지만 s 자체는 lvalue입니다!
    foo(s); // foo(const std::string&)
    foo(std::move(s)); // foo(std::string&&)
}

int main()
{
    // prints 'lvalue rvalue'
    test("만약 rvalue reference 파라미터를 생성자로 '이동'해야한다면 std::move를 사용해주세요");
}
```

### <a name='tip23'></a>Template 타입 추론 (ft. std::forward와 universal reference)
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

    // std::forward는 static_cast<T&&>를 수행하는데, 여기에 위에서 언급한 reference collapsing rule 중 마지막 두 개가 사용됩니다.
    {
        static_assert(std::is_same_v<forwardType<int&>, int&>);
        static_assert(std::is_same_v<forwardType<int&&>, int&&>);
        static_assert(std::is_same_v<forwardType<const int&>, const int&>);
        static_assert(std::is_same_v<forwardType<const int&&>, const int&&>);
    }

    // std::move는 const 여부를 유지한 채 전부 &&로 만들어버립니다.
    {
        static_assert(std::is_same_v<moveType<int&>, int&&>);
        static_assert(std::is_same_v<moveType<int&&>, int&&>);
        static_assert(std::is_same_v<moveType<const int&>, const int&&>);
        static_assert(std::is_same_v<moveType<const int&&>, const int&&>);
    }

    // 함수의 템플릿 파라미터가 T, T&, const T&, 그리고 T&&로 선언되었을 때 T가 어떻게 추론되는지 알아봅
    int i = 1;
    const int ci = 1;
    int& ri = i;
    const int& cri = ci;

    // test(T param): const나 reference를 모두 떼어버리고 기본 타입만 남깁니다.
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

    // testR(T& param): rvalue를 제외한 모든 것을 const 여부를 유지한 채 레퍼런스로 받습니다.
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

    // testCR(const T& param): 모든 것을 const reference로 만들어버립니다.
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
    
    // testUR(T&& param): 넘긴 타입과 T가 동일합니다! (T&&는 그래서 'universal reference'라고 불립니다)
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

### <a name='tip31'></a>왜 universal reference에 std::forward까지 필요한가요? (ft. perfect forwarding)
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

// universal reference만 사용한 경우
template<typename T>
constexpr int goo1(T&& arg)
{
    return foo(arg);
}

// std::forward를 함께 사용한 경우
template<typename T>
constexpr int goo2(T&& arg)
{
    return foo(std::forward<T>(arg));
}

int main()
{
    Test t;

    // Universal reference를 사용하면 T에 우리가 넘긴 타입이 정확히 들어가니까
    // goo1처럼 구현해도 rvalue와 lvalue를 구분할 수 있을 것만 같습니다.
    // 하지만 goo1은 무조건 foo(Test&)를 호출합니다.
    // 'arg'라는 파라미터 자체는 lvalue이기 때문이죠!
    static_assert(goo1(Test{}) == goo1(t));

    // 반면에 goo2는 rvalue가 넘어왔을 때 foo(Test&&)를 호출합니다.
    // 이건 std::forward가 rvalue에 대해서는 std::move()와 같은 효과를 내고 lvalue는 그대로 두기 때문입니다 (reference collapsing).
    // 참고: T가 Test&&인 경우 std::forward<T>는 static_cast<Test&& &&>를 수행하는데, 이게 Test&&와 동일합니다.
    //
    // Forwarding은 foo와 goo에서 한 것처럼 파라미터로 넘어온걸 그대로 전달하는 행동을 말하는데요,
    // universal reference와 std::forward를 같이 쓰면 이걸 완벽하게 할 수 있다보니
    // 이런 방식의 구현을 'perfect forwarding'이라고 부른답니다.
    static_assert(goo2(Test{}) != goo2(t));
}
```

### <a name='tip24'></a>람다에서 perfect forwarding 구현하는 방법
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
generic lambda가 있음에도 template lambda를 추가로 도입한 데에는 [몇 가지 이유](https://stackoverflow.com/questions/54126204/what-is-the-need-of-template-lambda-introduced-in-c20-when-c14-already-has-g)가 있다고 합니다.

## Linkage and Scope
### <a name='tip4'></a>추가적인 스코프로 변수 이름 가리기
```c++
int main()
{
    // 필요하다면 스코프를 추가로 만들어서 변수 이름을 재사용할 수 있습니다.
    // desc1, desc2, result1, result2처럼 용도는 같은데 어쩔 수 없이 이름을 다르게 해야 하는 경우에 유용합니다.
    // 물론 구별 가능하고 유의미한 이름을 각각 지어줄 수 있다면 그게 최고겠죠...
    {
        int result = someComplexFunction();
        // Handle result
    }
    {
        std::vector<int> result = anotherComplexFunction();
        // Handle result
    }
{
```

### <a name='tip6'></a>모든 translation unit에서 변수 공유되게 만들기
Counter.h
```c++
// 'inline'키워드를 붙이면 같은 정의가 반복되는 것이 허용됩니다!
// 만약 'inline' 없이 이렇게 하면 ODR을 어겼다며 컴파일러가 혼내줍니다.
inline int counter = 0;

// 'static'키워드는 조금 다릅니다.
// 'inline'처럼 ODR 관련해선 뭐라 안 하는건 똑같은데 얘는 internal linkage를 가져서
// translation unit마다 counter2의 값이 다를 수 있습니다.
// cpp파일 하나마다 자기만의 counter2가 하나씩 생긴다고 생각하시면 편합니다!
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
    std::cout << foo2() << foo2() << counter2 << std::end; // prints 120 (Foo.cpp의 counter2만 증가하고 main.cpp의 counter2는 그대로 0)
}
```

### <a name='tip19'></a>External linkage vs internal linkage (with examples)
Test.h
```c++
// External linkage:
//     한 translation unit에서 선언된 변수를 다른 translation unit에서도 접근 가능합니다.
//     이런 변수는 모든 translation unit에서 같은 값을 가집니다.
//     프로그램 하나 당 객체 하나가 생긴다고 생각하시면 됩니다.
//     external linkage를 갖는 symbol은 ODR (One Definition Rule)을 준수해야 합니다.
//
// Internal linkage:
//     symbol이 선언된 translation unit 안에서만 접근 가능합니다.
//     이름이 같더라도 다른 translation unit에서는 다른 값을 가질 수 있습니다.
//     translation unit마다 객체 하나가 생긴다고 생각하시면 됩니다.


// 전역 변수는 기본적으로 external.
// 만약 여러 파일에서 #include를 해버리면 ODR을 위반하게됩니다.
// 혹시나 그럴 일이 있다면 한 곳에서만 변수를 정의하고 다른 곳에서는 'extern' 키워드를 붙여주세요.
/*extern*/ int case1 = 1234; // external linkage

// 'static'키워드는 전역 변수가 internal linkage를 갖도록 만듭니다.
// 익명 namespace안에 변수를 선언하는 것과 동일한 효과를 가집니다.
static int case2 = 1234; // internal linkage

// static global variable은 internal linkage를 가지므로 inline을 붙여도 아무 효과가 없습니다.
static inline int case3 = 1234; // internal linkage

// 전역 함수는 기본적으로 external.
// 여러 translation unit에서 #include해버리면 ODR violation이 일어납니다.
// 그러니까 헤더 파일에서 함수를 정의하는 일은 피해주세요!
// 정 그렇게 해야한다면 static 함수나 익명 namespace 안에 넣는걸 고려해보세요.
int case4() // external linkage
{
    return 1234;
}

// 전역 변수처럼 전역 함수도 'static'을 붙이면 internal linkage를 가집니다.
static int case5()
{
    return 1234;
}

// 클래스는 익명 namespace안에서 선언되지 않는 이상 external linkage를 가집니다.
class External // external linkage
{
public:
    // 멤버 함수는 기본적으로 external.
    int case6(); // external linkage
    
    // 클래스 선언 안에서 정의한 멤버 함수는 기본적으로 inline.
    // 덕분에 헤더 파일 안에서 정의해도 ODR 문제가 없습니다.
    /*inline*/ int case7() // external linkage
    {
        return 1234;
    }

    // static 멤버 변수는 클래스의 linkage를 따라갑니다.
    static inline int case8 = 1234; // external linkage

private:
    // non-static 멤버 변수의 linkage를 얘기하는 것은 의미가 없습니다.
    // translation unit 수준이 아니라 객체 단위로 다뤄지기 때문이죠.
    //
    // 헤더 파일 안에 다음과 같은 두 가지 선언이 있다고 생각해봅시다:
    //     extern External e1;
    //     static External e2;
    // case9이라는 멤버 변수는 internal linkage와 external linkage 중 어느 쪽이라고 해야할까요?
    // e1은 external이고 e2는 internal이니까 어느쪽도 정답이 될 수 없습니다!
    int case9 = 1234;
};

// 여러 translation unit에서 #include 했다면 ODR violation이 일어납니다.
// External::case7과 다르게 inline 멤버 함수가 아니기 때문입니다.
// 이런 방식의 멤버 함수 정의는 보통 대응되는 cpp파일에 (e.g., External.cpp) 들어갑니다.
int External::case6()
{
    return 1234;
}

// 익명 namespace는 안에 선언된 symbol이 internal linkage를 갖도록 만듭니다.
namespace
{
    // 전역 스코프에서 'static int case10 = 1234'라고 선언하는 것과 동일한 효과를 냅니다.
    int case10 = 1234; // internal linkage
    
    // 전역 스코프에서 'static int case11() { ... }'라고 선언하는 것과 동일한 효과를 냅니다.
    int case11() // internal linkage
    {
        return 1234;
    }

    // 참고: 'inline'키워드는 클래스 선언에 사용할 수 없습니다 (i.e. 'inline class'는 유효한 선언이 아닙니다)
    class Internal // internal linkage
    {
    public:
        // 클래스가 internal linkage를 가지므로 static 멤버 변수도 internal.
        static inline int case12 = 1234; // internal linkage
    };
}
```
[stackoverflow 질문](https://stackoverflow.com/questions/46103512/do-member-variables-have-external-linkage)에서 멤버 변수의 linkage에 대한 내용을 찾아보실 수 있습니다.<br>
[cppreference page](https://en.cppreference.com/w/cpp/language/static)의 'static data member'항목애 static 멤버 변수의 linkage에 대한 설명이 있습니다.<br>
이 [stackoverflow 질문](https://stackoverflow.com/questions/154469/unnamed-anonymous-namespaces-vs-static-functions)은 static 함수와 익명 namespace안의 함수를 비교해줍니다.

## Tricky Behaviors
### <a name='tip10'></a>다단계 암시적 변환은 허용되지 않습니다
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
    parseFile(File{"file1"}); // OK: const reference에는 임시 객체를 넘길 수 있습니다.
    parseFile("file2"sv); // OK: string_view -> File 한 단계의 암시적 변환은 괜찮습니다.
    parseFile("file3"); // Error: invalid initialization of reference of type const File& from expression of type const char[6]
}
```
[stackoverflow 질문](https://stackoverflow.com/questions/12847272/multiple-implicit-conversions-on-custom-types-not-allowed)에서 conversion 규칙에 대해 더욱 자세한 내용을 확인하실 수 있습니다.

### <a name='tip11'></a>암시적 변환은 혼란을 야기할 수 있습니다: 'explicit'을 사용해보세요
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

    // 의도한대로 잘 작동합니다
    auto r = Resource{"file1"};
    rm.addResource(std::move(r)); // case 1) lvalue를 std::move하기
    rm.addResource(Resource{"file2"}); // case 2) 임시 객체는 그 자체로 rvalue

    // 의도했는지는 모르겠지만 암시적 변환이 일어납니다.
    // '이게 왜 문자열을 받지?' 또는 '왜 문자열을 여기에 넘기지?'라는 의문이 생길 수 있습니다.
    //
    // case 4처럼 생성자의 파라미터가 하나인 경우 아무런 조짐 없이 암시적 변환이 일어납니다.
    // 만약 이런 상황을 막고 싶다면 생성자에 explicit을 붙여 아래 두 줄이 컴파일되는 것을 방지할 수 있습니다.
    // 
    // 참고) case 4에서 문자열 리터럴을 넘기려고 하면 multi-step implicit conversion이 불가능해서 컴파일되지 않습니다.
    // 자세히는 string literal -> std::string_view -> Resource를 거쳐야해서 암시적으로 Resource를 생성할 수가 없습니다.
    rm.addResource({"file3"}); // case 3) initializer list를 사용한 암시적 변환 (중괄호 덕분에 약간은 식별하기 쉽습니다)
    rm.addResource("file4"sv); // case 4) 함수 정의를 찾아보지 않는 이상 알아채기 힘든 암시적 변환
    
    // 생성자의 파라미터가 두 개 이상이라면 보통 문제가 없습니다.
    // initializer list를 사용하지 않고는 case 4같은 상황이 발생할 수 없기 때문입니다.
    rm.addResource({5, "Hello, world!"}); // case 5) initializer list를 사용한 암시적 변환
    rm.addResource(6, "you can't do this"); // case 6) Error: no matching function for call to ResourceManager::addResource(int, const char [18])
}
```
explicit 생성자에 대한 여러 의견이 궁금하시면 [stackoverflow 질문](https://stackoverflow.com/questions/12437241/c-always-use-explicit-constructor)을 읽어보시는걸 추천합니다.

### <a name='tip16'></a>static_cast로 하는 downcasting이 위험한 이유
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
    // Base와 Derived가 상속 관계이기 때문에 컴파일이 되긴 합니다만
    // pb로 뭐가 넘어올지 알 수 없기 때문에 성공은 장담할 수 없습니다.
    // 만약 우리가 접근하려는 함수나 변수가 없는 객체가 넘어올 경우
    // 다른 메모리 영역을 침범할 위험이 있습니다.
    Derived* pd = static_cast<Derived*>(pb);
    pd->goo();
}

int main()
{
    // 결과 분석에는 godbolt.org와 x86-64 gcc 12.2가 사용되었습니다!
    //
    // 메모리 구조:
    // ---- 높은 주소 ----
    // d.derivedMember 12345 <- [rbp - 4]
    // d.baseMember    5555  <- [rbp - 8]  (&d)
    // b.baseMember    3333  <- [rbp - 12] (&b)
    //                       <- [rsp] (i.e. 스택 꼭대기)
    // ---- 낮은 주소 ----
    //
    // baseMember는 offset이 0이고 derivedMember는 offset이 4임을 기억해주세요.
    Derived d(5555, 12345);
    Base b(3333);

    // [rbp - 8]로 함수 호출:
    // 1. goo()는 pd->baseMember가 [rbp - 8]에 있다고 가정합니다 (valid: d.baseMember에 접근합니다)
    // 2. goo()는 pd->derivedMember가 [rbp - 4]에 있다고 가정합니다 (valid: d.derivedMember에 접근합니다)
    test(&d); // prints 5555, 12345

    // Called with address [rbp - 12]:
    // 1. goo()는 pd->baseMember가 [rbp - 12]에 있다고 가정합니다 (valid: b.baseMember에 접근합니다 )
    // 2. goo()는 pd->derivedMember가 [rbp - 8]에 있다고 가정합니다 (INVALID: d.baseMember의 메모리 주소입니다!)
    test(&b); // prints 3333, 5555
}
```

### <a name='tip28'></a>람다에서 캡처한 변수의 mutability
```c++
#include <utility>

int main()
{
    auto i = 12345;
    const auto ci = 12345;

    // capture by value (기본적으로 const)
    [x = i]{};             // const int x
    [x = ci]{};            // const int x
    [x = ci]() mutable {}; // int x ('mutable'키워드를 붙여주면 캡처한 값을 수정할 수 있습니다!)

    // capture by reference (원본의 const 여부를 따라감)
    [&x = i]{};                // int& x
    [&x = ci]{};               // const int& x
    [&x = std::as_const(i)]{}; // const int& x (std::as_const()를 사용하면 const로 캡처합니다)

    // example) state가 있는 람다를 활용한 숫자 생성기
    auto generator = [count = 0]() mutable { return ++count; };
    generator(); // 1
    generator(); // 2
    generator(); // 3
}
```

### <a name='tip30'></a>std::vector<bool>은 bool을 저장하지 않습니다
```c++
#include <vector>
#include <array>

int main()
{
    // std::vector<bool>은 저장공간 효율을 위해 bool대신 bit를 사용합니다.
    // 이게 저장된 원소에 대한 bool&를 갖기 어렵게 만드는데요,
    // bit단위 레퍼런스는 언어 차원에서 제공하지 않기 때문입니다.
    // 실제로 std::vector<bool>::operator[]가 리턴하는 객체는 std::vector<bool>::reference라는 프록시입니다.
    //
    // 아래 예시는 std::vector<bool>::reference를 rvalue bool로 변환하는데,
    // 결과적으론 임시 bool 객체를 레퍼런스 변수에 묶으려는 것과 동일해서 허용되지 않습니다.
    std::vector<bool> v{true, false};
    bool& b1 = v[0]; // Error: cannot bind non-const lvalue reference of type bool& to an rvalue of type bool
    bool& b2 = true; // Same error!

    std::array<bool, 2> a{true, false};
    bool& b3 = a[0]; // std::array는 진짜 bool을 사용해서 잘 작동합니다.
}
```
[Microsoft's C++ documentation](https://learn.microsoft.com/en-us/cpp/standard-library/vector-bool-class?view=msvc-170)에도 해당 내용이 명시되어있습니다.

## Concurrency
### <a name='tip29'></a>뮤텍스를 수동으로 잠그고 푸는 것은 위험합니다
```c++
#include <mutex>

// 예외가 발생할 수도 있는 어떤 비동기 작업
void critical_section()
{
    throw std::exception();
}

int main()
{
    // 어떤 자원을 이 뮤텍스로 보호하고 있다고 가정합시다.
    std::mutex m;

    try
    {
        // Case 1) 수동 lock/unlock
        {
            m.lock();

            // 실제 상황에서는 여기서 예외가 발생하지 않을거라고 단정짓기 쉽지 않습니다.
            // 특히 남이 만든 함수를 호출하는데 그게 길고 복잡하다면 말이죠.
            // 이 예시에서는 예외가 발생합니다.
            critical_section();

            // 예외가 발생하면 즉시 catch절로 넘어가기 때문에
            // 아래 코드가 실행되지 않고 m은 영원히 잠긴 상태로 유지됩니다.
            m.unlock();
        }

        // Case 2) RAII style lock/unlock
        {
            // 예외가 발생해서 lg가 스코프 밖으로 나가면 소멸자에서 자동으로 unlock해줍니다.
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
exception safety를 위해 std::lock_guard나 std::unique_lock를 사용해주세요.

### <a name='tip34'></a>멀티스레딩을 할 때 특정 신호를 기다리거나 동기화 지점을 만드는 방법
```C++
#include <thread>
#include <mutex>
#include <barrier>
#include <condition_variable>
#include <queue>
#include <atomic>
#include <format>
#include <iostream>

// 원하는 동작:
// - 한 스레드에서 다른 스레드에서 신호를 보낼 때까지 기다리는 것
//
// 사용되는 도구:
// - std::mutex
// - std::condition_variable
// - std::unique_lock
//
// 필요한 c++ 표준:
// - C++11 이상
// 
// 용례:
// - producer 스레드는 클라이언트가 보내는 요청 패킷을 큐에 쌓아둠
// - consumer 스레드는 큐에서 패킷을 하나씩 꺼내고 처리함
void wait_for_signal()
{
    // producer 스레드가 끝났다고 알려주는 용도
    auto producer_done = std::atomic_bool{ false };

    // producer에서 consumer로 정보를 전달해주는 객체
    auto items = std::queue<int>{};

    // 동기화에 사용되는 객체
    auto m = std::mutex{};
    auto cv = std::condition_variable{};

    // producer 스레드는 500ms마다 items에 숫자를 하나씩 넣습니다
    auto producer = std::thread([&] {
        for (int i = 0; i < 3; ++i)
        {
            std::this_thread::sleep_for(std::chrono::milliseconds(100));

            // Critical section
            {
                auto lg = std::lock_guard(m);
                items.push(i);
            }

            // m을 해제하기 전에 신호를 보내도 cv.wait()에서 lock에 성공할 때까지 기다리기 때문에 큰 문제는 없지만
            // 이왕이면 바로 m을 잠글 수 있도록 해제한 상태에서 신호를 보내는게 좋다고 생각합니다.
            cv.notify_one();
        }

        producer_done = true;
    });

    // consumer 스레드는 items에 숫자가 들어올 때마다 하나씩 빼내고 출력합니다
    auto consumer = std::thread([&] {
        auto exit = false;
        while(!exit)
        {
            // 1. m을 잠금
            auto ul = std::unique_lock(m);

            // 2. 두 번째 파라미터로 넘긴 predicate를 검사하고 만약 true를 리턴하면 다음 줄로 넘어감
            //    false를 리턴한 경우 m을 해제하고 cv에 신호가 올때까지 기다림
            // 3. 신호가 오면 m을 잠그고 두 번째 파라미터로 넘긴 조건을 검사해서 'spurious wakeup'이 아닌지 확인
            //    만약 false를 리턴하면 다시 m을 해제하고 신호가 올때까지 기다림 (3번 반복)
            //    만약 true를 리턴하면 m이 잠긴 상태로 다음 줄로 넘어감
            // 
            // cv.wait(ul, pred)는 while (!pred()) cv.wait(ul)과 동일하며 condition variable의 취약점 두 가지를 보완해줍니다.
            // 
            // 1) spurious wakeup
            // - cv.wait(ul)은 신호를 보내지 않아도 블록이 해제될 수 있습니다 (정확한 원인은 저도 모르겠습니다...)
            // - 조건 체크 루프를 사용하면 가짜 신호가 온 경우 다시 대기 상태로 만들 수 있습니다
            // 
            // 2) lost wakeup: 
            // - wait()중인 스레드가 없는 경우 notify_one() 또는 notify_all()로 보낸 신호를 놓치게됩니다
            // - predicate가 없는 wait을 사용하면 무조건 블록되며 signal이 오거나 spurious wakeup이 발생하지 않는 한 깨어나지 않습니다.
            // - while (!pred()) 루프를 사용할 경우 wait 상태로 전환하기 전에 일단 조건을 체크하므로
            //   신호를 놓치더라도 스레드가 영원히 블록되는 상황을 방지할 수 있습니다
            cv.wait(ul, [&] { return items.size() > 0; });

            auto item = items.front();
            items.pop();

            // producer가 작업을 끝냈고 consumer도 items를 모두 처리했다면 루프 탈출
            if (producer_done && items.empty())
            {
                exit = true;
            }

            ul.unlock();

            // producer에서 넘겨준 값 처리
            std::cout << item << std::endl;
        }
    });

    producer.join();
    consumer.join();
}

// 원하는 동작:
// - 여러 스레드가 특정 지점에 도달할 때까지 기다리는 것
//
// 사용되는 도구:
// - std::barrier
// - std::latch (일회용 std::barrier)
//
// 필요한 c++ 표준:
// - C++20 이상
// 
// 용례:
// - 여러 오브젝트가 update()와 render()를 수행해야하는 게임을 만드는 중
// - update()는 병렬로 실행할 수 있어서 스레드 풀을 사용할 예정
// - render()는 모든 update()가 끝난 다음에 실행되어야 함
void wait_for_tasks()
{
    constexpr auto num_workers = 3;

    // 초기 카운트와 완료 callback으로 barrier를 생성합니다
    // arrive_and_wait()가 실행될 때마다 count가 줄어들고
    // count가 0이 되는 순간 callback이 실행됩니다.
    // Note: callback 함수는 'noexcept'이어야 합니다
    auto sync_point = std::barrier(num_workers, []() noexcept {
        std::cout << "everyone reached sync_point\n";
    });

    // worker 스레드를 생성합니다
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

## Other Tips
### <a name='tip22'></a>템플릿 타입의 중첩된 symbol을 타입 이름으로 사용하기
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

// T::Something은 type일수도 있고 non-type일수도 있습니다 (e.g. static 멤버 변수):
//   Int::Something  -> int
//   Haha::Something -> 컴파일 타임 상수 1234.
//
// 컴파일러에게 type인지 non-type인지 알려주는 방법:
// 1. 만약 type이라면 앞에 'typename'을 붙여주세요
// 2. 만약 non-type이라면 'typename' 없이 사용해주세요
template<typename T>
constexpr auto asType()
{
    return typename T::Something{123}; // 'typename T::Something' 전체가 하나의 타입을 가리킵니다
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

### <a name='tip17'></a>switch문 안에서 변수 선언하기
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
        std::string msg = "Hello, world!"; // 'case'절에 스코프를 넣어주지 않으면 오류 발생
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
    // 컴파일러가 msg를 스코프 없이 선언하게 허용했는데
    // 여기서 'std::cout << msg.size()'같은 코드를 실행한다고 생각해보세요.
    // 그럼 우리는 msg의 초기화를 건너뛰고 msg.size()를 실행하게 되는겁니다!
    // 같은 이유로 goto는 label사이에 변수를 선언하지 못하게 합니다.
exit:
    return 0;
}
```
더욱 자세한 내용은 [stackoverflow 질문](https://stackoverflow.com/questions/92396/why-cant-variables-be-declared-in-a-switch-statement)에서 다룹니다.

### <a name='tip14'></a>const 키워드는 제일 앞에 오지 않는 이상 왼쪽 토큰에 적용됩니다
```c++
// 타입을 오른쪽에서 왼쪽으로 읽으면 이해하기 쉽습니다.
const int* i1 = nullptr; // A pointer to an integer that is constant
int const* i2 = nullptr; // A pointer to a constant integer (same as i1)
const int* const i3 = nullptr; // A constant pointer to an integer that is constant
int const* const i4 = nullptr; // A constant pointer to a constant integer (same as i3)

// 가장 왼쪽에 있는 const는 바로 오른쪽 토큰에 적용되므로
// 아래의 예시는 'constant인 constant integer에 대한 pointer'가 됩니다...
const int const* i5 = nullptr; // Error: duplicate 'const'

// 포인터와 다르게 reference variable은 다른 대상을 가리키도록 바꿀 수 없습니다.
// 그러니까 레퍼런스 자체가 const하도록 붙이는건 의미가 없습니다 ('T& const').
// 실제로 'constant reference to ...'처럼 해석되는 타입은 선언할 수 없습니다.
int& const i6 = 0; // Error: 'const' qualifiers cannot be applied to 'int&'
const int& const i7 = 0; // Error: 'const' qualifiers cannot be applied to 'const int&'

// 그나저나 레퍼런스에 대한 포인터도 & const처럼 금지되어있더군요
int&* i8 = nullptr; // Error: cannot declare a pointer to 'int&'
```
'& const'에 대한 내용은 [stackoverflow 질문](https://stackoverflow.com/questions/54359088/const-qualifiers-cannot-be-applied-to-stdvectorlong-unsigned-int)에서 다루고 있습니다.<br>
이 [stackoverflow 질문](https://stackoverflow.com/questions/1898524/difference-between-pointer-to-a-reference-and-reference-to-a-pointer)은 '&*'를 더욱 자세히 설명해줍니다.

### <a name='tip26'></a>런타임 오버헤드 없는 다형성 (ft. CRTP)
```c++
#include <iostream>
#include <memory>

// 함수를 통한 다형성
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

// 템플릿과 상속을 이용한 다형성 흉내내기.
// 이런 방식의 상속을 CRTP (Curiously Recurring Template Pattern)라고 부릅니다.
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

// b가 가리키는 객체는 런타임에 결정됩니다.
// 어떤 foo를 사용할지 결정하기 위해 virtual function table을 참고합니다.
void test(const Base& b)
{
    b.foo();
}

// T가 컴파일 시간에 결정되기 때문에 vtable lookup같은 런타임 검사가 필요하지 않습니다.
// 이 방식의 단점은 이 트릭을 사용하는 모든 곳에서 템플릿을 써야 한다는 것입니다.
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

### <a name='tip3'></a>람다를 만드는 것은 operator()를 정의한 구조체를 만드는 것과 비슷합니다
```c++
#include <iostream>

struct Lambda
{
    int captureThis = 234; // 멤버 변수로 저장된 캡처된 값.
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

### <a name='tip32'></a>멤버 변수의 타입을 결정하는 법
```c++
#include <memory>

// T가 int같은 fundamental type이 아닌 경우
// 멤버 변수로 어떤 세부 타입을 사용해야 할지 결정해봅시다.
//
// 멤버 변수의 타입으로는 다음과 같은 다섯 가지를 가장 많이 사용합니다:
// - T
// - T*
// - T&
// - std::shared_ptr<T>
// - std::unique_ptr<T>
//
// 각각의 질문에 예/아니요로 답하면서 따라가보세요.
//
// 객체의 소유권이 뭔지 정확히 모르겠는 분들은
// 어떤 객체의 수명이 자신에게 종속적이라면 그 객체를 소유해야 한다고 가정해주세요.
//
// ex) 엔진은 자동차 없이는 의미 없기 때문에 자동차는 엔진을 소유해야 합니다.
//
// 반대로 객체를 참조할 때 그 객체가 항상 유효하다고 보장할 수 있으면 소유하지 않아도 됩니다.
//
// ex) Player::name()만 호출하는 함수는 Player 객체를 소유하지 않아도 됩니다.
//     적어도 함수 내부에서는 넘겨진 Player객체가 갑자기 사라지거나 파괴될 걱정이 없기 때문입니다.
//
// 물론 객체를 소유하지 않아도 되는 경우가 있지만, 그렇다고 소유해서는 안된다는건 아닙니다.
// 소유권을 갖는 것은 비용이 따르지만 장점도 있거든요.
// 예를 들어 dangling reference같은 문제를 아예 예방할 수 있습니다!
//
// 만약 null pointer exception같은 상황이 확실히 일어나지 않고
// 성능에도 신경써야 한다면 소유권이 없는 참조를 사용해보세요.
//
// 주의: 개인적인 가이드라인이라 모든 상황에 적절하지는 않을 수 있습니다.
template<typename T>
void member_type_guideline()
{
    // T 객체를 소유해야하나요?
    if (true)
    {
        // 다른 클래스에서 동일한 T 객체를 함께 소유해야하나요?
        if (true)
        {
            // 주의: 순환 참조가 일어나면 메모리 누수로 이어집니다. 이런 경우에는 std::weak_ptr를 사용해주세요.
            std::shared_ptr<T> member_variable;
        }
        else
        {
            // T가 polymorphic한가요?
            // 참고: 가상 함수가 있는 클래스를 polymorphic하다고 합니다.
            if (true)
            {
                // 이유: 다형성은 레퍼런스나 포인터로만 구현됩니다.
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
        // 복사 생성자나 복사 대입 연산자를 제공해야하나요?
        if (true)
        {
            // 이유: 레퍼런스 타입의 멤버 변수를 갖는 클래스는 제대로 복사할 수 없습니다 (레퍼런스 객체의 복사가 필연적으로 일어납니다!)
            T* member_variable;
        }
        else
        {
            // 멤버 변수에 새로운 값을 대입할 가능성이 있나요?
            if (true)
            {
                // 이유: 레퍼런스 타입은 다른 객체를 가리키도록 reassign할 수 없습니다 (무조건 복사)
                T* member_variable;
            }
            else
            {
                // 이 경우, 생성자에서 멤버 변수를 무조건 초기화해줘야하므로 생성자 파라미터에 T 객체를 넘겨받아야 할겁니다.
                T t;
                T& member_variable = t;
            }
        }
    }
}
```

### <a name='tip33'></a>```set(CMAKE_CXX_STANDARD ??)```는 꼭 ```project()``` 뒤에 나와야합니다
Bad
```cmake
cmake_minimum_required(VERSION 3.10)

# project() 전에 적으면 설정이 무시되더군요...?
# 왜 이게 적용이 안되는지 의문이었는데 한참 뒤에야 이걸 발견했네요
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
