---
title: Modern C++ Design 笔记
date: 2023-03-03 01:55:00 -0500
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

# 第一章 策略类

整个这一章讲的都是策略类的使用。这一部分分散在模板笔记当中。包括设计模式目前没有整理，后续会整理。

核心要点就是让类可以定制化。当把一个类拆分为多个策略的时候，首先要把设计的功能模块抽离。同时要注意寻找正交的策略，也就是彼此之间无交互，可以独立更改的策略。



