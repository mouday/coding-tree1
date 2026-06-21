# 汇编语言 - 宏

宏（Macro）是 NASM 提供的一种代码复用机制，它允许你在编译时展开代码模板，减少重复编写相似代码的需求。

---

## 什么是宏

宏（Macro） 是一种编译时文本替换机制，由汇编器的预处理器在编译前展开。

与过程（Procedure）不同，宏不会有 CALL/RET 开销——每次使用宏时，编译器直接将宏的内容复制到使用位置。

宏不是函数调用，没有栈帧开销；但可执行文件体积会更大，因为每次展开都会复制一份代码。

---

## 单行宏：%define

`%define` 是单行宏，语法简单：

## 实例

```asm
; 单行宏示例  

; 定义常量  

%define MAX_SIZE 256  

%define APP_NAME 'runoob'  

; 定义带参数的宏（宏函数）  

%define mul_by_2(x) (x * 2)  

%define sum3(a, b, c) ((a) + (b) + (c))  

section .text  

    global _start  

_start:  

    mov eax, MAX_SIZE            ; eax = 256  

    mov eax, mul_by_2(10)        ; eax = (10 * 2) = 20  

    mov eax, sum3(1, 2, 3)       ; eax = ((1)+(2)+(3)) = 6  

    ; 重新定义（%define 可以改变）  

    %define MAX_SIZE 512  

    mov eax, MAX_SIZE            ; eax = 512  

    ; 取消定义  

    %undef MAX_SIZE  

    ; mov eax, MAX_SIZE          ; 错误：MAX_SIZE 未定义  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
> `%define` 宏展开时是纯文本替换。例如 `mul_by_2(2+3)` 展开为 `(2+3 * 2)`，由于运算符优先级，结果不是 10 而是 8。这就是宏参数一定要加括号的原因。
---

## 多行宏：%macro / %endmacro

`%macro` 用于定义包含多条指令的复杂宏：

## 实例

```asm
; 文件路径：macro_multi.asm  

; 多行宏示例  

; 宏定义：退出程序  

; 参数 argc：宏接受多少个参数  

%macro exit_program 1  

    mov eax, 1                   ; sys_exit  

    mov ebx, %1                  ; 第 1 个参数作为退出码  

    int 0x80  

%endmacro  

; 宏定义：打印字符串  

; 接受 2 个参数：字符串地址、长度  

%macro print_string 2  

    push eax  

    push ebx  

    push ecx  

    push edx  

    mov eax, 4                   ; sys_write  

    mov ebx, 1                   ; stdout  

    mov ecx, %1                  ; 字符串地址  

    mov edx, %2                  ; 字符串长度  

    int 0x80  

    pop edx  

    pop ecx  

    pop ebx  

    pop eax  

%endmacro  

section .data  

    msg db 'Hello, RUNOOB!', 0xA  

    msg_len equ $ - msg  

section .text  

    global _start  

_start:  

    print_string msg, msg_len     ; 使用宏打印消息  

    print_string msg, msg_len     ; 再次调用  

    exit_program 0                ; 退出程序  
```

---

## 宏参数的高级用法

NASM 宏支持默认参数、参数计数和条件展开：

## 实例

```asm
; 高级宏参数示例  

; 带默认参数的宏（参数范围 2-3）  

%macro debug_print 2-3 1        ; 至少 2 个参数，最多 3 个，第 3 个默认 = 1  

    %if %3 = 1                   ; 如果第 3 个参数 = 1（debug 模式开启）  

        push eax  

        push ebx  

        push ecx  

        push edx  

        mov eax, 4  

        mov ebx, 1  

        mov ecx, %1  

        mov edx, %2  

        int 0x80  

        pop edx  

        pop ecx  

        pop ebx  

        pop eax  

    %endif  

%endmacro  

; 不定数量参数的宏  

%macro push_registers 1-*        ; 1 到任意多个参数  

    %rep %0                      ; %0 是参数个数  

        push %1                  ; 展开第1个  

        %rotate 1                ; 向左旋转参数列表  

    %endrep  

%endmacro  

section .text  

    global _start  

_start:  

    ; 使用 push_registers 保存多个寄存器  

    push_registers eax, ebx, ecx, edx  

    ; 展开为：  

    ; push eax  

    ; push ebx  

    ; push ecx  

    ; push edx  

    ; 对应地弹出  

    pop edx  

    pop ecx  

    pop ebx  

    pop eax  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## 宏中的局部标签

宏中使用 `%%label` 定义局部标签，避免多次展开时标签冲突：

## 实例

```asm
; 宏中的局部标签  

; 比较两个值并设置最小值  

%macro min_val 2  

    mov eax, %1  

    mov ebx, %2  

    cmp eax, ebx  

    jle %%skip                   ; ★ 局部标签，每次展开会生成唯一名  

    mov eax, ebx  

%%skip:  

%endmacro  

; 如果不使用局部标签，展开两次后会出现重复的 skip 标签  

section .text  

    global _start  

_start:  

    min_val 10, 5               ; eax = 5  

    min_val eax, 3              ; eax = 3  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
> 普通标签在宏中展开两次会导致标签重名错误。局部标签 `%%label` 让 NASM 每次展开都生成唯一的标签名（如 `..@0001.skip`），避免冲突。
---

## 宏与过程的对比

| 特性   | 宏（Macro）         | 过程（Procedure）          |
| ---- | ---------------- | ---------------------- |
| 实现方式 | 编译时文本展开          | 运行时 CALL/RET           |
| 执行开销 | 无调用开销（直接内联）      | 有 CALL/RET 开销（约几个时钟周期） |
| 代码大小 | 每次展开增加体积         | 一份代码多次调用               |
| 参数类型 | 任意文本（寄存器、立即数、内存） | 运行时值                   |
| 调试   | 难（展开后无痕迹）        | 易（有函数调用栈）              |
| 适用场景 | 短小频繁调用、类型灵活的代码   | 复杂逻辑、代码量大的函数           |

---

## 条件编译：%if / %elif / %else / %endif

宏预处理器支持条件编译，可以根据符号定义选择生成不同的代码：

## 实例

```asm
; 文件路径：cond_compile.asm  

; 条件编译示例：调试和发布模式  

%define DEBUG 1                  ; 1=调试模式, 0=发布模式  

section .data  

    msg db 'Program running...', 0xA  

    msg_len equ $ - msg  

    debug_msg db '[DEBUG] Entering function', 0xA  

    debug_len equ $ - debug_msg  

section .text  

    global _start  

_start:  

    %if DEBUG = 1  

        ; 调试模式：输出调试信息  

        mov eax, 4  

        mov ebx, 1  

        mov ecx, debug_msg  

        mov edx, debug_len  

        int 0x80  

    %endif  

    ; 正常业务逻辑  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, msg  

    mov edx, msg_len  

    int 0x80  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

条件编译常用场景：

| 场景      | 示例                                |
| ------- | --------------------------------- |
| 调试/发布切换 | `%if DEBUG` / `%else` / `%endif`  |
| 平台适配    | `%ifdef LINUX` / `%ifdef WINDOWS` |
| 功能开关    | 通过符号存在与否控制特性                      |

---

## %rep 重复块

`%rep` 用于生成重复的代码或数据：

## 实例

```asm
; %rep 重复块示例  

section .data  

    ; 生成 256 字节的查找表  

    ; 0, 1, 4, 9, 16, 25, ...（平方数表）  

    %assign i 0  

    ; 这里不能用 %rep 生成平方表来进行复杂计算  

    ; 简单示例：生成 0-9 的 ASCII 表  

    digits: db '0', '1', '2', '3', '4', '5', '6', '7', '8', '9'  

section .text  

    global _start  

_start:  

    ; %rep 在代码中重复指令  

    mov eax, 0  

    %rep 5                      ; 重复 5 次  

        inc eax                  ; eax 每次加 1  

    %endrep  

    ; eax = 5  

    mov ebx, eax  

    mov eax, 1  

    int 0x80  
```
> 过度使用宏会让代码难以阅读和调试。当宏体超过 10 行时，考虑改为过程调用。在性能敏感的热路径中，短小宏的内联效果才有明显好处。
