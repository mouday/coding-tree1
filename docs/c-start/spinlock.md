# 自旋锁

实现方式：

- TAS
- CAS

## TAS

原理：

向内存变量写入1，然后返回内存变量的原值

伪代码

```cpp
// Test And Set
int TAS(int *lock)
{
    int temp = *lock; // 读取旧值
    *lock = 1;        // 无条件设置为1
    return temp;      // 返回旧值
}

void acquire_lock(int *lock)
{
    while (TAS(lock) == 1)
    {
        continue;
    }
}

void release_lock(int *lock)
{
    *lock = 0;
}
```

## CAS

原理：

比较锁种的值和期望值

- 如果锁中的值和期望值相同，则设置为新值，返回true
- 否则不设置新值，返回false

伪代码

```cpp
// Compare And Swap
bool CAS(int *lock, int expected, int value)
{
    if (*lock == expected)
    {
        *lock = value;
        return true;
    }
    else
    {
        return false;
    }
}

void acquire_lock(int *lock)
{
    while (!CAS(lock, 0, 1))
    {
        continue;
    }
}

void release_lock(int *lock)
{
    *lock = 0;
}
```

示例

```cpp
#include "stdbool.h"
#include "stdatomic.h"
#include "stdio.h"
#include "pthread.h"
#include "unistd.h"

typedef struct
{
    atomic_int lock;
} spinlock_t;

void acquire_lock(spinlock_t *lock)
{
    int expected = 0;
    while (!atomic_compare_exchange_weak_explicit(
        &lock->lock,
        &expected,
        1,
        memory_order_acquire,
        memory_order_relaxed))
    {
        expected = 0;
        // 减少忙等待
    }
}

void release_lock(spinlock_t *lock)
{
    atomic_store_explicit(&lock->lock, 0, memory_order_release);
}

int count = 0;
spinlock_t lock = {0};

void *run_task(void *arg)
{
    for (size_t i = 0; i < 10000; i++)
    {
        acquire_lock(&lock);
        count++;
        release_lock(&lock);
    }
    return NULL;
}

int main(int argc, char const *argv[])
{
    const int nthread = 100;
    pthread_t t[nthread];
    for (int i = 0; i < nthread; i++)
    {
        int ret = pthread_create(&t[i], NULL, run_task, NULL);
        if (ret == 0)
        {
            printf("create thread success\n");
        }
        else
        {
            printf("create error, ret: %d\n", ret);
        }
    }

    for (int i = 0; i < nthread; i++)
    {
        pthread_join(t[i], NULL);
    }

    printf("count: %d\n", count);

    return 0;
}
```

## 区别

- TAS：不管原来是什么，都写1
- CAS: 只有原来是0，才会写1

## TTAS

优化 test and test and set

先读取，发现锁空闲再进行CAS

```cpp
void acquire_lock(int *lock)
{
    for(;;){
        while(atomic_load_explicit(lock, memory_order_relaxed) !=0 ){
            continue;
        }

        if (!CAS(lock, 0, 1))
        {
            return;
        }
    }
}
```
