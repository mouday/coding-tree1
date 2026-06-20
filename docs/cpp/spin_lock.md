# 自旋锁

锁实现的困难点：不能同时读/写共享内存

- load (环顾四周) 的时候不能写，只能看一眼，看到的东西立马就过时了
- store (改变状态) 的时候不能读，只能盲改，也不知道把啥呢么改成了什么

项目结构

```shell
main.c
spin_lock.c
spin_lock.h
```

代码实现

spin_lock.h

```cpp
typedef int spin_lock_t;

void spin_lock(spin_lock_t *lock);

void spin_unlock(spin_lock_t *lock);
```

spin_lock.c

```cpp
// spin_lock.c
#include "spin_lock.h"

static inline int atomic_xchg(volatile int *addr, int newval)
{
    int result;
    asm volatile("lock xchg %0, %1" : "+m"(*addr), "=a"(result) : "1"(newval) : "memory");
    return result;
}

void spin_lock(spin_lock_t *lock)
{
    while (1)
    {
        if (atomic_xchg(lock, 1) == 0)
        {
            break;
        }
    }
}

void spin_unlock(spin_lock_t *lock)
{
    atomic_xchg(lock, 0);
}

```

main.c

```cpp
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include "spin_lock.h"

#define N 1000000

spin_lock_t lock;
long n, sum = 0;

void *get_sum(void *arg)
{   
    for (int i = 0; i < n; i++)
    {
        spin_lock(&lock);
        sum++;
        spin_unlock(&lock);
    }

    return NULL;
}

int main(int argc, char const *argv[])
{
    printf("argc: %d\n", argc);
    if (argc != 2){
        return 1;
    }

    int nthread = atoi(argv[1]);
    n = N / nthread;
    pthread_t t[nthread];
    for (int i = 0; i < nthread; i++)
    {
        pthread_create(&t[i], NULL, get_sum, NULL);
    }

    for (int i = 0; i < nthread; i++)
    {
        pthread_join(t[i], NULL);
    }

    printf("sum: %ld\n", sum);
    return 0;
}
```

输出结果

```shell
gcc -g spin_lock.c main.c -o main -lpthread && time ./main 100
```

统计结果

| n | real | user | sys
| - | - | - | -
| 1 | 0m0.032s | 0m0.030s | 0m0.001s
| 10 | 0m0.446s | 0m3.379s | 0m0.014s
| 100 | 0m2.983s | 0m23.499s | 0m0.167s
| 1000 | 0m0.236s | 0m0.049s | 0m0.470s
| 10000 | 0m2.196s | 0m0.185s | 0m4.123s

结论：自旋锁的存在，线程数越多，耗时越久

使用自旋锁的场景：

1. 临界区同一时间只有一个线程进入
2. 操作系统内核的并发数据结构（短临界区）

对比：pthread_mutex_t

| n | real | user | sys
| - | - | - | -
| 1 | 0m0.031s | 0m0.027s | 0m0.002s
| 10 | 0m0.083s | 0m0.151s | 0m0.240s
| 100 | 0m0.095s | 0m0.192s | 0m0.382s
| 1000 | 0m0.241s | 0m0.043s | 0m0.469s
| 10000 | 0m2.427s | 0m0.205s | 0m4.609s
