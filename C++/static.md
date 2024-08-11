
# static

## 函数内

在 C++ 中，在函数内部定义一个 static 变量意味着这个变量只会被初始化一次，并且它的生命周期会延续到程序结束。这类变量通常用来保持其在函数调用之间的状态不变。以下是几个关键点：

- **初始化时机**：static 变量在它第一次被使用时初始化，而不是每次函数被调用时。初始化只执行一次。

- **默认值**：如果在初始化时没有明确赋值，static 变量会被自动初始化为零。对于基本类型如 int，static int x; 将自动初始化为 0。对于类类型，将调用默认构造函数进行初始化。

- **存储位置**：虽然 static 变量在函数内定义，但它实际上并不存储在栈上，而是存储在程序的全局/静态存储区域中。这意味着它们不会随着函数调用的结束而被销毁。

- **作用域**：static 变量的作用域限定在它被声明的函数内部，但它的存在期限是整个程序的生命周期。

- **用途**：这种变量常用于实现依赖于之前函数调用状态的功能，如生成唯一的ID、计数函数调用次数、或实现设计模式（例如单例模式）。

举一个简单的例子，如果你想在函数中计算该函数被调用了多少次：

```c++
#include <iostream>

void countCalls() {
    static int count = 0;  // 只初始化一次
    count++;
    std::cout << "Function has been called " << count << " times." << std::endl;
}

int main() {
    countCalls(); // 输出: Function has been called 1 times.
    countCalls(); // 输出: Function has been called 2 times.
    countCalls(); // 输出: Function has been called 3 times.
    return 0;
}
```

在这个例子中，每次调用 countCalls 时，count 变量都会增加，但不会重新初始化。这是 static 变量的典型用途。