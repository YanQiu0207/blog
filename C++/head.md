# 头文件

## Include Guard

Include Guard：防止头文件被多次包含

```c++
#ifndef HEADER_FILE_NAME_H
#define HEADER_FILE_NAME_H

// 头文件的内容

#endif
```

头文件可能被多个源文件包含时，编译器首次遇到头文件时，HEADER_FILE_NAME_H 尚未定义，所以 #ifndef HEADER_FILE_NAME_H 为真，编译器继续处理 #define HEADER_FILE_NAME_H 和头文件的其他内容。当头文件第二次被包含时，HEADER_FILE_NAME_H 已经定义了，#ifndef HEADER_FILE_NAME_H 为假，编译器跳过头文件的内容，从而避免了重复定义和潜在的编译错误。

头文件被多次包含可能会引起多种编译问题和逻辑错误，具体包括：

（1）重复定义：如果头文件中包含了函数定义或者变量定义（而非声明），多次包含头文件会导致同一符号在同一个编译单元中多次定义，这将违反 C++ 的 One Definition Rule（ODR），导致编译错误。

（2）编译效率降低：如果头文件被不必要地多次包含，它会增加编译器的工作量，因此会降低编译效率。

（3）命名冲突：多次包含可能导致命名空间污染，尤其是在大型项目中，不同的头文件可能不经意间使用了相同的宏定义或变量名，导致难以追踪的编译错误或行为异常。

（4）增加调试难度：在复杂的项目中，如果头文件包含顺序不当或重复包含，可能导致预处理器输出结果不符合预期，增加了代码维护和调试的难度。

## pragma once

与传统的 Include Guard 相比，它更简洁，因为你只需要在头文件的最顶部添加一行代码即可

```c++
#pragma once

// 头文件的内容
```

