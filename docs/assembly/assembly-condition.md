# 汇编语言 - 条件判断

条件判断是程序控制流的基石。汇编语言通过比较指令和条件跳转指令来实现 if-else、switch 等判断逻辑。

---

## CMP - 比较指令

`CMP` 对两个操作数做减法运算（目标 - 源），但不保存结果，只更新标志位。

本质上 `CMP dest, src` 等同于 `SUB dest, src` 但丢弃计算结果。

## 实例

```asm
; CMP 比较指令示例  

section .text  

    global _start  

_start:  

    mov eax, 10  

    cmp eax, 10                 ; eax == 10?  

    ; ZF = 1（相等，结果为零）  

    ; CF = 0（无借位）  

    ; SF = 0（结果非负）  

    mov eax, 5  

    cmp eax, 10                 ; eax == 10?  

    ; ZF = 0（不等，结果非零）  

    ; CF = 1（有借位：5 < 10）  

    mov eax, 20  

    cmp eax, 10                 ; eax == 10?  

    ; ZF = 0（不等）  

    ; CF = 0（无借位：20 >= 10）  

    ; SF = 0（结果非负：10 > 0）  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## 条件跳转指令

条件跳转指令根据标志位的状态决定是否跳转。如果条件成立，跳转到目标标签；否则继续执行下一条指令。

| 指令        | 含义           | 检查的标志位        | 适用场景                  |
| --------- | ------------ | ------------- | --------------------- |
| JE / JZ   | 相等 / 为零时跳转   | ZF = 1        | `cmp a, b` 后检查 a == b |
| JNE / JNZ | 不相等 / 不为零时跳转 | ZF = 0        | `cmp a, b` 后检查 a != b |
| JG / JNLE | 大于时跳转（有符号）   | ZF=0 且 SF=OF  | 有符号数 a > b            |
| JGE / JNL | 大于等于时跳转（有符号） | SF = OF       | 有符号数 a >= b           |
| JL / JNGE | 小于时跳转（有符号）   | SF != OF      | 有符号数 a < b            |
| JLE / JNG | 小于等于时跳转（有符号） | ZF=1 或 SF!=OF | 有符号数 a <= b           |
| JA / JNBE | 大于时跳转（无符号）   | CF=0 且 ZF=0   | 无符号数 a > b            |
| JAE / JNB | 大于等于时跳转（无符号） | CF = 0        | 无符号数 a >= b           |
| JB / JNAE | 小于时跳转（无符号）   | CF = 1        | 无符号数 a < b            |
| JBE / JNA | 小于等于时跳转（无符号） | CF=1 或 ZF=1   | 无符号数 a <= b           |

> 很多指令有两个别名（如 JE 和 JZ），它们在机器码层面完全一样。使用哪个取决于语境：比较后用 JE/JNE，运算结果检查后用 JZ/JNZ。这样代码可读性更好。
---

## 单分支 IF 结构

## 实例

```asm
; 文件路径：if_demo.asm  

; 实现：if (x > 10) x = 10;  

section .data  

    x dd 15                     ; 测试值  

    limit dd 10  

section .text  

    global _start  

_start:  

    mov eax, [x]                ; 加载 x 到 eax  

    cmp eax, [limit]            ; x > 10 ?  

    jle skip_update             ; 如果 x <= 10，跳过更新  

    mov dword [x], 10           ; x = 10  

skip_update:  

    ; 程序继续...（这里 x 已经是 10 了）  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## IF-ELSE 双分支结构

## 实例

```asm
; 文件路径：if_else_demo.asm  

; 实现：if (score >= 60) grade = 'P' else grade = 'F'  

section .data  

    score dd 75                 ; 考试分数  

    grade db 0                  ; 成绩等级  

    PASS_SCORE equ 60  

section .text  

    global _start  

_start:  

    mov eax, [score]            ; 加载分数  

    cmp eax, PASS_SCORE         ; score >= 60 ?  

    jge pass_label              ; 如果 >= 60，跳到 pass  

    ; else 分支：不及格  

    mov byte [grade], 'F'       ; grade = 'F'  

    jmp end_if                  ; 跳过 if 分支  

pass_label:  

    ; if 分支：及格  

    mov byte [grade], 'P'       ; grade = 'P'  

end_if:  

    ; grade 变量现在已设置好  

    ; 输出成绩等级  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, grade  

    mov edx, 1  

    int 0x80  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

运行结果：

```
$ nasm -f elf32 if_else_demo.asm -o if_else_demo.o
$ ld -m elf_i386 if_else_demo.o -o if_else_demo
$ ./if_else_demo
P
```

---

## IF-ELSE IF-ELSE 多分支结构

下面以分数评级系统为例，展示多分支判断的流程：

![条件判断多分支流程图](assets/conditional-flow.svg)

## 实例

```asm
; 文件路径：multi_branch.asm  

; 分数评级：>=90 -> A, >=80 -> B, >=70 -> C, >=60 -> D, <60 -> F  

section .data  

    score dd 85  

    result db 0  

    newline db 0xA  

section .text  

    global _start  

_start:  

    mov eax, [score]            ; 加载分数  

    ; 检查 >= 90  

    cmp eax, 90  

    jl check_80                 ; 如果 < 90，继续检查  

    mov byte [result], 'A'  

    jmp print_result  

check_80:  

    cmp eax, 80  

    jl check_70  

    mov byte [result], 'B'  

    jmp print_result  

check_70:  

    cmp eax, 70  

    jl check_60  

    mov byte [result], 'C'  

    jmp print_result  

check_60:  

    cmp eax, 60  

    jl fail_label  

    mov byte [result], 'D'  

    jmp print_result  

fail_label:  

    mov byte [result], 'F'  

print_result:  

    ; 输出评级  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, result  

    mov edx, 1  

    int 0x80  

    ; 输出换行  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, newline  

    mov edx, 1  

    int 0x80  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## TEST - 非破坏性测试指令

`TEST` 执行按位 AND 但不保存结果，只更新标志位。

`TEST` 常用于检查特定位是否为 1，或者检查寄存器是否为 0。

## 实例

```asm
; TEST 指令示例  

    ; 检查 eax 是否为 0（比 CMP eax, 0 更高效）  

    test eax, eax               ; eax & eax = eax，只更新 ZF  

    jz  eax_is_zero             ; 若 ZF=1，说明 eax=0  

    ; 检查特定位  

    test al, 0x01               ; 检查 bit0 是否为 1  

    jnz bit0_is_set             ; 若 bit0=1，跳转  

    ; 检查多个位  

    test al, 0x03               ; 检查 bit0 和 bit1 是否有至少一个为 1  

    jnz some_bit_set  
```

---

## SETcc - 条件设置指令

不跳转，而是根据条件将目标字节设为 1 或 0：

## 实例

```asm
; SETcc 条件设置示例  

    ; 将 a > b 的结果存入 al  

    mov eax, 10  

    cmp eax, 5                  ; 10 > 5 ?  

    setg al                     ; al = 1（大于成立）  

    ; 如果 eax = 3，则 al = 0  

    ; 其他 SETcc 指令：  

    ; sete / setz   -> 相等/为零  

    ; setne / setnz -> 不等/不为零  

    ; setl          -> 小于（有符号）  

    ; setb          -> 小于（无符号）  

    ; setg          -> 大于（有符号）  

    ; seta          -> 大于（无符号）  
```

---

## 完整示例：判断奇偶和正负

## 实例

```asm
; 文件路径：number_check.asm  

; 判断数字的奇偶、正负  

section .data  

    number dd -42  

    msg_even db 'Even', 0xA  

    msg_even_len equ $ - msg_even  

    msg_odd db 'Odd', 0xA  

    msg_odd_len equ $ - msg_odd  

    msg_pos db 'Positive', 0xA  

    msg_pos_len equ $ - msg_pos  

    msg_neg db 'Negative', 0xA  

    msg_neg_len equ $ - msg_neg  

    msg_zero db 'Zero', 0xA  

    msg_zero_len equ $ - msg_zero  

section .text  

    global _start  

_start:  

    mov eax, [number]  

    ; 判断是否为 0  

    cmp eax, 0  

    jne check_sign  

    ; 为零  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, msg_zero  

    mov edx, msg_zero_len  

    int 0x80  

    jmp exit  

check_sign:  

    ; 判断正负  

    cmp eax, 0  

    jg  is_positive             ; eax > 0  

    ; 负数  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, msg_neg  

    mov edx, msg_neg_len  

    int 0x80  

    jmp check_parity  

is_positive:  

    ; 正数  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, msg_pos  

    mov edx, msg_pos_len  

    int 0x80  

check_parity:  

    ; 判断奇偶（只需看最低位）  

    mov eax, [number]  

    test eax, 1                 ; 检查 bit0  

    jnz is_odd  

    ; 偶数  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, msg_even  

    mov edx, msg_even_len  

    int 0x80  

    jmp exit  

is_odd:  

    ; 奇数  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, msg_odd  

    mov edx, msg_odd_len  

    int 0x80  

exit:  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
> 条件跳转指令只能跳转到当前代码段内的标签，跳转范围通常受限于约 128 字节（短跳转）到 2GB（近跳转）。NASM 会自动选择合适的跳转编码。
