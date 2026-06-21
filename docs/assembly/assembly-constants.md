# 汇编语言 - 常量

常量是在编译时确定且程序运行期间不可改变的值。在汇编中使用常量可以让代码更易读、更易维护。

---

## 什么是汇编中的常量

在 NASM 中，常量是通过 伪指令 定义的符号，汇编器在编译时将所有常量引用替换为实际值。

与变量不同，常量不占用数据段内存，它们在 编译时 就被替换，相当于 C 语言中的 `#define`。

---

## EQU - 等值常量

`EQU` 是最常用的定义常量方式，定义后不可重新定义：

## 实例

```asm
; EQU 常量定义示例  

; 数值常量  

MAX_SIZE equ 256                 ; 缓冲区最大大小  

DEFAULT_PORT equ 8080            ; 默认端口号  

TRUE equ 1                       ; 真值  

FALSE equ 0                      ; 假值  

; 字符常量  

LF equ 0xA                       ; 换行符  

CR equ 0xD                       ; 回车符  

NULL equ 0                       ; 空字符  

; 表达式常量  

ARRAY_SIZE equ 100 * 4           ; 100 个双字的字节数  

BUFFER_SIZE equ MAX_SIZE * 2     ; 引用另一个常量  

TOTAL equ 10 + 20 + 30           ; 计算结果  

section .data  

    ; 使用常量  

    buffer db MAX_SIZE dup(0)    ; 定义 MAX_SIZE 字节的缓冲区  

section .text  

    global _start  

_start:  

    mov eax, MAX_SIZE            ; eax = 256  

    mov ebx, BUFFER_SIZE         ; ebx = 512  

    mov ecx, ARRAY_SIZE          ; ecx = 400  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
> `EQU` 定义的常量不能在程序中修改，也不能重新定义。如果需要可以重新定义的常量（类似宏变量），使用 `%define`。
---

## %define - 宏定义常量

`%define` 比 EQU 更灵活，支持重新定义和作用域控制：

## 实例

```asm
; %define 常量定义与重定义示例  

%define VERSION 1                ; 定义版本号  

%define APP_NAME 'runoob'        ; 定义应用名称  

; 使用 %define 定义的常量  

    mov eax, VERSION             ; eax = 1  

; 可以重新定义（EQU 不允许这样做）  

%define VERSION 2                ; 版本升级  

    mov eax, VERSION             ; 现在 eax = 2  

; %undef 取消定义  

%undef VERSION                   ; VERSION 不再有效  

; mov eax, VERSION               ; 错误：VERSION 未定义  

; %define 可以定义多行（带括号的宏）  

%define sum(a, b) (a + b)  

    mov eax, sum(10, 20)         ; mov eax, (10+20) -> eax = 30  
```

| 特性     | EQU        | %define   |
| ------ | ---------- | --------- |
| 可否重新定义 | 否          | 是         |
| 可取消定义  | 否          | 是（%undef） |
| 支持参数   | 否          | 是（宏函数）    |
| 使用场景   | 真正的常量（不变值） | 可变的配置、宏替换 |

---

## %assign - 可赋值的常量

`%assign` 用于定义可以在编译时多次修改的数值常量：

## 实例

```asm
; %assign 编译时可变常量  

%assign counter 0                ; 初始为 0  

section .data  

    ; 配合 %rep 重复块使用  

    ; 生成 table_0, table_1, ..., table_9  

    %rep 10  

        db counter               ; 使用当前 counter 值  

        %assign counter counter + 1  

    %endrep  
```

---

## 常量在程序中的实际应用

## 实例

```asm
; 文件路径：constants_practice.asm  

; 综合使用常量提高代码可读性  

; 系统调用号常量  

SYS_EXIT equ 1  

SYS_FORK equ 2  

SYS_READ equ 3  

SYS_WRITE equ 4  

SYS_OPEN equ 5  

SYS_CLOSE equ 6  

; 文件描述符常量  

STDIN equ 0  

STDOUT equ 1  

STDERR equ 2  

; 程序常量  

MAX_INPUT equ 256  

EXIT_SUCCESS equ 0  

EXIT_FAILURE equ 1  

section .data  

    msg db 'Hello, RUNOOB!', 0xA  

    msg_len equ $ - msg  

section .bss  

    buffer resb MAX_INPUT        ; 使用常量定义缓冲区大小  

section .text  

    global _start  

_start:  

    ; 使用常量名代替数字，代码意图一目了然  

    mov eax, SYS_WRITE           ; 清晰：这是写操作  

    mov ebx, STDOUT              ; 清晰：输出到屏幕  

    mov ecx, msg  

    mov edx, msg_len  

    int 0x80  

    mov eax, SYS_EXIT            ; 清晰：退出程序  

    mov ebx, EXIT_SUCCESS        ; 清晰：正常退出  

    int 0x80  
```
> 使用常量名代替"魔法数字"是一个好习惯。三个月后，`mov eax, SYS_WRITE` 比 `mov eax, 4` 更容易理解。
---

## 字符串常量

NASM 支持几种定义字符串常量的方式：

## 实例

```asm
; 字符串常量的定义方式  

section .data  

    ; 直接定义（可用 EQU 表示长度）  

    s1 db 'hello', 0  

    s1_len equ $ - s1  

    ; 带转义字符  

    s2 db 'Line1', 0xA, 'Line2', 0   ; 0xA 是换行符  

    ; 使用反引号支持 C 风格转义  

    s3 db `hello\nworld\n`, 0         ; \n 自动替换为 0xA  

    ; 多行字符串  

    s4 db 'first line', 0xA  

       db 'second line', 0xA  

       db 'third line', 0  
```
> NASM 的反引号字符串支持 `\n`（换行）、`\t`（制表符）、`\\`（反斜杠）、`\'`（单引号）等标准 C 转义序列。
