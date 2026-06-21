# 汇编语言 - 文件管理

文件管理是程序与外部存储交互的基础。

通过系统调用，汇编程序可以打开、读取、写入和关闭文件，与高级语言一样灵活地处理文件 IO。

---

## 文件操作流程总览

文件操作遵循打开→读写→关闭的基本流程，以及三个标准文件描述符：

![文件操作流程图](assets/file-io-flow.svg)

---

## 文件操作相关的系统调用

| 系统调用        | 调用号 | 用途        | 主要参数                     |
| ----------- | --- | --------- | ------------------------ |
| sys\_open   | 5   | 打开/创建文件   | EBX=文件名, ECX=标志, EDX=权限  |
| sys\_read   | 3   | 读取文件      | EBX=fd, ECX=缓冲区, EDX=字节数 |
| sys\_write  | 4   | 写入文件      | EBX=fd, ECX=缓冲区, EDX=字节数 |
| sys\_close  | 6   | 关闭文件      | EBX=fd                   |
| sys\_creat  | 8   | 创建文件（旧方式） | EBX=文件名, ECX=权限          |
| sys\_lseek  | 19  | 移动文件指针    | EBX=fd, ECX=偏移, EDX=起始位置 |
| sys\_unlink | 10  | 删除文件      | EBX=文件名                  |

---

## 文件打开标志

| 常量        | 值           | 说明        |
| --------- | ----------- | --------- |
| O\_RDONLY | 0           | 只读        |
| O\_WRONLY | 1           | 只写        |
| O\_RDWR   | 2           | 读写        |
| O\_CREAT  | 64（0x40）    | 若文件不存在则创建 |
| O\_TRUNC  | 512（0x200）  | 打开时清空文件内容 |
| O\_APPEND | 1024（0x400） | 追加到文件末尾   |

多个标志用 OR 组合，例如 `O_WRONLY | O_CREAT | O_TRUNC = 1 | 64 | 512 = 577`

---

## 创建并写入文件

## 实例

```asm
; 文件路径：file_write.asm  

; 创建文件并写入内容  

section .data  

    filename db 'runoob_output.txt', 0    ; 文件名（null 结尾）  

    content db 'Hello, RUNOOB!', 0xA       ; 要写入的内容  

    content_len equ $ - content  

    create_msg db 'File created successfully!', 0xA  

    create_msg_len equ $ - create_msg  

section .bss  

    fd resd 1                               ; 文件描述符  

section .text  

    global _start  

_start:  

    ; 1. 创建文件  

    mov eax, 8                  ; sys_creat  

    mov ebx, filename           ; 文件名  

    mov ecx, 0o644              ; 权限：rw-r--r--  

    ; 0o644 = 所有者读写、组只读、其他只读  

    int 0x80  

    mov [fd], eax               ; 保存文件描述符  

    ; 2. 写入内容到文件  

    mov eax, 4                  ; sys_write  

    mov ebx, [fd]               ; 文件描述符  

    mov ecx, content            ; 数据地址  

    mov edx, content_len        ; 数据长度  

    int 0x80  

    ; 3. 关闭文件  

    mov eax, 6                  ; sys_close  

    mov ebx, [fd]               ; 文件描述符  

    int 0x80  

    ; 4. 提示成功  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, create_msg  

    mov edx, create_msg_len  

    int 0x80  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

运行结果：

```
$ nasm -f elf32 file_write.asm -o file_write.o
$ ld -m elf_i386 file_write.o -o file_write
$ ./file_write
File created successfully!
$ cat runoob_output.txt
Hello, RUNOOB!
```

---

## 读取文件内容

## 实例

```asm
; 文件路径：file_read.asm  

; 打开并读取文件内容  

section .data  

    filename db 'runoob_output.txt', 0  

    open_error db 'Error: cannot open file', 0xA  

    open_error_len equ $ - open_error  

    read_success db 'File content:', 0xA  

    read_success_len equ $ - read_success  

section .bss  

    fd resd 1  

    buffer resb 1024                 ; 读取缓冲区  

section .text  

    global _start  

_start:  

    ; 1. 打开文件（只读）  

    mov eax, 5                  ; sys_open  

    mov ebx, filename           ; 文件名  

    mov ecx, 0                  ; 只读模式  

    mov edx, 0                  ; 权限（只读时忽略）  

    int 0x80  

    cmp eax, 0                  ; 文件描述符 < 0 表示错误  

    jl  open_failed  

    mov [fd], eax               ; 保存文件描述符  

    ; 2. 读取文件内容  

    mov eax, 3                  ; sys_read  

    mov ebx, [fd]               ; 文件描述符  

    mov ecx, buffer             ; 缓冲区  

    mov edx, 1024               ; 最多读取 1024 字节  

    int 0x80  

    ; 返回值 eax = 实际读取的字节数  

    mov esi, eax                ; 保存读取字节数  

    ; 3. 关闭文件  

    mov eax, 6                  ; sys_close  

    mov ebx, [fd]  

    int 0x80  

    ; 4. 输出提示信息  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, read_success  

    mov edx, read_success_len  

    int 0x80  

    ; 5. 输出文件内容到屏幕  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, buffer  

    mov edx, esi                ; 使用实际读取的字节数  

    int 0x80  

    jmp exit  

open_failed:  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, open_error  

    mov edx, open_error_len  

    int 0x80  

exit:  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

运行结果：

```
$ nasm -f elf32 file_read.asm -o file_read.o
$ ld -m elf_i386 file_read.o -o file_read
$ ./file_read
File content:
Hello, RUNOOB!
```

---

## 追加写入文件

## 实例

```asm
; 文件路径：file_append.asm  

; 以追加模式打开文件并写入  

section .data  

    filename db 'runoob_log.txt', 0  

    log_entry db '[INFO] Program executed successfully.', 0xA  

    log_len equ $ - log_entry  

section .bss  

    fd resd 1  

section .text  

    global _start  

_start:  

    ; 打开文件（创建 + 追加模式）  

    mov eax, 5                  ; sys_open  

    mov ebx, filename           ; 文件名  

    mov ecx, 0x441              ; O_WRONLY | O_CREAT | O_APPEND  

    ; O_WRONLY=1, O_CREAT=0x40, O_APPEND=0x400  

    ; 组合：1 | 0x40 | 0x400 = 0x441  

    mov edx, 0o644              ; 创建时的权限  

    int 0x80  

    mov [fd], eax  

    ; 追加写入  

    mov eax, 4                  ; sys_write  

    mov ebx, [fd]               ; 文件描述符  

    mov ecx, log_entry          ; 日志内容  

    mov edx, log_len  

    int 0x80  

    ; 关闭文件  

    mov eax, 6  

    mov ebx, [fd]  

    int 0x80  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```

---

## 复制文件

一个综合示例：将一个文件的内容复制到另一个文件：

## 实例

```asm
; 文件路径：file_copy.asm  

; 复制文件：将 runoob_input.txt 复制到 runoob_copy.txt  

section .data  

    src_file db 'runoob_input.txt', 0  

    dst_file db 'runoob_copy.txt', 0  

    success_msg db 'File copied successfully!', 0xA  

    success_len equ $ - success_msg  

    error_msg db 'Error during file copy', 0xA  

    error_len equ $ - error_msg  

section .bss  

    src_fd resd 1  

    dst_fd resd 1  

    buffer resb 4096              ; 4KB 复制缓冲区  

section .text  

    global _start  

_start:  

    ; 1. 打开源文件（只读）  

    mov eax, 5  

    mov ebx, src_file  

    mov ecx, 0                   ; O_RDONLY  

    mov edx, 0  

    int 0x80  

    cmp eax, 0  

    jl  error_exit  

    mov [src_fd], eax  

    ; 2. 创建目标文件  

    mov eax, 8                   ; sys_creat  

    mov ebx, dst_file  

    mov ecx, 0o644  

    int 0x80  

    cmp eax, 0  

    jl  error_exit  

    mov [dst_fd], eax  

copy_loop:  

    ; 3. 从源文件读取  

    mov eax, 3                   ; sys_read  

    mov ebx, [src_fd]  

    mov ecx, buffer  

    mov edx, 4096  

    int 0x80  

    ; eax = 实际读取的字节数  

    cmp eax, 0                   ; 读到 0 字节？  

    jle copy_done                ; 是，文件结束  

    ; 4. 写入目标文件  

    mov esi, eax                 ; 保存读取的字节数  

    mov eax, 4                   ; sys_write  

    mov ebx, [dst_fd]  

    mov ecx, buffer  

    mov edx, esi                 ; 写入实际读取的字节数  

    int 0x80  

    jmp copy_loop                ; 继续循环  

copy_done:  

    ; 5. 关闭文件  

    mov eax, 6                   ; sys_close  

    mov ebx, [src_fd]  

    int 0x80  

    mov eax, 6  

    mov ebx, [dst_fd]  

    int 0x80  

    ; 6. 输出成功信息  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, success_msg  

    mov edx, success_len  

    int 0x80  

    jmp exit  

error_exit:  

    mov eax, 4  

    mov ebx, 1  

    mov ecx, error_msg  

    mov edx, error_len  

    int 0x80  

exit:  

    mov eax, 1  

    mov ebx, 0  

    int 0x80  
```
> 文件操作前后务必检查系统调用的返回值（在 EAX 中）。负值表示错误（如 -2 是 ENOENT 文件不存在，-13 是 EACCES 权限不足），忽略错误返回值是文件操作 bug 的最大来源。
