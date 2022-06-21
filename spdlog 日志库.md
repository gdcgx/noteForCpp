# 一. 概述

`spdlog`日志库自身带有包括控制台日志记录、基础文件日志记录、循环文件日志记录、每日文件日志记录等在内的日志记录方式，能满足日常不同的情景需求。
本文主要介绍`spdlog`日志库的基本使用，包括创建日志记录器`（logger）`、创建日志记录器槽`（sink）`、设置日志输出内容格式`（pattern）`、设置日志输出等级`（level）`等。

# 二. 基础概念

## 2.1、level_enum

日志等级枚举，包括 `trace`、`info`、`debug`、`warn`、`err`、`critical`、`off`、`n_levels`。

```c_cpp
enum level_enum
{
    trace = SPDLOG_LEVEL_TRACE,
    debug = SPDLOG_LEVEL_DEBUG,
    info = SPDLOG_LEVEL_INFO,
    warn = SPDLOG_LEVEL_WARN,
    err = SPDLOG_LEVEL_ERROR,
    critical = SPDLOG_LEVEL_CRITICAL,
    off = SPDLOG_LEVEL_OFF,
    n_levels
};
```

## 2.2、sink

日志记录器槽，进行底层操作（比如格式化内容、输出内容到控制台/文件）的类。

`sink`是实际将日志写入目标位置的对象。每一个`sink`仅应负责写一个目标文件（比如 `file`，`console`，`db`），并且每一个`sink`有专属的私有格式化器`formatter`实例。

`sink`类主要使用的函数包括：

> set_pattern(const std::string&) ：设置日志输出的内容格式。
> 
> set_level(level_enum) ： 设置日志输出的最低等级。
> 
> log(log_msg) ：由logger自动调用，外部不会主动调用。

### 可用的sink

#### 1> rotating_file_sink

达到最大文件大小时，关闭文件，重命名文件并创建新文件。 最大文件大小和最大文件数都可以在构造函数中配置。

> 注意：用户应该负责去创建任何他们需要的文件夹，spdlog除了文件不会尝试创建任何文件夹。（旧版本）
> 
> 目前的新版本spdlog会创建文件夹

```c_cpp
// create a thread safe sink which will keep its file size to a maximum of 5MB and a maximum of 3 rotated files.
#include "spdlog/sinks/rotating_file_sink.h"
...
auto file_logger = spdlog::rotating_logger_mt("file_logger", "logs/mylogfile", 1048576 * 5, 3);
```

或者手动创建 `sink` 并将它传递给 `logger` ：

```c_cpp
#include "spdlog/sinks/rotating_file_sink.h"
...
auto rotating = make_shared<spdlog::sinks::rotating_file_sink_mt> ("log_filename", "log", 1024*1024, 5, false);
auto file_logger = make_shared<spdlog::logger>("my_logger", rotating);
```

#### 2> daily_file_sink

每天在一个特别的时间创建一个新的日志文件，并在文件名字上添加一个时间戳

> 参数设置为14，55代表将会在每天的14:55创建一个新的日志文件，并创建一个线程安全的sink

```c_cpp
#include "spdlog/sinks/daily_file_sink.h"
..
auto daily_logger = spdlog::daily_logger_mt("daily_logger", "logs/daily", 14, 55);
```

#### 3> simple_file_sink

无任何限制的向一个日志文件中写入

```c_cpp
#include "spdlog/sinks/basic_file_sink.h"
...
auto logger = spdlog::basic_logger_mt("mylogger", "log.txt");
```

#### 4> stdout_sink/stderr_sink with colors

```c_cpp
#include "spdlog/sinks/stdout_color_sinks.h"
...
auto console = spdlog::stdout_color_mt("console");
auto err_console = spdlog::color_logger_mt("console");
```

或者直接创建 `sink` ：

```c_cpp
auto sink = std::make_shared<spdlog::sinks::stdout_color_sink_mt>();
```

#### 5> ostream_sink

```c_cpp
#include "spdlog/sinks/ostream_sink.h "
...
std::ostringstream oss;
auto ostream_sink = std::make_shared<spdlog::sinks::ostream_sink_mt> (oss);
auto logger = std::make_shared<spdlog::logger>("my_logger", ostream_sink);
```

#### 6> null_sink

会丢弃所有到它的日志

```c_cpp
#include "spdlog/sinks/null_sink.h"
#### ...
auto logger = spdlog::create<spdlog::sinks::null_sink_st>("null_logger");
```

#### 7> syslog_sink

`POSIX syslog(3)` 发送日志到 `syslog`

```c_cpp
#include "spdlog/sinks/syslog_sink.h"
...
auto syslog_logger = spdlog::syslog_logger("syslog", "my_ident");
```

#### 8> dist_sink

将日志消息分发到其他接收器列表

```c_cpp
#include "spdlog/sinks/syslog_sink.h"
...
auto dist_sink = make_shared<spdlog::sinks::dist_sink_st>();
auto sink1 = make_shared<spdlog::sinks::stdout_sink_st>();
auto sink2 = make_shared<spdlog::sinks::simple_file_sink_st>("mylog.log");
dist_sink->add_sink(sink1);
dist_sink->add_sink(sink2);
```

#### 9> msvc_sink

`Windows debug sink` (使用 `OutputDebugStringA` 窗口输出日志)

```c_cpp
#include "spdlog/sinks/msvc_sink.h"
auto sink = std::make_shared<spdlog::sinks::msvc_sink_mt>();
auto logger = std::make_shared<spdlog::logger>("msvc_logger", sink);
```

### 实现自己的sink

> 实现自己的sink，需要实现sink类的接口
> 
> 一种推荐的方式是继承自base_sink类
> 
> 该类已经处理了线程锁，使得实现一个线程安全的sink非常容易

```c_cpp
#include "spdlog/sinks/base_sink.h"

template<typename Mutex>
class my_sink : public spdlog::sinks::base_sink <Mutex>
{
...
protected:
    void sink_it_(const spdlog::details::log_msg& msg) override
    {

    // log_msg is a struct containing the log entry info like level, timestamp, thread id etc.
    // Log_msg是一个包含日志条目信息的结构体，如级别、时间戳、线程id等。
    // msg.raw contains pre formatted log
    // msg.raw 原始包含预格式化的日志
    
    // If needed (very likely but not mandatory), the sink formats the message before sending it to its final destination:
    // 如果需要(很有可能但不是强制的)，接收器会在将消息发送到最终目的地之前对其进行格式化:
    fmt::memory_buffer formatted;
    sink::formatter_->format(msg, formatted);
    std::cout << fmt::to_string(formatted);
    }

    void flush_() override 
    {
       std::cout << std::flush;
    }
};

#include "spdlog/details/null_mutex.h"
#include <mutex>
using my_sink_mt = my_sink<std::mutex>;
using my_sink_st = my_sink<spdlog::details::null_mutex>;
```

## 2.3、logger

日志记录器，被顶层调用来输出日志的类。一个`logger`对象中存储有多个`sink`，当调用`logger`的日志输出函数时，`logger`会调用自身存储的所有`sink`对象的`log`(`log_msg`) 函数进行输出。与自带的`sink`对应，`spdlog`也自带了几种`logger`。`logger`类主要使用的函数包括：

```c_cpp
// 设置logger包含的所有sink的日志输出内容格式
set_pattern(const std::string&)

// 设置logger日志输出最低等级，如果logger包含的sink没有设置日志等级的话，则会为其设置日志等级
set_level(level_enum)

// 按照level等级进行输出content，logger其中日志输出最低等级小于或等于level的sink会进行执行输出操作
log(level_enum level,log_msg content) 

// 同上面的log，与其不同的是，将loc设置为spdlog::source_loc{__FILE__, __LINE__, SPDLOG_FUNCTION}
// 可以比上面的log多输出一个文件+打印日志的行号
log(source_loc loc, level::level_enum lvl, string_view_t msg)

// 按照trace等级进行输出，输出内容由content与后面的参数格式化而成。同类的函数还包括：debug/info/warn…
trace(content,arg1,arg2…) 
```

## 2.4、st/mt

对象版本，`spdlog`中的`logger`对象和`sink`对象都有两种版本，一种是以`st`结尾的单线程版本，以及以`mt`结尾的多线程版本。

> st：单线程版本，不用加锁，效率更高。
> 
> mt：多线程版本，用于多线程程序是线程安全的。


# 三. 使用示例

`spdlog` 是个只有头文件的库，只需要将头文件拷贝到你的工程就可以使用了，编译器需要支持 `C++11` ,它使用一个类似 `python` 的格式 `API`库 `fmt`：

```c_cpp
logger->info("Hello {} {} !!", "param1", 123.4);
```

`spdlog` 支持使用最小集的方式，意味着你只用包含你实际需要的头文件，而不是全部，比如说你只需要使用 `rotating logger`，那么只需要

```c_cpp
#include <spdlog/sinks/rotating_file_sink.h>
```

对于异步特性，还需要

```c_cpp
#include <spdlog/asynch.h>
```

## 3.1、创建logger

> 每一个 `logger` 中包含一个存有一个或多个 `std::shared_ptr<spdlog::sink>` 的 `vector`
> 
> `logger` 在记录每一条日志时（如果是有效的级别），将会调用每一个 `std::shared_ptr<spdlog::sink>` 中的 `sink(log_msg)` 函数
> 
> `spdlog` 用 `_mt`(多线程) 和 `_st`(单线程) 后缀标识`sink`是否是线程安全
> 
> 单线程的 `sink` 不可以在多线程中使用，它的速度会更快，因为没有锁竞争


### （1）使用spdlog自带的logger

```c_cpp
 //<1.创建名称为LoggerName1、内容输出到控制台的单线程版本日志记录器,将自身注册到spdlog，返回自身的智能指针
auto logger1 = spdlog::stdout_color_st("LoggerName1");

//<2.创建名称为LoggerName2、内容输出到Logs/BasicFileLog.txt的多线程版本日志记录器
auto logger2 = spdlog::basic_logger_mt("LoggerName2", "Logs/BasicFileLog.txt");

//<3.创建名称为LoggerName3、内容输出到Logs/RotatingFileLog.txt的多线程版本日志记录器
//参数1024*1024*5设置了文件最大容量为5mb，参数3设置了文件最大数量为3
//当日志文件存储超过5mb时，将重命名日志文件并且新增另外一个日志文件
//当日志文件数量超过3时，将删除第一个日志文件
auto logger3 = spdlog::rotating_logger_mt("LoggerName3", "Logs/RotatingFileLog.txt", 1024 * 1024 * 5, 3);

//<4.创建名称为LoggerName4、内容输出到Logs/DailyFileLog.txt的多线程版本日志记录器
//参数2和30指定每天生成日志文件的时间为凌晨2点30分
auto logger4 = spdlog::daily_logger_mt("LoggerName4", "Logs/DailyFileLog.txt", 2, 30);
```

### （2）使用sink创建logger

```c_cpp
//<1.创建名称为LoggerName1、内容输出到控制台的单线程版本日志记录器
auto logger1 = std::make_shared<spdlog::logger>("LoggerName1", 
      std::make_shared<spdlog::sinks::stdout_color_sink_st>());

//<2.创建名称为LoggerName2、内容输出到Logs/BasicFileLog.txt的多线程版本日志记录器
auto logger2 = std::make_shared<spdlog::logger>("LoggerName2", 
      std::make_shared<spdlog::sinks::basic_file_sink_mt>("Logs/BasicFileLog.txt"));

//<3.创建名称为LoggerName3、内容输出到Logs/RotatingFileLog.txt的多线程版本日志记录器
//参数1024*1024*5设置了文件最大容量为5mb，参数3设置了文件最大数量为3
//当日志文件存储超过5mb时，将重命名日志文件并且新增另外一个日志文件
//当日志文件数量超过3时，将删除第一个日志文件
auto logger3 = std::make_shared<spdlog::logger>("LoggerName3", 
      std::make_shared<spdlog::sinks::rotating_file_sink_mt>("Logs/RotatingFileLog.txt",1024 * 1024 * 5, 3));

//<4.创建名称为LoggerName4、内容输出到Logs/DailyFileLog.txt的多线程版本日志记录器
//参数2和30指定每天生成日志文件的时间为凌晨2点30分
auto logger4 = std::make_shared<spdlog::logger>("LoggerName4", 
      std::make_shared<spdlog::sinks::daily_file_sink_mt>("Logs/DailyFileLog.txt", 2,30));
```

### （3）创建使用同个sink的多个logger

```c_cpp
#include <iostream>
#include "spdlog/spdlog.h"
#include "spdlog/sinks/daily_file_sink.h"
int main(int, char* [])
{
    try
    {
        //<1.创建sink
        auto daily_sink = std::make_shared<spdlog::sinks::daily_file_sink_mt>("RotatingFileLog.txt", 23, 59);
        //<2.创建共同使用sink的多个logger，这些logger把内容一起输出到RotatingFileLog.txt中
        auto net_logger = std::make_shared<spdlog::logger>("net", daily_sink);
        auto hw_logger  = std::make_shared<spdlog::logger>("hw",  daily_sink);
        auto db_logger  = std::make_shared<spdlog::logger>("db",  daily_sink);      

        net_logger->set_level(spdlog::level::critical); // independent levels
        hw_logger->set_level(spdlog::level::debug);
         
        // 全局注册日志记录器，这样就可以使用spdlog::get(logger_name)访问日志记录器。
        spdlog::register_logger(net_logger);
    }
    catch (const spdlog::spdlog_ex& ex)
    {
        std::cout << "Log initialization failed: " << ex.what() << std::endl;
    }
}
```

### （4）创建使用多个sink的单个logger

```c_cpp
//
// Logger with console and file output.
// the console will show only warnings or worse, while the file will log all messages.
// 
#include <iostream>
#include "spdlog/spdlog.h"
#include "spdlog/sinks/stdout_color_sinks.h" // or "../stdout_sinks.h" if no colors needed
#include "spdlog/sinks/basic_file_sink.h"
int main(int, char* [])
{
    try
    {
        //<1.创建多个sink
        auto console_sink = std::make_shared<spdlog::sinks::stdout_color_sink_mt>();
        console_sink->set_level(spdlog::level::warn);
        console_sink->set_pattern("[multi_sink_example] [%^%l%$] %v");

        auto file_sink = std::make_shared<spdlog::sinks::basic_file_sink_mt>("logs/multisink.txt", true);
        file_sink->set_level(spdlog::level::trace);
        
        //<2.创建使用多个sink的单个logger，logger会把内容输出到不同位置，此处是控制台以及multisink.txt
        //std::vector< spdlog::sink_ptr> sinks = { console_sink,file_sink };
        //auto logger = std::make_shared<spdlog::logger>("LoggerName", sinks.begin(), sinks.end());
        spdlog::logger logger("multi_sink", {console_sink, file_sink});
        logger.set_level(spdlog::level::debug);
        logger.warn("this should appear in both console and file");
        logger.info("this message should not appear in the console, only in the file");
    }
    catch (const spdlog::spdlog_ex& ex)
    {
        std::cout << "Log initialization failed: " << ex.what() << std::endl;
    }
}
```

### （5）使用工厂函数创建异步logger：

```c_cpp
#include "spdlog/async.h" //support for async logging.
#include "spdlog/sinks/basic_file_sink.h"

int main(int, char* [])
{
	try
	{        
		auto async_file = spdlog::basic_logger_mt<spdlog::async_factory>("async_file_logger", "logs/async_log.txt");
		for (int i = 1; i < 101; ++i)
		{
			async_file->info("Async message #{}", i);
		}
		// Under VisualStudio, this must be called before main finishes to workaround a known VS issue
		// 在VisualStudio下，必须在main完成之前调用这个函数，以解决一个已知的VS问题
		spdlog::drop_all(); 
	}
	catch (const spdlog::spdlog_ex& ex)
	{
		std::cout << "Log initialization failed: " << ex.what() << std::endl;
	}
}
```

### （6）创建异步logger并改变线程池设置

> 对于异步日志记录，`spdlog` 使用具有专用消息队列的共享全局线程池。
> 
> 为此，它在消息队列中创建固定数量的预分配的插槽（64位中每个插槽大约256个字节），并且可以使用 `spdlog::init_thread_pool(queue_size，backing_threads_count)` 进行修改。
> 
> 当尝试记录一条日志时，并且队列已满，那么调用默认会被阻塞，并且默认直到一个插槽可用时，或者立即移除队列中最旧的日志信息，并追加最新的日志信息（如果 `logger` 以` async_overflow_policy==overrun_oldest` 构造）

```c_cpp
#include "spdlog/async.h" //support for async logging.
#include "spdlog/sinks/basic_file_sink.h"
int main(int, char* [])
{
    try
    {                                        
        auto daily_sink = std::make_shared<spdlog::sinks::daily_file_sink_mt>("logfile", 23, 59);
       // default thread pool settings can be modified *before* creating the async logger:
       // 默认线程池设置可以在创建异步日志程序之前修改:
        spdlog::init_thread_pool(10000, 1); // queue with 10K items and 1 backing thread.队列中有10K个条目和1个后台线程。
        auto async_file = spdlog::basic_logger_mt<spdlog::async_factory>("async_file_logger", "logs/async_log.txt");       
        spdlog::drop_all(); 
    }
    catch (const spdlog::spdlog_ex& ex)
    {
        std::cout << "Log initialization failed: " << ex.what() << std::endl;
    }
}
```

## 3.2、设置logger

### （1）设置pattern控制输出内容格式，设置level控制输出日志等级

每一个`logger`的`sink`都有一个格式化器，用来格式化消息为目标格式，`spdlog`默认的日志格式为：

```
[2019-04-18 13:31:59.678] [info] [my_loggername] Some message
```

有两种方式可以自定义`logger`的格式：

> 实现自定义格式器，实现`formatter`接口，并调用

```
set_formatter(std::make_shared<my_custom_formatter>());
```

```c_cpp
#include "spdlog/pattern_formatter.h"
class my_formatter_flag : public spdlog::custom_flag_formatter
{
public:
    void format(const spdlog::details::log_msg &, const std::tm &, spdlog::memory_buf_t &dest) override
    {
        std::string some_txt = "custom-flag";
        dest.append(some_txt.data(), some_txt.data() + some_txt.size());
    }

    std::unique_ptr<custom_flag_formatter> clone() const override
    {
        return spdlog::details::make_unique<my_formatter_flag>();
    }
};

void custom_flags_example()
{
    auto formatter = spdlog::details::make_unique::make_unique<spdlog::pattern_formatter>(); // for pre c++14
    formatter->add_flag<my_formatter_flag>('*').set_pattern("[%n] [%*] [%^%l%$] %v");
    auto logger = spdlog::stdout_color_st("LoggerName1");
    logger->set_formatter(std::move(formatter));
    logger->info("hello world");
}
```

> 设置模式字符串（推荐）: `set_pattern(pattern_string);`

```c_cpp
 //<1.创建多个sink
auto sink1 = std::make_shared<spdlog::sinks::stdout_color_sink_mt>();
auto sink2 = std::make_shared<spdlog::sinks::rotating_file_sink_mt>("Logs/RotatingFileLog.txt", 1024 * 1024 * 5, 3);
std::vector< spdlog::sink_ptr> sinks = { sink1,sink2 };

//<2.创建使用多个sink的单个logger，logger会把内容输出到不同位置，此处是控制台以及RotatingFileLog.txt
auto logger = std::make_shared<spdlog::logger>("LoggerName", sinks.begin(), sinks.end());

//<3.设置sink的pattern和level
spdlog::set_pattern("*** [%H:%M:%S %z] [thread %t] %v ***");//格式应用到所有被注册的logger
logger->set_pattern(">>>>>>>>> %H:%M:%S %z %v <<<<<<<<<");//格式应用到一个特定的logger对象
sink1->set_pattern("[%Y-%m-%d %H:%M:%S.%e][%l]>>>%v");//格式应用到一个特定的sink对象
sink1->set_level(spdlog::level::debug);
sink2->set_pattern("[%Y-%m-%d][%L]%v");
sink2->set_level(spdlog::level::info);
logger->sinks()[0]->set_pattern(">>>>>>>>> %H:%M:%S %z %v <<<<<<<<<");
logger->sinks()[1]->set_pattern("..");

//<4.输出日志
logger->debug("The message will only be shown on stdout.");//因为日志等级为debug，所以只会调用sink1进行输出
logger->info("The message will only be shown on stdout.");//日志等级为info，sink1与sink2都会进行输出
```

### （2）刷新策略

缺省情况下，`spdlog` 允许基础 `libc` 在合适时刷新，也选用以下选项：

#### 手动刷新

```c_cpp
// logger将依次在每个sink上调用flush
logger->flush()

// 当前, 无需在关闭前显式调logger->flush()或spdlog::shutdown(), 因其会在程序退出时自动销毁。
// 但是, 若要在“立即”退出函数(abort或exit)前手动刷新所有异步looger，则需此前调spdlog::shutdown()
```

#### 基于级别刷新

```c_cpp
// 可设置触发自动刷新的最低日志级别, 例：每当记录错误或更严重的消息时刷新
my_logger->flush_on(spdlog::level::err); 
```

#### 基于间隔刷新

```c_cpp
// spdlg支持设置刷新间隔, 其由单个工作线程定期在每个logger上调用flush实现, 例：定期为所有注册的logger隔5秒刷新
spdlog::flush_every(std::chrono::seconds(5));
```

### （3）常用的pattern标记

|flag|meaning|example|
|:--:|:--:|:--:|
|%v|要记录的实际文本|“some user text”|
|%t|线程id|“1232”|
|%P|进程id|“3456”|
|%n|logger的名字|“some logger name”|
|%l|日志记录等级|“debug”, “info”...|
|%L|日志记录等级简写|"D","I"...|
|%a|缩写工作日的名字|“Thu”|
|%A|完整的工作日的名字|“Thursday”|
|%b|缩写月份的名字|“Aug”|
|%B|完整的月份的名字|“August”|
|%c|日期和时间表示|“Thu Aug 23 15:35:46 2014”|
|%C|两位数的年份|"22"|
|%Y|四位数的年份|"2022"|
|%D or %x|短的MM / DD / YY日期|"01/05/22"|
|%m|1-12月|"10"|
|%d|一个月的1-31号|"11"|
|%H|24小时制的小时|"23"|
|%I|12小时制的小时|"11"|
|%M|0-59分钟|"59"|
|%S|0-59秒|"58"|
|%e|毫秒部分，0-999|"123"|
|%f|微秒部分，0-999999|"123456"|
|%F|纳秒部分，0-999999999|"123456789"|
|%p|AM/PM|“AM”|
|%r|12小时时钟|“02:55:02 pm”|
|%R|24小时HH:MM时间，相当于%H:%M|“23:55”|
|%T or %X|ISO 8601时间格式(HH:MM:SS)，相当于%H:%M:%S|“23:55:59”|
|%z|ISO 8601从UTC时区偏移量([+/-]HH:MM)|“+02:00”|
|%E|纪元以来的秒数|“1528834770”|
|%i|消息序列号(默认禁用-编辑' tweak .h '使能)|“1154”|
|%%|输出%符号|“%”|
|%+|spdlog的默认格式|“[2014-10-31 23:46:59.678] [mylogger] [info] message”|
|%^|开始颜色范围|“[mylogger] [info(green)] Some message”|
|%$|结束颜色范围(例如%^%l%$ %v)|[info(green)] Some message|
|%@|源文件和行|my_file.cpp:123|
|%s|源文件|my_file.cpp|
|%#|行|123|
|%!|源函数|my_func|

### （4）对齐

每一个模式标记可以通过预先添加一个宽度标记来对齐（上限128）

使用 `-`（左对齐）或者 `=`（中间对齐）去控制向哪边对齐：

|align|meaning|example|result|
|:--:|:--:|:--:|:--:|
|%<width><flag>|右对齐|%8l|"    info"|
|%-<width><flag>|左对齐|%-8l|"info    "|
|%=<width><flag>|中间对齐|%=8l|"  info  "|

## 3.3、使用logger

### （1）通过spdlog::get(std::string)获取指定名称的logger进行使用

要使用 `get` 获取 `logger` ，必须先 `register_logger`

`loggers` 可以在任何地方使用线程安全的 `spdlog::get("logger_name")` 来进行访问，返回智能指针

注意：spdlog::get可能会拖慢你的程序，因为它内部维护了一把锁，所以要谨慎使用。比较推荐的用法是保存返回的shared_ptr<spdlog::logger>，直接使用它，至少在频繁访问的代码中。

```c_cpp
//<1.创建logger
//A.cpp
auto logger1 = std::make_shared<spdlog::logger>("LoggerName1", 
          std::make_shared<spdlog::sinks::stdout_color_sink_mt>());
auto logger2 = std::make_shared<spdlog::logger>("LoggerName2", 
          std::make_shared<spdlog::sinks::stdout_color_sink_mt>());
//应注意get前需先register_logger, 因内含锁, 故建议存下返回值以利用
spdlog::register(logger2);
auto logger3 = spdlog::stdout_color_st("LoggerName3");

//<2.使用spdlog::get(std::string)获取logger
//B.cpp
auto logger1 = spdlog::get("LoggerName1");//logger1为空，因为logger1在A.cpp中没被注册
if (logger1)
{
    logger1->info("Hello world!");
}
auto logger2 = spdlog::get("LoggerName2");//logger2不为空，因为logger2在A.cpp中已被注册
if (logger2)
{
    logger2->info("Good night.");//输出
}
auto logger3 = spdlog::get("LoggerName3");//logger2不为空，因为spdlog自带的logger对象在构造时会自动注册
if (logger3)
{
    logger3->info("Good night.");//输出
}
```

### （2）通过spdlog::default_logger()调用默认的logger进行使用

```c_cpp
//<1.创建logger，并且将logger设置为默认的日志记录器
//A.cpp
auto logger = spdlog::stdout_color_mt("LoggerName");
spdlog::set_default_logger(logger);

//<2.使用spdlog::default_logger()获取默认的日志记录器
//B.cpp
auto logger = spdlog::default_logger();
if (logger)
{
    logger->info("Gigigigigi");//输出
}
//同样使用default_logger进行输出，更加方便的调用方式；未作default_logger是否为空的判断，会有程序中断的可能。
spdlog::info("Taco Tuesday");
```

### （3）使用宏调用默认的logger的方法进行日志输出

在包含 `*spdlog.h` 之前，添加 `SPDLOG_ACTIVE_LEVEL `宏定义可以设置期望的日志级别

```c_cpp
#define SPDLOG_ACTIVE_LEVEL SPDLOG_LEVEL_INFO //只输出info级别以上的日志
#include <spdlog/spdlog.h>
SPDLOG_INFO("This message will be shown.");//输出
SPDLOG_DEBUG("This message will not be shown.");//不输出
SPDLOG_LOGGER_TRACE(file_logger , "Some trace message that will not be evaluated.{} ,{}", 1, 3.23);//不输出
SPDLOG_LOGGER_DEBUG(file_logger , "Some Debug message that will be evaluated.. {} ,{}", 1, 3.23);//不输出
//SPDLOG_DEBUG和SPDLOG_LOGGER_DEBUG都可以输出日志，区别在于后者可以指定logger
```

### （4）使用fmt库格式化字符串

```c_cpp
//[2021-04-13 22:06:21.219] [info] Support for string.For example,hello,Leborn Jame.
spdlog::info("Support for string.For example,hello,{}.", "Leborn Jame");

//[2021-04-13 22:07:48.865] [warning] Supprot for integer.For example,00000023.
spdlog::warn("Supprot for integer.For example,{:08d}.", 23);

//[2021-04-13 22:07:48.865] [error] Support for float.For example,1.2.
spdlog::error("Support for float.For example,{:3.2}.", 1.23456);

//[2021-04-13 22:07:48.866] [critical] Support for position parameter.For example,cat and dog.
spdlog::critical("Support for position parameter.For example,{1} and {0}.", "dog", "cat");
```

# 四.特殊操作

## 4.1、长数据 二进制转十六进制

```c_cpp
// Log binary data as hex.
// Many types of std::container<char> types can be used.
// Iterator ranges are supported too.
// Format flags:
// {:X} - print in uppercase.    // 用大写字母打印
// {:s} - don't separate each byte with space.    // 不用空格分隔每个字节
// {:p} - don't print the position on each line start.       // 不用在每行开始处打印位置信息
// {:n} - don't split the output to lines.        // 不用将输出拆分成几行
// {:a} - show ASCII if :n is not set             // 显示ASCII
#include "spdlog/fmt/bin_to_hex.h"
void binary_example()
{
    std::vector<char> buf(80);
    for (int i = 0; i < 80; i++)
    {
        buf.push_back(static_cast<char>(i & 0xff));
    }
    spdlog::info("Binary example: {}", spdlog::to_hex(buf));
    spdlog::info("Another binary example:{:n}", spdlog::to_hex(std::begin(buf), std::begin(buf) + 10));
    // more examples:
    spdlog::info("uppercase: {:X}", spdlog::to_hex(buf));    // 大写
    spdlog::info("uppercase, no delimiters: {:Xs}", spdlog::to_hex(buf));    // 大写 且 不留空格
    spdlog::info("uppercase, no delimiters, no position info: {:Xsp}", spdlog::to_hex(buf));    // 大写 且 不留空格 且 无位置信息
    spdlog::info("hexdump style: {:a}", spdlog::to_hex(buf));    // 显示ASCII
    spdlog::info("hexdump style, 20 chars per line {:a}", spdlog::to_hex(buf, 20));    // 显示ASCII, 20字符一行
}
```

运行结果：

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/9c9a0a750f481736c6c67da7382b7125.png)

## 4.2、定义logger后自定义宏调用输出

```c_cpp
#define LTrace(...)     myLog::getInstance()->getLogger()->log(spdlog::source_loc{__FILE__, __LINE__, SPDLOG_FUNCTION}, spdlog::level::trace, __VA_ARGS__)
#define LDebug(...)     myLog::getInstance()->getLogger()->log(spdlog::source_loc{__FILE__, __LINE__, SPDLOG_FUNCTION}, spdlog::level::debug, __VA_ARGS__)
#define LInfo(...)      myLog::getInstance()->getLogger()->log(spdlog::source_loc{__FILE__, __LINE__, SPDLOG_FUNCTION}, spdlog::level::info, __VA_ARGS__)
#define LWarn(...)      myLog::getInstance()->getLogger()->log(spdlog::source_loc{__FILE__, __LINE__, SPDLOG_FUNCTION}, spdlog::level::warn, __VA_ARGS__)
#define LError(...)     myLog::getInstance()->getLogger()->log(spdlog::source_loc{__FILE__, __LINE__, SPDLOG_FUNCTION}, spdlog::level::err, __VA_ARGS__)
#define LCritical(...)  myLog::getInstance()->getLogger()->log(spdlog::source_loc{__FILE__, __LINE__, SPDLOG_FUNCTION}, spdlog::level::critical, __VA_ARGS__)

void trace_example()
{
    auto sink_stdout = std::make_shared<spdlog::sinks::stdout_color_sink_mt>();
    sink_stdout->set_pattern("[%Y-%m-%d %H:%M.%S.%e][%n][%^%-8l%$][%s:%#] %v");
    auto sink_rotate = std::make_shared<spdlog::sinks::rotating_file_sink_mt>(filepath,1024*1024*5,3);
    sink_rotate->set_pattern("[%Y-%m-%d %H:%M.%S.%e][%n][%-8l][%s:%#] %v");
    sinks = {sink_rotate,sink_stdout};
    auto m_logger = std::make_shared<spdlog::logger>(name,sinks.begin(),sinks.end());
    m_logger->set_level(spdlog::level::trace);
    m_logger->set_error_handler([](const std::string& msg){
      throw std::runtime_error(msg);
    });
    m_logger->flush_on(spdlog::level::warn);
    spdlog::register_logger(m_logger);
}

void test(){
    LTrace("dfhj{}","dfk");
    LDebug("dfhj{}","dfk");
    LWarn("dfhj{}","dfk");
    LInfo("dfhj{}","dfk");
    LError("dfhj{}","dfk");
    LCritical("dfhj{}","dfk");
}
```

## 4.3、查看运行时间

```c_cpp
// stopwatch example
#include "spdlog/stopwatch.h"
#include <thread>
void stopwatch_example()
{
    spdlog::stopwatch sw;
    std::this_thread::sleep_for(std::chrono::milliseconds(123));
    spdlog::info("Stopwatch: {} seconds", sw);
}
```

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/1a3b4c834fdb42e22a93b2d0bd52862a.png)

## 4.4、自定义类型来输出日志

```c_cpp
// User defined types logging by implementing operator<<
#include "spdlog/fmt/ostr.h" // must be included
struct my_type
{
    int i;
    template<typename OStream>
    friend OStream &operator<<(OStream &os, const my_type &c)
    {
        return os << "[my_type i=" << c.i << "]";
    }
};

void user_defined_example()
{
    spdlog::info("user defined type: {}", my_type{14});
}
```

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/b1e01e85511b7222cf083ae34c5d1277.png)

## 4.5、自定义错误处理程序【将在日志失败时触发】

```c_cpp
// 自定义错误处理程序, 将在日志失败时触发
void err_handler_example()
{
    // can be set globally or per logger(logger->set_error_handler(..))
    spdlog::set_error_handler([](const std::string &msg) { 
    	printf("*** Custom log error handler: %s ***\n", msg.c_str()); 
    });
}
```

# 五、单例模式简单封装spdlog

```c_cpp
/*************************************************************
 * 基于spdlog的简单封装，后续待完善
 * **********************************************************/
#include <spdlog/spdlog.h>
#include <spdlog/sinks/rotating_file_sink.h>
#include <spdlog/sinks/basic_file_sink.h>
#include <spdlog/sinks/daily_file_sink.h>
#include <spdlog/sinks/stdout_color_sinks.h>
#include <mutex>

class myLog{
public:
    static myLog* getInstance(){
        if(!m_mylog){
            std::lock_guard<std::mutex> locker(m_mtx);
            if(nullptr == m_mylog){
                m_mylog = new myLog();
            }
        } 
        return m_mylog;   
    }

    // spdlog日志初始化
    // @name : 日志名
    // @filepath : 生成日志的路径
    void init(std::string name,std::string filepath){
        try{
#ifdef STDOUT_FOR_DEBUG
            // 通过定义宏控制是否将日志打印到控制台输出，用于调试
            auto sink_stdout = std::make_shared<spdlog::sinks::stdout_color_sink_mt>();
            sink_stdout->set_pattern("[%Y-%m-%d %H:%M.%S.%e][%n][%^%-8l%$][%s:%!:%#] %v");
            // sink_stdout->set_level(spdlog::level::trace);
            sinks.emplace_back(sink_stdout);
#endif
            auto sink_rotate = std::make_shared<spdlog::sinks::rotating_file_sink_mt>(filepath,1024*1024*5,3);
            sink_rotate->set_pattern("[%Y-%m-%d %H:%M.%S.%e][%n][%-8l][%s:%!:%#] %v");
            // sink_rotate->set_level(spdlog::level::warn);
            sinks.emplace_back(sink_rotate);
            m_logger = std::make_shared<spdlog::logger>(name,sinks.begin(),sinks.end());

#ifdef STDOUT_FOR_DEBUG
            m_logger->set_level(spdlog::level::trace);
#else
            m_logger->set_level(spdlog::level::info);
#endif
            m_logger->set_error_handler([](const std::string& msg){
                throw std::runtime_error(msg);
            });
            m_logger->flush_on(spdlog::level::err);
            spdlog::register_logger(m_logger);
            spdlog::flush_every(std::chrono::seconds(3));
        }
        catch(const spdlog::spdlog_ex& ex){
            std::cout << "Log initialization failed: " << ex.what() << std::endl;
        }
    }

    // 刷新日志
    void flush(){
        m_logger->flush();
    }

    auto getLogger(){
        return m_logger;
    }

    static void unInstance(){
        if(m_mylog){
            std::lock_guard<std::mutex> locker(m_mtx);
            if(m_mylog){
                delete m_mylog;
                m_mylog = nullptr;
            }
        }
    }
    
protected:
    void stopLogger(){
        spdlog::shutdown();
    }

    myLog(){
        sinks.clear();
    }	

    ~myLog(){
        stopLogger();
    }

	myLog(const myLog&) = delete;
	myLog& operator=(const myLog&) = delete;
	
private:
    static myLog* m_mylog;
    static std::mutex m_mtx;
    std::shared_ptr<spdlog::logger> m_logger = nullptr;
    std::vector<spdlog::sink_ptr> sinks;
};

myLog* myLog::m_mylog = nullptr;
std::mutex myLog::m_mtx;

#define LTrace(...)     myLog::getInstance()->getLogger()->log(spdlog::source_loc{__FILE__, __LINE__, SPDLOG_FUNCTION}, spdlog::level::trace, __VA_ARGS__)
#define LDebug(...)     myLog::getInstance()->getLogger()->log(spdlog::source_loc{__FILE__, __LINE__, SPDLOG_FUNCTION}, spdlog::level::debug, __VA_ARGS__)
#define LInfo(...)      myLog::getInstance()->getLogger()->log(spdlog::source_loc{__FILE__, __LINE__, SPDLOG_FUNCTION}, spdlog::level::info, __VA_ARGS__)
#define LWarn(...)      myLog::getInstance()->getLogger()->log(spdlog::source_loc{__FILE__, __LINE__, SPDLOG_FUNCTION}, spdlog::level::warn, __VA_ARGS__)
#define LError(...)     myLog::getInstance()->getLogger()->log(spdlog::source_loc{__FILE__, __LINE__, SPDLOG_FUNCTION}, spdlog::level::err, __VA_ARGS__)
#define LCritical(...)  myLog::getInstance()->getLogger()->log(spdlog::source_loc{__FILE__, __LINE__, SPDLOG_FUNCTION}, spdlog::level::critical, __VA_ARGS__)
```
