---
title: More Effective C++ 笔记
date: 2023-02-27 01:55:00 -0500
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

# 基础议题
## 条款4：非必要不提供默认构造

这一点可能和我们之前的认知有差异。但是也是正确的。原因是在语义上或者是设计上，针对某一些类型，如果我们不能提供一个初值来初始化一个对象，那么通过默认构造实例化出来的这个对象会是无意义的。

当然了，还有很多类型是允许有默认对象的，比如空的容器之类的。

- 所以，如果类型展示出：从无到有生成对象是合理的 的语义，则应该有默认构造。
- 但是如果类型展示出：必须有外来信息才能生成对象 的语义，则不应该有默认构造。

但是为了表现出这种清晰的语义，会有诸多限制。

- 当然，需要注意有些函数或容器强调参数或元素必须是可默认构造的。
- 同时，在继承环境下，如果基类不是可默认构造的，那么就需要显式调用基类的构造。非常头疼。

# 操作符
## 条款8

查看memory3



# 效率

## 条款17 考虑使用 缓式评估

缓式评估就行copy on write 写时复制一样。当我们调用拷贝构造的时候并不一定拷贝的副本立刻被使用，有可能从不使用或者是从不更改。所以此时可以单独的做一个标记，当真的对副本进行修改的时候再进行构造动作。

## 条款18 分期偿还预期的计算成本（超急评估）

其实就是和17反过来。如果某些大概率或一定会使用的数据，尤其当这些数据使用频繁的时候，尝试设计一种数据结构进行预加载，当做一种缓存。

## 条款21 考虑利用重载来避免隐式类型转换造成的临时对象。

**我们在这一条中讨论的不是是否禁止隐式类型转换，主要是讨论如何降低开销。**

假设我们有这样的简朴的代码：

```c++
struct myclass{
    int val;
    myclass(int x):val(x){
        cout <<"const" << endl;
    };
};

const myclass operator+(const myclass& lhs, const myclass& rhs){
    return myclass(lhs.val + rhs.val);
}
int main(){
    myclass a(20);
    a+20;
    20+a;
}
```

这段代码会一共构造五次。第一次是`a`，第二次是`20`的隐式类型转换。第三次是`a+20`的返回值。第四次是`20`的隐式类型转换，第五次是`20+a`的返回值。

我们此时可以**多**写出两个重载的版本。

```c++
const myclass operator+(const myclass& lhs, int rhs){
    return myclass(lhs.val + rhs);
}

const myclass operator+(int lhs, const myclass&  rhs){
    return myclass(lhs + rhs.val);
}
```

之后，执行代码就只会构造三次。分别是`a`，`a+20`和`20+a`。没有隐式转换造成的临时对象的开销。

## 条款22 针对操作符，考虑同时提供复合形式(+=或-=)和单独形式（+，-）

主要原因是如`+=`和`-=`形式的符合操作符是直接作用于自身，所以返回的是`T&`形式。但是单独形式的操作符一般都是`const T`形式。所以显然前者效率较高。

同时，应该以复合形式为基础，实现单独形式 。也就是在`operator+`内部使用`operator+=`。因为这样只需要更改`operator+=`就可以改变其行为。

## 条款26 限制某个class所能产生的对象数量

主要讲了单例模式 和 对象数量计数器。需要计算数量的对象可以继承自对象数量计数器基类。行为有一点像侵入式智能指针。

计数器基类大概长这样**（利用了CRTP）**：

```c++
template<typename T>
struct counts{
    static int count;
    void increase(){
        count++;
    }
    void decrease(){
        count--;
    }
    static void get(){
        cout << count << endl;
    }
};
template<typename T>
int counts<T>::count = 0;

class myobj : counts<myobj>{ //这是一种CRTP。
    //具体内容  
};
class myobjanother : counts<myobjanother>{
  //具体内容  
};

```

既可以手动分离处理加减，也可以直接在计数器基类的构造或析构函数中计算加减。一个是手动一个是自动而已。

## 条款27 要求或禁止对象产生在堆中

### 要求对象产生在堆中

- 将对象的析构函数声明为`protected`。如果有继承，则必须要同时声明为`virtual`。
  - 因为外部无法访问对象的析构函数，则编译器禁止在栈上创建对象
  - 声明为`protected`而不是`private`的目的是让子类可以访问析构函数。
  - 注意子类也应该将析构函数声明为`protected`，否则子类对象会被允许创建在栈上。
  - 注意杂记4中提到的自动储存期限。所以子类如果在栈上，则子类的父类部分也在栈上。子类在堆上，则子类的父类部分也在堆上。
- 声明一个`destroy`函数，用于调用析构函数。
  - `destroy`函数是否为`virtual`不重要，因为`delete`会调用对应的`virtual`析构函数。

```c++
struct myclass{

    myclass(){
        cout <<"myclass const" << endl;
    }

    void destroy(){
        delete this;
    }

    protected: //protected
    virtual ~myclass(){ //虚函数
        cout <<"myclass dest" << endl;
    }
};


struct derive: public myclass{
    derive(){
        cout << "derive const" << endl;
    }
    void destroy(){
        delete this;
    }
    protected: //protected
    ~derive(){
        cout <<"derive dest" << endl;
    }

};


int main(){

    myclass* p = new myclass();
    p->destroy();
    derive* pp = new derive();
    pp->destroy();

    myclass* ppp = new derive();
    ppp->destroy();

    myclass pppp; 	//禁止
    derive ppppp;	//禁止
    return 0;
}
```



### 禁止对象产生在堆中

- 非常简单。只需要把`operator new` 和` operator delete`声明为`private`即可。
  - 也可以同时把`operator new[]` 和` operator delete[]`声明为`private`

```c++
struct myclass{

    myclass(){
        cout <<"myclass const" << endl;
    }
    ~myclass(){
        cout <<"myclass dest" << endl;
    }

    private:
        static void* operator new(size_t size); //私有
        static void operator delete(void* ptr); //私有

};


struct derive: public myclass{
    derive(){
        cout << "derive const" << endl;
    }
    ~derive(){
        cout <<"derive dest" << endl;
    }

};

int main(){
    derive obj;
    derive* p = new derive(); //错误
    return 0;
}
```

## 条款28 智能指针

都在智能指针章节。



## 条款29 引用计数

大概和智能指针的引用计数理论一致。额外的就是COW相关的实现。还有就是`operator[]`的语义，这部分写在effSTL的map的`[]`部分了。



## 条款 30 代理类

书里的例子是尝试用别的方法实现重载`operator[][]`的语义。因为压根没有`operator[][]`。

```c++
struct arr2{
    struct arr1{ //代理类
        arr1() = default;
        arr1(int x, int y):pivot(x), pivot2(y){};  //假设每个arr1拥有两个元素
        int pivot = 20;
        int pivot2 = 30;
        int& operator[](size_t index){ //返回实际元素的引用，满足赋值要求。
            cout <<"arr1 []" << endl;
            if(index == 0){
                return pivot;
            }
            else{
                return pivot2;
            }
        }
    };
    vector<arr1> arrs{arr1(1,2),arr1(3,4), arr1(5,6)};//我们假设arr2拥有三个arr1
    arr2() = default;
    arr1& operator[](size_t index){ //代理operator[]，直接返回对应下标的arr1对象的引用。满足赋值要求
        cout <<"arr2 []" << endl;
        return arrs[index];
    }
};

int main(){
    arr2 obj;
    cout << obj[2][1] << endl;
    cout << (obj.operator[](2)).operator[](1) << endl; //等同于上面


    obj[2][1] = 300;
    cout << obj[2][1] << endl; //赋值也没问题

    arr2::arr1 temp = obj[2]; //单独提取出第一维对象也没问题。
    //第一个operator[]返回的是arr1对象。所以如果再次链式调用operator[]就会自然匹配到arr1的那个而不是arr2的
    return 0;
}
```

我们想模拟二维数组的语义，但是这里模拟的还是有问题。图一乐就行

`obj[2][1]`的语义是`(obj.operator[](2)).operator[](1);`

我们核心想法是让第一层`arr2`储存一堆的`arr1`。然后`arr2`的`operator[]`会返回对应的`arr1`对象。此时，如果链式调用，**第二个`operator[]`自然会是`arr1`的。因为当前的operator`[]`是作用在`arr2`的`operator[]`返回的`arr1`上面的那个**

### 让operator[]可以区分左值和右值（不是真正区分，而是区分行为）

一般来说，左值是写入行为，右值是读取行为。这个例子只能用书里的。

```c++
class String {
public:
    //代理类用于区分operator[]的读写操作
    class CharProxy { // proxies for string chars
    public:
        CharProxy(String& str, int index); // creation
        CharProxy& operator=(const CharProxy& rhs); // 拷贝赋值。左值运用的场景，这种适用于 s1[3] = s2[8]的场景
        CharProxy& operator=(char c); // 拷贝赋值。左值运用的场景，这种适用于 s2[5] = 'x'的场景
        operator char() const;  //右值运用的场景。
    private:
        String& theString; //用于操作String,并在适当时机开辟新内存并复制
        int charIndex;
    };
    const CharProxy operator[](int index) const; // string类的 operator[]重载
    CharProxy operator[](int index); // string类的 operator[]重载
    ...
    friend class CharProxy;
private:
    RCPtr<StringValue> value;//见条款29
};
```

- 就像书里的例子。`s2[5] = 'x';`由于下标访问运算符重载为返回一个代理对象，所以此时的赋值会调用代理类对象的赋值。也就是`CharProxy& operator=(char c);`这个函数。这时候`s2[5]`的语义是左值。


- 针对`s1[3] = s2[8]`这个场景，虽然我们知道`s2[8]`没有写入，应该是个右值。但是注意，因为书中String实现了写时复制，所以`operator[]`本身是无成本的。因为代理对象只保有数据的引用。随后直到调用了左侧的代理对象的`CharProxy& operator=(const CharProxy& rhs); `这个函数，这时候才开始相应的动作。所以这里我们本身无法区分左值和右值，但是我们可以从行为上区分左值和右值。


```c++
String::CharProxy& String::CharProxy::operator=(const CharProxy& rhs)
{
    if (theString.value->isShared()) {
        theString.value = new StringValue(theString.value->data);
    }
    theString.value->data[charIndex] = rhs.theString.value->data[rhs.charIndex];
    return *this;
}
```

- 针对如`cout << s1[2];`这种场景，非常明确`s1[2]`的行为是右值行为，也就是读取行为。所以说压根没有必要做出任何的额外成本动作

```c++
String::CharProxy::operator char() const
{
return theString.value->data[charIndex]; //单纯的返回一个字符。这个data的类型是一个char*数组。所以这个下标访问是内置的。
}
```

- 潜在的问题是代理类非常复杂，常常会造成语义的改变。因为目标对象和代理对象的行为常常有细微差异。比如在上文的例子中`char* p = &s[2]`就无法通过。
  - 因为取出来的地址的类型是代理类类型。无法赋值给`char`类。由于我们目标类的`operator[]`返回的是代理类对象，所以这时候我们必须重载代理类的取地址运算符`operator&`

- 但是这不能解决所有问题。假设有一个类`A`引用了上面的这个蕴含代理类的目标对象，那么直接针对这个`A`使用目标对象的`operator[]`依旧返回的是代理类对象。此时如果我们想进行函数调用，那么就会出现问题。因为取回来的并不是类`A`对象，代理类对象并没有这个特定的成员函数。所以需要在所有的函数上都进行重载让他们也适用于代理类对象。

- 同时，我不知道为啥上面的目标类的`operator[]`一定要返回代理类的对象而不是代理类的引用。则此时在赋值方面会出现问题。

- 最后一个问题是隐式类型转换。隐式类型转换中，每一个步骤都只能执行一次。也就是每个步骤都只能进行一个层次的转换。比如`a`可以由`int`构造，`b`可以由`a`构造。那么如果一个函数接受一个`b`类对象，可以传入`a`，但是传入`int`就不可以。
