---
title: Effective C++ 笔记
date: 2023-02-25 01:55:00 -0500
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

# 条款25 swap函数的实现方式

具体原因没啥好说的，都能理解。概括一下使用方式。

- 当标准库的`std::swap`对某种自定义类型效率很低的时候，提供一个`swap`成员函数，并且确定该函数不抛出异常。
  - 具体自定义类应该使用pimpl手法，也就是保有资源指针。所以成员`swap`仅仅需要交换指针。
- 由于通常来说，我们的保有资源指针是`private`的，所以应该再提供一个非成员函数的`swap`，让这个`swap`调用那个非成员函数的`swap`。
- 对于非模板类类型，可以考虑特化`std::swap`
- 针对类模板的非成员`swap`函数，应该使用`using std::swap`引入标准库的`swap`函数。
  - 这样在`T`类有合适的swap的时候，自然会使用`T`类自己的`swap`。如果`T`类没有自己的`swap`函数，则会匹配至`std::swap`标准库的`swap`函数。
- 最后，调用`swap`的时候不要加任何命名空间修饰符。



