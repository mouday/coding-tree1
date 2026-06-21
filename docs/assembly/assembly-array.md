# 汇编语言 - 数组

数组是连续存放的同类型数据集合。在汇编中，数组就是内存中一段连续的空间，通过基址加偏移的方式来访问每个元素。

---

## 一维数组定义

## 实例

```asm
; 数组的定义方式  

section .data  

    ; 方式1：逐个列出元素  

    arr1 dd 10, 20, 30, 40, 50  ; 5 个双字元素的数组  

    ; 方式2：用 dup 初始化  

    arr2 dd 10 dup(0)            ; 10 个双字，全为 0  

    ; 方式3：字节数组  

    bytes db 1, 2, 3, 4, 5, 6   ; 6 个字节  

    ; 方式4：字符数组（字符串）  

    chars db 'runoob', 0         ; 7 字节  

    ; 数组长度计算（编译时）  

    arr1_len equ ($ - arr1) / 4  ; 双字数组的元素个数  

    arr2_len equ ($ - arr2) / 4  

    bytes_len equ ($ - bytes)    ; 字节数组，不需除法  

section .bss  

    ; 未初始化的数组  

    buffer resd 100              ; 预留 100 个双字的空间  
```

---

## 数组元素访问

访问数组元素使用 基址 + 索引 × 元素大小 的寻址方式：

## 实例

```asm
; 文件路径：array_access.asm  

; 数组元素的读取和写入  

section .data  

    nums dd 100, 200, 300, 400, 500   ; 5 个元素  

    nums_len equ ($ - nums) / 4  

section .text  

    global _start  

_start:  

    ; 访问第 0 个元素（下标 0）  

    mov eax, [nums]              ; eax = 100  

    ; 访问第 2 个元素（下标 2）  

    mov eax, [nums + 2 * 4]      ; eax = nums[2] = 300  

    ; 2 * 4 = 8，从 nums 偏移 8 字节  

    ; 使用寄存器作为下标  

    mov esi, 3                   ; 下标 = 3  

    mov eax, [nums + esi * 4]    ; eax = nums[3] = 400  

    ; 修改数组元素  

    mov dword [nums + 4], 250    ; nums[1] = 250  

    ; 数组中现在：100, 250, 300, 400, 500  

    ; 使用 EBX 作为基址寄存器  

    mov ebx, nums                ; ebx = 数组基址  

    mov eax, [ebx + 4 * 4]       ; eax = nums[4] = 500  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
> 注意：`[nums + esi * 4]` 中的乘以 4 是因为每个元素是 4 字节（双字）。如果是字节数组（db），直接写成 `[arr + esi]`；字数组（dw），写成 `[arr + esi * 2]`。
---

## 数组遍历

## 实例

```asm
; 文件路径：array_traverse.asm  

; 遍历数组并计算总和  

section .data  

    numbers dd 5, 10, 15, 20, 25, 30, 35, 40, 45, 50  

    count equ ($ - numbers) / 4   ; 元素个数  

section .text  

    global _start  

_start:  

    mov ecx, count                ; 循环次数  

    mov esi, 0                    ; 下标（从 0 开始）  

    mov eax, 0                    ; 累加和  

sum_loop:  

    add eax, [numbers + esi * 4]  ; 累加数组元素  

    inc esi                       ; 下标 +1  

    loop sum_loop  

    ; eax = 5+10+15+...+50 = 275  

    ; 另一种遍历方式：使用指针  

    mov ecx, count  

    mov ebx, numbers              ; ebx 指向数组起始  

    mov eax, 0                    ; 累加和  

sum_loop2:  

    add eax, [ebx]                ; 累加当前元素  

    add ebx, 4                    ; 指针移动到下一个元素  

    loop sum_loop2  

    ; eax = 275  

    mov ebx, eax                  ; 退出码 = 累加和  

    mov eax, 1  

    int 0x80  
```

---

## 数组查找

## 实例

```asm
; 文件路径：array_search.asm  

; 在数组中查找指定值  

section .data  

    data dd 12, 45, 67, 23, 89, 34, 56, 78, 90, 11  

    data_len equ ($ - data) / 4  

    target dd 23                   ; 要查找的值  

    found_msg db 'Found at index: ', 0  

    found_len equ $ - found_msg  

    notfound_msg db 'Not found (runoob)', 0xA  

    notfound_len equ $ - notfound_msg  

    newline db 0xA  

section .text  

    global _start  

_start:  

    mov ecx, data_len             ; 循环次数  

    mov esi, 0                    ; 当前下标  

search_loop:  

    mov eax, [data + esi * 4]     ; 加载当前元素  

    cmp eax, [target]             ; 是否等于目标值  

    je  found                     ; 找到了  

    inc esi                       ; 下标 +1  

    loop search_loop  

    ; 没找到  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, notfound_msg  

    mov edx, notfound_len  

    int 0x80  

    jmp exit  

found:  

    ; 找到了（esi 是下标）  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, found_msg  

    mov edx, found_len  

    int 0x80  

    ; 将下标转为 ASCII 字符输出  

    add esi, '0'                  ; 单数字下标转字符  

    push esi                      ; 压栈作为临时存储  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, esp                  ; 栈顶地址  

    mov edx, 1  

    int 0x80  

    pop esi  

    ; 输出换行  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, newline  

    mov edx, 1  

    int 0x80  

exit:  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## 二维数组

二维数组在内存中按行展开为一维存储。访问 `arr[i][j]` 的地址公式为：

```
地址 = 基址 + (i*列数 + j) * 元素大小
```

## 实例

```asm
; 文件路径：2d_array.asm  

; 二维数组的定义和访问  

section .data  

    ; 3 行 4 列的二维数组  

    matrix dd 1,  2,  3,  4  

           dd 5,  6,  7,  8  

           dd 9, 10, 11, 12  

    rows equ 3  

    cols equ 4  

    elem_size equ 4              ; 双字 = 4 字节  

section .text  

    global _start  

_start:  

    ; 访问 matrix[1][2] = 7（第2行第3列）  

    ; 地址 = matrix + (1*4 + 2) * 4 = matrix + 24  

    mov eax, [matrix + (1 * cols + 2) * elem_size]  

    ; eax = 7  

    ; 使用寄存器动态计算（假设 i=2, j=1）  

    ; matrix[2][1] = 10（第3行第2列）  

    mov esi, 2                   ; 行 i = 2  

    mov edi, 1                   ; 列 j = 1  

    mov eax, cols                ; 列数  

    mul esi                      ; eax = i * cols = 2*4 = 8  

    add eax, edi                 ; eax = i*cols + j = 8+1 = 9  

    ; eax = eax * elem_size  

    ; 注意：MUL 的结果在 EAX 中，这里直接使用  

    mov eax, [matrix + eax * elem_size]  

    ; eax = 10  

    ; 遍历二维数组所有元素  

    mov ecx, rows * cols         ; 总元素数 = 12  

    mov esi, 0                   ; 下标  

    mov ebx, 0                   ; 累加和  

traverse:  

    add ebx, [matrix + esi * elem_size]  

    inc esi  

    loop traverse  

    ; ebx = 1+2+3+...+12 = 78  

    mov eax, 1  

    int 0x80  
```

---

## 冒泡排序完整示例

## 实例

```asm
; 文件路径：bubble_sort.asm  

; 冒泡排序算法  

section .data  

    array dd 64, 34, 25, 12, 22, 11, 90, 78  

    array_len equ ($ - array) / 4  

section .text  

    global _start  

_start:  

    mov ecx, array_len            ; 外层循环：n 次  

    dec ecx                       ; 外层只需 n-1 次  

outer_loop:  

    push ecx                      ; 保存外层计数器  

    mov esi, 0                    ; 内层下标从 0 开始  

    mov ecx, array_len - 1        ; 内层循环次数  

inner_loop:  

    mov eax, [array + esi * 4]    ; a[j]  

    mov ebx, [array + esi * 4 + 4]  ; a[j+1]  

    cmp eax, ebx                  ; a[j] > a[j+1] ?  

    jle no_swap                   ; 否，不交换  

    ; 交换 a[j] 和 a[j+1]  

    mov [array + esi * 4], ebx  

    mov [array + esi * 4 + 4], eax  

no_swap:  

    inc esi  

    loop inner_loop  

    pop ecx  

    loop outer_loop  

    ; 数组现在已排序：11, 12, 22, 25, 34, 64, 78, 90  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
> 汇编语言中的数组没有任何边界检查。访问越界的索引不会报错，而是会静默地读写邻近内存中的数据，这可能导致难以调试的 bug。务必自己确保索引在合法范围内。
