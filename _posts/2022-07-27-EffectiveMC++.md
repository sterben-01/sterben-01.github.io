---
title: Effective Modern C++ 笔记
date: 2022-07-20 01:55:00 -0500
categories: [笔记]
tags: [C++]
pin: false
author: 01

toc: true
comments: true
typora-root-url: ../../Sterben-01.github.io
math: false
mermaid: true
  
---

# Effective Modern C++ 笔记

# 条款18需要显式所有权的资源管理时，用`std::unique_ptr`



















# 万能引用，右值引用，完美转发等等在杂记1.有时间会整理至此

# 条款26 避免重载万能引用

**2023.2.21：直接看模板的6.2**

第一次看书给我看乐了。核心就一句话。根据C++的重载决议规则，万能引用版本总会被优先匹配。万能引用很jb贪。。它们会在具现过程中，和几乎任何实参型别都会产生精确匹配。而且在重载过程当中，万能引用模板还会和构造函数拷贝构造函数竞争。例子不写了，看书吧。反正别重载万能引用就行。

**要点:**

- 把万能引用作为重载候选型别，几乎总会让该重载版本在始料未及的情况下被调用到
- （完美转发）构造函数(使用了万能引用）的问题尤其严重，因为对于非常量的左值型别而言，它们一般都会形成相对于复制构造函数的更佳匹配，并且它们还会劫持子类中对父类的复制和移动构造函数的调用

# 条款27：26的解决方案

## 1. 使用常量左值引用做为形参（`const &`)

常量左值引用可以接受任意类型的参数。（常量左值，左值，常量右值，右值）。虽然效率低一些，但是可以正确使用。

## 2. 传值

有些人忽略一点。以值传递是可以接受右值的

```c++
void num(int a){
    cout << a << endl;
}
int main(){
    int digit = 5;
    num(digit);	//	OK 
    num(800); 	//	OK
    return 0;
}
```

所以：

```c++
class Person {
public:
    explicit Person(std::string n) //值传递
    : name(std::move(n)) {}

    explicit Person(int idx)
    : name(nameFromIdx(idx)) {}
    ...
private:
    std::string name;
};

```

没有效率损失的原因：如果实参是左值，那么实参到形参是一次复制，形参到`name`是一次移动，相比普适引用只多了一次移动；如果实参是右值，那么实参到形参是一次移动，形参到`name`还是一次移动，相比普适引用还是只多一次移动，可以认为没有效率损失。

## 3.使用标签。

**2023.2.21：直接看模板的标记派发。**

这里很有意思。侯捷老师讲STL的时候提到过，当时没有理解。现在有点理解了。

我们重新实现` logAndAdd`把它委托给另外两个函数，**一个接受整型值，另一个接受其他所有型别**。而` logAndAdd` 本身则接受所有型别的实参，无论整型和非整型都来者不拒。

改动前的原始版本：

```c++
std::multiset<std::string> names;
template <typename T>
void logAndAdd(T&& name) {
    names.emplace(std::forward<T>(name)); //具体代码细节无须在意
    //...
}
```
接近实现正确的版本：

```c++
template <typename T>
void logAndAdd(T&& name) {
    logAndAddImpl(std::forward<T>(name), 
                  std::is_integral<T>()//检查T的类型是否为整型
    );
}
```

上面的问题是：当实参是左值时，`T`会被推导为左值引用，即如果实参类型是`int`，那么`T`就是`int&`，（杂记的完美转发推导有写）`std::is_integral<T>()`就会返回false（此函数判断是否为整型。但是**所有的引用型别都不是整型**）。**这里我们需要把`T`可能的引用性去掉**：

```c++
template <typename T>
void logAndAdd(T&& name) {
    logAndAddImpl(
        std::forward<T>(name),
        std::is_integral<typename std::remove_reference<T>::type>() //检查T的类型是否为整型
    );
}
```
然后`logAndAddImpl`提供两个特化版本：
```c++
template <typename T>
void logAndAddImpl(T&& name, std::false_type) { //如果不是整型
    names.emplace(std::forward<T>(name)); //具体代码细节无须在意
}

template <typename T>
void logAndAddImpl(T&& name, std::true_type) {	//如果是整型
    logAndAdd(nameFromIdx(idx)); //具体代码细节无须在意
}
```

为什么用`std::true_type`/`std::false_type`而不用`true/false`？前者是编译期值，后者是运行时值。

注意这里我们都没有给`logAndAddImpl`的第二个参数起名字，说明它就是一个Tag。这种方法常用于模板元编程。它们在运行期不起任何作用。
