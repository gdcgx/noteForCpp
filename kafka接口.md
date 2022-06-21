# kafka之C++接口

## 天才第一步：在docker上部署kafka

### 一、拉取kafka镜像

> docker pull wurstmeister/zookeeper
> docker pull wurstmeister/kafka

### 二、定义docker-compose.yml

```yml
version: '3'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"
  kafka:
    image: wurstmeister/kafka
    depends_on: [ zookeeper ]
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 192.168.218.130
      KAFKA_CREATE_TOPICS: "test:1:1"
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /data/product/zj_bigdata/data/kafka/docker.sock:/var/run/docker.sock
```

在docker-compose.yml文件目录进行服务打包

```bash
[root@VM_0_16_centos kafka] # docker-compose build
zookeeper uses an image, skipping
kafka uses an image, skipping
```

### 三、启动服务

```bash
[root@VM_0_16_centos kafka]# docker-compose up -d
Starting kafka_kafka_1     ... done
Starting kafka_zookeeper_1 ... done
```

### 四、测试

#### 1、进入`kafka`容器

记住启动的启动名称，`kafka`为 `kafka_kafka_1` ，`zookeeper` 为 `kafka_zookeeper_1` 。如果`docker-compose`正常启动，此时`docker ps`会看到以上两个容器。

```bash
docker exec -it kafka_kafka_1 bash
```

#### 2、创建一个`topic`（主题）

```bash
$KAFKA_HOME/bin/kafka-topics.sh --create --topic daianna --partitions 4 --zookeeper kafka_zookeeper_1:2181 --replication-factor 1 
```

注意`--zookeeper`后面的参数为容器的`name`，查看刚刚创建的`topic`

```bash
$KAFKA_HOME/bin/kafka-topics.sh --zookeeper kafka_zookeeper_1:2181 --describe --topic daianna
```

查看到的结果如下：

```BASH
Topic: daianna	PartitionCount: 4	ReplicationFactor: 1	Configs: 
	Topic: daianna	Partition: 0	Leader: 1001	Replicas: 1001	Isr: 1001
	Topic: daianna	Partition: 1	Leader: 1001	Replicas: 1001	Isr: 1001
	Topic: daianna	Partition: 2	Leader: 1001	Replicas: 1001	Isr: 1001
	Topic: daianna	Partition: 3	Leader: 1001	Replicas: 1001	Isr: 1001
```

#### 3、发布信息

```bash
bash-4.4# $KAFKA_HOME/bin/kafka-console-producer.sh --topic=test --broker-list kafka_kafka_1:9092
>ni
>haha
```

同样注意--broker-list后面的参数

#### 4、接收消息

```bash
bash-4.4# $KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server kafka_kafka_1:9092 --from-beginning --topic test
>ni
>haha
```

## 天才第二步：通过C++操作kafka

### 一、安装 librdkafka 库

#### **1、github安装指导**

在 `Mac OSX` 上，使用自制软件安装 `librdkafka`：

```bash
$ brew install librdkafka
```

在 `Debian` 和 `Ubuntu` 上，从 `Confluent APT` 存储库安装 `librdkafka`，请参阅[此处的](https://docs.confluent.io/current/installation/installing_cp/deb-ubuntu.html#get-the-software)说明，然后安装 `librdkafka`：

```bash
$ apt install librdkafka-dev
```

在 `RedHat`、`CentOS`、`Fedora` 上，从 `Confluent YUM` 存储库安装 `librdkafka` ，说明[在这里](https://docs.confluent.io/current/installation/installing_cp/rhel-centos.html#get-the-software)，然后安装 `librdkafka`：

```bash
$ yum install librdkafka-devel
```

#### **2、使用 vcpkg 安装 librdkafka**

使用[vcpkg](https://github.com/Microsoft/vcpkg)依赖项管理器下载并安装 `librdkafka` ：

```bash
#如果尚未安装，请安装 vcpkg
$ git clone https://github.com/Microsoft/vcpkg.git
$ cd vcpkg
$ ./bootstrap-vcpkg.sh
$ ./vcpkg 集成安装

#安装 librdkafka 
$ vcpkg install librdkafka
```

#### **3、从源代码构建**

**要求**

```bash
The GNU toolchain
GNU make
pthreads
zlib-dev (optional, for gzip compression support)
libssl-dev (optional, for SSL and SASL SCRAM support)
libsasl2-dev (optional, for SASL GSSAPI support)
libzstd-dev (optional, for ZStd compression support)
```

注意：生产者中 ZStd 的静态链接（需要 zstd >= 1.2.1）可以在压缩帧头中对原始大小进行编码，这将加速消费者。使用`STATIC_LIB_libzstd=/path/to/libzstd.a ./configure --enable-static` 使静态ZStd链接。MacOSX 示例： `STATIC_LIB_libzstd=$(brew ls -v zstd | grep libzstd.a$) ./configure --enable-static`

**安装**

从`GitHub`[kafka官方](https://github.com/edenhill/librdkafka/)上拉取源码进行如下安装

```bash
  ./configure
  # Or, to automatically install dependencies using the system's package manager:
  # ./configure --install-deps
  # Or, build dependencies from source:
  # ./configure --install-deps --source-deps-only

  make
  sudo make install
```

#### **4、安装包安装**

1、在[github官方](https://github.com/edenhill/librdkafka/)下载`librdkafka`安装包

2、执行如下命令，安装librdkafka库

```bash
cd librdkafka-master 	//cd到刚刚解压到的文件目录下
chmod 777 configure lds-gen.py
./confgure
make
make install
```

### 二、c++代码示例

需要在代码中设置`broker`和`topic`

```bash
std::string brokers="xx.xx.xx.xx:9092"; 	 //地址为kafka服务器地址，这里是随便写的
std::string topic="xxxx" 				//主题为kafka中存在的主题，若不存在则会自动创建
```

**注意：**编译代码时加入 `-lstdc++ -lrdkafka -lrdkafka++ -lrt -lpthread -L.`（测试发现只要加`-lstdc++ -lrdkafka -lrdkafka++ -lpthread`即可）

**生产者**代码示例如下：

```cpp
//producer.cpp
#include <iostream>
#include <string>
#include <cstdlib>
#include <cstdio>
#include <csignal>
#include <cstring>
#if _AIX
#include <unistd.h>
#endif

#include "librdkafka/rdkafkacpp.h"

static volatile sig_atomic_t run = 1;

static void sigterm(int sig)
{
    run = 0;
}

//成功发送一次信息调用一次回调函数
class ExampleDeliveryReportCb : public RdKafka::DeliveryReportCb
{
public:
    void dr_cb(RdKafka::Message &message)
    {
        /* If message.err() is non-zero the message delivery failed permanently for the message. */
        if (message.err())
            std::cerr << "% Message delivery failed: " << message.errstr() << std::endl;
        else
            std::cerr << "% Message delivered to topic " << message.topic_name() << " [" << message.partition() << "] at offset " << message.offset() << std::endl;
    }
};

int main()
{

    std::string brokers = "192.168.218.130:9092"; //这里的IP地址应该设置为kafka所在服务器地址
    std::string topic = "gogogo";                 //设置主题
    std::string errstr;

    //设置配置
    RdKafka::Conf *conf = RdKafka::Conf::create(RdKafka::Conf::CONF_GLOBAL);

    if (conf->set("bootstrap.servers", brokers, errstr) !=
        RdKafka::Conf::CONF_OK)
    {
        std::cerr << errstr << std::endl;
        exit(1);
    }

    signal(SIGINT, sigterm);
    signal(SIGTERM, sigterm);

    ExampleDeliveryReportCb ex_dr_cb;

    if (conf->set("dr_cb", &ex_dr_cb, errstr) != RdKafka::Conf::CONF_OK)
    {
        std::cerr << errstr << std::endl;
        exit(1);
    }

    //创建 producer
    RdKafka::Producer *producer = RdKafka::Producer::create(conf, errstr);
    if (!producer)
    {
        std::cerr << "Failed to create producer: " << errstr << std::endl;
        exit(1);
    }

    delete conf;

    /*
 * Read messages from stdin and produce to broker.
 */
    std::cout << "% Type message value and hit enter "
              << "to produce message." << std::endl;

    for (std::string line; run && std::getline(std::cin, line);)
    {
        if (line.empty())
        {
            producer->poll(0);
            continue;
        }

    //将message传入队列中
    retry:
        RdKafka::ErrorCode err =
            producer->produce(topic,
                              RdKafka::Topic::PARTITION_UA, //随机分区
                              /* Make a copy of the value */
                              RdKafka::Producer::RK_MSG_COPY /* Copy payload */,
                              /* Value */
                              const_cast<char *>(line.c_str()), line.size(), //需要写入的数据
                                                                             /* Key */
                              NULL, 0,
                              /* Timestamp (defaults to current time) */
                              0,
                              /* Message headers, if any */
                              NULL,
                              /* Per-message opaque value passed to
                              * delivery report */
                              NULL);

        if (err != RdKafka::ERR_NO_ERROR)
        {
            std::cerr << "% Failed to produce to topic " << topic << ": " << RdKafka::err2str(err) << std::endl;

            if (err == RdKafka::ERR__QUEUE_FULL)
            { //存放数据的队列已满，将执行等待
                producer->poll(1000);
                goto retry;
            }
        }
        else
        {
            std::cerr << "% Enqueued message (" << line.size() << " bytes) "
                      << "for topic " << topic << std::endl;
        }

        producer->poll(0); //保持常态
    }

    std::cerr << "% Flushing final messages..." << std::endl;
    producer->flush(10 * 1000 /* wait for max 10 seconds */);

    run = true; //之前因为没有加这个就报错了

    if (producer->outq_len() > 0) //发送数据
        std::cerr << "% " << producer->outq_len() << " message(s) were not delivered" << std::endl;

    delete producer;

    return 0;
}
```

**消费者**代码试例如下：

```cpp
#include <iostream>
#include <string>
#include <cstdlib>
#include <cstdio>
#include <csignal>
#include <cstring>
#include <sys/time.h>
#include <getopt.h>
#include <unistd.h>

#include "librdkafka/rdkafkacpp.h"

static bool run = true;
static bool exit_eof = true;
static int eof_cnt = 0;
static int partition_cnt = 0;
static int verbosity = 1;
static long msg_cnt = 0;
static int64_t msg_bytes = 0;

static void sigterm(int sig)
{
    run = false;
}

class ExampleEventCb : public RdKafka::EventCb
{
public:
    void event_cb(RdKafka::Event &event)
    {
        switch (event.type())
        {
        case RdKafka::Event::EVENT_ERROR:
            std::cerr << "ERROR (" << RdKafka::err2str(event.err()) << "): " << event.str() << std::endl;
            if (event.err() == RdKafka::ERR__ALL_BROKERS_DOWN)
                run = false;
            break;

        case RdKafka::Event::EVENT_STATS:
            std::cerr << "\"STATS\": " << event.str() << std::endl;
            break;

        case RdKafka::Event::EVENT_LOG:
            fprintf(stderr, "LOG-%i-%s: %s\n",
                    event.severity(), event.fac().c_str(), event.str().c_str());
            break;

        case RdKafka::Event::EVENT_THROTTLE:
            std::cerr << "THROTTLED: " << event.throttle_time() << "ms by " << event.broker_name() << " id " << (int)event.broker_id() << std::endl;
            break;

        default:
            std::cerr << "EVENT " << event.type() << " (" << RdKafka::err2str(event.err()) << "): " << event.str() << std::endl;
            break;
        }
    }
};

void msg_consume(RdKafka::Message *message, void *opaque)
{
    switch (message->err())
    {
    case RdKafka::ERR__TIMED_OUT:
        //std::cerr << "RdKafka::ERR__TIMED_OUT"<<std::endl;
        break;

    case RdKafka::ERR_NO_ERROR:
        /* Real message */
        msg_cnt++;
        msg_bytes += message->len();
        if (verbosity >= 3)
            std::cerr << "Read msg at offset " << message->offset() << std::endl;
        RdKafka::MessageTimestamp ts;
        ts = message->timestamp();
        if (verbosity >= 2 &&
            ts.type != RdKafka::MessageTimestamp::MSG_TIMESTAMP_NOT_AVAILABLE)
        {
            std::string tsname = "?";
            if (ts.type == RdKafka::MessageTimestamp::MSG_TIMESTAMP_CREATE_TIME)
                tsname = "create time";
            else if (ts.type == RdKafka::MessageTimestamp::MSG_TIMESTAMP_LOG_APPEND_TIME)
                tsname = "log append time";
            std::cout << "Timestamp: " << tsname << " " << ts.timestamp << std::endl;
        }
        if (verbosity >= 2 && message->key())
        {
            std::cout << "Key: " << *message->key() << std::endl;
        }
        if (verbosity >= 1)
        {
            printf("%.*s\n",
                   static_cast<int>(message->len()),
                   static_cast<const char *>(message->payload()));
        }
        break;

    case RdKafka::ERR__PARTITION_EOF:
        /* Last message */
        if (exit_eof && ++eof_cnt == partition_cnt)
        {
            std::cerr << "%% EOF reached for all " << partition_cnt << " partition(s)" << std::endl;
            run = false;
        }
        break;

    case RdKafka::ERR__UNKNOWN_TOPIC:
    case RdKafka::ERR__UNKNOWN_PARTITION:
        std::cerr << "Consume failed: " << message->errstr() << std::endl;
        run = false;
        break;

    default:
        /* Errors */
        std::cerr << "Consume failed: " << message->errstr() << std::endl;
        run = false;
    }
}

class ExampleConsumeCb : public RdKafka::ConsumeCb
{
public:
    void consume_cb(RdKafka::Message &msg, void *opaque)
    {
        msg_consume(&msg, opaque);
    }
};

int main()
{
    std::string brokers = "192.168.218.130:9092";
    std::string errstr;
    std::string topic_str = "gogogo";
    std::vector<std::string> topics;
    std::string group_id = "1001";

    RdKafka::Conf *conf = RdKafka::Conf::create(RdKafka::Conf::CONF_GLOBAL);
    RdKafka::Conf *tconf = RdKafka::Conf::create(RdKafka::Conf::CONF_TOPIC);
    //group.id必须设置
    if (conf->set("group.id", group_id, errstr) != RdKafka::Conf::CONF_OK)
    {
        std::cerr << errstr << std::endl;
        exit(1);
    }

    topics.push_back(topic_str);
    //bootstrap.servers可以替换为metadata.broker.list
    conf->set("bootstrap.servers", brokers, errstr);

    ExampleConsumeCb ex_consume_cb;
    conf->set("consume_cb", &ex_consume_cb, errstr);

    ExampleEventCb ex_event_cb;
    conf->set("event_cb", &ex_event_cb, errstr);
    conf->set("default_topic_conf", tconf, errstr);

    signal(SIGINT, sigterm);
    signal(SIGTERM, sigterm);

    RdKafka::KafkaConsumer *consumer = RdKafka::KafkaConsumer::create(conf, errstr);
    if (!consumer)
    {
        std::cerr << "Failed to create consumer: " << errstr << std::endl;
        exit(1);
    }
    std::cout << "% Created consumer " << consumer->name() << std::endl;

    RdKafka::ErrorCode err = consumer->subscribe(topics);
    if (err)
    {
        std::cerr << "Failed to subscribe to " << topics.size() << " topics: "
                  << RdKafka::err2str(err) << std::endl;
        exit(1);
    }

    while (run)
    {
        //5000毫秒未订阅到消息，触发RdKafka::ERR__TIMED_OUT
        RdKafka::Message *msg = consumer->consume(5000);
        msg_consume(msg, NULL);
        delete msg;
    }

    consumer->close();

    delete conf;
    delete tconf;
    delete consumer;

    std::cerr << "% Consumed " << msg_cnt << " messages ("
              << msg_bytes << " bytes)" << std::endl;

    //应用退出之前等待rdkafka清理资源
    RdKafka::wait_destroyed(5000);

    return 0;
}
```

