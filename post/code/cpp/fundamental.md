---
mathjax: true
title: C++ 基础
date: 2017-12-23 20:40:43
tags: C++
categories: 编程
---

### new delete
在C++ 中可以使用关键词 new 和 delete 进行动态内存的管理。

```cpp
// 几种方法都可以
int *p1 = new int(3);
int *p2 = new int[2];
int *p3 = new int;
int *points = new Point[4];
```

```cpp
// 对于对象数组来说
delete points // 代表用来释放内存，且只用来释放points指向的内存。
delete[] points // 逐一调用数组中每个对象的析构函数。

// 但对于内置类型来说，下面两个效果是一样的。
delete p2
delete[] p2
```

### 引用
#### 引用作为函数形参
联想到 c 语言的swap 函数

<!--more-->

```c
void swap(int a, int b) {
    int t = a;
    a = b;
    b = t;
}

void main() {
    int a = 2, b = 3;
    swap(a, b);
}

```

这个函数是不会真正交换 main 函数中的变量 a, b的值的。如果要实现交换值就要使用指针。

```c
void swap_x(int *a, int *b) {
    int t = *a;
    *a = *b;
    *b = a;
}

void main() {
    int a = 2, b = 3;
    swap(&a, &b);
}
```

但是也发现了仅仅实现这么简单的功能也要使用指针，太麻烦了。
在C++ 中利用引用可以解决这个问题。

```cpp
void swap(int &a, int &b) {
    int t = a;
    a = b;
    b = t;
}

int main() {
    int a = 2, b = 3;
    swap(a, b);
}
```
这就是引用的最简单的应用。

##### 引用作为函数返回值
当函数返回值是引用时，函数也就能够作为左值。
具体看代码：

```cpp
const int maxn = 1000;
int a[maxn]; // 为了简化代码直接使用了全局变量

int& at(int i) {
    return a[i];
}

// 一个错误的示例
int at_wrong(int i) {
    return a[i];
}

void test() {
    at(0) = 1;
    at(1) = 6;
    at(2) = 4;
    at(3) = 8;
    at(4) = 8;

    // 下面一行代码会报错。因为 int 是不能作为左值的。
    at_wrong(5) = 1;
}
```

引用作为函数返回值这个 很重要。比如：自定义一个数据结构的时候，
需要提供接口，用来修改数据。比如 标准库中的 `vector`

下面举一个自定义的 类来说明这个特性的重要性。

```cpp
const unsigned maxn = 10000;

template<class T>
class my_vector {
    T a[maxn];
    unsigned cur = 0;
public:
    void add(T item) {
        a[cur++] = item;
    }

    T& operator [] (int i) {
        return a[i];
    }
};

int main() {
    my_vector<int> a;
    a.add(1);
    a.add(6);
    a.add(4);
    a.add(8);
    a.add(8);
    a.add(3);

    a[6] = 1;
}

```

重载了 `[]` 运算符。提供和数组类似的数据访问方式。

### const 修饰符
#### 对于指针的 const
对于指针的 const 有一些特殊的地方。下面做一些讨论。
注意：对于引用，并没有这种不同，两种定义的都是，底层 const。

```cpp
int a = 2;
int b = 3;

const int *p1 = &a;
// 底层（low-level） const
// 不允许指针改变变量的值
// *p1 = 2;
p1 = &b; // 允许

int * const p2 = &a;
// 顶层（top-level） const
// 不允许改变指针的值
// p2 = &b;
*p2 = 3; // 允许

const int * const p3 = &a;
// 都不允许
// p3 = &b;
// *p3 = 3;
```

#### 对结构体（类）的引用

```cpp
struct Point {
    int x, y;
};

// 加上const 修饰符保护数据
void test(const Point &point) {
    point.x = 3; // 错误
}

// 重点：这种情况下会发生重载。
// 具体见下一节
void test(Point &point) {
    point.x = 3; // 正确
}
```
### 函数特性
#### 内联函数
利用内联函数，相对于正常的函数来说，一个优点就是执行效率会有所提升。
将指定的函数体插入并取代每一处调用该函数的地方。这样会减少调用函数（我认为是 压栈出栈所消耗的时间）的时间吧。

注意：对递归函数的内联扩展可能引起部分编译器的无穷编译。

```cpp
int max(int x, int y) {
    return x > y ? x : y;
}

// 使用内联函数的 max 函数
inline max_x(int x, int y) {
    return x > y ? x : y;
}
```

#### 函数重载
重载的条件：同名函数的形式参数（指参数的个数、类型或者顺序）不同。

利用一个函数名去执行不同的功能。

```cpp
// add 能够加 两个或者三个数。
// 形式参数个数不同
void add(int a, int b, int c);
void add(int a, int b);
```

请注意下面一种情况：

下面程序表明 `const Point &` 这个类型和 `Point &` 是不同的。

```cpp
struct Point {
    int x, y;
};

// 发生函数重载。const 关键字修饰
void show(const Point &point) {
    cout << "const: " << point.x << "," << point.y << endl;
}

// 调用后将 x 坐标变为 0
void show(Point &point) {
    cout << "not const: " << point.x << "," << point.y << endl;
    point.x = 0;
}

int main() {
    Point point1 = {1, 2};
    const Point point2 = {2, 3};
    show(point1);
    show(point1);
    show(point2);
}
```

程序运行的结果是。

```
not const: 1,2
not const: 0,2
const: 2,3
```

#### 函数参数默认值
使用函数默认值，调用的时候就可以简化传参，有时候也避免了再写一个函数去重载。

```cpp
// 计算两个数，a ope b。ope 默认为 +
void compute(int a, int b, char ope = '+');

void compute(int a, int b, char ope) {
    ...
}
```

注意：函数默认值应该在声明的时候写上去。

具有默认值的应该在函数形参列表的最右边。

```cpp
void add(int a = 0, int b); // 这样子是错的
```

[1] Lippman S B. C++ Primer[M]. Pearson Education India, 2005.

[2] [new_and_delete](http://www.cnblogs.com/hazir/p/new_and_delete.html)
