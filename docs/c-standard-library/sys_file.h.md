# sys/file.h

```cpp
#include <sys/file.h>
```

## flock

flock 是 Linux/Unix 系统下一个轻量级的文件锁工具，核心作用是为整个文件提供“建议性锁”（Advisory Lock）。

```cpp
/**
 * fd: 文件描述符
 * operation: 指定要执行的操作，包括锁类型和可选的行为标志
 */
int flock(int fd, int operation);
```

组合用法示例

| 操作值 | 含义 |
|--------|------|
| `LOCK_SH` | 获取共享锁，若无法立即获得则阻塞等待 |
| `LOCK_EX` | 获取独占锁，若无法立即获得则阻塞等待 |
| `LOCK_UN` | 释放锁 |
| `LOCK_SH \| LOCK_NB` | 尝试获取共享锁，若无法立即获得则立即返回错误 |
| `LOCK_EX \| LOCK_NB` | 尝试获取独占锁，若无法立即获得则立即返回错误 |

示例

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <sys/file.h> // 提供 flock() 和相关宏
#include <unistd.h>   // 提供 STDOUT_FILENO, sleep()
#include <fcntl.h>    // 提供 open() 的常量标志

int main()
{
    // 1. 打开（或创建）一个文件，作为锁文件
    int fd = open("/tmp/myapp.lock", O_RDWR | O_CREAT, 0644);
    if (fd < 0)
    {
        perror("open file failed");
        exit(EXIT_FAILURE);
    }

    printf("PID %d: 尝试获取独占锁...\n", getpid());

    // 2. 尝试获取一个独占锁（排他锁），如果锁被占用，则在此阻塞等待
    if (flock(fd, LOCK_EX) == 0)
    {
        printf("PID %d: 成功获得锁！开始执行关键操作...\n", getpid());

        // 模拟一段需要保护的关键区域，比如写文件或处理数据
        sleep(5);

        // 3. 操作完成，释放锁
        flock(fd, LOCK_UN);
        printf("PID %d: 锁已释放。\n", getpid());
    }
    else
    {
        perror("flock failed");
    }

    // 4. 关闭文件，锁也会被自动释放
    close(fd);
    return 0;
}
```
