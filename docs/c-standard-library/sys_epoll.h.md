# sys/epoll.h

macOS 并不原生支持 Linux 的 epoll 机制。这是因为 epoll 是 Linux 内核特有的 I/O 事件通知机制，而 macOS 作为基于 BSD 的系统，使用了其自己的高性能 I/O 多路复用实现 —— kqueue

为了简化跨平台开发，避免直接处理 epoll 和 kqueue 之间的差异，使用成熟的跨平台网络库通常是更高效、更稳定的选择。这些库在底层封装了不同操作系统的特定 API，为你提供统一的编程接口。

libevent: 一个广受欢迎的高性能网络库，支持 epoll, kqueue, select, poll 等多种后端

https://libevent.org/

Linux系统中通常使用epoll模型搭建这种活跃连接较少的服务器，相比select/poll的主动查询，epoll模型采用基于事件的通知方式，事先为建立连接的文件描述符注册事件，一旦该文件描述符就绪，内核会采用类似callback的回调机制，将文件描述符加入到epoll的指定的文件描述符集中，之后进程再根据该集合中文件描述符的数量，对客户端请求逐一进行处理。

虽然epoll机制中返回的同样是就绪文件描述符的数量，但epoll中的文件描述符集只存储了就绪的文件描述符，服务器进程无需再扫描所有已连接的文件描述符；且epoll机制使用内存映射机制（类似共享内存），不必再将内核中的文件描述符集复制到内存空间；此外，epoll机制不受进程可打开最大文件描述符数量的限制（只与系统内存有关），可连接远超过默认FD_SETSIZE的进程。

头文件

```cpp
#include <sys/poll.h>
```

## epoll_create

epoll_create()函数用于创建一个epoll句柄，并请求内核为该实例后期需存储的文件描述符及对应事件预先分配存储空间

```cpp
int epoll_create(int size);
```

函数中的参数size为在该epoll中可监听的文件描述符的最大个数，若该函数调用成功，将返回一个用于引用epoll的句柄；若调用失败，则返回-1，并设置errno。

当所有与该epoll相关的文件描述符都关闭后，内核会销毁epoll实例并释放相关资源，但若该函数返回的epoll句柄不再被使用，用户应主动调用close()函数将其关闭。

## epoll_ctl

epoll_ctl()是epoll的事件注册函数，用于将文件描述符添加到epoll的文件描述符集中，或从集合中删除指定文件描述符

```cpp
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

参数epfd为函数epoll_create()返回的epoll句柄；

参数op表示epoll_ctl()的动作，该动作的取值由三个宏指定，这些宏及其含义分别如下：

- EPOLL_CTL_ADD表示epoll_ctl()将会在epfd中为新fd注册事件；

- EPOLL_CTL_MOD表示epoll_ctl()将会修改已注册的fd监听事件；

- EPOLL_CTL_DEL表示epoll_ctl()将会删除epfd中监听的fd。

参数fd用于指定待操作的文件描述符；

参数event用于设定要监听的事件，该参数是一个`struct epoll_event` 类型的指针

```cpp
struct epoll_event {
  __uint32_t  events;   //epoll事件
  epoll_data_t data;    //用户数据变量
};
```

struct epoll_event结构体中的成员events表示要监控的事件，该成员可以是由一些单一事件组成的位集，这些单一事件由一组宏表示，宏及其含义分别如下：

- EPOLLIN表示监控文件描述符fd的读事件（包括socket正常关闭）；

- EPOLLOUT表示监控fd的写事件；

- EPOLLPRI表示监控fd的紧急可读事件（有优先数据到达时触发）；

- EPOLLERR表示监控fd的错误事件；

- EPOLLHUP表示监控fd的挂断事件；

- EPOLLET表示将epoll设置为边缘触发（Edge Triggered）模式；

- EPOLLONESHOT表示只监听一次事件，当此次事件监听完成后，若要再次监听该fd，需将其再次添加到epoll队列中。

struct epoll_event结构体成员data的数据类型是共用体epoll_data_t，其类型定义如下：

```cpp
typedef union epoll_data {
  void    *ptr;
  int     fd;
  __uint32_t  u32;
  __uint64_t  u64;
} epoll_data_t;
```

返回

- 若epoll_ctl()函数调用成功时会返回0；
- 若调用失败，则返回-1，并设置errno。

不同于select/poll机制在监听事件时才确定事件的类型，epoll机制在连接建立后便会指定要监控的事件。

## epoll_wait

epoll_wait()函数用于等待epoll句柄epfd中所监控事件的发生，当有一个或多个事件发生或等待超时后epoll_wait()返回

```cpp
int epoll_wait(int epfd, struct epoll_event *events,
​              int maxevents, int timeout);
```

参数epfd为epoll_create()函数返回的句柄；

参数events指向发生epoll_create()调用时系统事先预备的空间，当有监听的事件发生时，内核会将该事件复制到此段空间中；

参数maxevents表示events的大小，该值不能超过调用epoll_create()时所传参数size的大小；

参数timeout的单位为毫秒，用于设置epoll_wait()的工作方式：

- 若设置为0则立即返回，
- 设置为-1则使epoll无限期等待，
- 设置为大于0的值表示epoll等待一定的时长。

返回

- 若epoll_wait()函数调用成功时返回就绪文件描述符的数量；
- 若等待超时后并无就绪文件描述符则返回0；
- 若调用失败则返回-1，并设置errno。

## 工作模式

epoll有两种工作模式

- 边缘触发（Edge Triggered）模式
- 水平触发（LevelTriggered）模式

所谓边缘触发，指只有当文件描述符就绪时会触发通知，即便此次通知后系统执行I/O操作只读取了部分数据，文件描述符中仍有数据剩余，也不会再有通知递达，直到该文件描述符从当前的就绪态变为非就绪态，再由非就绪态再次变为就绪态，才会触发第二次通知；此外，接收缓冲区大小为5字节，也就是说ET模式下若只进行一次I/O操作，每次只能接收到5字节的数据。因此系统在收到就绪通知后，应尽量多次地执行I/O操作，直到无法再读出数据为止。

而水平触发与边缘触发有所不同，即便就绪通知已发送，内核仍会多次检测文件描述符状态，只要文件描述符为就绪态，内核就会继续发送通知。

## 示例

案例5：使用epoll模型搭建多路I/O转接服务器，使服务器可接收客户端数据，并将接收到的数据转为大写，写回客户端；使客户端可向服务器发送数据，并将服务器返回的数据打印到终端。

案例实现如下：

服务器端

```cpp
// epoll_s.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <sys/epoll.h>
#include <errno.h>

#define MAXLINE 80
#define SERV_PORT 8000
#define OPEN_MAX 1024

int main()
{
    int i, j, maxi, listenfd, connfd, sockfd;
    int nready, efd, res;
    ssize_t n;
    char buf[MAXLINE], str[INET_ADDRSTRLEN];
    socklen_t clilen;
    int client[OPEN_MAX];
    struct sockaddr_in cliaddr, servaddr;
    struct epoll_event tep, ep[OPEN_MAX];

    listenfd = socket(AF_INET, SOCK_STREAM, 0);
    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(SERV_PORT);
    bind(listenfd, (struct sockaddr *)&servaddr, sizeof(servaddr));
    listen(listenfd, 20);

    // 初始化client集合
    for (i = 0; i < OPEN_MAX; i++)
        client[i] = -1;
    maxi = -1; // 初始化maxi

    efd = epoll_create(OPEN_MAX); // 创建epoll句柄
    if (efd == -1)
        perr_exit("epoll_create");

    // 初始化tep
    tep.events = EPOLLIN;
    tep.data.fd = listenfd;
    // 为服务器进程注册事件（listenfd）
    res = epoll_ctl(efd, EPOLL_CTL_ADD, listenfd, &tep);
    if (res == -1)
        perr_exit("epoll_ctl");
    for (;;)
    {
        nready = epoll_wait(efd, ep, OPEN_MAX, -1); // 阻塞监听
        if (nready == -1)
            perr_exit("epoll_wait");
        // 处理就绪事件
        for (i = 0; i < nready; i++)
        {
            if (!(ep[i].events & EPOLLIN))
                continue;
            // 若fd为listenfd，表示有连接请求到达
            if (ep[i].data.fd == listenfd)
            {
                clilen = sizeof(cliaddr);
                connfd = accept(listenfd, (struct sockaddr *)&cliaddr,
                                &clilen);
                printf("received from %s at PORT %d\n", inet_ntop(AF_INET, &cliaddr.sin_addr, str, sizeof(str)),
                       ntohs(cliaddr.sin_port)); // 字节序转换
                                                 // 将accept获取到的文件描述符保存到client[]数组中
                for (j = 0; j < OPEN_MAX; j++)
                    if (client[j] < 0)
                    {
                        client[j] = connfd;
                        break;
                    }
                if (j == OPEN_MAX)
                    perr_exit("too many clients");
                if (j > maxi)
                    maxi = j; // 更新最大文件描述符
                tep.events = EPOLLIN;
                tep.data.fd = connfd;
                // 为新建立连接的进程注册事件
                res = epoll_ctl(efd, EPOLL_CTL_ADD, connfd, &tep);
                if (res == -1)
                    perr_exit("epoll_ctl");
            }
            else
            { // 若fd不等于listenfd，表示就绪的是各路连接
                sockfd = ep[i].data.fd;
                n = read(sockfd, buf, MAXLINE);
                if (n == 0)
                { // 若读取的字符个数为0表示对应客户端进程将关闭连接
                    for (j = 0; j <= maxi; j++)
                    {
                        if (client[j] == sockfd)
                        {
                            client[j] = -1;
                            break;
                        }
                    }
                    // 取消监听
                    res = epoll_ctl(efd, EPOLL_CTL_DEL, sockfd, NULL);
                    if (res == -1)
                        perr_exit("epoll_ctl");
                    close(sockfd); // 服务器端关闭连接
                    printf("client[%d] closed connection\n", j);
                }
                else
                {
                    for (j = 0; j < n; j++)
                        buf[j] = toupper(buf[j]);
                    writen(sockfd, buf, n);
                }
            }
        }
    }
    close(listenfd);
    close(efd);
    return 0;
}
```

客户端

```cpp
// epoll_c.c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <netinet/in.h>

#define MAXLINE 80
#define SERV_PORT 8000

int main()
{
    struct sockaddr_in servaddr;
    char buf[MAXLINE];
    int sockfd, n;
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    inet_pton(AF_INET, "127.0.0.1", &servaddr.sin_addr);
    servaddr.sin_port = htons(SERV_PORT);
    connect(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr));
    while (fgets(buf, MAXLINE, stdin) != NULL)
    {
        write(sockfd, buf, strlen(buf));
        n = read(sockfd, buf, MAXLINE);
        if (n == 0)
            printf("the other side has been closed.\n");
        else
            write(STDOUT_FILENO, buf, n);
    }
    close(sockfd);
    return 0;
}
```

```shell
gcc epoll_s.c -o epoll_s  && ./epoll_s

gcc epoll_c.c -o epoll_c  && ./epoll_c
```

## 附件`sys/epoll.h`

```cpp
// sys/epoll.h

/* Copyright (C) 2002-2012 Free Software Foundation, Inc.
   This file is part of the GNU C Library.

   The GNU C Library is free software; you can redistribute it and/or
   modify it under the terms of the GNU Lesser General Public
   License as published by the Free Software Foundation; either
   version 2.1 of the License, or (at your option) any later version.

   The GNU C Library is distributed in the hope that it will be useful,
   but WITHOUT ANY WARRANTY; without even the implied warranty of
   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
   Lesser General Public License for more details.

   You should have received a copy of the GNU Lesser General Public
   License along with the GNU C Library; if not, see
   <http://www.gnu.org/licenses/>.  */

#ifndef	_SYS_EPOLL_H
#define	_SYS_EPOLL_H	1

#include <stdint.h>
#include <sys/types.h>

/* Get __sigset_t.  */
#include <bits/sigset.h>

#ifndef __sigset_t_defined
# define __sigset_t_defined
typedef __sigset_t sigset_t;
#endif

/* Get the platform-dependent flags.  */
#include <bits/epoll.h>

#ifndef __EPOLL_PACKED
# define __EPOLL_PACKED
#endif


enum EPOLL_EVENTS
  {
    EPOLLIN = 0x001,
#define EPOLLIN EPOLLIN
    EPOLLPRI = 0x002,
#define EPOLLPRI EPOLLPRI
    EPOLLOUT = 0x004,
#define EPOLLOUT EPOLLOUT
    EPOLLRDNORM = 0x040,
#define EPOLLRDNORM EPOLLRDNORM
    EPOLLRDBAND = 0x080,
#define EPOLLRDBAND EPOLLRDBAND
    EPOLLWRNORM = 0x100,
#define EPOLLWRNORM EPOLLWRNORM
    EPOLLWRBAND = 0x200,
#define EPOLLWRBAND EPOLLWRBAND
    EPOLLMSG = 0x400,
#define EPOLLMSG EPOLLMSG
    EPOLLERR = 0x008,
#define EPOLLERR EPOLLERR
    EPOLLHUP = 0x010,
#define EPOLLHUP EPOLLHUP
    EPOLLRDHUP = 0x2000,
#define EPOLLRDHUP EPOLLRDHUP
    EPOLLWAKEUP = 1u << 29,
#define EPOLLWAKEUP EPOLLWAKEUP
    EPOLLONESHOT = 1u << 30,
#define EPOLLONESHOT EPOLLONESHOT
    EPOLLET = 1u << 31
#define EPOLLET EPOLLET
  };


/* Valid opcodes ( "op" parameter ) to issue to epoll_ctl().  */
#define EPOLL_CTL_ADD 1	/* Add a file descriptor to the interface.  */
#define EPOLL_CTL_DEL 2	/* Remove a file descriptor from the interface.  */
#define EPOLL_CTL_MOD 3	/* Change file descriptor epoll_event structure.  */


typedef union epoll_data
{
  void *ptr;
  int fd;
  uint32_t u32;
  uint64_t u64;
} epoll_data_t;

struct epoll_event
{
  uint32_t events;	/* Epoll events */
  epoll_data_t data;	/* User data variable */
} __EPOLL_PACKED;


__BEGIN_DECLS

/* Creates an epoll instance.  Returns an fd for the new instance.
   The "size" parameter is a hint specifying the number of file
   descriptors to be associated with the new instance.  The fd
   returned by epoll_create() should be closed with close().  */
extern int epoll_create (int __size) __THROW;

/* Same as epoll_create but with an FLAGS parameter.  The unused SIZE
   parameter has been dropped.  */
extern int epoll_create1 (int __flags) __THROW;


/* Manipulate an epoll instance "epfd". Returns 0 in case of success,
   -1 in case of error ( the "errno" variable will contain the
   specific error code ) The "op" parameter is one of the EPOLL_CTL_*
   constants defined above. The "fd" parameter is the target of the
   operation. The "event" parameter describes which events the caller
   is interested in and any associated user data.  */
extern int epoll_ctl (int __epfd, int __op, int __fd,
		      struct epoll_event *__event) __THROW;


/* Wait for events on an epoll instance "epfd". Returns the number of
   triggered events returned in "events" buffer. Or -1 in case of
   error with the "errno" variable set to the specific error code. The
   "events" parameter is a buffer that will contain triggered
   events. The "maxevents" is the maximum number of events to be
   returned ( usually size of "events" ). The "timeout" parameter
   specifies the maximum wait time in milliseconds (-1 == infinite).

   This function is a cancellation point and therefore not marked with
   __THROW.  */
extern int epoll_wait (int __epfd, struct epoll_event *__events,
		       int __maxevents, int __timeout);


/* Same as epoll_wait, but the thread's signal mask is temporarily
   and atomically replaced with the one provided as parameter.

   This function is a cancellation point and therefore not marked with
   __THROW.  */
extern int epoll_pwait (int __epfd, struct epoll_event *__events,
			int __maxevents, int __timeout,
			const __sigset_t *__ss);

__END_DECLS

#endif /* sys/epoll.h */

```