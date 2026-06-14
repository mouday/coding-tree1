# gcc

gcc 常用参数

- `-I <dir>` 头文件`include` 搜索路径
- `-L <dir>` 动态库`library` 搜索路径
- `-g` 保留源码debug信息
- `-o <file>` 编译产物输出文件
- `-std=<value>` 语言标准库版本
- `-E` 仅预处理

查看帮助

```shell
gcc --help | less
```

## 预处理

```shell
gcc -E main.c -o main.i
```

输入`main.c`

```cpp
int main(int argc, char const *argv[])
{
    return 0;
}

```

输出`main.i`

```cpp
# 0 "main.c"
# 0 "<built-in>"
# 0 "<command-line>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 0 "<command-line>" 2
# 1 "main.c"
int main(int argc, char const *argv[])
{
    return 0;
}

```

## 编译

gcc -S main.i -o main.s


输出

```asm
	.file	"main.c"
	.text
	.globl	main
	.type	main, @function
main:
.LFB0:
	.cfi_startproc
	endbr64
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movl	%edi, -4(%rbp)
	movq	%rsi, -16(%rbp)
	movl	$0, %eax
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size	main, .-main
	.ident	"GCC: (Ubuntu 11.4.0-1ubuntu1~22.04.3) 11.4.0"
	.section	.note.GNU-stack,"",@progbits
	.section	.note.gnu.property,"a"
	.align 8
	.long	1f - 0f
	.long	4f - 1f
	.long	5
0:
	.string	"GNU"
1:
	.align 8
	.long	0xc0000002
	.long	3f - 2f
2:
	.long	0x3
3:
	.align 8
4:
```

## 汇编(as)

gcc -c main.s -o main.o

输出二进制

## 链接（ld）

gcc main.o -o main

## 运行

``shell
./main
```
