# 汇编语言 - 算术指令

算术指令是 CPU 执行数学运算的基础。本章详细介绍 x86 架构中的加法、减法、乘法、除法以及相关进位运算指令。

---

## ADD - 加法指令

`ADD` 将源操作数和目标操作数相加，结果存入目标操作数。

## 实例

```asm
; 文件路径：add_demo.asm  

; ADD 指令示例：计算 100 + 200 + 300  

section .data  

    a dd 100  

    b dd 200  

    c dd 300  

    sum dd 0  

section .text  

    global _start  

_start:  

    ; 方式1：寄存器 += 立即数  

    mov eax, [a]                ; eax = 100  

    add eax, 200                ; eax = 100 + 200 = 300  

    ; 方式2：寄存器 += 寄存器  

    mov ebx, [b]                ; ebx = 200  

    add eax, ebx                ; eax = 300 + 200 = 500  

    ; 方式3：寄存器 += 内存  

    add eax, [c]                ; eax = 500 + 300 = 800  

    ; 存储结果  

    mov [sum], eax              ; sum = 800  

    ; 加法对标志位的影响  

    mov eax, 0xFFFFFFFF         ; eax = 最大的 32 位无符号数  

    add eax, 1                  ; eax = 0（溢出回绕）  

    ; CF = 1（产生进位）  

    ; ZF = 1（结果为 0）  

    ; OF = 0（有符号视角无溢出）  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## SUB - 减法指令

`SUB` 从目标操作数中减去源操作数，结果存入目标操作数。

## 实例

```asm
; 文件路径：sub_demo.asm  

; SUB 指令示例  

section .data  

    x dd 1000  

    y dd 300  

section .text  

    global _start  

_start:  

    ; 基本减法  

    mov eax, [x]                ; eax = 1000  

    sub eax, [y]                ; eax = 1000 - 300 = 700  

    ; 减法对标志位的影响  

    mov eax, 10  

    sub eax, 20                 ; eax = -10（即 0xFFFFFFF6）  

    ; CF = 1（产生借位：10 < 20）  

    ; SF = 1（结果为负）  

    ; ZF = 0（结果非零）  

    ; OF = 0（无符号溢出）  

    ; 减自身：常用于清零  

    mov eax, 12345  

    sub eax, eax                ; eax = 0  

    ; ZF = 1, CF = 0  

    ; 这是一条将寄存器清零的经典方式（比 mov eax, 0 效率高）  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
> `sub eax, eax` 是将寄存器清零的经典技巧，它只占用 2 字节，而 `mov eax, 0` 占用 5 字节。在需要极致优化体积的场景（如 shellcode）中常用。
---

## INC / DEC - 自增/自减指令

比 ADD/SUB 更简洁的加1和减1指令：

## 实例

```asm
; INC 和 DEC 示例  

section .data  

    counter dd 0  

section .text  

    global _start  

_start:  

    mov dword [counter], 0      ; counter = 0  

    inc dword [counter]         ; counter = 1（内存操作数）  

    inc dword [counter]         ; counter = 2  

    mov ecx, 10  

    dec ecx                     ; ecx = 9  

    dec ecx                     ; ecx = 8  

    ; INC/DEC 不影响 CF 标志位（这是与 ADD/SUB 的重要区别）  

    ; 但会影响 ZF、SF、OF、PF  

    ; 循环中使用 INC  

    mov ecx, 5  

    mov eax, 0  

loop_inc:  

    inc eax                     ; eax 每次加 1  

    loop loop_inc               ; 重复 5 次，eax 最终 = 5  

    mov ebx, eax                ; 返回值 = 5  

    mov eax, 1  

    int 0x80  
```

---

## MUL - 无符号乘法

`MUL` 执行无符号乘法。乘法的规则比较特殊：

| 操作数大小 | 乘数            | 被乘数（隐式） | 结果存放                |
| ----- | ------------- | ------- | ------------------- |
| 1 字节  | 任何 8 位寄存器或内存  | AL      | AX = AL × 操作数       |
| 2 字节  | 任何 16 位寄存器或内存 | AX      | DX:AX = AX × 操作数    |
| 4 字节  | 任何 32 位寄存器或内存 | EAX     | EDX:EAX = EAX × 操作数 |

## 实例

```asm
; 文件路径：mul_demo.asm  

; MUL 无符号乘法示例  

section .data  

    val1 dd 1000  

    val2 dd 2000  

    result_low dd 0  

    result_high dd 0  

section .text  

    global _start  

_start:  

    ; 32 位乘法：EDX:EAX = EAX × 操作数  

    mov eax, [val1]             ; eax = 1000  

    mul dword [val2]            ; edx:eax = 1000 × 2000 = 2,000,000  

    ; 结果 = 2,000,000 = 0x001E8480  

    ; EAX = 0x001E8480（低 32 位）  

    ; EDX = 0x00000000（高 32 位，因为结果没超过 32 位）  

    ; 大数乘法演示（结果超过 32 位）  

    mov eax, 0xFFFFFFFF         ; eax = 4,294,967,295  

    mov ebx, 2                  ; ebx = 2  

    mul ebx                     ; edx:eax = 0xFFFFFFFF × 2  

    ; EAX = 0xFFFFFFFE  

    ; EDX = 0x00000001（高 32 位）  

    ; CF = 1（结果超出 32 位）  

    ; 保存结果  

    mov [result_low], eax  

    mov [result_high], edx  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## IMUL - 有符号乘法

`IMUL` 用于有符号数的乘法，有三种形式：

## 实例

```asm
; 文件路径：imul_demo.asm  

; IMUL 有符号乘法示例  

section .data  

    a dd -100  

    b dd 3  

section .text  

    global _start  

_start:  

    ; 单操作数形式：同 MUL  

    mov eax, [a]                ; eax = -100  

    imul dword [b]              ; edx:eax = -100 × 3 = -300  

    ; EAX = -300（0xFFFFFED4）  

    ; EDX = 0xFFFFFFFF（符号扩展）  

    ; 双操作数形式：reg = reg × 操作数  

    mov ebx, [a]                ; ebx = -100  

    imul ebx, [b]               ; ebx = -100 × 3 = -300  

    ; 结果必须是 32 位能容纳的  

    ; 三操作数形式：reg = 操作数1 × 立即数  

    imul ecx, [a], 5            ; ecx = -100 × 5 = -500  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
> `MUL` 和 `IMUL` 的主要区别是对符号的处理：`MUL` 解释为无符号数，`IMUL` 解释为有符号数。比如 0xFFFFFFFF 在 MUL 中是 4294967295，在 IMUL 中是 -1。
---

## DIV - 无符号除法

`DIV` 执行无符号除法，规则与 MUL 对称：

| 除数大小 | 被除数（隐式） | 商   | 余数  |
| ---- | ------- | --- | --- |
| 1 字节 | AX      | AL  | AH  |
| 2 字节 | DX:AX   | AX  | DX  |
| 4 字节 | EDX:EAX | EAX | EDX |

## 实例

```asm
; 文件路径：div_demo.asm  

; DIV 无符号除法示例  

section .data  

    dividend dd 1000  

    divisor dd 7  

    quotient dd 0  

    remainder dd 0  

section .text  

    global _start  

_start:  

    ; 32 位除法：edx:eax / 除数  

    ; 被除数要先扩展到 EDX:EAX  

    mov eax, [dividend]         ; eax = 1000  

    mov edx, 0                  ; edx = 0（高 32 位清零）  

    div dword [divisor]         ; edx:eax / 7  

    ; EAX = 142（商）  

    ; EDX = 6（余数：1000 = 142×7 + 6）  

    mov [quotient], eax         ; quotient = 142  

    mov [remainder], edx        ; remainder = 6  

    ; 字节除法示例：55 / 4  

    mov ax, 55                  ; 被除数  

    mov bl, 4                   ; 除数  

    div bl                      ; al = 13（商）, ah = 3（余数）  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
> 做除法前务必用 `mov edx, 0` 或 `xor edx, edx` 清零 EDX！如果 EDX 中有旧数据，被除数的值就不对了。这是初学者最常见的除法 bug。
---

## IDIV - 有符号除法

`IDIV` 用于有符号除法。做除法前需要用 `CDQ` 指令将 EAX 符号扩展到 EDX:EAX：

## 实例

```asm
; IDIV 有符号除法示例  

    ; 计算 -100 / 3  

    mov eax, -100               ; eax = -100  

    cdq                         ; 符号扩展：edx:eax = -100  

                                ; cdq 将 eax 的符号位复制到 edx 的所有位  

    mov ebx, 3                  ; 除数  

    idiv ebx                    ; eax = -33（商）, edx = -1（余数）  

    ; 验证：-100 = -33 × 3 + (-1)  
```

---

## ADC / SBB - 带进位/借位运算

用于 大数运算（超过 32 位的数据）：

## 实例

```asm
; 文件路径：adc_sbb_demo.asm  

; 64 位加法：两个 64 位数相加  

section .data  

    ; 64 位数 a = 0x00000001 FFFFFFFF  

    a_low dd 0xFFFFFFFF  

    a_high dd 0x00000001  

    ; 64 位数 b = 0x00000000 00000005  

    b_low dd 0x00000005  

    b_high dd 0x00000000  

    ; 结果  

    result_low dd 0  

    result_high dd 0  

section .text  

    global _start  

_start:  

    ; 低 32 位相加  

    mov eax, [a_low]  

    add eax, [b_low]            ; eax = 0xFFFFFFFF + 5 = 0x00000004  

    ; CF = 1（产生进位！）  

    mov [result_low], eax       ; result_low = 0x00000004  

    ; 高 32 位加进位  

    mov eax, [a_high]  

    adc eax, [b_high]           ; adc = add + CF  

    ; eax = 1 + 0 + 1(进位) = 2  

    mov [result_high], eax      ; result_high = 2  

    ; 最终 64 位结果：0x00000002 00000004  

    ; 验证：0x1FFFFFFF + 5 = 0x200000004  

    ; SBB 减法同理（带借位）  

    mov eax, [a_low]  

    sub eax, [b_low]            ; 低 32 位相减  

    mov [result_low], eax  

    mov eax, [a_high]  

    sbb eax, [b_high]           ; sbb = sub - CF  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## 算术指令速查表

| 指令   | 格式              | 功能                     | 影响的标志位                 |
| ---- | --------------- | ---------------------- | ---------------------- |
| ADD  | `add dest, src` | dest = dest + src      | CF, ZF, SF, OF, PF     |
| SUB  | `sub dest, src` | dest = dest - src      | CF, ZF, SF, OF, PF     |
| INC  | `inc dest`      | dest = dest + 1        | ZF, SF, OF, PF（不影响 CF） |
| DEC  | `dec dest`      | dest = dest - 1        | ZF, SF, OF, PF（不影响 CF） |
| MUL  | `mul src`       | 无符号乘法                  | CF, OF                 |
| IMUL | `imul src`      | 有符号乘法                  | CF, OF                 |
| DIV  | `div src`       | 无符号除法                  | 无定义（不定）                |
| IDIV | `idiv src`      | 有符号除法                  | 无定义（不定）                |
| ADC  | `adc dest, src` | dest = dest + src + CF | CF, ZF, SF, OF, PF     |
| SBB  | `sbb dest, src` | dest = dest - src - CF | CF, ZF, SF, OF, PF     |
| NEG  | `neg dest`      | dest = -dest（取负）       | CF, ZF, SF, OF, PF     |
