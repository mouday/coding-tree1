# 汇编语言 - 变量

变量是程序中存储数据的基本单元。在汇编语言中定义和使用变量比高级语言更接近底层，你需要直接控制变量的大小、类型和内存布局。

---

## 汇编中的变量概念

在汇编语言中，变量 本质上是内存中一个有名字的存储位置。

与高级语言不同，汇编变量没有自动的类型检查——一个标签只是内存地址的别名，你可以用任何大小去读写它。

NASM 提供了三种段来存放不同类型的变量：`.data`（已初始化）、`.bss`（未初始化）和 `.rodata`（只读）。

---

## 在 .data 段定义已初始化变量

使用 DB、DW、DD、DQ、DT 伪指令定义不同大小的变量：

| 伪指令 | 数据大小  | 含义                | 示例                              |
| --- | ----- | ----------------- | ------------------------------- |
| DB  | 1 字节  | Define Byte       | `flag db 1`                     |
| DW  | 2 字节  | Define Word       | `count dw 1000`                 |
| DD  | 4 字节  | Define Doubleword | `price dd 9999`                 |
| DQ  | 8 字节  | Define Quadword   | `big_val dq 0x1234567890ABCDEF` |
| DT  | 10 字节 | Define Ten Bytes  | `ext_val dt 3.14`               |

## 实例

```asm
; 文件路径：variables_data.asm  

; 演示 .data 段中各种变量的定义  

section .data  

    ; 字节变量  

    status db 1                 ; 1 字节，值为 1  

    grade db 'A'                ; 1 字节，字符 'A' = 0x41  

    ; 字变量（2 字节）  

    year dw 2026                ; 2 字节，值为 2026  

    ; 双字变量（4 字节）  

    salary dd 50000             ; 4 字节，值为 50000  

    ; 多字节序列  

    msg db 'Hello, runoob!', 0  ; 字符串以 0 结尾（C 风格）  

    hex_bytes db 0x55, 0xAA, 0x00, 0xFF   ; 十六进制字节序列  

    ; 重复值  

    stars db 10 dup('*')        ; 10 个星号字符  

    ; 带名称的多个变量  

    x dd 10  

    y dd 20  

    z dd 30  

section .text  

    global _start  

_start:  

    ; 读取变量值到寄存器  

    mov al, [status]            ; al = 1（读取 1 字节）  

    mov ax, [year]              ; ax = 2026（读取 2 字节）  

    mov eax, [salary]           ; eax = 50000（读取 4 字节）  

    ; 修改变量值  

    mov byte [status], 0        ; status = 0  

    mov dword [x], 100          ; x = 100  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
> 使用 `[变量名]` 访问变量时，务必确保读取/写入的字节数与变量定义时的字节数匹配。用 `mov al, [status]` 读 1 字节，用 `mov eax, [salary]` 读 4 字节。NASM 会检查操作数大小是否一致。
---

## 在 .bss 段预留未初始化空间

使用 RESB、RESW、RESD 等伪指令预留空间（不初始化）：

| 伪指令  | 预留大小    | 示例                |
| ---- | ------- | ----------------- |
| RESB | 字节      | `buffer resb 256` |
| RESW | 字（2字节）  | `wbuf resw 100`   |
| RESD | 双字（4字节） | `dbuf resd 50`    |
| RESQ | 四字（8字节） | `qbuf resq 25`    |

## 实例

```asm
; 文件路径：variables_bss.asm  

; 演示 .bss 段中预留空间  

section .bss  

    input_buf resb 128          ; 预留 128 字节的输入缓冲区  

    numbers resd 100            ; 预留 100 个双字（400 字节）  

    temp resb 1                 ; 预留 1 字节的临时变量  

section .data  

    prompt db 'Enter a number: '  

    prompt_len equ $ - prompt  

section .text  

    global _start  

_start:  

    ; 输出提示  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, prompt  

    mov edx, prompt_len  

    int 0x80  

    ; 读取输入到 .bss 段的缓冲区  

    mov eax, 3  

    mov ebx, 0  

    mov ecx, input_buf           ; 使用 .bss 段预留的空间  

    mov edx, 128  

    int 0x80  

    ; 向 numbers 数组填入数据  

    mov dword [numbers], 42      ; numbers[0] = 42  

    mov dword [numbers + 4], 100 ; numbers[1] = 100  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## 字节、字、双字的内存布局

x86 使用 小端序（Little Endian）——低字节存放在低地址。

例如，当定义 `value dd 0x12345678` 时，内存中的布局为：

```
地址    ：[value]  [value+1]  [value+2]  [value+3]
内容    ：0x78     0x56       0x34       0x12
```

## 实例

```asm
; 文件路径：endianness.asm  

; 演示小端序存储  

section .data  

    value dd 0x12345678        ; 双字，4 字节  

section .text  

    global _start  

_start:  

    ; 按不同大小读取同一个变量  

    mov eax, [value]            ; 读 4 字节：eax = 0x12345678  

    mov ax, [value]             ; 读 2 字节：ax = 0x5678（低2字节）  

    mov al, [value]             ; 读 1 字节：al = 0x78（最低1字节）  

    ; 验证小端序：低地址存放低字节  

    mov al, [value]             ; al = 0x78  

    mov bl, [value + 1]         ; bl = 0x56  

    mov cl, [value + 2]         ; cl = 0x34  

    mov dl, [value + 3]         ; dl = 0x12  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## 变量的初始化与访问

汇编中变量的操作遵循"加载-运算-存储"三步模式：

## 实例

```asm
; 文件路径：var_operations.asm  

; 演示变量操作的三步模式  

section .data  

    a dd 100  

    b dd 200  

    result dd 0  

section .text  

    global _start  

_start:  

    ; result = a + b  

    ; 第1步：加载  

    mov eax, [a]                ; 加载 a 到 eax  

    ; 第2步：运算  

    add eax, [b]                ; eax = eax + b  

    ; 第3步：存储  

    mov [result], eax           ; 存储结果到 result  

    ; result = a * 2 - b  

    mov eax, [a]  

    imul eax, 2                 ; eax = a * 2  

    sub eax, [b]                ; eax = eax - b  

    mov [result], eax  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
> 汇编不支持 `mov [result], [a] + [b]` 这种高级语法。x86 的 mov 指令不能同时用两个内存操作数，必须先经过寄存器中转。
---

## 变量大小操作符

当汇编器无法推断操作数大小时，需要显式指定：

## 实例

```asm
; 使用大小操作符明确数据大小  

section .data  

    var dd 0  

section .text  

    global _start  

_start:  

    ; 大小操作符：byte, word, dword, qword  

    mov byte [var], 1           ; 写入 1 字节  

    mov word [var], 1000        ; 写入 2 字节  

    mov dword [var], 999999     ; 写入 4 字节  

    ; 当目标大小明确时（由寄存器决定），可以省略  

    mov al, [var]               ; al 是 1 字节，自动按 byte 读取  

    mov ax, [var]               ; ax 是 2 字节，自动按 word 读取  

    mov eax, [var]              ; eax 是 4 字节，自动按 dword 读取  

    ; 但下面这行会报错（大小不明确）：  

    ; mov [var], 1              ; 错误！1 可以是字节/字/双字  

    ; 必须写成：  

    mov dword [var], 1          ; 明确指定 4 字节  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
