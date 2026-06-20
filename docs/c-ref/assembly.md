# 汇编语言

汇编语言是机器指令的助记符

汇编语言的组成

1. 汇编指令（机器码的助记符）
2. 伪指令（由编译器执行）
3. 其他符号（由编译器识别，如：+ - * /）

总线

1. 地址总线：存储单元的地址
2. 数据总线：读或写的数据
3. 控制总线：读或写命令

寄存器

CPU = 运算器+控制器+【寄存器】

8086CPU有14个寄存器，名称分别为：
AX，BX，CX，DX，SI，DI，SP，BP，IP，CS，SS，DS，ES，PSW

x64dbg

x86汇编语言是理解计算机底层工作的绝佳入口。下面为你准备了一份极简但完整的入门指南，重点在**让你能看懂并动手写第一个程序**。

## 1. 核心概念

- **寄存器 (Registers)**：CPU内部的高速临时存储“盒子”。入门先记这四个：
  - `EAX, EBX, ECX, EDX`：通用寄存器，存数据，做运算。
  - `EIP` (指令指针)：指向下一条要执行的指令。
  - `ESP, EBP` (栈指针，基址指针)：用于管理函数调用和内存。
- **内存模型**：程序可以读写内存地址（如 `[0x1234]`），访问时间比寄存器长。**关键概念：栈 (Stack)**，一种“后进先出”的数据结构，用 `PUSH`（压栈）和 `POP`（出栈）操作，ESP 寄存器指向栈顶。
- **指令格式**：`操作码 (Opcode) 目标, 源`
  - `MOV EAX, 123`：把数字123放入`EAX`
  - `ADD EAX, 5`：把`EAX`的值加5

## 2. 环境搭建 (5分钟)

**方案A：最简单 - 在线模拟器** (如 [OneCompiler's Assembly](https://onecompiler.com/assembly))
**方案B：本地 - 安装NASM + QEMU** (推荐，真实体验)
- **Windows**：安装WSL或在CMD中通过winget安装：`winget install nasm -s winget`
- **macOS**：`brew install nasm qemu`
- **Linux**：`sudo apt install nasm qemu-system-x86`

## 环境搭建

vscode插件

- `x86 and x86_64 Assembly` 语法高亮

编译运行脚本

```shell
#!/bin/bash
# bash asm.sh hello.asm

filename=$1
# 移除后缀
filename=${filename%.*}

nasm -f elf32 -g -F dwarf ${filename}.asm -o ${filename}.o && \
ld -m elf_i386 ${filename}.o -o ${filename} && \
./"${filename}"
```

临时方案：https://onecompiler.com/assembly

## 3. 第一个程序：Hello World (32位)


保存为 `hello.asm`。这段程序在Linux系统调用下输出"Hello World"。

```shell
section .data               ; 数据段：已初始化的数据
    msg db 'Hello World', 0xa  ; db = 定义字节, 0xa = 换行符
    len equ $ - msg            ; equ = 常量, $ = 当前位置, 计算msg长度

section .text               ; 代码段
    global _start           ; 告诉链接器入口点是 _start

_start:
    ; 系统调用 write(1, msg, len)
    ; 1 = 标准输出 (stdout)
    mov eax, 4              ; 系统调用号 4 = sys_write
    mov ebx, 1              ; 文件描述符 1
    mov ecx, msg            ; 字符串地址
    mov edx, len            ; 字符串长度
    int 0x80                ; 触发系统调用中断

    ; 系统调用 exit(0)
    mov eax, 1              ; 系统调用号 1 = sys_exit
    xor ebx, ebx            ; 退出码 0 (xor清零)
    int 0x80
```

**编译运行 (Linux环境)**：
```bash
nasm -f elf32 hello.asm -o hello.o         # 编译成32位目标文件
ld -m elf_i386 hello.o -o hello            # 链接成可执行文件
./hello
```

## 4. 常用指令表

| 类别 | 指令 (示例) | 含义 |
|------|-------------|------|
| 数据传送 | `MOV EAX, EBX` | 将EBX的值复制到EAX |
| 数据传送 | `MOV EAX, [var]` | 读取变量var的值到EAX |
| 算术 | `ADD EAX, 5` | EAX = EAX + 5 |
| 算术 | `SUB EAX, 5` | EAX = EAX - 5 |
| 算术 | `INC EAX` | EAX++ |
| 逻辑/移位 | `AND/OR/XOR` | 按位与/或/异或 |
| 比较 | `CMP EAX, EBX` | 比较EAX和EBX，设置标志位 |
| 条件跳转 | `JE/JZ label` | 如果相等/为零，跳转到label |
| 条件跳转 | `JNE/JNZ label` | 如果不相等/非零，跳转 |
| 无条件跳转 | `JMP label` | 直接跳转 |
| 栈操作 | `PUSH EAX` | 将EAX压入栈顶，ESP减4 |
| 栈操作 | `POP EAX` | 栈顶值弹出到EAX，ESP加4 |
| 调用/返回 | `CALL func` | 调用函数，返回地址压栈 |
| 调用/返回 | `RET` | 从函数返回，地址出栈 |

## 5. 实践：写一个求和函数

```shell
section .text
global _start

_start:
    mov eax, 10
    mov ebx, 20
    call sum_func           ; 调用函数

    ; 此时EAX中是返回值(30)，作为退出码
    mov ebx, eax
    mov eax, 1
    int 0x80

sum_func:
    add eax, ebx            ; EAX = EAX + EBX
    ret                     ; 返回，EAX就是返回值
```
运行后 `echo $?` 会输出 30。

## 6. 调试工具：观察寄存器

使用 `gdb` 调试：
```bash
gdb ./hello
(gdb) break _start
(gdb) run
(gdb) info registers     ; 查看所有寄存器
(gdb) stepi              ; 单步执行一条指令
(gdb) x/s &msg           ; 以字符串形式查看msg地址的内容
```

## 7. 下一步学什么？

- **指令集选择**：先熟悉**32位 (IA-32)**，再学**64位 (x86-64，寄存器和调用约定有变化)**。
- **函数调用约定**：如 `cdecl` (32位) 和 `System V AMD64` (64位)，理解参数如何传递、栈帧。
- **浮点与向量**：`FPU`、`MMX`、`SSE`、`AVX`。
- **实践项目**：编写字符串反转函数、冒泡排序；尝试 **“C语言编译成汇编”** (`gcc -S hello.c`) 对照学习。

## 总结指令速记卡

- `MOV` - 复制
- `ADD/SUB/INC/DEC` - 加减
- `CMP` - 比较 (影响 ZF, CF 等标志位)
- `JMP/JE/JNE/JG/JL` - 跳转
- `PUSH/POP` - 栈操作
- `CALL/RET` - 函数调用
- `INT 0x80` (32位) / `syscall` (64位) - 系统调用

**记忆技巧**：Intel风格汇编的典型格式是 `INSTRUCTION DESTINATION, SOURCE`。

现在你可以从第一个Hello World开始了。如果遇到链接错误，请检查你的系统是32位还是64位，可能需要调整编译选项（比如 `-m32`）或改用64位系统调用（使用 `syscall` 而不是 `int 0x80`）。祝你编码愉快！


在使用 GDB 调试汇编程序时，“查看变量的值”通常指**查看内存中数据段（`.data`）或栈上变量的内容**，以及**查看寄存器的值**。下面以经典的 Hello World 汇编程序为例，介绍常用命令。

---

## 1. 准备工作：编译时添加调试信息

使用 NASM 编译时加上 **`-g -F dwarf`** 选项，这样 GDB 可以识别符号（变量名、函数名）。

```bash
nasm -f elf32 -g -F dwarf hello.asm -o hello.o
ld -m elf_i386 hello.o -o hello
```

---

## 2. 启动 GDB 并设置断点

```bash
gdb ./hello
(gdb) break _start      ; 在入口处打断点
(gdb) run               ; 运行程序，停在 _start
```

---

## 3. 查看寄存器

- **所有寄存器**：`info registers` 或简写 `info reg`
- **单个寄存器**：`p $eax`（`p` 是 `print` 的简写）

```shell
(gdb) info reg eax      ; 只显示 EAX
eax            0x0      0
(gdb) p $ebx            ; 另一种方式
$1 = 0
```

---

## 4. 查看内存变量（`.data` 段）

假设程序中有：

```shell
section .data
    msg db 'Hello World', 0xa   ; 字符串 + 换行
    len equ $ - msg
```

用 **`x`（examine）命令** 查看内存：

| 命令格式 | 含义 |
|---------|------|
| `x/s &msg` | 以字符串形式显示 `msg` 的内容 |
| `x/d &len` | 以十进制整数显示 `len`（注意 `len` 是常量，没有地址，但 GDB 会处理成立即数） |
| `x/10xb &msg` | 显示从 `msg` 起始的 10 个字节（十六进制） |
| `x/wx &msg` | 显示一个 4 字节字（十六进制） |

**示例会话：**

```shell
(gdb) x/s &msg
0x804a000 <msg>:        "Hello World\n"

(gdb) x/d &len          ; len 是 equ 常量，GDB 也能识别
0x804a00c:     12

(gdb) x/6xb &msg        ; 显示前 6 个字节的十六进制
0x804a000 <msg>:       0x48    0x65    0x6c    0x6c    0x6f    0x20
```

> 注意：`&msg` 取变量的地址，如果你直接写 `x/s msg`，GDB 会把 `msg` 当作地址值，效果相同。为清晰推荐 `&msg`。

---

## 5. 查看栈上的局部变量

栈变量通常在 `ebp` 或 `esp` 附近。例如函数内定义 `local_var equ -4`，则查看：

```shell
(gdb) x/d $ebp-4        ; 以十进制显示
(gdb) x/wx $esp+8       ; 显示栈上参数
```

更直观的方法：先用 `info frame` 查看栈帧信息，再配合 `x`。

---

## 6. 其他有用命令

- **反汇编当前函数**：`disas` 或 `disassemble`
- **单步执行一条机器指令**：`stepi` 或 `si`
- **查看下一条指令**：`x/i $eip`
- **自动显示某些值**：`display /s &msg`（每次停止时自动显示）

---

## 7. 注意事项

- **32 位 vs 64 位**：64 位汇编中寄存器名变为 `rax, rbx, ...`，地址长度 8 字节，查看时用 `x/gx`（8 字节十六进制）。
- **变量位置**：如果变量在 `.bss` 段（未初始化），需要用 `x` 命令查看，但内容可能是随机值。
- **符号未找到**：如果 GDB 提示 “No symbol ...”，检查编译时是否真的加了 `-g -F dwarf`，且变量名是否用 `global` 导出（对于 `.data` 段变量，不需要导出也能被 GDB 识别）。

---

## 快速参考表

| 目的 | GDB 命令 |
|------|----------|
| 查看所有寄存器 | `info reg` |
| 查看 EAX 的值 | `p $eax` |
| 查看字符串变量 `msg` | `x/s &msg` |
| 查看整数变量 `count` | `x/d &count` |
| 查看地址 `0x804a000` 开始的 4 个字节（十六进制） | `x/4wx 0x804a000` |
| 查看栈顶的 4 个字 | `x/4wx $esp` |
| 每次停止时自动显示 `msg` | `display /s &msg` |

掌握这几个命令，就能在调试汇编程序时轻松观察所有变量和寄存器的状态。
