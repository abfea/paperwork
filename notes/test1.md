# Test

some text here

Gnome 插入鼠标抽动关闭触控板

```shell
gsettings set org.gnome.desktop.peripherals.touchpad send-events disabled-on-external-mouse
gsettings set org.gnome.desktop.peripherals.touchpad send-events enabled
```

Themes and fonts

```shell
gsettings set org.gnome.desktop.interface gtk-theme    'Yaru'
gsettings set org.gnome.desktop.interface icon-theme   'Yaru'
gsettings set org.gnome.desktop.interface cursor-theme 'Yaru'
gsettings set org.gnome.desktop.interface font-name    'Your font name'
gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'
```

开启Gnome super+mouse_right 缩放窗口

```shell
gsettings set org.gnome.desktop.wm.preferences resize-with-right-button true
```

Gnome app 窗口按键

```shell
gsettings set  org.gnome.desktop.wm.preferences button-layout 'maximize,minimize,close:'

# My prefer
gsettings set  org.gnome.desktop.wm.preferences button-layout ':minimize,close'
gsettings set  org.gnome.desktop.wm.preferences button-layout ':close'

# 按键居左
gsettings set  org.gnome.desktop.wm.preferences button-layout 'close,minimize,maximize:'
```

Nautilus 关闭最近文件

```shell
gsettings set org.gnome.desktop.privacy remember-recent-files false
```


## 2.使用const, enum, inline 替换 `#define`

Prefer consts, enums, and inline to #define

```cpp
#define ASPECT_RATIO 1.653
// to
const double AspectRatio = 1.653;

const char* const authorName = "Scott Meyers";
const std::string authorName("ScottMeyers");

// 将常量限定在类作用域中

class GamePayer {
private:
   staic const int NumTurns = 5; // 常量声明式
   int scores[NumTurns];
   // the enum hack
   enum {AnotherNumTurns = 5};
   int AnotherScores[AnotherNumTurns];
   ...
}

const int GamePayer::NumTures; // 常量定义式
// 定义式放在实现文件中而非头文件，定义式无需给定初值，生命式已经给定初值

/*
   the num hack 更像#define 而不像const，
   例如， const 取地址是合法的，但是define不合法，
   同样，TNH也是不合法的。
*/

// 函数形态的宏
#define CALL_WITH_MAX(a, b) f((a) > (b) ?　(a): (b))
int a = 5;
int b = 6;
CALL_WITH_MAX(++a, b); // a 被累加两次
CALL_WITH_MAX(++a, b+10); // a 被累加一次

template<typename T>
inline void callWithMax(const T& a, const T& b) {
    f(a > b ? a : B);
}

```

* 最好用const对象或enums替代define
* 对于形似函数的宏(macros)，最好改用inline替代

## 3.尽可能的使用const

在const和non-const函数中避免重复

```cpp
class textBlock {
public:
    ...
    const char& operator[](std::size_t position) const {
        ... // 边界检验(bounds checking)
        ... // 记录数据访问(log access)
        ... // 检验数据完整性(verify data integrity)
        retrun text[position];
    }
    char& operator[](std::size_t position) {
    /*
        ...
        ...
        ...
        return text[position];
    */
        return
            const_cast<char&>(
                static_cast<const TextBlock&>(*this)[position]
            );
    /*
        第一次转型为*this添加const, 使接下来调用const版本operator[]
        第二次则是从const operator[]返回值剔除const
    */
    }
}
```

## 7.为多态(polyphonic)基类生命virtual析构函数

* 当derived class对象经由一个base class的指针被删除，
而该base class带着一个non-virtual析构函数，
其结果是未定义的。

* 当class不含virtual函数，通常表示它并不意图被用作一个基类。
这时，令构析函数为virtual往往是个馊主意。

## 8.别让异常逃离析构函数

Prevent exceptions from leaving destructors

* 析构函数绝对不要吐出异常。
如果一个别析构函数调用的函数可能抛出异常，
析构函数应该捕获异常，然后吞下它们（不传播）
或结束程序。

* 如果客户需要对某个操作函数运行期间抛出异常作出反应，
那么class应该提供一个普通函数，而非再构析函数中执行该操作。

## 9.绝不再构造和析构函数过程中调用virtual函数

Never call vitrual functions during construction or description.

在derived class 对象的base class 构造期间，对象类型是base class。
不只virtual函数会被编译器解析至(resolve to )base class

