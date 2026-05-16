# C++网络编程

## 课程简介

- 介绍网络编程的基础知识，socket 的库函数
- 介绍网络通讯的原理
- I/O 服用的模型，select/poll/epoll/非阻塞 IO

## 文件描述符

存放每个进程打开的 fd: `/proc/进程id/fd`

进程 id 可以通过如下命令查找

```shell
ps -ef | grep demo.cpp
```

Linux 进程默认打开了 3 个文件描述符：

- 0-标准输入（键盘）cin
- 1-标准输出（显示器）cout
- 2-标准错误（显示器）cerr

示例：标准输入、输出、错误

```cpp
#include <iostream>

using namespace std;

int main(int argc, char const *argv[])
{
    int num;
    cin >> num;
    cout << "cout: " << num << endl;
    cerr << "cerr: " << num << endl;
    return 0;
}
```

接收一个输入，并输出两次

```shell
% g++ stdio_demo.cpp && ./a.out
100
cout: 100
cerr: 100
```

示例：关闭标准流

```cpp
#include <iostream>
#include <unistd.h>

using namespace std;

int main(int argc, char const *argv[])
{
    close(0); // stdin
    close(1); // stdout
    close(2); // stderr

    int num;
    cin >> num;
    cout << "cout: " << num << endl;
    cerr << "cerr: " << num << endl;
    return 0;
}
```

再次运行，没有任何输出

```shell
g++ stdio_demo.cpp && ./a.out
```

文件描述符分配规则：

找到最小的，没有被占用的文件描述符

结论：

- 对 Linux 来说，socket 操作与文件操作没有区别
- 在网络传输数据的过程中，可以使用文件的 I/O 函数
- 文件描述符是 Linux 分配给文件或者 socket 的整数

socket 的读写操作，可以替换为文件的读写操作

例如：

```cpp
// 文件读写
read(socket_fd, buffer, sizeof(buffer));
write(socket_fd, buffer, len);

// socket读写
recv(socket_fd, buffer, sizeof(buffer), 0);
send(socket_fd, buffer, len, 0);
```

查看可以打开的文件数

```shell
ulimit -a
```

## socket 使用方式

```cpp
// 服务端
socket 创建socket
bind 指定通信ip地址和端口
listen 监听
accept 接受客户端连接
recv/send 接收/发送数据
close 关闭连接

// 客户端
socket 创建socket
connect 向服务端发起连接请求
recv/send 接收/发送数据
close 关闭连接
```

服务端

```cpp
// server.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <memory.h>
#include <assert.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

int main(int argc, char const *argv[])
{
    int ret = 0;

    // 1、创建socket
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    printf("socket ret: %d\n", server_fd);

    // 绑定前清理TIME_WAIT，查看：netstat -an | grep 8080
    int opt = 1;
    ret = setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
    printf("setsockopt ret: %d\n", ret);

    // 2、绑定IP和端口
    struct sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    // 端口
    int listen_port = 8080;
    server_addr.sin_port = htons(listen_port);
    // IP
    server_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    ret = bind(server_fd, (struct sockaddr *)&server_addr, sizeof(server_addr));
    printf("bind ret: %d\n", ret);
    assert(ret == 0);

    // 3、监听
    ret = listen(server_fd, 1);
    printf("listen ret: %d\n", ret);

    // 4、接受客户端连接
    struct sockaddr_in client_addr;
    socklen_t client_addr_len = sizeof(client_addr);
    int client_fd = accept(server_fd, (struct sockaddr *)&client_addr, &client_addr_len);
    printf("accept ret: %d\n", client_fd);

    // 打印客户端IP和端口
    char ip[INET_ADDRSTRLEN];
    inet_ntop(AF_INET, &client_addr.sin_addr, ip, sizeof(ip));
    printf("client ip: %s:%d\n", ip, ntohs(client_addr.sin_port));

    // 5、接收数据
    char buf[1024];
    ssize_t n = recv(client_fd, buf, sizeof(buf) - 1, 0);
    printf("recv ret: %ld\n", n);
    buf[n] = '\0';
    printf("data: %s\n", buf);

    // 6、发送数据
    char *result = "hello client";
    ret = send(client_fd, result, strlen(result), 0);
    printf("send ret: %d\n", ret);

    // 7、关闭连接
    ret = close(client_fd);
    printf("close client_fd ret: %d\n", ret);
    ret = close(server_fd);
    printf("close server_fd ret: %d\n", ret);

    return 0;
}
```

客户端

```cpp
// client.c
#include <stdio.h>
#include <unistd.h>
#include <memory.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <netdb.h>

int main(int argc, char const *argv[])
{
    int ret = 0;

    // 1、创建socket
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    printf("socket ret: %d\n", server_fd);

    // 2、连接服务端
    struct sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    // 端口
    int server_port = 8080;
    server_addr.sin_port = htons(server_port);
    // IP
    char *server_ip = "127.0.0.1";
    struct hostent *host_net = gethostbyname(server_ip);
    memcpy(&server_addr.sin_addr, host_net->h_addr, host_net->h_length);
    printf("connect: %s:%d\n", server_ip, server_port);
    ret = connect(server_fd, (struct sockaddr *)&server_addr, sizeof(server_addr));
    printf("connect ret: %d\n", ret);

    // 3、发送数据
    char *data = "hello server";
    ret = send(server_fd, data, strlen(data), 0);
    printf("send ret: %d\n", ret);

    // 4、接收数据
    char buf[1024];
    ret = recv(server_fd, buf, sizeof(buf) - 1, 0);
    printf("recv ret: %d\n", ret);
    buf[ret] = '\0';
    printf("data: %s\n", buf);

    // 5、关闭连接
    ret = close(server_fd);
    printf("close ret: %d\n", ret);

    return 0;
}
```

```shell
# 服务端
% gcc server.c && ./a.out
socket ret: 3
setsockopt ret: 0
bind ret: 0
listen ip: 127.0.0.1:8080
listen ret: 0
accept ret: 4
client ip: 127.0.0.1:49469
recv ret: 12
data: hello server
send ret: 12
close client_fd ret: 0
close server_fd ret: 0

# 客户端
% gcc client.c && ./a.out
socket ret: 3
connect: 127.0.0.1:8080
connect ret: 0
send ret: 12
recv ret: 12
data: hello client
close ret: 0
```

## C++对 socket 的封装

目录结构

```shell
% tree -I build
.
├── client
│   ├── CMakeLists.txt
│   └── client.cpp
├── server
│   ├── CMakeLists.txt
│   └── server.cpp
└── src
    ├── connection.h
    └── connection.cpp
```

src/connection.h

```cpp
#include <string>

class Connection
{
public:
    Connection();
    Connection(std::string ip, short port);
    ~Connection();

    bool send(const std::string &data);
    bool receive(std::string &buffer);
    void close();
    bool is_connected();
    // 客户端行为
    bool connect();
    // 服务端行为
    bool listen();
    std::shared_ptr<Connection> accept();

private:
    // 文件描述符
    int m_fd;
    // ip
    std::string m_ip;
    // 端口号
    short m_port;
};
```

src/connection.cpp

```cpp
#include <unistd.h>
#include <iostream>
#include <string>
#include <stdio.h>
#include <memory.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <netdb.h>
#include <stdlib.h>
#include <assert.h>

#include "connection.h"

using namespace std;

Connection::Connection()
{
}
Connection::Connection(std::string ip, short port)
{
    m_ip = ip;
    m_port = port;
}

Connection::~Connection()
{
    close();
}

/**
 * 发送数据
 */
bool Connection::send(const std::string &data)
{
    if (!is_connected() || data.size() <= 0)
    {
        return false;
    }

    int ret = ::send(m_fd, data.data(), data.size(), 0);
    printf("send ret: %d\n", ret);
    return ret > 0;
};

/**
 * 接收数据
 */
bool Connection::receive(std::string &buffer)
{
    if (!is_connected())
    {
        return false;
    }

    // 清空原有数据
    buffer.clear();
    // 扩容
    const size_t max_size=1024;
    buffer.resize(max_size);
    ssize_t n = recv(m_fd, &buffer[0], buffer.size(), 0);
    printf("recv ret: %ld\n", n);
    if (n <= 0)
    {
        buffer.clear();
        return false;
    }

    // 重置buffer大小
    buffer.resize(n);
    return true;
};

bool Connection::is_connected()
{
    return m_fd > -1;
}

void Connection::close()
{
    cout << "Connection::close" << endl;
    if (!is_connected())
    {
        return;
    }

    ::close(m_fd);
    m_fd = -1;
}

bool Connection::connect()
{
    int ret = 0;

    // 1、创建socket
    m_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (m_fd < 0)
    {
        return false;
    }
    printf("socket ret: %d\n", m_fd);

    // 2、连接服务端
    struct sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    // 端口
    server_addr.sin_port = htons(m_port);
    // IP
    struct hostent *host_net = gethostbyname(m_ip.data());
    if (host_net == nullptr)
    {
        return false;
    }

    memcpy(&server_addr.sin_addr, host_net->h_addr, host_net->h_length);

    printf("connect: %s:%d\n", m_ip.data(), m_port);
    ret = ::connect(m_fd, (struct sockaddr *)&server_addr, sizeof(server_addr));
    if (ret != 0)
    {
        return false;
    }

    return true;
}

/**
 * 开始监听
 */
bool Connection::listen()
{
    int ret = 0;

    // 1、创建socket
    m_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (m_fd < 0)
    {
        return false;
    }

    // 绑定前清理TIME_WAIT，查看：netstat -an | grep 8080
    int opt = 1;
    ret = setsockopt(m_fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
    if (ret != 0)
    {
        return false;
    }

    // 2、绑定IP和端口
    struct sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    // 端口
    server_addr.sin_port = htons(m_port);
    // IP
    server_addr.sin_addr.s_addr = inet_addr(m_ip.data());
    ret = ::bind(m_fd, (struct sockaddr *)&server_addr, sizeof(server_addr));
    printf("bind ret: %d\n", ret);

    // 3、监听
    printf("listen ip: %s:%d\n", m_ip.data(), m_port);
    ret = ::listen(m_fd, 3);
    printf("listen ret: %d\n", ret);

    return true;
}

/**
 * 接受客户端连接
 */
std::shared_ptr<Connection> Connection::accept()
{
    struct sockaddr_in client_addr;
    socklen_t client_addr_len = sizeof(client_addr);
    int client_fd = ::accept(m_fd, (struct sockaddr *)&client_addr, &client_addr_len);
    printf("accept ret: %d\n", client_fd);

    // 打印客户端IP和端口
    char ip[INET_ADDRSTRLEN];
    inet_ntop(AF_INET, &client_addr.sin_addr, ip, sizeof(ip));
    uint16_t port = ntohs(client_addr.sin_port);
    printf("client ip: %s: %d\n", ip, port);

    // 创建一个链接对象
    std::shared_ptr<Connection> connection = std::make_shared<Connection>();
    connection->m_ip = ip;
    connection->m_port = port;
    connection->m_fd = client_fd;
    return connection;
}

```

server/CMakeLists.txt

```shell
# CMakeLists.txt
# cmake -B build && cmake --build build && ./build/server
cmake_minimum_required(VERSION 3.29)
project(server)

set(CMAKE_CXX_STANDARD 11)

include_directories("../src")

aux_source_directory(./ SERVER_LIST)
aux_source_directory(../src SRC_LIST)

add_executable(server ${SERVER_LIST} ${SRC_LIST})
```

server/server.cpp

```cpp
#include <iostream>
#include <string>
#include <thread>
#include <list>
#include <fstream>

#include "connection.h"


using namespace std;

void client_handler(std::shared_ptr<Connection> client)
{
    string result;
    bool ret = client->receive(result);
    cout << "result: " << result << endl;
    client->send("hello client");
}

int main(int argc, char const *argv[])
{
    std::string server_ip = "0.0.0.0";
    short server_port = 8080;

    std::shared_ptr<Connection> server = std::make_shared<Connection>(server_ip, server_port);
    server->listen();

    while (true)
    {
        std::shared_ptr<Connection> client = server->accept();
        client_handler(client);
        // file_handler(client);
    }

    return 0;
}
```

client/CMakeLists.txt

```shell
# CMakeLists.txt
# cmake -B build && cmake --build build && ./build/client
cmake_minimum_required(VERSION 3.29)
project(client)

set(CMAKE_CXX_STANDARD 11)

include_directories("../src")

aux_source_directory(./ CLIENT_LIST)
aux_source_directory(../src SRC_LIST)

add_executable(client ${CLIENT_LIST} ${SRC_LIST})

```

client/client.cpp

```cpp
#include <iostream>
#include <fstream>
#include <string>

#include "connection.h"

using namespace std;

void send_text()
{
    std::string server_ip = "127.0.0.1";
    short server_port = 8080;
    std::shared_ptr<Connection> connection = std::make_shared<Connection>(server_ip, server_port);
    bool ret = connection->connect();
    if (ret)
    {
        connection->send("hello server");

        string result;
        bool ret = connection->receive(result);
        if (ret)
        {
            cout << "result: " << result << endl;
        }
    }
}

int main(int argc, char const *argv[])
{
    send_text();
    return 0;
}
```

## 发送传输

发送文件流程

```shell
先发送文件名和文件大小
等待服务端确认
发送文件内容
等待服务端确认接收完成
```

接收文件流程

```shell
接收文件名和大小信息
给客户端回复确认报文，表示客户端可以发送文件了
接收文件内容
给客户端回复确认报文，表示接收完成
```

## TCP 缓存

- Nagle 算法 尽可能发送大块数据
- ACK 延时机制 40ms

## IO 多路复用

一个进程/线程处理多个 TCP 连接，减少系统开销

三种模型：

- select 1024
- poll 数千
- epoll 百万

## select

```cpp
#include <sys/select.h>

FD_SET(fd, &fdset);

FD_CLR(fd, &fdset);

FD_ISSET(fd, &fdset);

FD_COPY(&fdset_orig, &fdset_copy);

FD_ZERO(&fdset);
```

读事件

1. 新的客户端连接
2. 有可读数据
3. 断开连接

写事件

1. 发送缓冲区可以写入数据

细节

- 写事件
- 水平触发
- 性能测试
- 存在的问题
  - FD_SETSIZE 默认 1024
  - 轮询效率较低
  - fd_set 拷贝多次

使用 select 示例

```cpp
void Connection::select(select_callback handle_select_event)
{
    fd_set watch_fds;         // 监视事件集合，1024位
    FD_ZERO(&watch_fds);      // 初始化，将bitmat每一位都设置为0
    FD_SET(m_fd, &watch_fds); // 添加监视对象
    int max_fd = m_fd;
    std::list<std::shared_ptr<Connection>> connections;

    while (true)
    {
        printf("select max_fd: %d\n", max_fd);
        fd_set read_fds = watch_fds; // 拷贝一份，会select修改
        int ready_fd_count = ::select(max_fd + 1, &read_fds, NULL, NULL, NULL);

        printf("ready_fd_count: %d\n", ready_fd_count);

        if (ready_fd_count < 0)
        {
            // 调用select失败
            break;
        }
        else if (ready_fd_count == 0)
        {
            // 超时
            break;
        }
        else
        {
            // ready_fd_count > 0 新的事件达到
            int max_event_fd = max_fd;
            for (int event_fd = 0; event_fd <= max_event_fd; event_fd++)
            {
                // 没有事件
                if (FD_ISSET(event_fd, &read_fds) == 0)
                {
                    continue;
                }

                printf("event_fd: %d\n", event_fd);
                if (event_fd == m_fd)
                {
                    // 新客户端连接
                    std::shared_ptr<Connection> connection = accept();
                    FD_SET(connection->m_fd, &watch_fds); // 添加监视对象
                    if (connection->m_fd > max_fd)
                    {
                        max_fd = connection->m_fd;
                    }
                    connections.push_back(connection);
                }
                else
                {
                    // 查找到对应 Connection对象
                    std::shared_ptr<Connection> connection = nullptr;
                    for (std::shared_ptr<Connection> item : connections)
                    {
                        if (item->m_fd == event_fd)
                        {
                            connection = item;
                            break;
                        }
                    }

                    // 收到新数据
                    bool ret = handle_select_event(connection);

                    if (!ret)
                    {
                        // 客户端断开连接
                        connections.remove(connection);
                        FD_CLR(event_fd, &watch_fds); // 移除监控

                        // 更新最大值，从后往前遍历
                        if (event_fd == max_fd)
                        {
                            for (int i = max_fd; i > 0; i--)
                            {
                                if (FD_ISSET(i, &watch_fds))
                                {
                                    max_fd = i;
                                    break;
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
```

## poll 模型

- select 拷贝 2 次
- poll 拷贝 1 次
- poll 没有 1024 限制，也存在遍历操作

## epoll

- 阻塞：等待返回，让出CPU使用权
- 非阻塞：立即返回

阻塞函数：

- connect
- accept
- send
- recv

水平触发和边缘触发

- 水平触发 LT
- 边缘触发 ET

应用

- select/poll采用水平触发
- epoll 水平触发（默认）和边缘触发

## 完整代码

```shell
.
├── client
│   ├── CMakeLists.txt
│   └── client.cpp
├── server
│   ├── CMakeLists.txt
│   └── server.cpp
├── server_epoll
│   ├── CMakeLists.txt
│   └── server.cpp
├── server_poll
│   ├── CMakeLists.txt
│   └── server.cpp
├── server_select
│   ├── CMakeLists.txt
│   └── server.cpp
└── src
    ├── connection.cpp
    └── connection.h

```

src/connection.h

```cpp
#include <string>
#include <memory>

class Connection;

// using connection_handler = bool (*)(std::shared_ptr<Connection> connection);

typedef bool (*connection_handler)(std::shared_ptr<Connection> connection);

class Connection
{
public:
    Connection();
    Connection(std::string ip, short port);
    ~Connection();

    bool send(const std::string &data);
    bool receive(std::string &buffer);
    void close();
    bool is_connected();
    // 客户端行为
    bool connect();
    // 服务端行为
    bool listen();
    std::shared_ptr<Connection> accept();

    void select(connection_handler handle_connection);

    void poll(connection_handler handle_connection);

    void epoll(connection_handler handle_connection);

private:

private:
    // 文件描述符
    int m_fd;
    // ip
    std::string m_ip;
    // 端口号
    short m_port;
};

```

src/connection.cpp

```cpp
#include <unistd.h>
#include <stdio.h>
#include <memory.h>
#include <netdb.h>
#include <stdlib.h>
#include <assert.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

#include <sys/select.h> // select
#include <poll.h>       // poll
#ifdef __linux__
#include <sys/epoll.h> // epoll
#endif

#include <unordered_map>
#include <iostream>
#include <string>

#include "connection.h"

using namespace std;

Connection::Connection()
{
}
Connection::Connection(std::string ip, short port)
{
    m_ip = ip;
    m_port = port;
}

Connection::~Connection()
{
    close();
}

/**
 * 发送数据
 */
bool Connection::send(const std::string &data)
{
    if (!is_connected() || data.size() <= 0)
    {
        return false;
    }

    int ret = ::send(m_fd, data.data(), data.size(), 0);
    printf("send ret: %d\n", ret);
    return ret > 0;
};

/**
 * 接收数据
 */
bool Connection::receive(std::string &buffer)
{
    if (!is_connected())
    {
        return false;
    }

    // 清空原有数据
    buffer.clear();
    // 扩容
    const size_t max_size = 1024;
    buffer.resize(max_size);
    ssize_t n = recv(m_fd, &buffer[0], buffer.size(), 0);
    printf("recv ret: %ld\n", n);
    if (n <= 0)
    {
        buffer.clear();

        return false;
    }

    // 重置buffer大小
    buffer.resize(n);
    return true;
};

bool Connection::is_connected()
{
    return m_fd > -1;
}

void Connection::close()
{
    cout << "Connection::close" << endl;
    if (!is_connected())
    {
        return;
    }

    ::close(m_fd);
    m_fd = -1;
}

bool Connection::connect()
{
    int ret = 0;

    // 1、创建socket
    m_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (m_fd < 0)
    {
        return false;
    }
    printf("socket ret: %d\n", m_fd);

    // 2、连接服务端
    struct sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    // 端口
    server_addr.sin_port = htons(m_port);
    // IP
    struct hostent *host_net = gethostbyname(m_ip.data());
    if (host_net == nullptr)
    {
        return false;
    }

    memcpy(&server_addr.sin_addr, host_net->h_addr, host_net->h_length);

    printf("connect: %s:%d\n", m_ip.data(), m_port);
    ret = ::connect(m_fd, (struct sockaddr *)&server_addr, sizeof(server_addr));
    if (ret != 0)
    {
        return false;
    }

    return true;
}

/**
 * 开始监听
 */
bool Connection::listen()
{
    int ret = 0;

    // 1、创建socket
    m_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (m_fd < 0)
    {
        return false;
    }

    // 绑定前清理TIME_WAIT，查看：netstat -an | grep 8080
    int opt = 1;
    ret = setsockopt(m_fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
    if (ret != 0)
    {
        return false;
    }

    // 2、绑定IP和端口
    struct sockaddr_in server_addr;
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    // 端口
    server_addr.sin_port = htons(m_port);
    // IP
    server_addr.sin_addr.s_addr = inet_addr(m_ip.data());
    ret = ::bind(m_fd, (struct sockaddr *)&server_addr, sizeof(server_addr));
    if (ret != 0)
    {
        return false;
    }
    printf("bind ret: %d\n", ret);

    // 3、监听
    printf("listen ip: %s:%d\n", m_ip.data(), m_port);
    ret = ::listen(m_fd, 3);
    printf("listen ret: %d\n", ret);

    return true;
}

/**
 * 接受客户端连接
 */
std::shared_ptr<Connection> Connection::accept()
{
    struct sockaddr_in client_addr;
    socklen_t client_addr_len = sizeof(client_addr);
    int client_fd = ::accept(m_fd, (struct sockaddr *)&client_addr, &client_addr_len);
    printf("accept ret: %d\n", client_fd);

    // 打印客户端IP和端口
    char ip[INET_ADDRSTRLEN];
    inet_ntop(AF_INET, &client_addr.sin_addr, ip, sizeof(ip));
    uint16_t port = ntohs(client_addr.sin_port);
    printf("client ip: %s: %d\n", ip, port);

    // 创建一个链接对象
    std::shared_ptr<Connection> connection = std::make_shared<Connection>();
    connection->m_ip = ip;
    connection->m_port = port;
    connection->m_fd = client_fd;
    return connection;
}

void Connection::select(connection_handler handle_connection)
{
    printf("Connection::select\n");

    fd_set listen_fds;         // 监视事件集合，1024位
    FD_ZERO(&listen_fds);      // 初始化，将bitmat每一位都设置为0
    FD_SET(m_fd, &listen_fds); // 添加监视对象
    int max_fd = m_fd;
    std::unordered_map<int, std::shared_ptr<Connection>> connections;

    while (true)
    {
        printf("select max_fd: %d\n", max_fd);
        fd_set read_fds = listen_fds; // 拷贝一份，会select修改
        int ready_fd_count = ::select(max_fd + 1, &read_fds, NULL, NULL, NULL);

        printf("ready_fd_count: %d\n", ready_fd_count);

        if (ready_fd_count < 0)
        {
            // 调用select失败
            break;
        }
        else if (ready_fd_count == 0)
        {
            // 超时
            break;
        }

        // ready_fd_count > 0 新的事件达到
        int max_event_fd = max_fd;
        for (int event_fd = 0; event_fd <= max_event_fd; event_fd++)
        {
            // 没有事件
            if (FD_ISSET(event_fd, &read_fds) == 0)
            {
                continue;
            }

            printf("event_fd: %d\n", event_fd);
            if (event_fd == m_fd)
            {
                // 新客户端连接
                std::shared_ptr<Connection> connection = accept();
                FD_SET(connection->m_fd, &listen_fds); // 添加监视对象
                if (connection->m_fd > max_fd)
                {
                    max_fd = connection->m_fd;
                }
                connections[connection->m_fd] = connection;
            }
            else
            {
                // 查找到对应 Connection对象
                std::shared_ptr<Connection> connection = nullptr;
                if (connections.find(event_fd) != connections.end())
                {
                    connection = connections[event_fd];
                }

                // 收到新数据
                bool ret = handle_connection(connection);

                if (!ret)
                {
                    // 客户端断开连接
                    connections.erase(connection->m_fd);
                    FD_CLR(event_fd, &listen_fds); // 移除监控

                    // 更新最大值，从后往前遍历
                    if (event_fd == max_fd)
                    {
                        for (int i = max_fd; i > 0; i--)
                        {
                            if (FD_ISSET(i, &listen_fds))
                            {
                                max_fd = i;
                                break;
                            }
                        }
                    }
                }
            }
        }
    }
}

void Connection::poll(connection_handler handle_connection)
{
    printf("Connection::poll\n");
#define LISTEN_LENGTH 1024 // 数组大小

    pollfd listen_fds[LISTEN_LENGTH];
    for (int i = 0; i < LISTEN_LENGTH; i++)
    {
        // poll会忽略 fd==-1
        listen_fds[i].fd = -1;
    }

    listen_fds[m_fd].fd = m_fd;
    listen_fds[m_fd].events = POLLIN; // POLLIN表示读事件
    int max_fd = m_fd;

    std::unordered_map<int, std::shared_ptr<Connection>> connections;

    while (true)
    {
        // 开始监听事件 -1表示：阻塞等待
        int ready_fd_count = ::poll(listen_fds, max_fd + 1, -1);
        printf("ready_fd_count: %d\n", ready_fd_count);

        if (ready_fd_count < 0)
        {
            // 调用失败
            break;
        }
        else if (ready_fd_count == 0)
        {
            // 超时
            break;
        }

        int max_event_fd = max_fd;
        for (int event_fd = 0; event_fd <= max_event_fd; event_fd++)
        {
            if (listen_fds[event_fd].fd < 0)
            {
                // 忽略
                continue;
            }

            if ((listen_fds[event_fd].revents & POLLIN) == 0)
            {
                // 没有读事件
                continue;
            }

            printf("event_fd: %d\n", event_fd);
            if (listen_fds[event_fd].fd == m_fd)
            {
                // 新客户端
                std::shared_ptr<Connection> connection = accept();

                // 更新监听数组
                listen_fds[connection->m_fd].fd = connection->m_fd;
                // POLLIN表示读事件
                listen_fds[connection->m_fd].events = POLLIN;
                if (connection->m_fd > max_fd)
                {
                    max_fd = connection->m_fd;
                }
                printf("update max_fd: %d\n", max_fd);
                connections[connection->m_fd] = connection;
            }
            else
            {
                // 查找到对应 Connection对象
                std::shared_ptr<Connection> connection = nullptr;

                if (connections.find(listen_fds[event_fd].fd) != connections.end())
                {
                    connection = connections[listen_fds[event_fd].fd];
                }

                // 收到新数据
                bool ret = handle_connection(connection);

                if (!ret)
                {
                    listen_fds[event_fd].fd = -1;
                    // 客户端断开连接
                    connections.erase(connection->m_fd);

                    // 更新最大值，从后往前遍历
                    if (event_fd == max_fd)
                    {
                        for (int i = max_fd; i > 0; i--)
                        {
                            if (listen_fds[event_fd].fd != -1)
                            {
                                max_fd = i;
                                break;
                            }
                        }
                    }
                }
            }
        }
    }
}

void Connection::epoll(connection_handler handle_connection)
{
#ifdef __linux__
    printf("Connection::epoll\n");

    // 创建epoll句柄
    int epoll_fd = ::epoll_create(1);

    // 事件的数据结构
    epoll_event ev;
    ev.data.fd = m_fd;   // 自定义数据
    ev.events = EPOLLIN; // 监视读事件

    // 加入要监视的fd到epoll_fd中
    ::epoll_ctl(epoll_fd, EPOLL_CTL_ADD, m_fd, &ev);

#define EVENT_LENGTH 10 // 数组大小

    // 存放返回事件
    epoll_event events[EVENT_LENGTH];

    std::unordered_map<int, std::shared_ptr<Connection>> connections;
    while (true)
    {
        // 等待事件发生
        int ready_fd_count = ::epoll_wait(epoll_fd, events, EVENT_LENGTH, -1);
        if (ready_fd_count < 0)
        {
            // 报错
            break;
        }
        else if (ready_fd_count == 0)
        {
            // 超时
            continue;
        }

        // 遍历有事件的数组
        for (int i = 0; i < ready_fd_count; i++)
        {
            if (events[i].data.fd == m_fd)
            {
                // 新客户端
                std::shared_ptr<Connection> connection = accept();

                // 将新客户端加入监听
                ev.data.fd = connection->m_fd;
                ev.events = EPOLLIN;

                ::epoll_ctl(epoll_fd, EPOLL_CTL_ADD, connection->m_fd, &ev);

                connections[connection->m_fd] = connection;
            }
            else
            {
                // 查找到对应 Connection对象
                std::shared_ptr<Connection> connection = nullptr;

                if (connections.find(events[i].data.fd) != connections.end())
                {
                    connection = connections[events[i].data.fd];
                }

                // 收到新数据
                bool ret = handle_connection(connection);

                if (!ret)
                {
                    // 客户端断开连接
                    connections.erase(connection->m_fd);
                }
            }
        }
    }
#else
    poll(handle_connection);
#endif
}

```

client/CMakeLists.txt

```shell
# CMakeLists.txt
# cmake -B build && cmake --build build && ./build/client
# mkdir build; cd build
# cmake .. && make  && ./client
cmake_minimum_required(VERSION 2.8)
project(client)

set(CMAKE_CXX_STANDARD 11)

# 设置C++11支持: CMake 2.8没有内置的C++11支持开关
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)

if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support.")
endif()

message("CMAKE_CXX_FLAGS: ${CMAKE_CXX_FLAGS}")

include_directories("../src")

aux_source_directory(./ CLIENT_LIST)
aux_source_directory(../src SRC_LIST)

add_executable(client ${CLIENT_LIST} ${SRC_LIST})

```

client/client.cpp

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <memory>

#include "connection.h"

using namespace std;

void send_text()
{
    std::string server_ip = "127.0.0.1";
    short server_port = 8080;
    Connection connection(server_ip, server_port);
    bool ret = connection.connect();
    if (ret)
    {
        connection.send("hello server");

        string result;
        bool ret = connection.receive(result);
        if (ret)
        {
            cout << "result: " << result << endl;
        }
    }
}

int main(int argc, char const *argv[])
{
    send_text();
    return 0;
}
```

server/CMakeLists.txt

```shell
# CMakeLists.txt
# cmake -B build && cmake --build build && ./build/server
# mkdir build && cd build && cmake .. && make  && ./server
cmake_minimum_required(VERSION 2.8)
project(server)

set(CMAKE_CXX_STANDARD 11)
# set(CMAKE_BUILD_TYPE Debug)


# 设置C++11支持: CMake 2.8没有内置的C++11支持开关
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)

if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support.")
endif()

message("CMAKE_CXX_FLAGS: ${CMAKE_CXX_FLAGS}")

include_directories("../src")

aux_source_directory(./ SERVER_LIST)
aux_source_directory(../src SRC_LIST)

add_executable(server ${SERVER_LIST} ${SRC_LIST})


```

server/server.cpp

```cpp
#include <iostream>
#include <string>
#include <thread>
#include <list>
#include <fstream>

#include "connection.h"


using namespace std;

void client_handler(std::shared_ptr<Connection> client)
{
    string result;
    bool ret = client->receive(result);
    cout << "result: " << result << endl;
    client->send("hello client");
}

int main(int argc, char const *argv[])
{
    std::string server_ip = "0.0.0.0";
    short server_port = 8080;

    std::shared_ptr<Connection> server = std::make_shared<Connection>(server_ip, server_port);
    server->listen();

    while (true)
    {
        std::shared_ptr<Connection> client = server->accept();
        client_handler(client);
        // file_handler(client);
    }

    return 0;
}
```

server_epoll/CMakeLists.txt

```shell
# CMakeLists.txt
# cmake -B build && cmake --build build && ./build/server
cmake_minimum_required(VERSION 2.8)
project(server)

set(CMAKE_CXX_STANDARD 11)

# ulimit -c unlimited
set(CMAKE_BUILD_TYPE Debug)


# 设置C++11支持: CMake 2.8没有内置的C++11支持开关
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)

if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support.")
endif()

message("CMAKE_CXX_FLAGS: ${CMAKE_CXX_FLAGS}")

include_directories("../src")

aux_source_directory(./ SERVER_LIST)
aux_source_directory(../src SRC_LIST)

add_executable(server ${SERVER_LIST} ${SRC_LIST})

```

server_epoll/server.cpp

```cpp
#include <iostream>
#include <string>
#include <thread>
#include <list>
#include <fstream>

#include "connection.h"

using namespace std;

bool client_handler(std::shared_ptr<Connection> client)
{
    string result;
    bool ret = client->receive(result);
    if (!ret)
    {
        return false;
    }
    cout << "result: " << result << endl;
    client->send("hello client");
    return true;
}

int main(int argc, char const *argv[])
{
    std::string server_ip = "0.0.0.0";
    short server_port = 8080;

    std::shared_ptr<Connection> server = std::make_shared<Connection>(server_ip, server_port);
    server->listen();
    server->epoll(client_handler);

    return 0;
}
```

server_poll/CMakeLists.txt

```shell
# CMakeLists.txt
# cmake -B build && cmake --build build && ./build/server
cmake_minimum_required(VERSION 2.8)
project(server)

set(CMAKE_CXX_STANDARD 11)


# 设置C++11支持: CMake 2.8没有内置的C++11支持开关
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)

if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support.")
endif()

message("CMAKE_CXX_FLAGS: ${CMAKE_CXX_FLAGS}")


# ulimit -c unlimited
set(CMAKE_BUILD_TYPE Debug)


include_directories("../src")

aux_source_directory(./ SERVER_LIST)
aux_source_directory(../src SRC_LIST)

add_executable(server ${SERVER_LIST} ${SRC_LIST})

```

server_poll/server.cpp

```cpp
#include <iostream>
#include <string>
#include <thread>
#include <list>
#include <fstream>

#include "connection.h"

using namespace std;

bool client_handler(std::shared_ptr<Connection> client)
{
    string result;
    bool ret = client->receive(result);
    if (!ret)
    {
        return false;
    }
    cout << "result: " << result << endl;
    client->send("hello client");
    return true;
}

int main(int argc, char const *argv[])
{
    std::string server_ip = "0.0.0.0";
    short server_port = 8080;

    std::shared_ptr<Connection> server = std::make_shared<Connection>(server_ip, server_port);
    server->listen();
    server->poll(client_handler);

    return 0;
}
```

server_select/CMakeLists.txt

```shell
# CMakeLists.txt
# cmake -B build && cmake --build build && ./build/server
cmake_minimum_required(VERSION 2.8)
project(server)

set(CMAKE_CXX_STANDARD 11)

# ulimit -c unlimited
set(CMAKE_BUILD_TYPE Debug)

# 设置C++11支持: CMake 2.8没有内置的C++11支持开关
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)

if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support.")
endif()

message("CMAKE_CXX_FLAGS: ${CMAKE_CXX_FLAGS}")


include_directories("../src")

aux_source_directory(./ SERVER_LIST)
aux_source_directory(../src SRC_LIST)

add_executable(server ${SERVER_LIST} ${SRC_LIST})

```

server_select/server.cpp

```cpp
#include <iostream>
#include <string>
#include <thread>
#include <list>
#include <fstream>

#include "connection.h"

using namespace std;

bool client_handler(std::shared_ptr<Connection> client)
{
    string result;
    bool ret = client->receive(result);
    if (!ret)
    {
        return false;
    }
    cout << "result: " << result << endl;
    client->send("hello client");
    return true;
}

int main(int argc, char const *argv[])
{
    std::string server_ip = "0.0.0.0";
    short server_port = 8080;

    std::shared_ptr<Connection> server = std::make_shared<Connection>(server_ip, server_port);
    server->listen();
    server->select(client_handler);

    return 0;
}
```
