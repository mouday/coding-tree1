# 汇编语言 - 逻辑指令

逻辑指令在汇编编程中用于位级操作，广泛用于掩码运算、位提取、标志位设置和高效乘除运算。

---

## AND - 按位与

`AND` 对两个操作数逐位执行"与"运算（都为 1 时才为 1）。

## 实例

```asm
; 文件路径：and_demo.asm  

; AND 指令示例  

section .text  

    global _start  

_start:  

    ; 基本按位与  

    mov eax, 0x0F0F             ; 0000 1111 0000 1111  

    and eax, 0x00FF             ; 0000 0000 1111 1111  

    ; 结果：eax = 0x000F         ; 0000 0000 0000 1111  

    ; 常用技巧1：屏蔽低 4 位（保留低 4 位）  

    mov eax, 0xAB               ; 1010 1011  

    and eax, 0x0F               ; 0000 1111  

    ; eax = 0x0B                ; 0000 1011  

    ; 常用技巧2：判断奇偶性（和 1 做 AND）  

    mov eax, 42                 ; 偶数  

    and eax, 1                  ; eax = 0（偶数）  

    ; 42 的二进制末尾是 0，42 & 1 = 0  

    mov eax, 43                 ; 奇数  

    and eax, 1                  ; eax = 1（奇数）  

    ; 43 的二进制末尾是 1，43 & 1 = 1  

    ; 常用技巧3：将寄存器清零  

    xor eax, eax                ; 等同于 mov eax, 0，但更快更短  

    ; AND 会设置标志位：CF=0, OF=0, ZF 和 SF 根据结果  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## OR - 按位或

`OR` 对两个操作数逐位执行"或"运算（只要有一个为 1 就是 1）。

## 实例

```asm
; OR 指令示例  

    ; 合并标志位  

    mov eax, 0x0F00             ; 0000 1111 0000 0000  

    or  eax, 0x00FF             ; 0000 0000 1111 1111  

    ; eax = 0x0FFF              ; 0000 1111 1111 1111  

    ; 设置某个位（把第 3 位设为 1）  

    mov eax, 0                  ; 0000 0000  

    or  eax, 0x08               ; 0000 1000  

    ; eax = 0x8                 ; 0000 1000  

    ; 大小写转换：大写转小写  

    mov al, 'A'                 ; al = 0x41 (0100 0001)  

    or  al, 0x20                ; 0x20 = 0010 0000  

    ; al = 0x61 = 'a'          ; 0110 0001  
```

---

## NOT - 按位取反

`NOT` 将操作数的每个位取反（0 变 1，1 变 0）。

## 实例

```asm
; NOT 指令示例  

    mov eax, 0x0F0F0F0F        ; 0000 1111 0000 1111 ...  

    not eax                     ; 1111 0000 1111 0000 ...  

    ; eax = 0xF0F0F0F0  

    ; NOT 不影响任何标志位（与 AND/OR/XOR 不同）  
```

---

## XOR - 按位异或

`XOR` 对两个操作数逐位执行"异或"运算（不同为 1，相同为 0）。

## 实例

```asm
; 文件路径：xor_demo.asm  

; XOR 指令示例  

section .text  

    global _start  

_start:  

    ; 基本异或  

    mov eax, 0x0F0F             ; 0000 1111 0000 1111  

    xor eax, 0x00FF             ; 0000 0000 1111 1111  

    ; 结果：eax = 0x0FF0         ; 0000 1111 1111 0000  

    ; 经典技巧1：寄存器清零（比 mov reg, 0 高效）  

    xor eax, eax                ; eax = 0，只占用 2 字节  

    xor ebx, ebx  

    xor ecx, ecx  

    ; 经典技巧2：交换两个寄存器（不需要第三个临时寄存器）  

    mov eax, 100                ; eax = 100  

    mov ebx, 200                ; ebx = 200  

    xor eax, ebx                ; eax = 100 xor 200  

    xor ebx, eax                ; ebx = 200 xor (100 xor 200) = 100  

    xor eax, ebx                ; eax = (100 xor 200) xor 100 = 200  

    ; 现在 eax = 200, ebx = 100（交换完成！）  

    ; 经典技巧3：简单的加密/解密  

    mov al, 'A'                 ; 原始字符 'A' = 0x41  

    xor al, 0x55                ; 加密：al = 'A' xor 0x55  

    ; al 现在是某个乱码值  

    xor al, 0x55                ; 解密：再 xor 0x55 恢复  

    ; al = 'A' 回来了！  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
> XOR 交换技巧虽然很酷，但在现代 CPU 上效率不如使用 `XCHG` 指令或临时寄存器。了解即可，不必执着使用。
---

## 移位指令

移位指令将二进制位向左或向右移动，常用于高效乘除运算（乘2、除2等）。

| 指令  | 功能                 | 示例                    |
| --- | ------------------ | --------------------- |
| SHL | 逻辑左移（左边出去，右边补0）    | `shl eax, 1`（乘以 2）    |
| SHR | 逻辑右移（右边出去，左边补0）    | `shr eax, 1`（无符号除以 2） |
| SAL | 算术左移（同 SHL）        | `sal eax, 1`          |
| SAR | 算术右移（右边出去，左边保持符号位） | `sar eax, 1`（有符号除以 2） |

## 实例

```asm
; 文件路径：shift_demo.asm  

; 移位指令示例  

section .text  

    global _start  

_start:  

    ; SHL：逻辑左移 = 乘以 2 的幂  

    mov eax, 10                 ; eax = 10 (1010)  

    shl eax, 1                  ; eax = 20 (10100)，即 10×2  

    shl eax, 2                  ; eax = 80 (1010000)，即 20×4  

    ; SHR：逻辑右移 = 无符号除以 2 的幂  

    mov eax, 80                 ; eax = 80  

    shr eax, 3                  ; eax = 10，即 80÷8  

    ; SAR：算术右移 = 有符号除以 2 的幂（保留符号位）  

    mov eax, -16                ; eax = -16 (0xFFFFFFF0)  

    sar eax, 2                  ; eax = -4 (0xFFFFFFFC)，即 -16÷4  

    ; SAR 对比 SHR：  

    ; 如果 eax = 0xFFFFFFF0 (-16)，SHR 会得到 0x3FFFFFFC (很大的正数)  

    ; 而 SAR 会得到 0xFFFFFFFC (-4)，保留了符号位  

    ; 移出的最后一位会进入 CF 标志位  

    mov eax, 5                  ; 0101  

    shr eax, 1                  ; eax = 2, CF = 1（末尾的1被移出）  

    jc carry_was_set            ; 如果 CF=1，说明原数是奇数  

carry_was_set:  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## 循环移位指令

循环移位将移出的位重新填入另一端，形成循环：

| 指令  | 功能              |
| --- | --------------- |
| ROL | 循环左移（绕过 CF）     |
| ROR | 循环右移（绕过 CF）     |
| RCL | 带进位的循环左移（通过 CF） |
| RCR | 带进位的循环右移（通过 CF） |

## 实例

```asm
; 循环移位示例  

    ; ROL：循环左移  

    mov al, 0x85                ; 1000 0101  

    rol al, 1                   ; 左移 1 位：0000 1011  

    ; CF = 1（最高位移出到 CF）  

    ; 同时 CF 原本的 1 被移入最低位，形成循环  

    ; ROL 实际效果：向左移 1 位，最高位同时进入 CF 和最低位  

    ; 1000 0101 -> ROL 1 -> 0000 1011  

    ; RCL：带进位循环左移（CF 参与循环）  

    clc                         ; 清除 CF = 0  

    mov al, 0x85                ; 1000 0101  

    rcl al, 1                   ; CF 参与：CF bit7...bit0 -> CF  

    ; 结果：0000 1010, CF = 1  

    ; 循环移位在加密算法和位操作中常用  
```

---

## 逻辑运算的实际用途

一个综合示例——使用逻辑运算实现简单的位标志系统：

## 实例

```asm
; 文件路径：bit_flags.asm  

; 使用逻辑运算实现位标志系统  

section .data  

    flags db 0                  ; 8 个标志位，初始全为 0  

    ; bit0: 是否激活  

    ; bit1: 是否可见  

    ; bit2: 是否需要保存  

    ; bit3: 是否已修改  

    FLAG_ACTIVE equ 1           ; 0000 0001  

    FLAG_VISIBLE equ 2          ; 0000 0010  

    FLAG_NEED_SAVE equ 4        ; 0000 0100  

    FLAG_MODIFIED equ 8         ; 0000 1000  

section .text  

    global _start  

_start:  

    ; 设置标志位：OR  

    mov al, [flags]  

    or  al, FLAG_ACTIVE         ; 设置 bit0  

    or  al, FLAG_VISIBLE        ; 设置 bit1  

    ; al = 0000 0011 = 3  

    mov [flags], al  

    ; 检查标志位：AND + TEST  

    test byte [flags], FLAG_ACTIVE    ; 测试 bit0 是否为 1  

    jnz is_active                     ; ZF=0 表示该位是 1  

is_active:  

    ; 清除标志位：AND + NOT  

    mov al, [flags]  

    and al, ~FLAG_VISIBLE       ; 清除 bit1（保留其他位不变）  

    ; al = 0000 0001 = 1  

    mov [flags], al  

    ; 切换标志位：XOR  

    mov al, [flags]  

    xor al, FLAG_MODIFIED       ; 切换 bit3  

    ; 如果原来是 0 变成 1，原来是 1 变成 0  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
> 位标志操作在系统编程中极为常见。操作系统内核、设备驱动和嵌入式系统中广泛使用这种技巧来管理状态。
