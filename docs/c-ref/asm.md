# 汇编语言

汇编语言（Assembly Language）

## 简介

汇编指令和机器指令之间存在 `一一对应` 的关系。

| 形式               | 示例                                         | 说明                     |
| ------------------ | -------------------------------------------- | ------------------------ |
| 机器码（二进制）   | 10111000 00101010 00000000 00000000 00000000 | CPU 直接执行的指令       |
| 机器码（十六进制） | B8 2A 00 00 00                               | 方便人类阅读的机器码表示 |
| 汇编语言           | `mov eax, 42`  | 人类可读的助记符表示     |

## 环境搭建

- Windows
  - WSL（Windows Subsystem for Linux）
  - Linux虚拟机

- macOS
  - Linux虚拟机
  - Docker 容器

- Ubuntu/Debian

```shell
# nasm
apt install nasm

# gcc
apt install gcc

# gdb
apt install gdb

# 在 64 位系统上编译 32 位汇编兼容库
apt install gcc-multilib
```

VSCode: Markdown Code blocks Assembly Syntax highlighting

hello world

```asm
; 文件路径：hello.asm
; 第一个汇编程序：打印 Hello, runoob!
; NASM 语法，Linux 32位

section .data                       ; 数据段：存放已初始化的数据
    msg db 'Hello, World!', 0xA     ; 定义字符串 msg，0xA 是换行符
    len equ $ - msg                 ; 计算字符串长度：当前地址 - msg 起始地址

section .text                       ; 代码段：存放可执行指令
    global _start                   ; 声明 _start 为程序入口点

_start:                             ; 程序入口
    ; 系统调用 sys_write (4)：向 stdout 输出字符串
    mov eax, 4                      ; 系统调用号 4 = sys_write
    mov ebx, 1                      ; 文件描述符 1 = stdout（标准输出）
    mov ecx, msg                    ; 要输出的字符串地址
    mov edx, len                    ; 字符串长度
    int 0x80                        ; 触发中断，执行系统调用

    ; 系统调用 sys_exit (1)：正常退出程序
    mov eax, 1                      ; 系统调用号 1 = sys_exit
    mov ebx, 0                      ; 返回值 0 = 正常退出
    int 0x80                        ; 触发中断，执行系统调用
```

编译运行

```shell
# 汇编
nasm -f elf32 -g -F dwarf hello.asm -o hello.o
# 链接
ld -m elf_i386 hello.o -o hello
# 运行
./hello
```

为了方便编译，可以写如下编译脚本

```shell
#!/bin/bash
# asm.sh

filename=$1
# 移除后缀
filename=${filename%.*}

nasm -f elf32 -g -F dwarf ${filename}.asm -o ${filename}.o && \
ld -m elf_i386 ${filename}.o -o ${filename} && \
./"${filename}"
```

使用

```shell
bash asm.sh hello.asm
```

GDB调试

```shell
$ gdb ./hello
(gdb) break _start                             # 在 _start 处设置断点
(gdb) run                                      # 运行程序
(gdb) stepi                                    # 单步执行一条指令
(gdb) info registers                           # 查看所有寄存器
(gdb) x/s msg                                  # 查看 msg 字符串内容
(gdb) i r  eax                                 # 查看单个寄存器的值
(gdb) quit                                     # 退出 GDB
```

| GDB 命令 | 说明 |
|------|----------|
| `p $eax` | 打印寄存器的值
| `x/s &msg` | 查看字符串变量 `msg` | 
| `p/d &len` | 查看整数变量 `len` |

## 基础语法

汇编程序的基本结构

```shell
section .data                       ; 数据段：存放已初始化的数据
    ; 这里定义变量和常量
    msg db 'Hello, RUNOOB!', 0xA
    len equ $ - msg

section .bss                        ; BSS 段：存放未初始化的数据
    ; 这里预留内存空间
    buffer resb 64

section .text                       ; 代码段：存放可执行指令
    global _start
```

汇编语句格式

```shell
[标签:] 指令助记符 [操作数1 [, 操作数2 [, 操作数3]]] [; 注释]
```

伪指令（Directives）

伪指令 是给汇编器的命令，不是给 CPU 的指令，用于控制汇编过程和定义数据结构。

|伪指令	| 用途	| 示例|
|-|-|-|
db	| 定义字节（1 字节）	| byte_val db 0x55
dw	| 定义字（2 字节）	| word_val dw 0x1234
dd	| 定义双字（4 字节）	| dword_val dd 0x12345678
equ	| 定义常量	| MAX_SIZE equ 256
resb|	预留字节空间	| buffer resb 128
resw|	预留字空间	| wbuf resw 64
resd	|预留双字空间	| dbuf resd 32
%define |	宏定义常量	| %define COUNT 10

大小写规范

- NASM 默认对标签和标识符 区分大小写：
- 指令助记符和寄存器名不区分大小写（MOV、Mov、mov 效果相同），但推荐的风格是统一小写。

数值表示方式

```shell
; NASM 中不同进制的表示方式

mov eax, 42             ; 十进制：直接写数字

mov eax, 0x2A           ; 十六进制：0x 前缀（推荐写法）
mov eax, 2Ah            ; 十六进制：h 后缀

mov eax, 0o52           ; 八进制：0o 前缀
mov eax, 52o            ; 八进制：o 后缀

mov eax, 101010b        ; 二进制：b 后缀
mov eax, 0b101010       ; 二进制：0b 前缀
```

示例：计算 1 到 10 的和并输出

```shell
; 文件路径：sum.asm
; 计算 1+2+...+10 并输出结果字符

section .data
    result db 3                 ; 存放计算结果（1字节）
    newline db 0xA              ; 换行符

section .text
    global _start

_start:
    ; 初始化寄存器和变量
    mov ecx, 10                 ; 循环计数器：从 10 开始倒数
    mov eax, 0                  ; eax 存放累加和，初始为 0

sum_loop:                       ; 循环开始标签
    add eax, ecx                ; eax = eax + ecx
    dec ecx                     ; ecx 减 1
    jnz sum_loop                ; 如果 ecx != 0，继续循环

    ; 此时 eax = 55（10+9+...+1）
    add eax, '0'                ; 将数字转为 ASCII 字符（'0'=48，55+48=103='g'，不对）
                                ; 实际演示需要更复杂的转换，见后续章节

    ; 这里只输出 result（简化演示）
    mov [result], al            ; 将累加结果存入 result

    ; 退出程序
    mov eax, 1
    mov ebx, 0
    int 0x80
```

## 内存分段

内存分段（Memory Segmentation）

将内存划分为 段（Segment）

物理地址计算公式(16 位寄存器)：

```shell
物理地址 = 段地址 × 16 + 偏移地址
```

三种基本段
一个典型的汇编程序使用三个主要段：

| 段名称	| 英文名| 	用途| 	对应的 Section
| - | - | - | -|
代码段	| Code Segment	| 存放可执行的机器指令，只读、可执行 | section .text
数据段	| Data Segment	| 存放已初始化的全局变量，可读可写 | section .data
栈段	| Stack Segment	| 存放函数调用信息、局部变量 | 操作系统自动管理

代码段（.text）

示例

```shell
; 文件路径：code_segment.asm
; 演示代码段使用

section .text                   ; 开始代码段
    global _start

_start:
    mov eax, 1                  ; 这些指令都存放在代码段中
    mov ebx, 0
    int 0x80
```

数据段（.data）

```shell
; 文件路径：data_segment.asm
; 演示数据段使用

section .data
    ; 定义各种类型的已初始化变量
    msg db 'Hello, world!', 0     ; 字符串变量（字节序列）
    count db 100                    ; 字节变量，初始值 100
    pi dd 314159                    ; 双字变量，存放 π 的近似值 × 100000
    array db 1, 2, 3, 4, 5         ; 字节数组

section .text
    global _start

_start:
    ; 读取数据段中的变量
    mov al, [count]                ; 将 count 的值（100）加载到 al 寄存器
    ; ...
    mov eax, 1
    mov ebx, 0
    int 0x80
```

BSS 段（.bss）

BSS（Block Started by Symbol）段存放 未初始化的全局变量。


```shell
; 文件路径：bss_segment.asm
; 演示 BSS 段使用

section .bss
    buffer resb 256                 ; 预留 256 字节的缓冲区（未初始化）
    num_array resd 100              ; 预留 100 个双字（400 字节）

section .data
    ; 已初始化数据

section .text
    global _start

_start:
    ; 使用 BSS 段的缓冲区
    mov byte [buffer], 'A'          ; 向缓冲区写入字符 'A'
    ; ...
    mov eax, 1
    mov ebx, 0
    int 0x80
```

段寄存器

|段寄存器	| 英文全称	| 用途
| - | - | - |
CS	| Code Segment	| 指向代码段，存放当前执行指令所在的段基址
DS	| Data Segment	| 指向数据段，存放大部分数据访问的段基址
SS	| Stack Segment	| 指向栈段，存放栈所在的段基址
ES	| Extra Segment	| 附加段寄存器，用于字符串操作等额外数据段
FS	| General Purpose	| 通用段寄存器，常用于线程局部存储
GS	| General Purpose	| 通用段寄存器，常用于线程局部存储

内存分段示意图

```shell
高地址
+-------------------+
|   栈 (Stack)      |  <-- SS:ESP 指向栈顶
|   向下增长         |
+-------------------+
|                   |
|   空闲内存         |
|                   |
+-------------------+
|   BSS 段 (.bss)   |  未初始化的全局变量
+-------------------+
|   数据段 (.data)   |  已初始化的全局变量
+-------------------+
|   代码段 (.text)   |  程序指令（只读）
+-------------------+
|   保留区域         |  操作系统使用
+-------------------+
低地址
```

## 寄存器

寄存器（Register）是 CPU 内部的高速存储单元

通用寄存器

通用寄存器（General Purpose Registers） 是最常用的寄存器，用于存放运算数据和临时结果。

x86 提供了 8 个 32 位通用寄存器：

| 32位|	16位|	低8位	|高8位（低16位）|	主要用途
|-|-|-|-|-|
EAX	| AX	| AL	| AH |  	累加器，存放函数返回值、算术运算结果
EBX	| BX	| BL	| BH |  	基址寄存器，常用于存放内存基地址
ECX	| CX	| CL	| CH |  	计数器，常用于循环计数和移位
EDX	| DX	| DL	| DH |  	数据寄存器，存放乘除法的高位结果
ESI	| SI	| SIL	| -	 |  源变址寄存器，字符串操作的源地址
EDI	| DI	| DIL	| -	 |  目的变址寄存器，字符串操作的目标地址
EBP	| BP	| BPL	| -	 |  基址指针，指向当前栈帧的底部
ESP	| SP	| SPL	| -	 |  栈指针，始终指向栈顶

名称规律：E 前缀表示 Extended（扩展到 32 位），X 后缀表示可拆分为高低字节。

```shell
; 文件路径：register_parts.asm
; 演示寄存器的各部分访问

section .text
    global _start

_start:
    mov eax, 0x12345678     ; 完整的 32 位寄存器
    ; 此时：EAX = 0x12345678
    ;       AX  = 0x5678      (低 16 位)
    ;       AH  = 0x56        (高 8 位，指 AX 的高 8 位)
    ;       AL  = 0x78        (低 8 位)

    mov ax, 0xAABB          ; 修改 AX（低 16 位）
    ; 此时：EAX = 0x1234AABB  (高 16 位保持不变！)
    ;       AX  = 0xAABB
    ;       AL  = 0xBB

    mov al, 0xCC            ; 修改 AL（最低 8 位）
    ; 此时：EAX = 0x1234AACC  (只有低 8 位变了)
    ;       AX  = 0xAACC
    ;       AL  = 0xCC

    mov eax, 1
    mov ebx, 0
    int 0x80
```

段寄存器

段寄存器用于指定当前使用的内存段：

| 寄存器 | 名称	| 用途
|-|-|-|
CS	  | 代码段寄存器	| 指向当前指令所在的段
DS	  | 数据段寄存器	| 指向数据所在的段
SS	  | 栈段寄存器	    | 指向栈所在的段
ES	  | 附加段寄存器	| 额外的数据段
FS	  | 附加段寄存器	| 通用，常用于线程局部存储
GS	  | 附加段寄存器	| 通用，常用于线程局部存储

指针和变址寄存器

这些寄存器主要用于访问内存，存放内存地址：

|寄存器|	全称	|用途
|-|-|-|
EIP	| 指令指针（Instruction Pointer）	|指向 CPU 下一条要执行的指令地址（不可直接访问）
ESP	| 栈指针（Stack Pointer）	|指向栈顶，PUSH/POP 指令自动调整它
EBP	| 基址指针（Base Pointer）	|指向当前函数栈帧的底部，用于访问函数参数和局部变量
ESI	| 源变址（Source Index）	|字符串/内存操作的源地址
EDI	| 目的变址（Destination Index）	|字符串/内存操作的目标地址


标志寄存器（EFLAGS）

| 标志位	| 名称	| 含义
| -	| -	| -
CF	| 进位标志（Carry Flag）	|无符号运算产生进位/借位时置 1
PF	| 奇偶标志（Parity Flag）	|结果低 8 位中 1 的个数为偶数时置 1
AF	| 辅助进位标志	|低 4 位向高 4 位进位/借位时置 1
ZF	| 零标志（Zero Flag）	|运算结果为 0 时置 1
SF	| 符号标志（Sign Flag）|	运算结果为负数时置 1（等于结果的最高位）
OF	| 溢出标志（Overflow Flag）	| 有符号运算溢出时置 1


```shell
; 文件路径：flags_demo.asm
; 演示运算对标志位的影响

section .text
    global _start

_start:
    mov eax, 10
    sub eax, 10         ; 10 - 10 = 0
    ; ZF = 1（结果为零）
    ; SF = 0（结果非负）
    ; CF = 0（无借位）

    mov eax, 0xFFFFFFFF
    add eax, 1          ; 0xFFFFFFFF + 1 = 0x100000000（超出了 32 位）
    ; ZF = 1（32位结果为0）
    ; CF = 1（产生进位）
    ; OF = 0（有符号角度看无溢出）

    mov eax, 1
    mov ebx, 0
    int 0x80
```

寄存器使用约定

在实际编程中，一些寄存器有约定俗成的用法——称为 调用约定（Calling Convention）：

| 寄存器 | 调用约定中的用途
|-|-
EAX|	存放函数返回值
ECX	|计数器（循环计数）
EDX	|存放除法的高位结果、扩展 EAX
EBX、ESI、EDI、EBP |	被调用函数必须保存和恢复（callee-saved）
EAX、ECX、EDX	| 调用者负责保存（caller-saved）

## 系统调用

系统调用（System Call）是用户程序与操作系统内核交互的唯一途径。

系统调用的流程是：

```shell
用户程序发起系统调用
 → 
CPU 切换到内核态
 → 
内核执行对应服务
 → 
返回用户态继续执行。
```

Linux 系统调用机制
在 Linux 32 位系统中，系统调用通过以下步骤完成：

- 将 `系统调用号` 放入 EAX 寄存器
- 将参数放入 EBX、ECX、EDX、ESI、EDI 等寄存器
- 执行 `int 0x80` 指令触发软件中断
- CPU 切换到内核态，内核根据 EAX 中的调用号执行对应服务
- 内核将返回值放入 EAX，切换回用户态继续执行

常用系统调用一览

|系统调用	| 调用号（EAX）	| 	功能		| 参数
|-|-|-|-|
sys_exit	| 1	|  退出程序	|  EBX = 返回值（exit code）
sys_fork	| 2	|  创建子进程	|  无参数
sys_read	| 3	|  读取文件/输入| 	EBX=fd, ECX=buf, EDX=count
sys_write	| 4	|  写入文件/输出| 	EBX=fd, ECX=buf, EDX=count
sys_open	| 5	|  打开文件	| EBX=filename, ECX=flags, EDX=mode
sys_close	| 6	|  关闭文件	| EBX=fd
sys_creat	| 8	|  创建文件	| EBX=filename, ECX=mode
sys_lseek	| 19| 	移动文件指针	| EBX=fd, ECX=offset, EDX=whence
sys_brk	   | 45	| 调整数据段大小（内存分配）	| EBX=新地址

完整系统调用列表 `/usr/include/asm/unistd_32.h`

sys_exit

```shell
section .text
    global _start

_start:
    mov eax, 1 ; sys_exit
    mov ebx, 6 ; exit code
    int 0X80
```

执行结果

```shell
bash asm.sh main.asm

echo $?
6
```

sys_write

```shell
; 文件路径：write_syscall.asm
; 使用 sys_write 输出字符串

section .data
    msg db 'Hello, World! Welcome to assembly.', 0xA
    len equ $ - msg

section .text
    global _start

_start:
    ; 调用 sys_write (4)
    mov eax, 4                  ; 系统调用号 4 = sys_write
    mov ebx, 1                  ; 文件描述符 1 = stdout（标准输出）
    mov ecx, msg                ; 要输出的数据地址
    mov edx, len                ; 数据长度（字节数）
    int 0x80                    ; 触发系统调用

    ; 成功时 EAX 返回实际写入的字节数

    ; 退出程序
    mov eax, 1
    mov ebx, 0
    int 0x80
```

运行结果

```shell
$ bash asm.sh write_syscall.asm

Hello, World! Welcome to assembly.
```

sys_read

```shell
; 文件路径：read_syscall.asm
; 使用 sys_read 读取用户输入并回显

section .bss
    buffer resb 64              ; 预留 64 字节输入缓冲区

section .data
    prompt db 'Please enter your name: '
    prompt_len equ $ - prompt
    output_msg db 'Hello, '
    output_msg_len equ $ - output_msg

section .text
    global _start

_start:
    ; 输出提示信息
    mov eax, 4
    mov ebx, 1
    mov ecx, prompt
    mov edx, prompt_len
    int 0x80

    ; 读取用户输入
    mov eax, 3                  ; 系统调用号 3 = sys_read
    mov ebx, 0                  ; 文件描述符 0 = stdin（标准输入）
    mov ecx, buffer             ; 输入缓冲区地址
    mov edx, 64                 ; 最多读取 64 字节
    int 0x80

    ; EAX 返回实际读取的字节数（包含末尾的换行符）
    mov esi, eax                ; 保存实际输入长度到 esi

    ; 输出 "Hello, "
    mov eax, 4
    mov ebx, 1
    mov ecx, output_msg
    mov edx, output_msg_len
    int 0x80

    ; 输出用户输入的内容
    mov eax, 4
    mov ebx, 1
    mov ecx, buffer
    mov edx, esi                ; 使用实际输入的字节数
    int 0x80

    ; 退出程序
    mov eax, 1
    mov ebx, 0
    int 0x80
```

运行结果

```shell
$ bash asm.sh read_syscall.asm

Please enter your name: Tom
Hello, Tom
```

系统调用通用模板

```shell
; 系统调用通用模板
; 适用于 Linux 32 位系统

; 步骤1：将系统调用号放入 EAX
mov eax, 系统调用号

; 步骤2：按顺序放入参数
mov ebx, 第1个参数
mov ecx, 第2个参数
mov edx, 第3个参数
mov esi, 第4个参数
mov edi, 第5个参数

; 步骤3：触发系统调用
int 0x80

; 步骤4：检查返回值（在 EAX 中）
; 通常负值表示错误
cmp eax, 0
jl error_handler              ; 如果 EAX < 0，跳转到错误处理
```

## 寻址方式

寻址方式（Addressing Mode）

寻址方式总览

| 寻址方式	| 语法格式	| 有效地址/值		| 典型用途
| -| -| -| -|
| 立即寻址	| mov eax, 42		| 42（常数值）	| 初始化、常量运算
| 寄存器寻址	| mov eax, ebx		| 寄存器 ebx 的值| 	寄存器间数据传递
| 直接寻址	| mov eax, [var]		| 内存地址 var 处的值	| 访问全局变量
| 间接寻址	| mov eax, [ebx]		| 地址 = ebx 的值	| 指针操作、遍历内存
| 基址寻址	| mov eax, [ebx+8]		| 地址 = ebx + 8	| 结构体成员访问
| 变址寻址	| mov eax, [arr+esi*4]		| 地址 = arr + esi × 4	| 一维数组访问
| 基址变址	| mov eax, [ebx+esi*4+8]		| 地址 = ebx + esi × 4 + 8	| 二维数组、复杂结构体