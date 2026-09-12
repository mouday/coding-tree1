# Shell

## printenv

c语言实现

通过main函数传入第3个参数，获取所有环境变量

```cpp
#include <stdio.h>
#include <stdbool.h>

int main(int argc, char const *argv[], const char *envp[])
{
    int i = 0;
    while (true)
    {
        if (envp[i] == NULL)
        {
            break;
        }
        printf("\x1b[33m%s\x1b[0m\n", envp[i]);
        i++;
    }
    return 0;
}
```

运行

```shell
gcc printenv.c -o printenv && ./printenv
```
