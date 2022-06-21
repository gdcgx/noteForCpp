# `C++11`之 `auto` 与 `decltype` 类型推导详解

## 一、`auto` 和 `decltype` 的区别

`auto` 和 `decltype` 都是 `C++11` 新增的关键字，都用于自动类型推导，但是它们的语法格式是有区别的，如下所示：

```cpp
auto varname = value;  			 //auto的语法格式
decltype(exp) varname [= value];  //decltype的语法格式
```

其中，`varname` 表示变量名，`value` 表示赋给变量的值，`exp` 表示一个表达式，方括号 `[]` 表示可有可无。

### 1、对类型推导的处理

`auto` 和 `decltype` 都会自动推导出变量 `varname` 的类型：

> `auto` 根据 `=` 右边的初始值 `value` 推导出变量的类型；
>
> `decltype` 根据 `exp` 表达式推导出变量的类型，跟 `=` 右边的 `value` 没有关系。

另外，`auto` 要求变量必须初始化，也就是在定义变量的同时必须给它赋值；而 `decltype` 不要求，初始化与否都不影响变量的类型。这很容易理解，因为 `auto` 是根据变量的初始值来推导出变量类型的，如果不初始化，变量的类型也就无法推导了。

### 2、对 `cv` 限定符的处理

`cv` 限定符是 `const` 和 `volatile` 关键字的统称：

> `const` 关键字用来表示数据是只读的，也就是不能被修改；
>
> `volatile` 和 `const` 是相反的，它用来表示数据是可变的、易变的，目的是不让 `CPU` 将数据缓存到寄存器，而是从原始的内存中读取。

在推导变量类型时，`auto` 和 `decltype` 对 `cv` 限制符的处理是不一样的。`decltype` 会保留 `cv` 限定符，而 `auto` 有可能会去掉 `cv` 限定符。

以下是 `auto` 关键字对 `cv` 限定符的推导规则：

> 如果表达式的类型不是指针或者引用，`auto` 会把 `cv` 限定符直接抛弃，推导成 `non-const` 或者 `non-volatile` 类型。
>
> 如果表达式的类型是指针或者引用，`auto` 将保留 `cv` 限定符。

下面的例子演示了对 `const` 限定符的推导：

```cpp
//非指针非引用类型const int n1 = 0;
auto n2 = 10;
n2 = 99;  //赋值不报错

decltype(n1) n3 = 20;
n3 = 5;  //赋值报错

//指针类型const int *p1 = &n1;
auto p2 = p1;
*p2 = 66;  //赋值报错

decltype(p1) p3 = p1;
*p3 = 19;  //赋值报错
```

在 `C++` 中无法将一个变量的完整类型输出，我们通过对变量赋值来判断它是否被 `const` 修饰；如果被 `const` 修饰那么赋值失败，如果不被 `const` 修饰那么赋值成功。虽然这种方案不太直观，但也是能达到目的的。

`n2` 赋值成功，说明不带 `const`，也就是 `const` 被 `auto` 抛弃了，这验证了 `auto` 的第一条推导规则。`p2` 赋值失败，说明是带 `const` 的，也就是 `const` 没有被 `auto` 抛弃，这验证了 `auto` 的第二条推导规则。

`n3` 和 `p3` 都赋值失败，说明 `decltype` 不会去掉表达式的 `const` 属性。

### 3、对引用的处理

当表达式的类型为引用时，`auto` 和 `decltype` 的推导规则也不一样；`decltype` 会保留引用类型，而 `auto` 会抛弃引用类型，直接推导出它的原始类型。

如下例：

```cpp
#include <iostream>
int main(){
    int n = 10;
    int &r1 = n;
    
    //auto推导 
    auto r2 = r1;
    r2 = 20;
    std::cout << n << ", " << r1 << ", " << r2 << std::endl;
    //decltype推导
    decltype(r1) r3 = n;
    r3 = 99;
    std::cout << n << ", " << r1 << ", " << r3 << std::endl;
    return 0;
}
//输出结果：
10, 10, 20
99, 99, 99
```

从运行结果可以发现，给 `r2` 赋值并没有改变 `n` 的值，这说明 `r2` 没有指向 `n`，而是自立门户，单独拥有了一块内存，这就证明 `r` 不再是引用类型，它的引用类型被 `auto` 抛弃了。

给 `r3` 赋值，`n` 的值也跟着改变了，这说明 `r3` 仍然指向 `n`，它的引用类型被 `decltype` 保留了。

### 4、总结

`auto` 虽然在书写格式上比 `decltype` 简单，但是它的推导规则复杂，有时候会改变表达式的原始类型；而 `decltype` 比较纯粹，它一般会坚持保留原始表达式的任何类型，让推导的结果更加原汁原味。

## 二、`auto`简介

编程时候常常需要把表达式的值付给变量,需要在声明变量的时候清楚的知道变量是什么类型。然而做到这一点并非那么容易(特别是模板中)，有时候根本做不到。为了解决这个问题， `C++11` 新标准就引入了 `auto` 类型说明符，用它就能让编译器替我们去分析表达式所属的类型。和原来那些只对应某种特定的类型说明符(例如 `int`)不同。`auto` 让编译器通过初始值来进行类型推演。从而获得定义变量的类型，所以说 `auto` 定义的变量必须有初始值。

如下例：

```cpp
#include <iostream>
#include <typeinfo>
#include <cxxabi.h>
#include <vector>
int main(){
    int a = 10;
    auto au_a = a;	//自动类型推断，au_a为int类型
    //typeid运算符可以输出变量的类型简写，如b(bool),c(char),i(int),f(float),d(double),l(long)等
    //__cxa_demangle可以实现复杂类型的直观输出,如结构体、容器适配器（eg. vector）等
    std::cout << typeid(au_a).name() << "->" << abi::__cxa_demangle(typeid(au_a).name(), NULL, NULL, NULL) <<  std::endl;
    std::vector<std::vector<float> > vecTest;
    std::cout<< typeid(vecTest).name() << "->" << abi::__cxa_demangle(typeid(vecTest).name(), NULL, NULL, NULL) << std::endl;
    return 0;
}
//输出结果：
i->int
St6vectorIS_IfSaIfEESaIS1_EE->std::vector<std::vector<float, std::allocator<float>>, std::allocator<std::vector<float, std::allocator<float>>>>
```

### 1、`auto` 的用法

用于代替冗长复杂、变量使用范围专一的变量声明。

假设没有 `auto` 的时候，操作标准库时经常需要这样：

```cpp
#include<string>
#include<vector>
int main()
{
    std::vector<std::string> vs;
    for (std::vector<std::string>::iterator i = vs.begin(); i != vs.end(); i++)
    {
        //...
    }
}
```

使用 `auto` 能简化代码：

```cpp
#include<string>
#include<vector>
int main()
{
    std::vector<std::string> vs;
    for (auto i = vs.begin(); i != vs.end(); i++)
    {
        //..
    }
}
```

`for` 循环中的 `i` 将在编译时自动推导其类型，而不用显式去定义那长长的一串。

**在定义模板函数时，用于声明依赖模板参数的变量类型。**

```cpp
template <typename _Tx, typename _Ty>
auto multiply(_Tx x, _Ty y)->decltype(x*y)
{
    return x*y;
}
```

当模板函数的返回值依赖于模板的参数时，依旧无法在编译代码前确定模板参数的类型，故也无从知道返回值的类型，这时可以使用 `auto` ，格式如上所示。

`decltype`操作符用于查询表达式的数据类型，也是`C++11`标准引入的新的运算符，其目的也是解决泛型编程中有些类型由模板参数决定，而难以表示它的问题。

`auto` 在这里的作用也称为 **返回值占位** ，它只是为函数返回值占了一个位置，真正的返回值是后面的 `decltype(_Tx*_Ty)` 。为何要将返回值后置呢？如果没有后置，则函数声明时为：

```c++
decltype(x*y) multiply(_Tx x, _Ty y)
```

而此时 `x,y` 还没声明呢，编译无法通过。

### 2、注意事项

`auto` 变量必须在定义时初始化，这类似于 `const` 关键字。定义在一个 `auto` 序列的变量必须始终推导成同一类型。例如：

```cpp
auto a4 = 10, a5 = 20, a6 = 30;		//正确
auto b4 = 10, b5 = 20.0, b6 = 'a';	//错误,没有推导为同一类型
```

使用 `auto` 关键字做类型自动推导时，依次施加一下规则：如果初始化表达式是引用，则去除引用语义。

```cpp
int a = 10;
int &b = a;

auto c = b;//c的类型为int而非int&（去除引用）
auto &d = b;//此时d的类型才为int&

c = 100;//a =10;
d = 100;//a =100;
```

如果初始化表达式为 `const` 或 `volatile`（或者两者兼有），则除去 `const/volatile` 语义。

```cpp
const int a1 = 10;
auto  b1= a1; //b1的类型为int而非const int（去除const）
const auto c1 = a1;//此时c1的类型为const int
b1 = 100;//合法
c1 = 100;//非法
```

如果 `auto` 关键字带上 `&` 号，则不去除 `const` 语意。

```cpp
const int a2 = 10;
auto &b2 = a2;//因为auto带上&，故不去除const，b2类型为const int
b2 = 10; //非法
```

初始化表达式为数组时，auto关键字推导类型为指针。

```cpp
#include <iostream>
#include <typeinfo>
#include <cxxabi.h>
int main(){
	int a3[3] = { 1, 2, 3 };
    auto b3 = a3;
    std::cout << typeid(b3).name() << "->" << abi::__cxa_demangle(typeid(b3).name(), NULL, NULL, NULL) << std::endl;
    return 0;
}
//输出结果：
Pi->int*
```

若表达式为数组且 `auto` 带上 `&` ，则推导类型为数组类型。

```cpp
#include <iostream>
#include <typeinfo>
#include <cxxabi.h>
int main(){
	int a7[3] = { 1, 2, 3 };
    auto &b7 = a7;
    std::cout << typeid(b7).name() << "->" << abi::__cxa_demangle(typeid(b7).name(), NULL, NULL, NULL) << std::endl;
    return 0;
}
//输出结果：
A3_i->int [3]
```

函数或者模板参数不能被声明为 `auto`

```cpp
void func(auto a)  //错误
{
	//... 
}
```

时刻要注意 `auto` 并不是一个真正的类型。`auto` 仅仅是一个占位符，它并不是一个真正的类型，不能使用一些以类型为操作数的操作符，如 `sizeof` 或者 `typeid`。

```cpp
cout << sizeof(auto) << endl;//错误
cout << typeid(auto).name() << endl;//错误
```

## 三、`decltype`简介

`decltype` 是 `C++11` 新增的一个关键字，它和 `auto` 的功能一样，都用来在编译时期进行自动类型推导。

`decltype` 是 `declare type` 的缩写，译为 **声明类型**。

既然已经有了 `auto` 关键字，为什么还需要 `decltype` 关键字呢？因为 `auto` 并不适用于所有的自动类型推导场景，在某些特殊情况下 `auto` 用起来非常不方便，甚至压根无法使用，所以 `decltype` 关键字也被引入到 `C++11` 中。

`auto` 和 `decltype` 关键字都可以自动推导出变量的类型，但它们的用法是有区别的：

> ```cpp
> auto varname = value;
> decltype(exp) varname = value;
> ```
>
> 其中，`varname` 表示变量名，`value` 表示赋给变量的值，`exp` 表示一个表达式。
>

`auto` 根据 `=` 右边的初始值 `value` 推导出变量的类型，而 `decltype` 根据 `exp` 表达式推导出变量的类型，跟 `=` 右边的 `value` 没有关系。

另外，`auto` 要求变量必须初始化，而 `decltype` 不要求。这很容易理解，`auto` 是根据变量的初始值来推导出变量类型的，如果不初始化，变量的类型也就无法推导了。`decltype` 可以写成下面的形式：

```cpp
decltype(exp) varname;
```

> `exp` 注意事项：
>
> 原则上讲，`exp` 就是一个普通的表达式，它可以是任意复杂的形式，但是我们必须要保证 `exp` 的结果是有类型的，不能是 `void`；
>
> 例如，当 `exp` 调用一个返回值类型为 `void` 的函数时，`exp` 的结果也是 `void` 类型，此时就会导致编译错误。

### 1、用法举例：

```cpp
int a = 0;
decltype(a) b = 1;			//b 被推导成了 int
decltype(10.8) x = 5.5;		//x 被推导成了 double
decltype(x + 100) y;		//y 被推导成了 double
```

可以看到，`decltype` 能够根据变量、字面量、带有运算符的表达式推导出变量的类型。请留意最后一行代码，`y` 没有被初始化。

### 2、推导规则

上面的例子初步感受了一下 `decltype` 的用法，但不要认为 `decltype` 就这么简单，它的玩法实际上可以非常复杂。当程序员使用 `decltype(exp)` 获取类型时，编译器将根据以下三条规则得出结果：

> 【规则1】 如果 `exp` 是一个不被括号`( )`包围的表达式，或者是一个类成员访问表达式，或者是一个单独的变量，那么 `decltype(exp)` 的类型就和 `exp` 一致，这是最普遍最常见的情况。
>
> 【规则2】 如果 `exp` 是函数调用，那么 `decltype(exp)` 的类型就和函数返回值的类型一致。
>
> 【规则3】 如果 `exp` 是一个左值，或者被括号`( )`包围，那么 `decltype(exp)` 的类型就是 `exp` 的引用；假设 `exp` 的类型为 `T`，那么 `decltype(exp)` 的类型就是 `T&`。

为了更好地理解 `decltype` 的推导规则，下面来看几个实际的例子：

**【实例1】**`exp` 是一个普通表达式：

```cpp
#include <iostream>
#include <typeinfo>
#include <cxxabi.h>

class Student{
public:
	static int total;
	std::string name;
	int age;
	float scores;
};
int Student::total = 0;

int main(){
	int n = 0;
	const int &r = n;
	Student stu;
	decltype(n) a = n;  //n 为 int 类型，a 被推导为 int 类型
    std::cout << "a->" << typeid(a).name() << "->" << abi::__cxa_demangle(typeid(a).name(), NULL, NULL, NULL) << std::endl;
	decltype(r) b = n;  //r 为 const int& 类型, b 被推导为 const int& 类型
    std::cout << "b->" << typeid(b).name() << "->" << abi::__cxa_demangle(typeid(b).name(), NULL, NULL, NULL) << std::endl;
	decltype(Student::total) c = 0; //total 为类 Student 的一个 int 类型的成员变量，c 被推导为 int 类型
    std::cout << "c->" << typeid(c).name() << "->" << abi::__cxa_demangle(typeid(c).name(), NULL, NULL, NULL) << std::endl;
	decltype(stu.name) url = "http://c.biancheng.net/cplus/";  //total 为类 Student 的一个 string 类型的成员变量，url被推导为string类型    
    std::cout << "url->" << typeid(url).name() << "->" << abi::__cxa_demangle(typeid(url).name(), NULL, NULL, NULL) << std::endl;
    return 0;
}
输出结果：
a->i->int
b->i->int
c->i->int
url->NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE->std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >
```

这段代码很简单，按照推导【规则1】，对于一般的表达式，`decltype` 的推导结果就和这个表达式的类型一致。

**【实例2】**`exp` 为函数调用：

```c++
#include <iostream>
#include <typeinfo>
#include <cxxabi.h>

//函数声明
int& func_int_r(int, char); //返回值为 int&
int&& func_int_rr(); //返回值为 int&&
int func_int(double); //返回值为 int
const int& fun_cint_r(int, int, int); //返回值为 const int&
const int&& func_cint_rr(); //返回值为 const int&&

int main(){
	//decltype类型推导
    int m = 100;
    decltype(func_int_r(100, 'A')) j = m; //j 的类型为 int&
    std::cout << "j->" << typeid(j).name() << "->" << abi::__cxa_demangle(typeid(j).name(), NULL, NULL, NULL) << std::endl;
    j = 1;
    std::cout << "m = " << m << "; j = " << j << std::endl;

    decltype(func_int_rr()) k = 0; //k 的类型为 int&&，右值
    std::cout << "k->" << typeid(k).name() << "->" << abi::__cxa_demangle(typeid(k).name(), NULL, NULL, NULL) << std::endl;

    decltype(func_int(10.5)) l = m; //l 的类型为 int
    std::cout << "l->" << typeid(l).name() << "->" << abi::__cxa_demangle(typeid(l).name(), NULL, NULL, NULL) << std::endl;
    l = 100;
    std::cout << "m = " << m << "; l = " << l << std::endl;

    decltype(fun_cint_r(1,2,3)) x = m; //x 的类型为 const int &
    std::cout << "x->" << typeid(x).name() << "->" << abi::__cxa_demangle(typeid(x).name(), NULL, NULL, NULL) << std::endl;
    //x = 20; //const修饰不可更改

    decltype(func_cint_rr()) y = 0; // y 的类型为 const int&&，右值
    std::cout << "y->" << typeid(y).name() << "->" << abi::__cxa_demangle(typeid(y).name(), NULL, NULL, NULL) << std::endl;
    //y = 90; //const修饰不可更改
    return 0;
}
输出结果：
j->i->int
m = 1; j = 1
k->i->int
l->i->int
m = 1; l = 100
x->i->int
y->i->int
```

**【实例3】**`exp` 是左值，或者被`( )`包围：

```c++
#include <iostream>
#include <typeinfo>
#include <cxxabi.h>

class Base
{
    public:
    Base():x(0){}
    int x;
};

int main(){
	const Base obj;
    int num = 100;

    //带有括号的表达式
    decltype(obj.x) a_ = num; //obj.x 为类的成员访问表达式，符合推导规则一，a 的类型为 int
    std::cout << "a_->" << typeid(a_).name() << "->" << abi::__cxa_demangle(typeid(a_).name(), NULL, NULL, NULL) << std::endl;
    a_ = 10;
    std::cout << "num = " << num << "; a_ = " << a_ << std::endl;

    decltype((obj.x)) b_ = num; //obj.x 带有括号，符合推导规则三，b 的类型为 const int&。
    std::cout << "b_->" << typeid(b_).name() << "->" << abi::__cxa_demangle(typeid(b_).name(), NULL, NULL, NULL) << std::endl;
    // b_ = 22;  表达式必须是可修改的左值,因为 Base 是 const 修饰的,取消 const 修饰则可以修改 b_ 的值
    // 但是上面 a_ 的类型为什么不是 const int 呢？？？
    // std::cout << "num = " << num << "; b_ = " << b_ << std::endl;


    //加法表达式
    int n_ = 0, m_ = 0, num_ = 100;
    decltype(n_ + m_) c_ = num_; //n+m 得到一个右值，符合推导规则一，所以推导结果为 int
    std::cout << "c_->" << typeid(c_).name() << "->" << abi::__cxa_demangle(typeid(c_).name(), NULL, NULL, NULL) << std::endl;
    c_ = 22;
    std::cout << "num_ = " << num_ << "; c_ = " << c_ << std::endl;

    decltype(n_ = n_ + m_) d_ = num_; //n=n+m 得到一个左值，符号推导规则三，所以推导结果为 int&
    std::cout << "d_->" << typeid(d_).name() << "->" << abi::__cxa_demangle(typeid(d_).name(), NULL, NULL, NULL) << std::endl;
    d_ = 22;
    std::cout << "num_ = " << num_ << "; d_ = " << d_ << std::endl;

    return 0;
}
输出结果：
a_->i->int
num = 100; a_ = 10
b_->i->int
c_->i->int
num = 100; c_ = 22
d_->i->int
num = 22; d_ = 22
```

### 3、实际应用

众所周知，`auto` 只能用于类的静态成员，不能用于类的非静态成员（普通成员），如果我们想推导非静态成员的类型，这个时候就必须使用 `decltype` 了。下面是一个模板的定义：

```c++
#include <vector>

template <typename T>
class Base 
{
public:
    void func(T& container) 
    {
        m_it = container.begin();
    }

private:
    typename T::iterator m_it; //注意这里
};

int main()
{
    // 传入 const 修饰的容器，编译报错
    const std::vector<int> v;
    Base<const std::vector<int>> obj;
    obj.func(v);
    
    // 如下则不会报错
    // std::vector<int> v;
    // Base<std::vector<int>> obj;
    // obj.func(v);

    return 0;
}
```

单独看 `Base` 类中 `m_it` 成员的定义，很难看出会有什么错误，但在使用 `Base` 类的时候，如果传入一个 `const` 类型的容器，编译器马上就会弹出一大堆错误信息。原因就在于，`T::iterator` 并不能包括所有的迭代器类型，当 `T` 是一个 `const` 容器时，应当使用 `const_iterator`。

要想解决这个问题，在之前的 `C++98/03` 版本下只能想办法把 `const` 类型的容器用模板特化单独处理，增加了不少工作量，看起来也非常晦涩。但是有了 `C++11` 的 `decltype` 关键字，就可以直接这样写：

```c++
#include <vector>

template <typename T>
class Base 
{
public:
    void func(T& container) 
    {
        m_it = container.begin();
    }

private:
    decltype(T().begin()) m_it; //注意这里，这么做的话就不会报错
};


int main()
{
    const std::vector<int> v;
    Base<const std::vector<int>> obj;
    obj.func(v);

    return 0;
}
```

## 四、实际使用示例

此处为线程池任务模板的示例，`auto`在此处作为返回值类型，之所以不用 `decltype` 是因为此时还没有定义函数 `f` 和参数 `args` ，无法通过 `decltype` 推导其返回值类型，因此将 `auto` 作为一个返回值占位符，其真正的返回值类型是 `std::future<decltype(f(args...))>` 这个 `std::future<>` 的类型，在 `future` 里面通过`decltype(f(args...))` 推导出其模板函数的返回值类型。

```cpp
/**
    * @brief 用线程池启用任务(F是function, Args是参数)
    *
    * @param 超时时间 ，单位ms (为0时不做超时控制) ；若任务超时，此任务将被丢弃
    * @param bind function
    * @return 返回任务的future对象, 可以通过这个对象来获取返回值
    */
    /*
    template <class F, class... Args>
    它是c++里新增的最强大的特性之一，它对参数进行了高度泛化，它能表示0到任意个数、任意类型的参数
    auto exec(F &&f, Args &&... args) -> std::future<decltype(f(args...))>
    std::future<decltype(f(args...))>：返回future，调用者可以通过future获取返回值
    返回值后置
    */
    template <class F, class... Args>
    auto exec(int64_t timeoutMs, F&& f, Args&&... args) -> std::future<decltype(f(args...))>
    {
        int64_t expireTime =  (timeoutMs == 0 ? 0 : TNOWMS + timeoutMs);  // 获取现在时间
        //定义返回值类型
        using RetType = decltype(f(args...));  // 推导返回值
        // 封装任务
        auto task = std::make_shared<std::packaged_task<RetType()>>(std::bind(std::forward<F>(f),
                                                              std::forward<Args>(args)...));

        TaskFuncPtr fPtr = std::make_shared<TaskFunc>(expireTime);  // 封装任务指针，设置过期时间
        fPtr->_func = [task]() {  // 具体执行的函数
            printf("do task-----------\n");
            (*task)();
        };

        std::unique_lock<std::mutex> lock(_mutex);
        _tasks.push(fPtr);              // 插入任务
        _condition.notify_one();        // 唤醒阻塞的线程，可以考虑只有任务队列为空的情况再去notify

        return task->get_future();;
    }
```

