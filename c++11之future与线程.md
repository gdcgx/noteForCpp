# 引言

`c++11` 在标准库中增加了线程库 `(std::thread)`，使得创建线程变得非常的简单，简单示例：

```c++
#include <thread>
#include <iostream>

void test(int num){ std::cout << num << std::endl; }

int main(){
	std::thread thr(test,5);
	thr.join();
	return 0;
}
```

但是线程毕竟是属于比较低层次的东西，有时候使用有些不便，比如希望获取线程函数的返回结果的时候，就不能直接通过 `thread.join()` 得到结果，这时就必须定义一个变量，在线程函数中去给这个变量赋值，然后 `join` ,最后得到结果，这个过程是比较繁琐的。

`c++11`还提供了异步接口 `std::async` ，通过这个异步接口可以很方便的获取线程函数的执行结果。`std::async` 会自动创建一个线程去调用线程函数，它返回一个 `std::future` ，这个 `future` 中存储了线程函数返回的结果，当需要线程函数的结果时，直接从 `future` 中获取，非常方便。

但是其实 `std::async` 提供的便利可不仅仅是这一点，它首先解耦了线程的创建和执行，使得可以在需要的时候获取异步操作的结果；其次它还提供了线程的创建策略（比如可以通过延迟加载的方式去创建线程），使得可以以多种方式去创建线程。

在介绍 `async` 具体用法以及为什么要用 `std::async` 代替线程的创建之前，先说一说 `std::future`、`std::promise` 和 `std::packaged_task`。

# 一、`std::future` 简介

`std::future` 是一个非常有用也很有意思的东西，简单说 `std::future` 提供了一种访问异步操作结果的机制，因此可以将它当成一种简单的线程间同步的一个手段。从字面意思来理解，它表示未来，这个名字非常贴切，因为一个异步操作我们是不可能马上就获取操作结果的，只能在未来某个时候获取，但是我们可以以同步等待的方式来获取结果，可以通过查询 `future` 的状态`(future_status)`来获取异步操作的结果。

`std::future` 通常由某个 `Provider` 创建，可以把 `Provider` 想象成一个异步任务的提供者，`Provider` 在某个线程中设置共享状态的值，与该共享状态相关联的 `std::future` 对象调用 `get()`（通常在另外一个线程中） 获取该值，如果共享状态的标志不为 `ready`，则调用 `std::future::get` 会阻塞当前的调用者，直到 `Provider` 设置了共享状态的值（此时共享状态的标志变为 `ready`），`std::future::get()` 返回异步任务的值或异常（如果发生了异常）。

一个有效 `(valid)` 的 `std::future` 对象通常由以下三种 `Provider` 创建，并和某个共享状态相关联。`Provider` 可以是函数或者类，它们分别是：

> - `std::async` 函数，本文后面会介绍 `std::async()` 函数。
> - `std::promise::get_future`，`get_future` 为 `promise` 类的成员函数。详见 [C++11 并发指南四( 详解一 std::promise 介绍)](http://www.cnblogs.com/haippy/p/3239248.html)。
> - `std::packaged_task::get_future`，此时 `get_future` 为 `packaged_task` 的成员函数。详见[C++11 并发指南四( 详解二 std::packaged_task 介绍)](http://www.cnblogs.com/haippy/p/3279565.html)。

一个 `std::future` 对象只有在有效 `(valid)` 的情况下才有用 `(useful)`，由 `std::future` 默认构造函数创建的 `future` 对象不是有效的（除非当前非有效的 `future` 对象被 `move` 赋值另一个有效的 `future` 对象）。

 在一个有效的`future`对象上调用`get`会阻塞当前调用者，直到`Provider`设置了共享状态的值或异常（此时共享状态的标志变为`ready`），`std::future::get` 将返回异步任务的值或异常（如果发生了异常）。

<future> 头文件中包含了以下几个类和函数：

> - `Providers` 类：`std::promise, std::package_task`
> - `Futures` 类：`std::future, shared_future`
> - `Providers` 函数：`std::async()`
> - 其他类型：`std::future_error, std::future_errc, std::future_status, std::launch`

## 1、成员函数

### 构造函数

`std::future` 一般由 `std::async, std::promise::get_future, std::packaged_task::get_future` 创建，不过也提供了构造函数，如下表所示：

| `default (1)`            | `future() noexcept;`                   |
| :----------------------- | :------------------------------------- |
| **`copy [deleted] (2)`** | **`future (const future&) = delete;`** |
| **`move (3)`**           | **`future (future&& x) noexcept;`**    |

不过 `std::future` 的拷贝构造函数是被禁用的，只提供了默认的构造函数和 `move` 构造函数（注：`C++` 新特新）。另外，`std::future` 的普通赋值操作也被禁用，只提供了 `move` 赋值操作。如下代码所示：

```cpp
std::future<int> fut;           // 默认构造函数
fut = std::async(do_some_task);   // move-赋值操作。
```

### `std::future::share()`

返回一个 `std::shared_future` 对象（本文后续内容将介绍 `std::shared_future` ），调用该函数之后，该 `std::future` 对象本身已经不和任何共享状态相关联，因此该 `std::future` 的状态不再是 `valid` 的了。

```cpp
#include <iostream>       // std::cout
#include <future>         // std::async, std::future, std::shared_future

int do_get_value() { return 10; }

int main ()
{
    std::future<int> fut = std::async(do_get_value);
    std::shared_future<int> shared_fut = fut.share();

    // 共享的 future 对象可以被多次访问.
    std::cout << "value: " << shared_fut.get() << '\n';
    std::cout << "its double: " << shared_fut.get()*2 << '\n';
	std::cout << fut.get() << std::endl;
    return 0;
}

输出结果：
value: 10
its double: 20
terminate called after throwing an instance of 'std::future_error'
  what():  std::future_error: No associated state
Aborted (core dumped)
```

### `std::future::get()`

`std::future::get` 一共有三种形式，如下表所示：

| `generic template (1)`             | `T get();`                                                   |
| ---------------------------------- | ------------------------------------------------------------ |
| **`reference specialization (2)`** | **`R& future<R&>::get();       // when T is a reference type (R&)`** |
| **`void specialization (3)`**      | **`void future<void>::get();   // when T is void`**          |

当与该 `std::future` 对象相关联的共享状态标志变为 `ready` 后，调用该函数将返回保存在共享状态中的值，如果共享状态的标志不为 `ready`，则调用该函数会阻塞当前的调用者，而此后一旦共享状态的标志变为 `ready`，`get` 返回 `Provider` 所设置的共享状态的值或者异常（如果抛出了异常）。

如下述程序：

```c++
#include <iostream>       // std::cin, std::cout, std::ios
#include <functional>     // std::ref
#include <thread>         // std::thread
#include <future>         // std::promise, std::future
#include <exception>      // std::exception, std::current_exception

void get_int(std::promise<int>& prom) {
    int x;
    std::cout << "Please enter an integer value: ";
    std::cin.exceptions (std::ios::failbit);   // 如果输入的内容不符合格式，则抛出异常
    try {
        std::cin >> x;                         // 输入的 x 不为 int 的时候设置为 failbit
        prom.set_value(x);
    } catch (std::exception&) {
        std::cout << "please enter a num,you entered is error." << std::endl;
        prom.set_exception(std::current_exception());
    }
}

void print_int(std::future<int>& fut) {
    try {
        int x = fut.get();
        std::cout << "value: " << x << '\n';
    } catch (std::exception& e) {
        std::cout << "[exception caught: " << e.what() << "]\n";
    }
}

int main ()
{
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();

    std::thread th1(get_int, std::ref(prom));
    std::thread th2(print_int, std::ref(fut));

    th1.join();
    th2.join();
    return 0;
}
```

### `std::future::valid()`

检查当前 `std::future` 对象是否有效，即释放与某个共享状态相关联。一个有效的 `std::future` 对象只能通过 `std::async(), std::future::get_future` 或者 `std::packaged_task::get_future` 来初始化。另外由 `std::future` 默认构造函数创建的 `std::future` 对象是无效`(invalid)`的，通过 `std::future` 的 `move` 赋值后该 `std::future` 对象也可以变为 `valid`。

```c++
#include <iostream>       // std::cout
#include <future>         // std::async, std::future
#include <utility>        // std::move

int do_get_value() { return 11; }

int main ()
{
    // 由默认构造函数创建的 std::future 对象,
    // 初始化时该 std::future 对象处于为 invalid 状态.
    std::future<int> foo, bar;
    foo = std::async(do_get_value); // move 赋值, foo 变为 valid.
    bar = std::move(foo); // move 赋值, bar 变为 valid, 而 move 赋值以后 foo 变为 invalid.

    if (foo.valid())
        std::cout << "foo's value: " << foo.get() << '\n';
    else
        std::cout << "foo is not valid\n";

    if (bar.valid())
        std::cout << "bar's value: " << bar.get() << '\n';
    else
        std::cout << "bar is not valid\n";

    return 0;
}

输出结果：
foo is not valid
bar's value: 11
```

### `std::future::wait()`

等待与当前 `std::future` 对象相关联的共享状态的标志变为 `ready`。如果共享状态的标志不是 `ready`（此时 `Provider` 没有在共享状态上设置值（或者异常）），调用该函数会被阻塞当前线程，直到共享状态的标志变为 `ready`。一旦共享状态的标志变为 `ready`，`wait()` 函数返回，当前线程被解除阻塞，但是 `wait()` 并不读取共享状态的值或者异常。

```c++
#include <iostream>                // std::cout
#include <future>                // std::async, std::future
#include <chrono>                // std::chrono::milliseconds

// a non-optimized way of checking for prime numbers:
bool do_check_prime(int x) // 为了体现效果, 该函数故意没有优化.
{
    for (int i = 2; i < x; ++i){
        std::this_thread::sleep_for(std::chrono::microseconds(1/100)); // 为了体现效果, 每次循环睡眠一小段时间.
        if (x % i == 0){
            return false;
        }
    }
    return true;
}

int main()
{
    std::future < bool > fut = std::async(do_check_prime, 194232491);
    std::cout << "Checking...\n";
    fut.wait();
    std::cout << "194232491 ";
    if (fut.get()){
        std::cout << "is prime.\n";
    }
    else{
        std::cout << "is not prime.\n";
    }
    return 0;
}
```

### `std::future::wait_for()`

与 `std::future::wait()` 的功能类似，即等待与该 `std::future` 对象相关联的共享状态的标志变为 `ready`，该函数原型如下：

```c++
template <class Rep, class Period>
  future_status wait_for (const chrono::duration<Rep,Period>& rel_time) const;
```

而与 `std::future::wait()` 不同的是，`wait_for()` 可以设置一个时间段 `rel_time`，如果共享状态的标志在该时间段结束之前没有被 `Provider` 设置为 `ready`，则调用 `wait_for` 的线程被阻塞，在等待了 `rel_time` 的时间长度后 `wait_until()` 返回，返回值如下：

| 返回值                    | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| `future_status::ready`    | 共享状态的标志已经变为 `ready`，即 `Provider` 在共享状态上设置了值或者异常。 |
| `future_status::timeout`  | 超时，即在规定的时间内共享状态的标志没有变为 `ready`。       |
| `future_status::deferred` | 共享状态包含一个 `deferred` 函数，表示异步操作还未开始。     |

```c++
// future example
#include <iostream>             // std::cout
#include <future>               // std::async, std::future
#include <chrono>               // std::chrono::milliseconds

// 一种判断是否为质数的非优化方法
bool is_prime(long long x)
{
    for (int i = 2; i < x; ++i)
        if (x % i == 0)
            return false;
    return true;
}

int main()
{
    // 异步调用函数
    std::future<bool> fut = std::async(is_prime, 444444443);
    // 在等待函数设置 future 的状态时输出 .
    std::cout << "checking, please wait ";
    while (fut.wait_for(std::chrono::milliseconds(100)) == std::future_status::timeout){
        std::cout << '.';
    }
    bool x = fut.get();         // 接受返回值
    std::cout << "\n444444443 " << (x ? "is" : "is not") << " prime.\n";
    return 0;
}

输出结果：
checking, please wait .......
444444443 is prime.
```

### `std::future::wait_until()`

与 `std::future::wait()` 的功能类似，即等待与该 `std::future` 对象相关联的共享状态的标志变为 `ready`，该函数原型如下：

```c++
template <class Rep, class Period>
  future_status wait_until (const chrono::time_point<Clock,Duration>& abs_time) const;
```

而 与 `std::future::wait()` 不同的是，`wait_until()` 可以设置一个系统绝对时间点 `abs_time`，如果共享状态的标志在该时间点到来之前没有被 `Provider` 设置为 `ready`，则调用 `wait_until` 的线程被阻塞，在 `abs_time` 这一时刻到来之后 `wait_for()` 返回，返回值与 `wait_for()` 一致。

```c++
#include <stdio.h>
#include <stdlib.h>
#include <cmath>
#include <chrono>
#include <future>
#include <iostream>

double ThreadTask(int n) {
    std::cout << std::this_thread::get_id() << " start computing..." << std::endl;

    double ret = 0;
    for (int i = 0; i <= n; i++) {
        ret += std::sin(i);
    }

    std::cout << std::this_thread::get_id() << " finished computing..." << std::endl;
    return ret;
}

int main(int argc, const char *argv[])
{
    std::future<double> f(std::async(std::launch::async, ThreadTask, 100000000));

    while(f.wait_until(std::chrono::system_clock::now() + std::chrono::seconds(1)) != std::future_status::ready) {
        std::cout << "task is running...\n";
    }

    std::cout << f.get() << std::endl;
    return 0;
}
```

## 2、与 `std::future` 相关的函数

### `std::shared_future` 

`std::shared_future` 与 `std::future` 类似，但是 `std::shared_future` 可以拷贝、多个 `std::shared_future` 可以共享某个共享状态的最终结果(即共享状态的某个值或者异常)。`shared_future` 可以通过`std::future`对象隐式转换（参见`std::shared_future`的构造函数），或者通过`std::future::share()`显示转换，无论哪种转换，被转换的那个 `std::future` 对象都会变为 `not-valid`。

**std::shared_future 构造函数**

`std::shared_future` 共有四种构造函数，如下表所示：

| `default`               | `shared_future() noexcept; `                       |
| ----------------------- | -------------------------------------------------- |
| **`copy`**              | **`shared_future (const shared_future& x); `**     |
| **`move`**              | **`shared_future (shared_future&& x) noexcept; `** |
| **`move from future `** | **`shared_future (future<T>&& x) noexcept;`**      |

最后 `move from future` 即从一个有效的 `std::future` 对象构造一个 `std::shared_future`，构造之后 `std::future` 对象 `x` 变为无效`(not-valid)`。

**std::shared_future 其他成员函数**

`std::shared_future` 的成员函数和 `std::future` 大部分相同，如下：

> - operator=
>
>   赋值操作符，与 std::future 的赋值操作不同，std::shared_future 除了支持 move 赋值操作外，还支持普通的赋值操作。 
>
> - get
>
>   获取与该 std::shared_future 对象相关联的共享状态的值（或者异常）。
>
> - valid
>
>   有效性检查。
>
> - wait
>
>   等待与该 std::shared_future 对象相关联的共享状态的标志变为 ready。
>
> - wait_for
>
>   等待与该 std::shared_future 对象相关联的共享状态的标志变为 ready。（等待一段时间，超过该时间段wait_for 返回。）
>
> - wait_until
>
>   等待与该 std::shared_future 对象相关联的共享状态的标志变为 ready。（在某一时刻前等待，超过该时刻 wait_until 返回。）

### `std::future_error` 

```c++
class future_error : public logic_error;
```

`std::future_error` 继承自 `C++` 标准异常体系中的 `logic_error`。

## 3、其他相关的函数

与 `std::future` 相关的函数主要是 `std::async()`，原型如下：

| `unspecified policy ` | `template <class Fn, class... Args>  future<typename result_of<Fn(Args...)>::type>  async(Fn&& fn, Args&&... args); ` |
| --------------------- | ------------------------------------------------------------ |
| **`specific policy`** | **`template <class Fn, class... Args>  future<typename result_of<Fn(Args...)>::type>  async(launch policy, Fn&& fn, Args&&... args); `** |

上面两组 `std::async()` 的不同之处是第一类 `std::async` 没有指定异步任务（即执行某一函数）的启动策略`(launch policy)`，而第二类函数指定了启动策略，详见 `std::launch` 枚举类型，指定启动策略的函数的 `policy` 参数可以是`launch::async`，`launch::deferred`，以及两者的按位或`( | )`。

`std::async()` 的 `fn` 和 `args` 参数用来指定异步任务及其参数。另外，`std::async()` 返回一个 `std::future` 对象，通过该对象可以获取异步任务的值或异常（如果异步任务抛出了异常）。

```c++
#include <stdio.h>
#include <stdlib.h>
#include <cmath>
#include <chrono>
#include <future>
#include <iostream>

double ThreadTask(int n) {
    std::cout << std::this_thread::get_id() << " start computing..." << std::endl;

    double ret = 0;
    for (int i = 0; i <= n; i++) {
        ret += std::sin(i);
    }

    std::cout << std::this_thread::get_id() << " finished computing..." << std::endl;
    return ret;
}

int main(int argc, const char *argv[])
{
    std::future<double> f(std::async(std::launch::async, ThreadTask, 100000000));

#if 0
    while(f.wait_until(std::chrono::system_clock::now() + std::chrono::seconds(1)) != std::future_status::ready) {
        std::cout << "task is running...\n";
    }
#else
    while(f.wait_for(std::chrono::seconds(1)) != std::future_status::ready) {
        std::cout << "task is running...\n";
    }
#endif

    std::cout << f.get() << std::endl;
    return EXIT_SUCCESS;
}
```

## 4、其他相关的枚举类

下面介绍与 `std::future` 相关的枚举类型。与 `std::future` 相关的枚举类型包括：

```
enum class future_errc;
enum class future_status;
enum class launch;
```

下面分别介绍以上三种枚举类型：

**`std::future_errc` 类型**

`std::future_errc` 类型描述如下：

| 类型                        | `取值` | 描述                                                         |
| --------------------------- | ------ | ------------------------------------------------------------ |
| `broken_promise`            | `0`    | 与该 `std::future` 共享状态相关联的 `std::promise` 对象在设置值或者异常之前一被销毁。 |
| `future_already_retrieved`  | `1`    | 与该`std::future`对象相关联的共享状态的值已经被当前`Provider`获取，即调用了 `std::future::get` 函数。 |
| `promise_already_satisfied` | `2`    | `std::promise` 对象已经对共享状态设置了某一值或者异常。      |
| `no_state`                  | `3`    | 无共享状态。                                                 |

**`std::future_status` 类型**

`std::future_status` 类型主要用在 `std::future`(或`std::shared_future`)中的 `wait_for` 和 `wait_until` 两个函数中的。

| 类型                      | `取值` | 描述                                                         |
| ------------------------- | ------ | ------------------------------------------------------------ |
| `future_status::ready`    | `0`    | `wait_for`(或`wait_until`) 因为共享状态的标志变为 `ready` 而返回。 |
| `future_status::timeout`  | `1`    | 超时，即 `wait_for`(或`wait_until`) 在指定的时间段（或时刻）内共享状态的标志依然没有变为 `ready` 而返回。 |
| `future_status::deferred` | `2`    | 共享状态包含了 `deferred` 函数。                             |

**`std::launch` 类型**

该枚举类型主要是在调用 `std::async` 设置异步任务的启动策略的。

| 类型               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| `launch::async`    | **Asynchronous:** 异步任务会在另外一个线程中调用，并通过共享状态返回异步任务的结果（一般是调用 `std::future::get()` 获取异步任务的结果）。 |
| `launch::deferred` | **Deferred:** 异步任务将会在共享状态被访问时调用，相当于 按需调用（即延迟(`deferred`)调用）。 |

```c++
#include <iostream>              // std::cout
#include <future>                // std::async, std::future, std::launch
#include <chrono>                // std::chrono::milliseconds
#include <thread>                // std::this_thread::sleep_for

void do_print_ten(char c, int ms)
{
    for (int i = 0; i < 10; ++i) {
        std::this_thread::sleep_for(std::chrono::milliseconds(ms));
        std::cout << c;
    }
}

int main()
{
    // launch::async在声明的时候就会创建线程去执行函数，其会自动阻塞等待线程结束，get直接获取其结果
    std::cout << "with launch::async:\n";
    std::future<void> foo = std::async(std::launch::async, do_print_ten, '*', 100);
    std::future<void> bar = std::async(std::launch::async, do_print_ten, '@', 200);
    // async "get" (wait for foo and bar to be ready):
    foo.get();
    bar.get();
    std::cout << "\n\n";

    // launch::deferred会将线程推迟到调用get方法时才会调用，等待线程结束获取到结果之后再执行get后面的程序如果不使用get或wait，就不会执行绑定的函数
    std::cout << "with launch::deferred:\n";
    foo = std::async(std::launch::deferred, do_print_ten, '*', 100);
    bar = std::async(std::launch::deferred, do_print_ten, '@', 200);
    // deferred "get" (perform the actual calls):
    foo.get();
    bar.get();
    std::cout << '\n';

    return 0;
}

输出结果：
with launch::async:
*@**@**@**@**@*@@@@@
with launch::deferred:
**********@@@@@@@@@@
```

# 二、`std::promise` 简介

`promise` 对象可以保存某一类型 `T` 的值，这个值可以是 `void` 类型，该值可被 `future` 对象读取（可能在另外一个线程中），因此 `promise` 也提供了一种线程同步的手段。在 `promise` 对象构造时可以和一个共享状态（通常是`std::future`）相关联，并可以在相关联的共享状态(`std::future`)上保存一个类型为 `T` 的值。

可以通过 `get_future` 来获取与该 `promise` 对象相关联的 `future` 对象，调用该函数之后，两个对象共享相同的共享状态(`shared state`)

> - `promise` 对象是异步 `Provider`，它可以在某一时刻设置共享状态的值。
> - `future` 对象可以异步返回共享状态的值，或者在必要的情况下阻塞调用者并等待共享状态标志变为 `ready`，然后才能获取共享状态的值。

下面以一个简单的例子来说明上述关系

```c++
#include <iostream>       // std::cout
#include <functional>     // std::ref
#include <thread>         // std::thread
#include <future>         // std::promise, std::future

void print_int(std::future<int>& fut) {
    int x = fut.get(); // 获取共享状态的值.
    std::cout << "value: " << x << '\n'; // 打印 value: 10.
}

int main ()
{
    std::promise<int> prom; // 生成一个 std::promise<int> 对象.
    std::future<int> fut = prom.get_future(); // 和 future 关联.
    std::thread t(print_int, std::ref(fut)); // 将 future 交给另外一个线程t.
    prom.set_value(10); // 设置共享状态的值, 此处和线程t保持同步.
    t.join();
    return 0;
}
```

## 1、成员函数

### 构造函数

| **`default`**         | **`promise(); `**                                            |
| --------------------- | ------------------------------------------------------------ |
| **`with allocator`**  | **`template <class Alloc> promise (allocator_arg_t aa, const Alloc& alloc); `** |
| **`copy [deleted]` ** | **`promise (const promise&) = delete; `**                    |
| **`move`**            | **`promise (promise&& x) noexcept; `**                       |

> 1. 默认构造函数，初始化一个空的共享状态。
> 2. 带自定义内存分配器的构造函数，与默认构造函数类似，但是使用自定义分配器来分配共享状态。
> 3. 拷贝构造函数，被禁用。
> 4. 移动构造函数。

另外，`std::promise` 的 `operator=` 没有拷贝语义，即 `std::promise` 普通的赋值操作被禁用，`operator=` 只有 `move` 语义，所以 `std::promise` 对象是禁止拷贝的。

```c++
#include <iostream>       // std::cout
#include <thread>         // std::thread
#include <future>         // std::promise, std::future

std::promise<int> prom;

void print_global_promise () {
    std::future<int> fut = prom.get_future();
    int x = fut.get();
    std::cout << "value: " << x << '\n';
}

int main ()
{
    std::thread th1(print_global_promise);
    prom.set_value(10);
    th1.join();

    prom = std::promise<int>();    // prom 被move赋值为一个新的 promise 对象.

    std::thread th2 (print_global_promise);
    prom.set_value (20);
    th2.join();

    return 0;
}

输出结果：
value: 10
value: 20
```

### `std::promise::get_future`

该函数返回一个与 `promise` 共享状态相关联的 `future` *。*返回的 `future` 对象可以访问由 `promise` 对象设置在共享状态上的值或者某个异常对象。只能从 `promise` 共享状态获取一个 `future` 对象。在调用该函数之后，`promise` 对象通常会在某个时间点准备好(设置一个值或者一个异常对象)，如果不设置值或者异常，`promise` 对象在析构时会自动地设置一个 `future_error` 异常(`broken_promise`)来设置其自身的准备状态。上面的例子中已经提到了 `get_future`，此处不再重复。

### `std::promise::set_value`

| `generic template`    | `void set_value (const T& val); void set_value (T&& val); `  |
| --------------------- | ------------------------------------------------------------ |
| **`specializations`** | **`void promise<R&>::set_value (R& val);   // when T is a reference type (R&) void promise<void>::set_value (void);   // when T is void`** |

设置共享状态的值，此后 `promise` 的共享状态标志变为 `ready`。因此 `set_value` 只能被调用一次，多次调用会抛出 `std::future_error` 异常。

一个 `std::promise` 实例只能与一个 `std::future` 关联共享状态，如果 `promise` 直到销毁时，都未设置过任何值，则 `promise` 会在析构时自动设置为`std::future_error`，这会造成 `std::future.get` 抛出 `std::future_error` 异常。

`std::promise<void>` 是合法的，此时 `std::promise.set_value` 不接受任何参数，仅用于通知关联的 `std::future.get()` 解除阻塞。

```c++
#include <iostream>
#include <future>
#include <thread>
#include <exception>  // std::make_exception_ptr
#include <stdexcept>  // std::logic_error

void catch_error(std::future<void> &future) {
    try {
        future.get();
    } 
    catch (std::logic_error &e) {
        std::cerr << "logic_error: " << e.what() << std::endl;
    }
}

int main() {
    std::promise<void> promise;
    std::future<void> future = promise.get_future();

    std::thread thread(catch_error, std::ref(future));
    // 自定义异常需要使用make_exception_ptr转换一下
    promise.set_exception(std::make_exception_ptr(std::logic_error("caught")));
    
    thread.join();
    return 0;
}
// 输出：logic_error: caught
```

###  `std::promise::set_exception` 

为 `promise` 设置异常，此后 `promise` 的共享状态变标志变为 `ready`，例子如下，线程 `1` 从终端接收一个整数，线程 `2` 将该整数打印出来，如果线程 `1` 接收一个非整数，则为 `promise` 设置一个异常`(failbit)` ，线程 `2` 在`std::future::get` 是抛出该异常。

```c++
#include <iostream>       // std::cin, std::cout, std::ios
#include <functional>     // std::ref
#include <thread>         // std::thread
#include <future>         // std::promise, std::future
#include <exception>      // std::exception, std::current_exception

void get_int(std::promise<int>& prom) {
    int x;
    std::cout << "Please, enter an integer value: ";
    std::cin.exceptions (std::ios::failbit);   // throw on failbit
    try {
        std::cin >> x;                         // sets failbit if input is not int
        prom.set_value(x);
    } catch (std::exception&) {
        prom.set_exception(std::current_exception());
    }
}

void print_int(std::future<int>& fut) {
    try {
        int x = fut.get();
        std::cout << "value: " << x << '\n';
    } catch (std::exception& e) {
        std::cout << "[exception caught: " << e.what() << "]\n";
    }
}

int main ()
{
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();
    std::thread th1(get_int, std::ref(prom));
    std::thread th2(print_int, std::ref(fut));
    th1.join();
    th2.join();
    return 0;
}

输出结果1：
Please, enter an integer value: 10
value: 10    
输出结果2：    
Please, enter an integer value: a
[exception caught: basic_ios::clear: iostream error]
```

### `std::promise::set_value_at_thread_exit` 

设置共享状态的值，但是不将共享状态的标志设置为 `ready`，当线程退出时该 `promise` 对象会自动设置为 `ready`。如果某个 `std::future` 对象与该 `promise` 对象的共享状态相关联，并且该 `future` 正在调用 `get`，则调用 `get` 的线程会被阻塞，当线程退出时，调用 `future::get` 的线程解除阻塞，同时 `get` 返回 `set_value_at_thread_exit` 所设置的值。注意，该函数已经设置了 `promise` 共享状态的值，如果在线程结束之前有其他设置或者修改共享状态的值的操作，则会抛出 `future_error( promise_already_satisfied )`。

### `std::promise::swap` 

交换 `promise` 的共享状态。

# 三、`std::packaged_task` 简介

`std::packaged_task` 包装一个可调用的对象，并且允许异步获取该可调用对象产生的结果，从包装可调用对象意义上来讲，`std::packaged_task` 与 `std::function` 类似，只不过 `std::packaged_task` 将其包装的可调用对象的执行结果传递给一个 `std::future` 对象（该对象通常在另外一个线程中获取 `std::packaged_task` 任务的执行结果）。

`std::packaged_task` 对象内部包含了两个最基本元素：

> 被包装的任务`(stored task)`，任务`(task)`是一个可调用的对象，如函数指针、成员函数指针或者函数对象，
>
> 共享状态`(shared state)`，用于保存任务的返回值，可以通过 `std::future` 对象来达到异步访问共享状态的效果。

可以通过 `std::packged_task::get_future` 来获取与共享状态相关联的 `std::future` 对象。在调用该函数后，两个对象共享相同的共享状态，具体如下：

> - `std::packaged_task` 对象是异步 `Provider`，它在某一时刻通过调用被包装的任务来设置共享状态的值。
> - `std::future` 对象是一个异步返回对象，通过它可以获得共享状态的值，当然在必要的时候需要等待共享状态标志变为 `ready`.

`std::packaged_task` 的共享状态的生命周期一直持续到最后一个与之相关联的对象被释放或者销毁为止。用法如下例：

```c++
#include <iostream>     // std::cout
#include <future>       // std::packaged_task, std::future
#include <chrono>       // std::chrono::seconds
#include <thread>       // std::thread, std::this_thread::sleep_for

// count down taking a second for each value:
int countdown (int from, int to) {
    for (int i=from; i!=to; --i) {
        std::cout << i << ' ';
        std::this_thread::sleep_for(std::chrono::seconds(1));
    }
    std::cout << "\nFinished!\n";
    return from - to;
}

int main ()
{
    std::packaged_task<int(int,int)> task(countdown); // 设置 packaged_task
    std::future<int> ret = task.get_future(); // 获得与 packaged_task 共享状态相关联的 future 对象.
    std::thread th(std::move(task), 10, 0);   //创建一个新线程完成计数任务.
    int value = ret.get();                    // 等待任务完成并获取结果.
    std::cout << "The countdown lasted for " << value << " seconds.\n";
    th.join();
    return 0;
}

输出结果：
10 9 8 7 6 5 4 3 2 1 
Finished!
The countdown lasted for 10 seconds.
```

## 1、成员函数

### 构造函数

| **`default`**        | **`packaged_task() noexcept; `**                             |
| -------------------- | ------------------------------------------------------------ |
| **`initialization`** | **`template <class Fn>  explicit packaged_task (Fn&& fn); `** |
| **`with allocator`** | **`template <class Fn, class Alloc>  explicit packaged_task (allocator_arg_t aa, const Alloc& alloc, Fn&& fn); `** |
| **`copy [deleted]`** | **`packaged_task (const packaged_task&) = delete; `**        |
| **`move`**           | **`packaged_task (packaged_task&& x) noexcept;`**            |

`std::packaged_task` 构造函数共有 `5`  种形式，不过拷贝构造已经被禁用了。下面简单地介绍一下上述几种构造函数的语义：

> 1. 默认构造函数，初始化一个空的共享状态，并且该 `packaged_task` 对象无包装任务。
> 2. 初始化一个共享状态，并且被包装任务由参数 `fn` 指定。
> 3. 带自定义内存分配器的构造函数，与默认构造函数类似，但是使用自定义分配器来分配共享状态。
> 4. 拷贝构造函数，被禁用。
> 5. 移动构造函数。

下面例子介绍了各类构造函数的用法：

```c++
#include <iostream>     // std::cout
#include <utility>      // std::move
#include <future>       // std::packaged_task, std::future
#include <thread>       // std::thread

int main ()
{
    std::packaged_task<int(int)> foo; // 默认构造函数.
    // 使用 lambda 表达式初始化一个 packaged_task 对象.
    std::packaged_task<int(int)> bar([](int x){return x*2;});
    foo = std::move(bar); // move-赋值操作，也是 C++11 中的新特性.
    // 获取与 packaged_task 共享状态相关联的 future 对象.
    std::future<int> ret = foo.get_future();
    std::thread(std::move(foo), 10).detach(); // 产生线程，调用被包装的任务.
    int value = ret.get(); // 等待任务完成并获取结果.
    std::cout << "The double of 10 is " << value << ".\n";
    return 0;
}
```

与 `std::promise` 类似， `std::packaged_task` 也禁用了普通的赋值操作运算，只允许 `move` 赋值运算。

### `std::packaged_task::valid` 

检查当前 `packaged_task` 是否和一个有效的共享状态相关联，对于由默认构造函数生成的 `packaged_task` 对象，该函数返回 `false`，除非中间进行了 `move` 赋值操作或者 `swap` 操作。

请看下例：

```c++
#include <iostream>     // std::cout
#include <utility>      // std::move
#include <future>       // std::packaged_task, std::future
#include <thread>       // std::thread

// 在新线程中启动一个 int(int) packaged_task.
std::future<int> launcher(std::packaged_task<int(int)>& tsk, int arg)
{
    if (tsk.valid()) {
        std::future<int> ret = tsk.get_future();
        std::thread (std::move(tsk),arg).detach();
        return ret;
    }
    else {
        return std::future<int>();
    }
}

int main ()
{
    std::packaged_task<int(int)> tsk([](int x){return x*2;});
    std::future<int> fut = launcher(tsk,25);
    std::cout << "The double of 25 is " << fut.get() << ".\n";
    return 0;
}
```

### `std::packaged_task::get_future` 

返回一个与 `packaged_task` 对象共享状态相关的 `future` 对象。返回的 `future` 对象可以获得由另外一个线程在该 `packaged_task` 对象的共享状态上设置的某个值或者异常。

请看例子(其实前面已经讲了 `get_future` 的例子)：

```
#include <iostream>     // std::cout
#include <utility>      // std::move
#include <future>       // std::packaged_task, std::future
#include <thread>       // std::thread

int main ()
{
    std::packaged_task<int(int)> tsk([](int x) { return x * 3; })); // package task

    std::future<int> fut = tsk.get_future();   // 获取 future 对象.

    std::thread(std::move(tsk), 100).detach();   // 生成新线程并调用packaged_task.

    int value = fut.get();                     // 等待任务完成, 并获取结果.

    std::cout << "The triple of 100 is " << value << ".\n";

    return 0;
}
```

### `std::packaged_task::operator()(Args... args)`

调用该 `packaged_task` 对象所包装的对象(通常为函数指针，函数对象，`lambda` 表达式等)，传入的参数为 `args`. 调用该函数一般会发生两种情况：

> - 如果成功调用 `packaged_task` 所包装的对象，则返回值（如果被包装的对象有返回值的话）被保存在 `packaged_task` 的共享状态中。
> - 如果调用 `packaged_task` 所包装的对象失败，并且抛出了异常，则异常也会被保存在 `packaged_task` 的共享状态中。

以上两种情况都使共享状态的标志变为 `ready`，因此其他等待该共享状态的线程可以获取共享状态的值或者异常并继续执行下去。

共享状态的值可以通过在 `future` 对象(由 `get_future`获得)上调用 `get` 来获得。

由于被包装的任务在 `packaged_task` 构造时指定，因此调用 `operator()` 的效果由 `packaged_task` 对象构造时所指定的可调用对象来决定：

> - 如果被包装的任务是函数指针或者函数对象，调用 `std::packaged_task::operator()` 只是将参数传递给被包装的对象。
> - 如果被包装的任务是指向类的非静态成员函数的指针，那么 `std::packaged_task::operator()` 的第一个参数应该指定为成员函数被调用的那个对象，剩余的参数作为该成员函数的参数。
> - 如果被包装的任务是指向类的非静态成员变量，那么 `std::packaged_task::operator()` 只允许单个参数。

### `std::packaged_task::make_ready_at_thread_exit`

该函数会调用被包装的任务，并向任务传递参数，类似 std::packaged_task 的 operator() 成员函数。但是与 operator() 函数不同的是，make_ready_at_thread_exit 并不会立即设置共享状态的标志为 ready，而是在线程退出时设置共享状态的标志。

如果与该 packaged_task 共享状态相关联的 future 对象在 future::get 处等待，则当前的 future::get 调用会被阻塞，直到线程退出。而一旦线程退出，future::get 调用继续执行，或者抛出异常。

注意，该函数已经设置了 promise 共享状态的值，如果在线程结束之前有其他设置或者修改共享状态的值的操作，则会抛出 future_error( promise_already_satisfied )。

### `std::packaged_task::reset()` 

重置 `packaged_task` 的共享状态，但是保留之前的被包装的任务。请看例子，该例子中，`packaged_task` 被重用了多次：

```c++
// packaged_task::get_future
#include <iostream>     // std::cout
#include <utility>      // std::move
#include <future>       // std::packaged_task, std::future
#include <thread>       // std::thread

// a simple task:
int triple (int x) { return x*3; }

int main ()
{
    std::packaged_task<int(int)> tsk (triple); // package task

    std::future<int> fut = tsk.get_future();
    tsk(33);
    std::cout << "The triple of 33 is " << fut.get() << ".\n";
    // re-use same task object:
    tsk.reset();
    fut = tsk.get_future();
    std::thread(std::move(tsk),99).detach();
    std::cout << "The triple of 99 is " << fut.get() << ".\n";
    return 0;
}
```

### `std::packaged_task::swap()`

交换 `packaged_task` 的共享状态。