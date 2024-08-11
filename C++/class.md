# 类

[TOC]

## 访问权限

在 C++ 中，public、private 和 protected 是用来定义类成员的访问权限的关键字，它们分别控制类成员的可见性和可访问性。正确地使用这些访问控制关键字对于封装和继承——面向对象编程的核心概念——至关重要。

**public**

- 定义：public 成员在任何地方都是可访问的。
- 应用：通常用来定义接口的一部分，即那些可以被外部代码访问和使用的函数和数据成员。例如，类的构造函数、析构函数和大多数成员函数（方法）都是公开的。

**private**

- 定义：private 成员只能被同一类中的函数访问。
- 应用：用于隐藏类的实现细节和内部状态。私有成员通常是只在类内部使用，不希望外部访问的数据成员和辅助函数。

**protected**

- 定义：protected 成员可以被该类及其派生类（子类）中的函数访问。
- 应用：主要用于当类被其他类继承时，这些成员需要对子类可见，但对其他外部类不可见。适合用于那些只有在继承链中才需要访问的成员。

```c++
class Rectangle {
private:
    double width, height;  // 只能被 Rectangle 类的方法访问

public:
    Rectangle(double w, double h) : width(w), height(h) {}  // 构造函数

    double area() const {  // 可以被任何人调用来获取矩形的面积
        return width * height;
    }

protected:
    void scale(double factor) {  // 可以被此类或继承此类的子类调用
        width *= factor;
        height *= factor;
    }
};

class Square : public Rectangle {
public:
    Square(double side) : Rectangle(side, side) {}

    void doubleSize() {
        scale(2);  // 调用了受保护的成员函数
    }
};
```

## 构造函数

### 基础

在 C++ 中，构造函数是一种特殊的类成员函数，**用于创建对象时初始化类的实例**。构造函数的特点包括：

- **构造函数的名称必须与类名完全相同**。
- **构造函数没有返回类型，也不返回任何值**。
- 构造函数可以有参数，允许在创建对象时传递值或数据。
- 可以有多个构造函数，每个构造函数有不同的参数列表（这称为构造函数重载）。
- 如果未显式定义任何构造函数，编译器将提供一个默认构造函数。

构造函数的类型

- **默认构造函数**：不带任何参数，或者每个参数都有默认值。
- **带参数的构造函数**：允许传递一个或多个参数，用于更灵活地初始化对象。
- **拷贝构造函数**：用一个同类型的对象来初始化另一个新对象。
- **移动构造函数**（C++11 及以后）：用于通过转移资源而不是拷贝资源来构造一个新对象，通常用于提高性能。

假设我们有一个 Point 类，代表二维空间中的点：

```c++
class Point {
public:
    int x, y;

    // 默认构造函数
    Point() : x(0), y(0) {}

    // 带参数的构造函数
    Point(int x, int y) : x(x), y(y) {}

    // 拷贝构造函数
    Point(const Point& other) : x(other.x), y(other.y) {}

    // 成员函数
    void move(int dx, int dy) {
        x += dx;
        y += dy;
    }
};
```

### 初始化列表

在上述示例中，构造函数使用了**初始化列表**（即冒号后跟的部分）来直接初始化成员变量。这种方式比在构造函数体中赋值更有效率，特别是对于复杂类型的成员变量。

### 移动构造函数

在 C++11 中引入了移动语义，它**允许资源（如动态分配的内存）从一个对象转移到另一个对象（减少了拷贝操作），这可以大幅提高某些操作的性能**。移动构造函数是实现移动语义的关键。

移动构造函数的特点：
- 它通常接受一个右值引用（Type&&）到同一类型的对象。
- 它应该将源对象的资源转移到新对象，然后将源对象置于一个有效但未定义的状态。
- 移动构造函数必须确保源对象的析构不会对新对象的状态造成影响。

如何定义移动构造函数

在之前的 Point 类例子中，移动构造函数实际上和拷贝构造函数的效果是一样的，因为 Point 类只包含基本类型的成员（int）。但为了展示如何定义一个移动构造函数，假设 Point 类中有一个动态分配的数组，我们就可以看到移动构造函数的实际用途：

```c++
class Point {
public:
    int *coords;
    int size;

    // 默认构造函数
    Point() : coords(nullptr), size(0) {}

    // 带参数的构造函数
    Point(int sz) : size(sz) {
        coords = new int[sz];
    }

    // 拷贝构造函数
    Point(const Point& other) : size(other.size) {
        coords = new int[size];
        std::copy(other.coords, other.coords + size, coords);
    }

    // 移动构造函数
    Point(Point&& other) : coords(other.coords), size(other.size) {
        other.coords = nullptr;  // 避免析构时删除内存
        other.size = 0;
    }

    // 析构函数
    ~Point() {
        delete[] coords;
    }

    // 成员函数
    void move(int dx, int dy) {
        if (size >= 2) {
            coords[0] += dx;
            coords[1] += dy;
        }
    }
};
```

说明：

移动构造函数使用了初始化列表来接管 other 对象的资源（在本例中是 coords 指针和 size），然后将 other 的指针设置为 nullptr 和大小设置为 0。这样做确保了当 other 的析构函数被调用时，它不会删除已经转移给新对象的内存。

这种方式可以大大提高性能，尤其是在涉及大型数据结构和容器类时，因为它避免了不必要的数据复制。

### explicit

在 C++ 中，explicit 关键字主要用于构造函数，用来防止隐式类型转换。它用于修饰单参数构造函数，以避免编译器在某些上下文中自动进行类型转换。具体来说，explicit 的作用是：

- **防止隐式转换**：当一个构造函数被声明为 explicit 时，该构造函数不能用于隐式转换。例如，当一个对象被赋值或传递给函数时，如果类型不匹配，编译器不会自动调用该构造函数进行转换。

- **增强代码的可读性和安全性**：使用 explicit 可以减少意外的类型转换错误，使得代码更清晰，因为所有的类型转换都是显式的。

以下是一个例子，说明 explicit 的作用：

在这个例子中，如果 MyClass 的构造函数不是 explicit 的，那么你可以通过 printValue(20); 调用函数，此时编译器会隐式地将 20 转换为 MyClass 对象。但是，由于构造函数被声明为 explicit，所以必须显式地创建 MyClass 对象。

```c++
#include <iostream>

class MyClass {
public:
    explicit MyClass(int x) {
        value = x;
    }

    int getValue() const {
        return value;
    }

private:
    int value;
};

void printValue(const MyClass& obj) {
    std::cout << "Value: " << obj.getValue() << std::endl;
}

int main() {
    MyClass obj1(10); // 正确：显式调用构造函数

    printValue(obj1); // 正确：传递 MyClass 对象

    // printValue(20); // 错误：无法进行隐式转换，因为构造函数是 explicit 的

    return 0;
}
```

总之，explicit 关键字**用于单参数的构造函数，避免意外的隐式转换，增强代码的安全性和可读性**

### Static

静态成员函数的使用场景

- 工具函数：可以在不需要对象的情况下，可以使用类的功能
- 全局状态或配置管理：通过静态成员函数访问静态成员变量来管理全局状态
- 工厂方法模式：创建和返回类的实例

## 析构函数

在 C++ 中，析构函数是一种特殊的类成员函数，用于在对象的生命周期结束时进行清理工作。析构函数的作用是**释放对象可能持有的资源，如动态分配的内存、文件句柄、网络连接等，以防止资源泄漏**。

析构函数的特点：
- **析构函数的名称是在类名前加一个波浪号（~）符号**。
- **析构函数不接受任何参数，也不返回任何值**。
- 每个类只能有一个析构函数。
- 如果没有为类显式定义析构函数，编译器将自动生成一个默认析构函数。默认析构函数什么也不做，仅调用成员和基类的析构函数。

如何定义析构函数

假设我们有一个 Point 类，我们已经在其中使用了动态分配的内存。我们将定义一个析构函数来确保这些资源被适当释放：

```c++
class Point {
public:
    int *coords;
    int size;

    // 默认构造函数
    Point() : coords(nullptr), size(0) {}

    // 带参数的构造函数
    Point(int sz) : size(sz) {
        coords = new int[sz];
    }

    // 拷贝构造函数
    Point(const Point& other) : size(other.size) {
        coords = new int[size];
        std::copy(other.coords, other.coords + size, coords);
    }

    // 移动构造函数
    Point(Point&& other) : coords(other.coords), size(other.size) {
        other.coords = nullptr;  // 避免析构时删除内存
        other.size = 0;
    }

    // 析构函数
    ~Point() {
        delete[] coords;  // 释放动态分配的内存
    }

    // 成员函数
    void move(int dx, int dy) {
        if (size >= 2) {
            coords[0] += dx;
            coords[1] += dy;
        }
    }
};
```

析构函数的调用时机
析构函数在对象生命周期结束时自动调用，这可能是由于：

- 对象作为局部变量离开其作用域时。
- 对象是通过 new 表达式动态创建的，并通过 delete 表达式删除时。
- 对象是容器的一部分，并且容器被销毁或从容器中删除对象时。
- 对象是通过智能指针管理，并且智能指针最终没有引用指向对象时。

使用析构函数是确保资源正确管理的关键部分，是良好的资源管理和防止内存泄漏的基本实践。

## 虚析构函数

在 C++ 中，当你在类层次结构中使用继承时，将析构函数声明为 virtual 是非常重要的，尤其是**当你打算通过基类指针删除一个派生类对象时。声明析构函数为 virtual 确保了对象的正确析构，即使是通过基类指针来进行的**。

为什么需要 virtual 析构函数
当一个基类的析构函数被声明为 virtual 后，C++ 运行时将确保无论通过何种类的指针（基类或派生类）来删除对象，都将调用正确的析构函数。这是通过动态绑定（多态）来实现的。

- **防止资源泄漏**：如果析构函数不是 virtual，那么删除通过基类指针指向的派生类对象时，只会调用基类的析构函数，而不会调用派生类的析构函数。这可能会导致派生类分配的资源（如动态内存）未被释放，从而导致内存泄漏。
- **确保正确的析构顺序**：通过使析构函数为 virtual，可以保证析构顺序是从派生类到基类，这对于资源管理非常重要。

示例

假设我们有一个基类 Shape 和一个派生类 Circle，Circle 类可能会有额外的资源需要管理（比如动态分配的内存）。

```c++
class Shape {
public:
    Shape() { std::cout << "Shape created." << std::endl; }
    virtual ~Shape() { std::cout << "Shape destroyed." << std::endl; }
};

class Circle : public Shape {
public:
    int* circleData; // 动态分配的资源

    Circle() {
        circleData = new int[100]; // 分配资源
        std::cout << "Circle created." << std::endl;
    }
    ~Circle() {
        delete[] circleData; // 释放资源
        std::cout << "Circle destroyed." << std::endl;
    }
};

void deleteShape(Shape* s) {
    delete s;
}
```

在这个示例中，如果你通过 Shape 类型的指针来删除一个 Circle 对象，如下所示：

```c++
Shape* s = new Circle();
deleteShape(s);
```

因为 Shape 的析构函数是 virtual 的，这将确保首先调用 Circle 的析构函数，然后才是 Shape 的析构函数。如果 Shape 的析构函数不是 virtual，则只会调用 Shape 的析构函数，导致 Circle 类中分配的内存未被释放。

总结
总之，将析构函数声明为 virtual 是处理多态对象生命周期的一种安全和正确的方式。当你设计一个可能会被继承的类，并且通过基类指针管理派生类对象时，你应该始终声明一个虚析构函数。这保证了无论对象的实际类型如何，都可以正确地进行清理。

## 带 const 的成员函数 

在 C++ 中，如果在类成员函数的声明后面加上 const 关键字，这表明该成员函数是一个常量成员函数。这意味着该函数不会修改其所属对象的任何数据成员（不包括被声明为 mutable 的成员）。

常量成员函数的特点和用途：

- **保证不修改对象状态**：在函数声明的末尾加上 const 关键字，这表明该函数不会改变对象的任何成员变量的状态（除了 mutable 成员）。这是通过编译时检查实现的，如果函数试图修改任何成员（除了 mutable 标记的成员），编译器将报错。

- **增加代码的安全性**：通过声明常量成员函数，你可以确保当函数被调用时不会意外地改变对象状态，这对于调试和维护大型程序非常有用。

- **允许在常量对象上调用**：只有常量成员函数可以被常量对象或常量对象引用/指针调用。这允许你在保证不会修改对象状态的情况下，从常量上下文中访问对象数据。

- **重载成员函数**：你可以根据函数是否是 const 来重载成员函数。这意味着你可以有两个相同名字和参数的函数，但一个是 const，另一个不是，编译器会根据对象是否为常量来选择调用哪一个版本。

```c++
class Point {
public:
    int x, y;

    Point(int x = 0, int y = 0) : x(x), y(y) {}

    // 常量成员函数，不修改任何成员变量
    int getX() const { return x; }
    int getY() const { return y; }

    // 非常量成员函数，允许修改成员变量
    void setX(int newX) { x = newX; }
    void setY(int newY) { y = newY; }

    // 一个重载的函数示例
    void print() const { // 用于常量对象
        std::cout << "Const Point: (" << x << ", " << y << ")" << std::endl;
    }

    void print() { // 用于非常量对象
        std::cout << "Mutable Point: (" << x << ", " << y << ")" << std::endl;
    }
};

void printPoint(const Point& p) {
    p.print(); // 将调用 const 版本
}

int main() {
    Point p1(10, 20);
    const Point p2(5, 5);

    p1.print(); // 调用非 const 版本
    p2.print(); // 调用 const 版本

    // p2.setX(10); // 编译错误，因为 p2 是常量对象
}
```

## 带 virtual 的成员函数

在 C++ 中，使用 virtual 关键字修饰类的成员函数是实现多态性的一种方式。当一个成员函数在基类中被声明为 virtual 时，**它可以在任何派生类中被重写（override），允许在运行时动态选择适当的函数版本进行调用**，这种机制称为动态绑定或晚绑定。

virtual 关键字的特点和用途：

- 实现多态：通过 virtual 关键字，基类可以定义一个接口，派生类可以根据自己的需求来实现或修改这个接口的行为。
- 运行时绑定：virtual 函数的调用不是在编译时决定的，而是在运行时。这意味着调用哪个函数版本是基于对象的实际类型决定的，而不仅仅是指针或引用的类型。
- 支持派生类的扩展性：派生类可以选择覆盖（override）基类中的 virtual 函数以提供特定的功能，而不改变基类接口。

使用 virtual 的示例：

假设我们有一个表示图形的基类 Shape 和两个派生类 Circle 和 Rectangle。我们希望每个图形都能计算其面积，但具体的计算方法取决于图形的类型。

```c++
class Shape {
public:
    virtual double area() const = 0;  // 纯虚函数，使得 Shape 成为抽象基类

    virtual ~Shape() {}  // 虚析构函数，确保派生类的析构函数被调用
};

class Circle : public Shape {
private:
    double radius;

public:
    Circle(double r) : radius(r) {}

    double area() const override {  // 重写基类的虚函数
        return 3.14159 * radius * radius;
    }
};

class Rectangle : public Shape {
private:
    double width, height;

public:
    Rectangle(double w, double h) : width(w), height(h) {}

    double area() const override {  // 重写基类的虚函数
        return width * height;
    }
};

void printArea(const Shape& shape) {
    std::cout << "Area: " << shape.area() << std::endl;
}

int main() {
    Circle c(5);
    Rectangle r(4, 5);

    printArea(c);
    printArea(r);
}
```

说明：
- 纯虚函数：在基类 Shape 中，area() 函数被声明为纯虚函数（virtual double area() const = 0;）。这意味着 Shape 是一个抽象基类，不能被实例化，任何派生自 Shape 的类都必须实现 area() 函数。
- 虚析构函数：提供一个虚析构函数是重要的，这样当通过基类指针删除派生类对象时，派生类的析构函数也会被调用，从而避免资源泄漏。

通过将成员函数声明为 virtual，可以使类的行为更加灵活和动态，**支持多种类型的对象以统一的方式进行操作**，这是实现多态的关键。

## 派生类

在 C++ 中，派生类的构造函数和析构函数需要特别注意与基类的关系，尤其是如何正确地调用基类的构造函数和确保基类的资源被适当释放。下面将分别解释派生类构造函数和析构函数的写法，并提供示例。

### 派生类构造函数

派生类的构造函数在执行时，**首先需要调用基类的构造函数，然后才是派生类自己的成员的初始化**。

如果没有明确指定，将调用基类的默认构造函数。

如果基类没有默认构造函数，或者你需要调用一个带参数的基类构造函数，你必须在派生类构造函数的初始化列表中显式地调用它。

示例：
假设有一个 Vehicle 基类和一个 Car 派生类，如下所示：

```c++
class Vehicle {
public:
    Vehicle() {
        cout << "Vehicle's constructor called" << endl;
    }

    Vehicle(int maxSpeed) {
        cout << "Vehicle's constructor called with maxSpeed " << maxSpeed << endl;
    }

    virtual ~Vehicle() {
        cout << "Vehicle's destructor called" << endl;
    }
};

class Car : public Vehicle {
private:
    int numDoors;

public:
    // 调用 Vehicle 的默认构造函数
    Car() : Vehicle(), numDoors(4) {
        cout << "Car's constructor called" << endl;
    }

    // 调用 Vehicle 的带参数的构造函数
    Car(int maxSpeed, int numDoors) : Vehicle(maxSpeed), numDoors(numDoors) {
        cout << "Car's constructor called with maxSpeed " << maxSpeed << " and numDoors " << numDoors << endl;
    }

    ~Car() {
        cout << "Car's destructor called" << endl;
    }
};
```

### 派生类析构函数

析构函数在类的对象被销毁时自动调用，用于清理资源。在派生类中，析构函数的调用顺序与构造函数相反。**当派生类的对象开始销毁时，首先执行派生类的析构函数，然后自动调用基类的析构函数**。

对于派生类析构函数，**通常不需要在析构函数体中显式调用基类的析构函数**，因为这一调用会自动进行。

示例中的析构函数：

```c++
~Car() {
    cout << "Car's destructor called" << endl;
    // 自动调用 Vehicle 的析构函数
}
```

重要注意点：

- 如果你打算通过基类指针来删除派生类对象，基类的析构函数应该是 virtual 的。这样可以确保调用正确的析构函数，从而避免资源泄漏。
- **构造函数中不能调用虚函数，因为这样做不会调用派生类中重写的版本**。如果需要在构造或析构过程中调用某些函数，这些函数不应该是虚函数。

总的来说，正确管理派生类和基类的构造函数和析构函数，是 C++ 中面向对象设计的一个关键部分，对于保证资源正确管理及避免内存泄漏非常重要。

## Static 函数

static const Entity * getEntity( const RangeListNode * pNode )
	{
		MF_ASSERT( pNode->isEntity() );
		return static_cast< const EntityRangeListNode * >( pNode )->getEntity();
	}