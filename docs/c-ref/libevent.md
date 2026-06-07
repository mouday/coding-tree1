# Libevent

Libevent 是一个轻量级、高性能的 C 语言事件通知库，它封装了系统底层的 I/O 多路复用机制（如 `epoll`, `kqueue`, `select`），让你能轻松构建高并发、异步的网络应用。

https://libevent.org/

## 🧠 核心概念与 Reactor 模式

在使用前，先认识它的几个核心概念：

- **`event_base` (事件循环引擎)**：整个库的核心，相当于一个“反应堆”，负责管理和调度所有事件。它内部封装了 `epoll_wait` 等系统调用，等待并分发事件。
- **`event` (事件)**：你需要关注的对象，每个 `event` 绑定了一个文件描述符 (fd)、一个事件类型 (读/写/信号/超时) 以及触发时执行的**回调函数**。
- **`event_base_dispatch` (事件循环)**：启动后，程序会进入一个无限循环，持续检测注册的事件是否就绪，一旦就绪便立即调用对应的回调函数。

这种设计正是经典的 **Reactor 模式**：`event_base` 即是 Reactor，负责事件的分发与处理。

## 下载安装

https://github.com/libevent/libevent/releases


```shell
./configure --prefix=/Users/wang/local/libevent
make && make verify && make install
```

## 🧑‍💻 从零开始：快速上手

下面通过代码感受一下它的工作方式。编译时记得链接 `-levent`。

```c
#include <stdio.h>
#include <unistd.h>     // 提供 STDIN_FILENO
#include <signal.h>     // 提供 SIGINT
#include <event2/event.h>

// 标准输入（STDIN）的回调函数
void stdin_cb(evutil_socket_t fd, short events, void *arg) {
    char buf[256];
    int len = read(fd, buf, sizeof(buf) - 1);
    if (len > 0) {
        buf[len] = '\0';
        printf("输入: %s", buf);
    }
}

// 定时器回调函数
void timeout_cb(evutil_socket_t fd, short events, void *arg) {
    printf("⏰ 定时器触发！\n");
}

// 信号（Ctrl+C）回调函数
void signal_cb(evutil_socket_t sig, short events, void *arg) {
    struct event_base *base = (struct event_base *)arg;
    printf("\n接收到信号 %d，准备退出\n", sig);
    event_base_loopbreak(base);  // 退出事件循环
}

int main() {
    // 1. 初始化 event_base
    struct event_base *base = event_base_new();
    if (!base) return -1;

    // 2. 创建并注册一个标准输入读事件（EV_PERSIST 使其持续生效）
    struct event *ev_stdin = event_new(base, STDIN_FILENO, EV_READ | EV_PERSIST, stdin_cb, NULL);
    event_add(ev_stdin, NULL);

    // 3. 创建并注册一个定时器（5秒后触发一次）
    struct event *ev_timer = evtimer_new(base, timeout_cb, NULL);
    struct timeval tv = {5, 0};
    event_add(ev_timer, &tv);

    // 4. 创建并注册一个信号事件（捕获 Ctrl+C）
    struct event *ev_signal = evsignal_new(base, SIGINT, signal_cb, base);
    event_add(ev_signal, NULL);

    printf("已启动事件循环，请在终端输入字符、按 Ctrl+C 或等待 5 秒...\n");
    event_base_dispatch(base);  // 开始事件循环，本行会阻塞

    // 5. 清理资源
    event_free(ev_signal);
    event_free(ev_timer);
    event_free(ev_stdin);
    event_base_free(base);
    printf("优雅退出。\n");
    return 0;
}
```

上面这个程序核心步骤是**创建事件循环引擎、创建并注册事件、启动事件循环**，展示了如何同时监听多种事件源，这正是异步编程的魅力所在。

## 📚 进阶示例：构建 TCP Echo 服务器

下面实现一个更常见的网络服务器：回显 (Echo) 服务。它会接收客户端连接并将收到的数据原样发回。

```c
// libevent_demo.c 
#include <stdio.h>
#include <string.h>
#include <event2/event.h>
#include <event2/listener.h>
#include <event2/bufferevent.h>
#include <event2/buffer.h>
#include <netinet/in.h>
#include <arpa/inet.h>

// 读取回调：有数据可读时调用
void echo_read_cb(struct bufferevent *bev, void *ctx)
{
    struct evbuffer *input = bufferevent_get_input(bev);
    struct evbuffer *output = bufferevent_get_output(bev);
    // 将输入缓冲区的数据拷贝到输出缓冲区，实现回显
    evbuffer_add_buffer(output, input);
}

// 事件回调：处理连接、错误等状态变化
void echo_event_cb(struct bufferevent *bev, short events, void *ctx)
{
    if (events & BEV_EVENT_EOF)
    {
        printf("客户端连接已关闭。\n");
        bufferevent_free(bev);
    }
    else if (events & BEV_EVENT_ERROR)
    {
        perror("客户端连接出错");
        bufferevent_free(bev);
    }
}

// 接受新连接的回调
void accept_conn_cb(struct evconnlistener *listener, evutil_socket_t fd,
                    struct sockaddr *address, int socklen, void *ctx)
{
    struct event_base *base = evconnlistener_get_base(listener);
    // 为新的客户端连接创建 bufferevent
    struct bufferevent *bev = bufferevent_socket_new(base, fd, BEV_OPT_CLOSE_ON_FREE);
    // 设置回调
    bufferevent_setcb(bev, echo_read_cb, NULL, echo_event_cb, NULL);
    // 启用读事件
    bufferevent_enable(bev, EV_READ);
    printf("新客户端连接已建立。\n");
}

int main()
{
    struct event_base *base = event_base_new();
    if (!base)
        return -1;

    struct sockaddr_in sin;
    memset(&sin, 0, sizeof(sin));
    sin.sin_family = AF_INET;
    sin.sin_port = htons(9999); // 监听 9999 端口
    sin.sin_addr.s_addr = htonl(INADDR_ANY);

    // 创建并绑定一个监听器
    struct evconnlistener *listener = evconnlistener_new_bind(
        base,                                      /* event_base */
        accept_conn_cb,                            /* evconnlistener_cb */
        NULL,                                      /* ptr */
        LEV_OPT_CLOSE_ON_FREE | LEV_OPT_REUSEABLE, /* flags */
        -1,                                        /* backlog */
        (struct sockaddr *)&sin,                   /* sockaddr */
        sizeof(sin)                                /* socklen */
    );
    if (!listener)
    {
        perror("无法绑定监听器");
        event_base_free(base);
        return -1;
    }

    printf("Echo 服务器启动，监听端口 9999\n");
    event_base_dispatch(base); // 进入事件循环

    evconnlistener_free(listener);
    event_base_free(base);
    return 0;
}
```

编译运行

```shell
gcc -g libevent_demo.c  -levent -o libevent_demo && ./libevent_demo
```

在这个示例中，`bufferevent` 模块自动处理了数据的读写缓冲，开发者只需关注业务逻辑，极大地简化了开发。你可以用 `telnet 127.0.0.1 9999` 命令来测试它。

## 🛠️ 核心 API 速查

掌握了上面的例子后，可以参考这个速查表加深记忆：

| 模块/函数                                             | 用途                           | 补充说明                                                     |
| :---------------------------------------------------- | :----------------------------- | :----------------------------------------------------------- |
| **事件管理**                                          |
| `event_base *event_base_new()`                        | 创建一个新的事件循环引擎       | 核心对象，所有事件依赖于此                                   |
| `int event_base_dispatch()`                           | 启动事件循环，开始分发事件     | 会阻塞线程                                                   |
| **事件操作**                                          |
| `event *event_new(base, fd, flags, cb, arg)`          | 创建一个新的事件               | `flags` 定义事件类型，如 `EV_READ`、`EV_WRITE`、`EV_PERSIST` |
| `int event_add(event, timeout)`                       | 将事件注册到事件循环中         | 开始监听该事件                                               |
| `int event_del(event)`                                | 将事件从事件循环中移除         | 停止监听                                                     |
| `void event_free(event)`                              | 释放事件对象                   | 清理资源                                                     |
| **缓冲事件 (Bufferevent)**                            |
| `bufferevent *bufferevent_socket_new(base, fd, opts)` | 创建一个基于套接字的缓冲事件   | 自带输入/输出缓冲区，读写更方便                              |
| `void bufferevent_setcb(...)`                         | 设置缓冲事件的读、写、事件回调 | 推荐使用此模块进行网络编程                                   |
| **监听器 (evconnlistener)**                           |
| `evconnlistener *evconnlistener_new_bind(...)`        | 创建一个监听器并绑定到地址     | 用于快速构建 TCP 服务器                                      |
| **定时器**                                            |
| `event *evtimer_new(base, callback, arg)`             | 创建一个定时器事件             | 本质是 `EV_TIMEOUT` 事件的封装                               |

## 💎 总结

Libevent 通过事件驱动和 Reactor 模式，将复杂、平台相关的 I/O 多路复用封装成统一、简单的接口，让你能专注于业务逻辑，写出高性能、高可扩展的网络程序，并兼具跨平台性、轻量级和高性能等特点。

作为 C 程序员，它几乎是高性能网络编程的必备工具，被广泛应用于 Memcached、Nginx 等众多知名的开源项目中。

如果想深入了解某个函数的具体用法，或者想尝试构建其他类型的网络服务，都可以随时再问我。
