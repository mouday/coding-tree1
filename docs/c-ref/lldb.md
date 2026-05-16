# lldb

生成core文件

```shell
# 编译程序，需要加`-g`参数
gcc -g main.c

# cmake打开debug模式
set(CMAKE_BUILD_TYPE Debug)

# 系统设置core文件大小无限制
ulimit -c unlimited
```

打开core文件

```shell
lldb -c  /cores/core.67020
```
