# **C++：Lambda表达式详解**

之前写代码有看到过一段神奇的代码，没有函数名也没有参数列表，只有几个类型，乍一看真是吓我一跳。后来才知道这叫`lambda`匿名函数，据说超级好用，但是之前因为摸鱼一直懒得整理。今天突发奇想准备学习、整理一下`C++`的匿名函数部分，以期提高一下自己的`C++`水准。在这里对`lambda`做一个简单的学习总结，后续如果有新的见解也会加入到这里面来。学无止境~											

## **function函数**

`std::function`是一个函数包装器模板，来自`fubctional`库。该函数包装器模板能包装**任何类型**的**可调用元素**（`callable element`）例如普通函数和函数对象。

包装器对象可以进行拷贝，并且包装器类型仅仅只依赖于其调用特征（`call signature`），而不依赖于可调用元素自身的类型。

一个`std::function`类型对象实例可以包装下列这几种可调用元素类型：函数、函数指针、类成员函数指针或任意类型的函数对象（例如定义了`operator()`操作

并拥有函数闭包）。`std::function`对象可被拷贝和转移，并且可以使用指定的调用特征来直接调用目标元素。

当`std::function`对象未包裹任何实际的可调用元素，调用该`std::function`对象将抛出`std::bad_function_call`异常。

头文件包含在`#include<functional>`里面

### **functiona函数使用**

和`vector`的使用类似

> std::function<函数返回类型（参数1类型，参数2类型，.......)> [funcName]

```cpp
#include<functional>

int fun1(int n, int m)
{
    return n + m;
}

void fun2(int n)
{
    printf("fun2\n");
}

int main()
{
    std::function<int(int, int)> call1 = fun1;
    int ans = call1(1, 2);
    printf("%d\n", ans);

    std::function<void(int)> call2 = fun2;
    call2(3);
    return 0;
}
```

## **lambda函数**

### **一段简单的Code**

我也不是文艺的人，对于`Lambda`的历史，以及`Lambda`与`C++`的那段渊源，我也不是很熟悉，技术人，讲究拿代码说事。

```c++
#include<iostream>
using namespace std;
 
int main()
{
  int a = 1;
  int b = 2;
 
  auto func = [=, &b](int c)->int {return b += a + c;};
  return 0;
}
```

当我第一次看到这段代码时，我直接凌乱了，直接看不懂啊。

### **基本语法**

上面已经简单介绍了`function`函数，但是用`function`去封装普通函数可能还看不出来使用`function`函数的意义，现在在介绍一下封装`lambda`表达式，更好理解`function`函数。

`C++11`提供了对匿名函数的支持,称为`Lambda`函数(也叫`Lambda`表达式)，还有一个名字式匿名函数，它有四部组成。其简单来说也就是一个函数，并没有很复杂。其语法定义如下：

> `[ caputrue ] ( params ) opt -> ret { body; } ;`
>
> `[ 外部变量捕获列表 ] ( 参数表 ) 特殊操作符 -> 返回值类型 { 函数体; };`

```c++
int n = 5;
//匿名函数
function call=[n/*外部变量捕获列表*/](/*参数列表*/int a,int b) -> int/*返回值类型*/{
    //函数体
    printf("%d\n",n+a+b);
    return 2;
};
```

> + 捕获列表`capture`：捕捉列表总是出现在`Lambda`函数的开始处。实际上，`[]`是`Lambda`引出符。编译器根据该引出符判断接下来的代码是否是`Lambda`函数。捕捉列表能够捕捉上下文中的变量以供`Lambda`函数使用;
> + 参数列表`params`(选填)：与普通函数的参数列表一致。如果不需要参数传递，则可以连同括号`()`一起省略;
>
> + 函数选项`opt`：可以填`mutable,exception,attribute`（选填）
>
>   + 修饰符`mutable`：说明`lambda`表达式体内的代码可以修改被捕获的变量，并且可以访问被捕获的对象的`non-const`方法。`mutable`可以取消其常量性。在使用该修饰符时，参数列表不可省略（即使参数为空）
>
>   + 修饰符`exception`：说明`lambda`表达式是否抛出异常以及何种异常。
>
>   + 修饰符`attribute`：用来声明属性。
>
> + 返回值类型`ret`(选填)：用追踪返回类型形式声明函数的返回类型。我们可以在不需要返回值的时候也可以连同符号”->”一起省略。此外，在返回类型明确的情况下，也可以省略该部分，让编译器对返回类型进行推导;
>
> + 函数体`body`：内容与普通函数一样，不过除了可以使用参数之外，还可以使用所有捕获的变量。

与普通函数最大的区别是，除了可以使用参数以外，`Lambda`函数还可以通过捕获列表访问一些上下文中的数据。具体地，捕捉列表描述了上下文中哪些数据可以被`Lambda`使用，以及使用方式（以值传递的方式或引用传递的方式）。

简单来说就是`lambda`表达式的捕获列表精细控制了`lambda`表达式能够访问的外部变量，以及如何访问这些变量。

语法上，在`[]`包括起来的是捕捉列表，捕捉列表由多个捕捉项组成，并以逗号分隔。捕捉列表有以下几种形式：

> + `[]`：不捕获任何变量
>
> + `[&]`：引用传递方式捕捉所有父作用域的变量（包括`this`）
>
> + `[=]`：值传递方式捕获父作用域中所有变量，并作为副本在函数体中使用（包括`this`）
>
> + `[var]`：值传递方式捕获`var`变量，同时不捕获其他变量
>
> + `[&var]`：引用传递方式捕获变量`var`
>
> + `[this]`：值传递方式捕获当前类中的`this`指针，让`lambda`表达式拥有和当前类成员函数同样的访问权限
>
>   如果已经使用了`&`或者 `=` ，就默认含有此选项
>
>   捕获`this`的目的是可以在`lamda`中使用当前类的成员函数和成员变量

上面提到了一个父作用域，也就是包含`Lambda`函数的语句块，说通俗点就是包含`Lambda`的`{}`代码块。上面的捕捉列表还可以进行组合，例如：

> + `[=,&a,&b]`：表示以引用传递的方式捕捉变量`a`和`b`，以值传递方式捕捉其它所有变量
> + `[&,a,this]`：表示以值传递的方式捕捉变量`a`和`this`，引用传递方式捕捉其它所有变量

不过值得注意的是，捕捉列表不允许变量重复传递。下面一些例子就是典型的重复，会导致编译时期的错误。例如：

> `[=,a]`：这里已经以值传递方式捕捉了所有变量，但是重复捕捉`a`了，会报错的
> `[&,&this]`：这里`&`已经以引用传递方式捕捉了所有变量，再捕捉`this`也是一种重复

### **lambda 示例代码**

```c++
[] (int x, int y) { return x + y; } 	// 隐式返回类型
[] (int& x) { ++x;  } 				   // 没有 return 语句 -> Lambda 函数的返回类型是 'void'
[] () { ++global_x;  } 				   // 没有参数，仅访问某个全局变量
[] { ++global_x; } 					  // 与上一个相同，省略了 (操作符重载函数参数)
```

可以像下面这样显示指定返回类型：

```c++
[] (int x, int y) -> int { int z = x + y; return z; }
```

在这个例子中创建了一个临时变量`z`来存储中间值。和普通函数一样，这个中间值不会保存到下次调用。什么也不返回的`Lambda` 函数可以省略返回类型，而不需要使用`-> void`形式。

#### **示例1**

```c++
#include <iostream>
#include <vector>
#include <algorithm>		//提供操作std::for_each()
int main(){
    std::vector<int> some_list;
    int total = 0;
    for (int i = 0; i < 5; ++i) some_list.push_back(i);
    std::for_each(begin(some_list), end(some_list), [&total](int x){
        total += x;
    });
    std::cout << total << std::endl;
}
//输出结果：10
```

此例计算 `some_list` 中所有元素的总和。变量 `total` 被存为 `Lambda` 函数闭包的一部分。因为它是栈变量（局部变量）`total` 引用，所以可以改变它的值。

#### **示例2**

```c++
#include <iostream>
#include <vector>
#include <algorithm>
class test{
    public:
    int a;
    int some_func(){
        a = 7;
        return a;
    }
    void xixi(){
        std::vector<int> some_list;
        for (int i = 0; i < 5; ++i) some_list.push_back(i);
        int total = 0;
        int value = 5;
        std::for_each(begin(some_list), end(some_list), [&](int x){
        total += x * value * this->some_func();
        });
        std::cout << total << std::endl;
    }
};
int main(){
    test sum_test;
    sum_test.xixi();
}
//输出结果：350
```

此例中 `total` 会存为引用, `value` 则会存一份值拷贝。对 `this` 的捕获比较特殊，它只能按值捕获。`this` 只有当包含它的最靠近它的函数不是静态成员函数时才能被捕获。对 `protect` 和 `private` 成员来说，这个 `Lambda` 函数与创建它的成员函数有相同的访问控制。如果 `this` 被捕获了，不管是显式还是隐式的，那么它的类的作用域对 `Lambda` 函数就是可见的。访问 `this` 的成员不必使用 `this->` 语法，可以直接访问。

### **lambda 和 function 联合使用**

`lambda`函数是通常和`function`函数一起使用，`function`函数定义一个封装函数的变量名称，`lambda`实现函数的封装，这样配合使用可以减少函数指针的使用以及资源的消耗，达到将一个函数当成一个变量去传参、储存，同时还能实现函数功能的效果

```cpp
#include<functional>
#include<iostream>
#include<vector>
using namespace std;
typedef function< void() > fun;
vector<fun>p;
void GetSum(int x, int y)
{
    cout << x + y << endl;
}
void add(fun fun1)
{
    p.push_back(fun1);
}

int main()
{
    int n;
    cin >> n;

    for (int i = 0; i < n; i++)
    {
        int x, y;
        cin >> x >> y;
        add([x, y]() {GetSum(x, y); });
    }
    for (int i = 0; i < n; i++)
    {
        p[i]();
    }
    system("pause");
    return 0;
}
```

### **lambda 代码实例**

对于`Lambda`的使用，说实话，我没有什么多说的，个人理解，在没有`Lambda`之前的`C++` , 我们也是那样好好的使用，并没有对缺少`Lambda`的`C++`有什么抱怨，而现在有了`Lambda`表达式，只是更多的方便了我们去写代码。不知道大家是否记得`C++ STL`库中的仿函数对象，仿函数想对于普通函数来说，仿函数可以拥有初始化状态，而这些初始化状态是在声明仿函数对象时，通过参数指定的，一般都是保存在仿函数对象的私有变量中;在`C++`中，对于要求具有状态的函数，我们一般都是使用仿函数来实现，比如以下代码：

```c++
#include<iostream>
using namespace std;
 
typedef enum
{
    add = 0,
    sub,
    mul,
    divi
}type;
 
class Calc
{
public:
    Calc(int x, int y):m_x(x), m_y(y){}
    int operator()(type i)
    {
        switch (i)
        {
            case add:
                return m_x + m_y;
            case sub:
                return m_x - m_y;
            case mul:
                return m_x * m_y;
            case divi:
                return m_x / m_y;
        }
    }
private:
    int m_x;
    int m_y;
};
 
int main()
{
    Calc addObj(10, 20);
    cout << addObj(add) << endl; // 发现C++11中，enum类型的使用也变了，更“强”了                            
    return 0;
}
```

现在我们有了`Lambda`这个利器，那是不是可以重写上面的实现呢？看代码：

```c++
#include<iostream>
using namespace std;
      
typedef enum
{     
    add = 0,
    sub,
    mul,
    divi
}type;
      
int main()
{     
    int a = 10;
    int b = 20;
    auto func = [=](type i)->int {
        switch (i)
        {
            case add:
                return a + b;
            case sub:
                return a - b;
            case mul:
                return a * b;
            case divi:
                return a / b;
        }
    };
    cout << func(add) << endl;
}
```

