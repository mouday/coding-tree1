# core dump

下面是一个简单的 C 语言程序，它会故意触发 **段错误 (Segmentation Fault)**，从而生成一个 **core dump** 文件。你可以用它来测试 core 文件的生成和调试。

## 代码示例：`core_demo.c`

```c
#include <stdio.h>

int main() {
    int *p = NULL;          // 空指针
    printf("即将访问空指针...\n");
    *p = 42;                // 对空指针写入 -> 段错误，产生 core dump
    return 0;
}
```

## 编译与运行（生成 core 文件）

### 1. 编译（保留调试符号，方便 gdb 分析）

```bash
gcc -g -o core_demo core_demo.c
```

### 2. 允许生成 core 文件（默认可能被限制）

```bash
ulimit -c unlimited       # 设置 core 文件大小无限制
```

### 3. 运行程序

```bash
./core_demo
```

你会看到类似下面的输出，并退出：

```
即将访问空指针...
段错误 (核心已转储)
```

### 4. 查找 core 文件

- 默认通常生成在当前目录，文件名为 `core` 或 `core.<pid>`。
- 可以通过以下命令查看 core 文件生成位置：

```bash
sysctl kernel.core_pattern   # 查看系统 core 文件命名规则
```

常见位置：`/var/lib/systemd/coredump/`（systemd 系统）或当前目录。

## 使用 gdb 分析 core 文件

假设 core 文件名为 `core` 或 `core.12345`：

```bash
gdb ./core_demo core
```

在 gdb 内部：

```bash
(gdb) bt          # 查看崩溃时的调用栈
(gdb) frame 0     # 切换到崩溃的那一帧
(gdb) info locals # 查看局部变量
(gdb) p p         # 打印指针 p 的值（应为 NULL）
(gdb) quit
```

## 其他常见触发 core 的方式

- **除零错误**（整数除法）：

```c
int a = 1, b = 0;
int c = a / b;    // SIGFPE
```

- **栈溢出**（无限递归）：

```c
void foo() { foo(); }
```

- **非法指令**（往代码区写数据）：

```c
char *code = (char*)main;
*code = 0xCC;     // 写代码段
```

## 注意事项

- **生产环境**：core 文件可能包含敏感信息，建议仅在开发/调试环境中启用。
- **systemd 系统**：core 文件可能被压缩存放在 `/var/lib/systemd/coredump/`，可以用 `coredumpctl list` 和 `coredumpctl gdb ./core_demo` 查看。
- **Docker 容器内**：默认通常不生成 core 文件，需要添加 `--ulimit core=-1` 并挂载 core 文件存放路径。

如果你需要特定场景（如 Docker 内产生 core）的完整示例，可以告诉我，我再补充详细步骤。
