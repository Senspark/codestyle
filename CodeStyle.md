
## C++
### Forward declarations

- Forward declarations (tiền khai báo) các lớp nếu có thể.

```
// Bad.
// MainScene.hpp
#include "MenuButton.hpp"

class MainScene {
public:
    MenuButton* button;
};
```

```
// OK.
// MainScene.hpp
class MenuButton;

class MainScene {
public:
    MenuButton* button;
};

// MenuScene.cpp
#include "MenuScene.hpp"
#include "MenuButton.hpp"

// ...
```

### Order of includes

- Ví dụ trong file: `MenuScene.cpp` thứ tự `include`:

 - `MainScene.hpp`
 - 1 dòng trống
 - Headers thư viện C/C++
 - 1 dòng trống
 - Headers của các thư viện khác
 - Headers trong project

```
#include "MainScene.hpp"

#include <math.h>
#include <cstdio>
#include <vector>

#include <cocos2d.h>
#include <EEHeader.hpp>

#include "GameScene.hpp"
#include "ShopScene.hpp"
```

### Namespace

- Không dùng `using namespace std`, `USING_NS_CC` hoặc tương tự trong các file headers (`.hpp`) và không khuyến khích dùng trong các file implementation (`.cpp`).

- Có thể sử dụng trong các hàm nếu cần thiết:

```
std::chrono::seconds MyClass::calculate() {
    using namespace std::chrono_literals;
    auto day = 24h;
    auto halfhour = 0.5h;
    return day + halfhour;
}
```

### Unnamed namespaces and static variables

- Khi khai báo hàm/biến trong file `.cpp` mà không có trong file `.hpp` thì đặt trong [`unnamed namespace`](http://en.cppreference.com/w/cpp/language/namespace#Unnamed_namespaces)(nên dùng) hoặc khai báo `static` để [`internal linkage`](http://en.cppreference.com/w/cpp/language/storage_duration#Linkage)

```
// Trong file ABC.cpp
// Bad
int myVariable;
int calculate() {
    return 123;
}
```

```
// ABC.cpp
// dùng unnamed namespace
namespace {
int myVariable;
int calculate() {
    return 123;
}
} // namespace

// dùng static.
static int myVariable;
static int calculate() {
    return 123;
}
```

### Class constructors

- Không được gọi các hàm `virtual` hoặc sử dụng `dynamic_cast` trong constructors [Stackoverflow](https://stackoverflow.com/questions/962132/calling-virtual-functions-inside-constructors)

- Luôn sử dụng `explicit` khi khai báo constructors có tham số:

```
class A {
public:
    A(); // OK
    A(int i); // Bad
    explicit A(int i); // OK
    explicit A(int i, float j); // OK
};
```

### Functions

- Tránh việc indent quá nhiều trong phần thân của các hàm:

```
// Bad
void process() {
    if (isBar()) {
        if (isFoo()) {
            if (isCloud()) {
                if (isSky()) {
                    // ...
                }
            }
        }
    }
}

// OK
void process() {
    if (!isBar()) {
        return;
    }
    if (!isFoor()) {
        return;
    }
    if (!isCloud()) {
        return;
    }
    if (!isSky()) {
        return;
    }
    // ...
}

// Bad.
void anotherProcess() {
    for (int i = 0; i < getSize(); ++i) {
        if (isValid(i)) {
            for (int j = 0; j < getSize(); ++j) {
                if (isValid(j)) {
                    doJob(i, j);
                    // ...
                }
            }
        }
    }
}

// OK.
void anotherProcess() {
    for (int i = 0; i < getSize(); ++i) {
        if (!isValid(i)) {
            continue;
        }
        for (int j = 0; j < getSize(); ++j) {
            if (!isValid(j)) {
                continue;
            }
            doJob(i, j);
            // ...
        }
    }
}

```

### Casting

- Không dùng kiểu cast cũ của `C`.
- Sử dụng `static_cast` cho các kiểu cơ bản của `C++` (`int`, `float`, ...)
- Dùng `reinterpret_cast` nếu biết mình đang làm gì và hiểu rõ được cơ chế [`alias`](http://en.cppreference.com/w/c/language/object#Strict_aliasing)
- Còn lại dùng `dynamic_cast`.

```
float x = 1;
int a1 = int(x); // Bad
int a2 = (int)x; // Bad
int a3 = static_cast<int>(x); // OK
```

```
struct A {};
struct B : A {};

void calculate(A* a) {
    auto b1 = (B*)a; // Bad
    auto b2 = static_cast<B*>(a); // Bad
    auto b2 = dynamic_cast<B*>(a); // OK    
}
```

### Preprocessor macros

- Tránh việc lạm dụng macro, chỉ sử dụng macro khi muốn biên dịch/không biên dịch một đoạn code hoặc những việc mà code không làm được:

```
// OK
#if CC_TARGET_PLATFORM == CC_PLATFORM_ANDROID
    do_something();
#else if CC_TARGET_PLATFORM == CC_PLATFORM_IOS
    do_something_else();
#endif

#define MY_SPEED 320 // Bad
constexpr float MY_SPEED = 320; // OK
```

### auto

- Sử dụng `auto` để tránh việc phải gõ tên những kiểu/lớp dài hoặc không cần thiết:

```
cocos2d::Sprite* sprite = cocos2d::Sprite::create(); // Bad
auto sprite = cocos2d::Sprite::create(); // Ok
```

### Enum class

- Sử dụng `enum class` thay cho `enum`:

```
// Bad
enum Indent {
    IndentLeft,
    IndentRight,
    IndentCenter
};

// OK
enum class Indent {
    Left,
    Right,
    Center,
};
```

### Common mistakes

##### Virtual destructor

- Trường hợp:

```
struct A {
    ~A() {}
};

struct B {
    ~B() {}
    // ...
};

int main() {
    A* x = new B();
    delete x; // destructor được khai báo trong B sẽ không được gọi, có thể dẫn đến leak bộ nhớ.
}
```

- Giải thích: khi gọi `delete x` thì chỉ có destructor trong A gọi: `x.~A()` vì `~A()` không được khai báo `virtual`.

- Cách sửa:

```
struct A {
    virtual ~A() {}
};
```

##### Global variables/functions:

- Trường hợp:

```
/// Utils.hpp
int sqr(int x) {
    return x * x;
}
```

```
/// A.cpp
#include "Utils.hpp"

int func1(int x) {
    return sqr(x);
}
```

```
/// B.cpp
#include "Utils.hpp"

int func2(int x) {
    return sqr(x);
}
```

Khi link xảy ra lỗi:

```
duplicate symbol ... in: ...
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

- Cách sửa:

```
/// Không khuyến khích
/// Utils.hpp
inline int sqr(int x) {
    return x * x;
}
```

## Cocos2d-x

### Namespace

- Các lớp trong project phải nằm trong một namespace chung:

```
// GameScene.hpp

// Bad
class GameScene {
public:
    // ...
};

// OK
NS_GAME_BEGIN
class GameScene {
public:
    // ...
};
NS_GAME_END

// NS_GAME_BEGIN và NS_GAME_END được define sẵn
```