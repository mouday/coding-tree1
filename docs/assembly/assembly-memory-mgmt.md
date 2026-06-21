# 汇编语言 - 内存管理

内存管理是系统编程的重要部分。在汇编语言中，你可以通过系统调用动态分配和释放内存，直接操作内存地址，实现高效的内存管理。

---

## 程序的内存布局

一个运行中的程序在内存中的分布如下：

![程序内存布局图](assets/memory-layout.svg)

一个 Linux 进程在内存中的布局如下：

```
高地址 (0xFFFFFFFF)
+----------------------+
|    内核空间           |  用户程序不可访问
+----------------------+  (0xC0000000)
|    栈 (Stack)        |  <-- ESP 指向栈顶
|    向下增长           |
+----------------------+
|                      |
|    内存映射区域        |  mmap 分配的区域
|                      |
+----------------------+
|    堆 (Heap)         |  <-- brk/sbrk 管理
|    向上增长           |
+----------------------+
|    BSS 段 (.bss)     |  未初始化全局变量
+----------------------+
|    数据段 (.data)     |  已初始化全局变量
+----------------------+
|    代码段 (.text)     |  程序指令（只读）
+----------------------+
低地址 (0x08048000)
```

---

## 动态内存分配：brk 系统调用

`sys_brk`（系统调用号 45）通过调整程序的 数据段边界（program break） 来分配/释放内存。

program break 是数据段（包括 .data、.bss 和堆）的结束位置，在其之上的内存程序不能访问。

| 调用方式        | 说明            | 返回值                        |
| ----------- | ------------- | -------------------------- |
| `ebx = 0`   | 获取当前 break 地址 | EAX = 当前 break 地址          |
| `ebx = 新地址` | 设置新的 break 地址 | EAX = 新的 break 地址（失败返回原地址） |

## 实例

```asm
; 文件路径：brk_demo.asm  

; 使用 sys_brk 动态分配内存  

section .data  

    alloc_msg db 'Memory allocated successfully at: 0x'  

    alloc_len equ $ - alloc_msg  

    newline db 0xA  

section .bss  

    orig_break resd 1            ; 保存原始 break 地址  

section .text  

    global _start  

_start:  

    ; 1. 获取当前 program break  

    mov eax, 45                  ; sys_brk  

    mov ebx, 0                   ; ebx=0 表示查询当前 break  

    int 0x80  

    mov [orig_break], eax        ; 保存原始 break 地址  

    ; 2. 分配 4096 字节（4KB）内存  

    mov ebx, eax                 ; 当前 break 地址  

    add ebx, 4096                ; 增加 4096 字节  

    mov eax, 45                  ; sys_brk  

    int 0x80  

    ; EAX = 新的 break 地址（即分配后的区域末尾）  

    ; 3. 新分配的内存在 orig_break 到 eax-1 之间  

    ; 可以安全地读写这片内存  

    mov ebx, [orig_break]        ; 获取分配区域的起始地址  

    mov dword [ebx], 42          ; 在新内存中写入 42  

    mov eax, [ebx]               ; 读取回来  

    ; 4. 释放内存：将 break 恢复到原位  

    mov eax, 45                  ; sys_brk  

    mov ebx, [orig_break]        ; 恢复到原始 break  

    int 0x80  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
> sys_brk 只能调整连续的数据段边界，不能释放中间的内存。要释放某块已分配的内存，必须先释放之后分配的所有内存。这就是为什么现代程序更多使用 mmap 或 malloc 库函数。
---

## 使用 mmap 分配内存（推荐）

`sys_mmap2`（系统调用号 192）更灵活，可以独立分配和释放多块内存：

## 实例

```asm
; 文件路径：mmap_demo.asm  

; 使用 mmap 分配可独立释放的内存  

section .data  

    ; mmap 相关常量  

    PROT_READ equ 1  

    PROT_WRITE equ 2  

    MAP_PRIVATE equ 2  

    MAP_ANONYMOUS equ 0x20  

section .bss  

    mem_block resd 1             ; 保存分配的内存地址  

section .text  

    global _start  

_start:  

    ; mmap 调用：分配 4096 字节的匿名内存  

    mov eax, 192                 ; sys_mmap2  

    mov ebx, 0                   ; 让内核选择地址  

    mov ecx, 4096                ; 分配大小：4KB  

    mov edx, PROT_READ | PROT_WRITE  ; 可读可写  

    mov esi, MAP_PRIVATE | MAP_ANONYMOUS  ; 私有匿名映射  

    mov edi, -1                  ; 文件描述符（匿名映射用 -1）  

    mov ebp, 0                   ; 偏移量（匿名映射用 0）  

    int 0x80  

    ; EAX = 分配的内存地址（失败返回负值）  

    cmp eax, -4096               ; 检查是否失败  

    ja  mmap_failed              ; 如果在 -4095 ~ -1 之间，失败  

    mov [mem_block], eax         ; 保存内存地址  

    ; 使用分配的内存：写入数据  

    mov ebx, [mem_block]  

    mov dword [ebx], 0x12345678  ; 写入 4 字节  

    mov dword [ebx + 4], 'runo'  ; 写入 "runo"  

    mov dword [ebx + 8], 'ob!!'  ; 写入 "ob!!"  

    ; 释放内存：munmap  

    mov eax, 91                  ; sys_munmap  

    mov ebx, [mem_block]         ; 内存地址  

    mov ecx, 4096                ; 释放大小  

    int 0x80  

    jmp exit  

mmap_failed:  

    ; 处理错误...  

exit:  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## 内存读写操作

汇编语言中，所有内存访问通过 `mov` 指令配合方括号完成：

## 实例

```asm
; 内存读写基本操作  

section .data  

    var1 db 0x55                ; 1 字节  

    var2 dw 0x1234              ; 2 字节  

    var3 dd 0x12345678          ; 4 字节  

section .text  

    global _start  

_start:  

    ; 读取不同大小的内存  

    mov al, [var1]              ; 读取 1 字节：al = 0x55  

    mov ax, [var2]              ; 读取 2 字节：ax = 0x1234  

    mov eax, [var3]             ; 读取 4 字节：eax = 0x12345678  

    ; 写入不同大小的内存  

    mov byte [var1], 0xAA       ; 写入 1 字节  

    mov word [var2], 0xABCD     ; 写入 2 字节  

    mov dword [var3], 0xDEADBEEF ; 写入 4 字节  

    ; 通过指针操作内存  

    mov ebx, var3               ; ebx 指向 var3  

    mov eax, [ebx]              ; 间接读取 var3 的值  

    add dword [ebx], 1          ; var3 = var3 + 1  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## 内存操作的安全规范

在汇编中操作内存没有"安全网"，以下规范需要牢记：

| 规范        | 说明                | 违反后果                    |
| --------- | ----------------- | ----------------------- |
| 不越界访问     | 读写不超过变量/缓冲区的大小    | 数据损坏、段错误                |
| 不读未初始化内存  | .bss 段的值不确定       | 不可预测的结果                 |
| 不在无效地址上读写 | 确保指针指向有效内存        | 段错误（Segmentation Fault） |
| 不写只读内存    | .text 段和常量区不可写    | 段错误                     |
| 地址对齐      | 字访问用偶地址，双字用 4 的倍数 | 性能下降或总线错误               |

## 实例

```asm
; 常见内存错误示例（警告：会导致崩溃！）  

section .data  

    small_buf db 0, 0, 0, 0     ; 只有 4 字节  

section .text  

    global _start  

_start:  

    ; 危险操作1：越界写入  

    ; mov dword [small_buf + 3], 0x12345678  

    ; 这会覆盖 small_buf 之后 3 字节的数据！  

    ; 危险操作2：写入代码段（只读）  

    ; mov dword [_start], 0x90  ; 尝试修改代码，会导致段错误  

    ; 危险操作3：空指针/无效地址  

    ; mov eax, [0]              ; 读取地址 0，会导致段错误  

    ; mov [0], eax              ; 写入地址 0，会导致段错误  

    ; 安全做法：始终在分配的内存范围内操作  

    mov dword [small_buf], 'runo'  ; 正确：在 4 字节内操作  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## 栈内存管理

栈是另一种重要的内存区域，由 PUSH/POP 指令和 ESP 寄存器管理：

## 实例

```asm
; 栈操作全面示例  

section .data  

    original_esp dd 0  

section .text  

    global _start  

_start:  

    mov [original_esp], esp      ; 保存原始栈指针  

    ; PUSH：压栈（ESP 减 4，写入数据）  

    push dword 100               ; 压入 100  

    ; ESP = ESP - 4, [ESP] = 100  

    push dword 200               ; 压入 200  

    push dword 300               ; 压入 300  

    ; 栈布局：  

    ; ESP+0: 300  

    ; ESP+4: 200  

    ; ESP+8: 100  

    ; POP：弹栈（读取数据，ESP 加 4）  

    pop eax                      ; eax = 300, ESP += 4  

    pop ebx                      ; ebx = 200, ESP += 4  

    pop ecx                      ; ecx = 100, ESP += 4  

    ; 通过栈分配局部空间  

    sub esp, 256                 ; 在栈上分配 256 字节  

    ; 现在 ESP 到 ESP+255 这 256 字节可以安全使用  

    mov dword [esp], 42          ; 在局部空间写入 42  

    add esp, 256                 ; 释放局部空间  

    ; 确保 ESP 回到原始位置！  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
> 栈操作最重要的规则：PUSH 和 POP 必须配对。函数返回前必须确保 ESP 回到正确的位置，否则 RET 会弹出错误的返回地址，导致程序崩溃或执行任意代码。这是汇编编程中最致命的 bug 之一。
---

## 内存管理方案对比

| 方案               | 灵活性        | 复杂度 | 适用场景      |
| ---------------- | ---------- | --- | --------- |
| 全局变量（.data/.bss） | 低（编译时固定）   | 最低  | 固定大小的数据   |
| 栈分配（sub esp）     | 中（函数内动态）   | 低   | 函数局部变量    |
| brk/sbrk         | 中（只能增长/收缩） | 中   | 连续内存区域    |
| mmap             | 高（任意分配释放）  | 高   | 需要灵活管理内存时 |
