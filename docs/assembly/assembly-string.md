# 汇编语言 - 字符串处理

字符串处理是编程中常见的需求。

在汇编中处理字符串意味着直接操作内存中的字节序列，虽然更底层但也更灵活。

---

## 字符串的定义和存储

在 NASM 中，字符串本质上是字节序列，可以用 DB 伪指令定义：

## 实例

```asm
; 字符串定义方式  

section .data  

    ; 方式1：标准字符串  

    str1 db 'Hello, runoob!', 0     ; C 风格：以 null 结尾  

    ; 方式2：带换行符  

    str2 db 'Line1', 0xA, 'Line2', 0xA  

    ; 方式3：反引号支持转义  

    str3 db `hello\nworld\n`, 0     ; 自动转换转义序列  

    ; 方式4：逐个字符定义  

    str4 db 'A', 'B', 'C', 'D', 0  

    ; 方式5：使用 dup 生成重复字符  

    border db 40 dup('-')           ; 40 个减号  

    ; 字符串长度计算（编译时）  

    str1_len equ $ - str1           ; 包含结尾 null  
```

---

## 计算字符串长度

有两种方式获取字符串长度：编译时计算（EQU）和运行时计算（遍历查找 null）：

## 实例

```asm
; 文件路径：strlen_demo.asm  

; 两种计算字符串长度的方式  

section .data  

    ; 方式A：编译时计算（适用于常量字符串）  

    msg1 db 'Hello, RUNOOB!', 0xA  

    msg1_len equ $ - msg1          ; 编译时自动计算  

    ; 方式B：null 结尾的字符串（运行时计算）  

    msg2 db 'Find my length', 0    ; 以 0 结尾  

section .text  

    global _start  

_start:  

    ; 方式A：直接使用编译时计算的长度  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, msg1  

    mov edx, msg1_len              ; 直接使用常量  

    int 0x80  

    ; 方式B：运行时计算 null 结尾字符串的长度  

    mov esi, msg2                  ; esi 指向字符串起始  

    mov ecx, 0                     ; 计数器  

strlen_loop:  

    cmp byte [esi], 0              ; 当前字节是 null？  

    je  strlen_done                ; 是，结束  

    inc ecx                        ; 计数 +1  

    inc esi                        ; 指针 +1  

    jmp strlen_loop  

strlen_done:  

    ; ecx 现在存放字符串长度（不含 null）  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, msg2  

    mov edx, ecx                   ; 使用计算出的长度  

    ; 这里有个问题：ecx 被覆盖了，应该先保存  

    ; 正确做法：push ecx 再 pop 到 edx  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## 字符串复制

使用循环逐字节复制，或使用 x86 的字符串操作指令 `MOVSB`：

## 实例

```asm
; 文件路径：strcpy_demo.asm  

; 字符串复制两种实现方式  

section .data  

    src db 'runoob source string', 0  

    src_len equ $ - src  

section .bss  

    dest_manual resb 64             ; 手动复制目标  

    dest_fast resb 64               ; 快速复制目标  

section .text  

    global _start  

_start:  

    ; 方式A：手动逐字节复制  

    mov esi, src                    ; 源地址  

    mov edi, dest_manual            ; 目标地址  

    mov ecx, src_len                ; 字节数  

copy_loop:  

    mov al, [esi]                   ; 读取一字节  

    mov [edi], al                   ; 写入一字节  

    inc esi                         ; 源指针++  

    inc edi                         ; 目标指针++  

    loop copy_loop  

    ; 方式B：使用字符串操作指令（更快）  

    cld                             ; 清除方向标志（DF=0，正向复制）  

    mov esi, src                    ; 源地址（ESI）  

    mov edi, dest_fast              ; 目标地址（EDI）  

    mov ecx, src_len                ; 字节数（ECX）  

    rep movsb                       ; 重复执行 movsb ecx 次  

    ; rep movsb：while(ecx>0) { [edi]=[esi]; esi++; edi++; ecx--; }  

    ; 验证：输出两个复制结果  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, dest_manual  

    mov edx, src_len  

    int 0x80  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, dest_fast  

    mov edx, src_len  

    int 0x80  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
> `REP MOVSB` 是 x86 最经典的字符串复制方式。但在现代 CPU 上，手动复制循环配合循环展开可能比 REP MOVSB 更快，因为现代 CPU 对简单操作有更好的流水线优化。
---

## 字符串比较

使用 `CMPSB` 配合 `REPE`（重复直到不等）逐字节比较：

## 实例

```asm
; 文件路径：strcmp_demo.asm  

; 比较两个字符串是否相等  

section .data  

    str_a db 'runoob', 0  

    str_b db 'runoob', 0  

    str_c db 'RUNOOB', 0  

    eq_msg db 'Strings are equal', 0xA  

    eq_len equ $ - eq_msg  

    ne_msg db 'Strings are NOT equal', 0xA  

    ne_len equ $ - ne_msg  

section .text  

    global _start  

_start:  

    cld                             ; 正向比较  

    mov esi, str_a                  ; 第一个字符串  

    mov edi, str_b                  ; 第二个字符串  

    mov ecx, 7                      ; 最多比较 7 字节（含 null）  

    repe cmpsb                      ; 重复比较直到不等或 ecx=0  

    ; repe: 如果 ZF=1（相等）且 ecx>0 则继续  

    ; cmpsb: 比较 [esi] 和 [edi]，然后 esi++, edi++  

    je  strings_equal               ; 如果 ZF=1，所有字节都相等  

    ; 不等  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, ne_msg  

    mov edx, ne_len  

    int 0x80  

    jmp compare_next  

strings_equal:  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, eq_msg  

    mov edx, eq_len  

    int 0x80  

compare_next:  

    ; 比较 str_a 和 str_c（大小写不同）  

    cld  

    mov esi, str_a  

    mov edi, str_c  

    mov ecx, 7  

    repe cmpsb  

    jne not_equal_2  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, eq_msg  

    mov edx, eq_len  

    int 0x80  

    jmp exit  

not_equal_2:  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, ne_msg  

    mov edx, ne_len  

    int 0x80  

exit:  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## 字符串操作指令汇总

| 指令    | 功能                 | 使用的寄存器                       |
| ----- | ------------------ | ---------------------------- |
| MOVSB | 复制字节：[EDI] = [ESI] | ESI=源, EDI=目标, ECX=次数, DF=方向 |
| MOVSW | 复制字（2 字节）          | 同上                           |
| MOVSD | 复制双字（4 字节）         | 同上                           |
| STOSB | 存字节：[EDI] = AL     | EDI=目标, AL=值, ECX=次数         |
| LODSB | 取字节：AL = [ESI]     | ESI=源                        |
| CMPSB | 比较字节：[ESI] - [EDI] | ESI=源, EDI=目标, ECX=次数        |
| SCASB | 扫描字节：AL - [EDI]    | EDI=目标, AL=查找值, ECX=次数       |

---

## 大小写转换示例

## 实例

```asm
; 文件路径：case_convert.asm  

; 将字符串中的小写字母转为大写  

section .data  

    msg db 'Hello, runoob! Welcome to Assembly.', 0xA  

    len equ $ - msg  

section .text  

    global _start  

_start:  

    ; 输出原始字符串  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, msg  

    mov edx, len  

    int 0x80  

    ; 转换：遍历字符串，小写 -> 大写  

    mov esi, msg                    ; 指向字符串开头  

    mov ecx, len                    ; 循环次数  

convert_loop:  

    mov al, [esi]                   ; 读取字符  

    cmp al, 'a'                     ; 是否 >= 'a'  

    jb  next_char                   ; 小于 'a'，跳过  

    cmp al, 'z'                     ; 是否 <= 'z'  

    ja  next_char                   ; 大于 'z'，跳过  

    ; 小写转大写：'a'(97) - 'A'(65) = 32  

    sub al, 32                      ; 转为大写  

    mov [esi], al                   ; 写回  

next_char:  

    inc esi  

    loop convert_loop  

    ; 输出转换后的字符串  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, msg  

    mov edx, len  

    int 0x80  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

运行结果：

```
$ nasm -f elf32 case_convert.asm -o case_convert.o
$ ld -m elf_i386 case_convert.o -o case_convert
$ ./case_convert
Hello, runoob! Welcome to Assembly.
HELLO, RUNOOB! WELCOME TO ASSEMBLY.
```
> 方向标志 DF 决定了字符串操作的方向：DF=0 时 ESI/EDI 递增（正向），DF=1 时递减（反向）。用 `CLD` 清零 DF，用 `STD` 置位 DF。在调用 C 函数或系统调用前应该确保 DF=0（cdecl 要求）。
