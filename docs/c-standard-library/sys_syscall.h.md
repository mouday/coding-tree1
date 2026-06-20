# sys/syscall.h

头文件

```cpp
#include <sys/syscall.h>
```

## syscall

```cpp
#include <stdio.h>
#include <unistd.h>
#include <sys/syscall.h>

int main(int argc, char const *argv[])
{
    pid_t tid = syscall(SYS_gettid);
    printf("tid: %d\n", tid);
    return 0;
}
```

输出

```shell
gcc -g thread.c -o thread  && ./thread

tid: 139
```
