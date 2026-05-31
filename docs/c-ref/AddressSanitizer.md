# AddressSanitizer内存检测

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

## AddressSanitizer内存检测工具

gcc自带，无需额外安装

常见问题

```cpp
#include <stdlib.h>

// 越界访问
void stack_buffer_overflow()
{
    char buffer[1];
    int i = 10;
    buffer[i] = 'A'; // 访问越界
}

// 野指针
void use_after_free()
{
    char *text = (char *)malloc(sizeof(char) * 5);
    free(text);

    text[0] = '1'; // 访问已释放内存
}

// 内存泄漏
void leak_memory()
{
    char *text = (char *)malloc(sizeof(char) * 5);
}

```

示例

```cpp
// main.c
#include <stdlib.h>

// 野指针
void use_after_free()
{
    char *text = (char *)malloc(sizeof(char) * 5);
    free(text);

    text[0] = '1'; // 访问已释放内存
}

int main(int argc, char const *argv[])
{
    use_after_free();
    return 0;
}
```

运行就能输出对应的错误信息

```shell
$ gcc main.c -o main -fsanitize=address  -g && ./main

=================================================================
==25561==ERROR: AddressSanitizer: heap-use-after-free on address 0x6020000000d0 at pc 0x000109372ee1 bp 0x7ff7b6b8cef0 sp 0x7ff7b6b8cee8
WRITE of size 1 at 0x6020000000d0 thread T0
    #0 0x109372ee0 in use_after_free main.c:17
    #1 0x109372f2a in main main.c:28
    #2 0x7ff80bb45365 in start+0x795 (dyld:x86_64+0xfffffffffff5c365)

0x6020000000d0 is located 0 bytes inside of 5-byte region [0x6020000000d0,0x6020000000d5)
freed by thread T0 here:
    #0 0x109e18b69 in wrap_free+0xa9 (libclang_rt.asan_osx_dynamic.dylib:x86_64h+0xdcb69)
    #1 0x109372e9e in use_after_free main.c:15
    #2 0x109372f2a in main main.c:28
    #3 0x7ff80bb45365 in start+0x795 (dyld:x86_64+0xfffffffffff5c365)

previously allocated by thread T0 here:
    #0 0x109e18a20 in wrap_malloc+0xa0 (libclang_rt.asan_osx_dynamic.dylib:x86_64h+0xdca20)
    #1 0x109372e91 in use_after_free main.c:14
    #2 0x109372f2a in main main.c:28
    #3 0x7ff80bb45365 in start+0x795 (dyld:x86_64+0xfffffffffff5c365)

SUMMARY: AddressSanitizer: heap-use-after-free main.c:17 in use_after_free
Shadow bytes around the buggy address:
  0x601ffffffe00: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x601ffffffe80: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x601fffffff00: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x601fffffff80: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x602000000000: fa fa fd fd fa fa 00 00 fa fa 00 04 fa fa 00 00
=>0x602000000080: fa fa 00 04 fa fa 00 00 fa fa[fd]fa fa fa fa fa
  0x602000000100: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x602000000180: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x602000000200: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x602000000280: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x602000000300: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==25561==ABORTING
zsh: abort      ./main
```

## CMake 中集成 AddressSanitizer

在 CMake 中集成 AddressSanitizer (ASan) 是发现 `double free` 这类内存错误的得力助手，它相比 jemalloc 或 Valgrind 更加直接高效。下面是几种常见且易于维护的实现方式。

### ⚙️ CMake集成方法对比

这里整理了三种主流方法，你可以根据项目需求选择：

#### 环境变量法

- **实现方式**：在 `cmake` 命令前设置 `CFLAGS`/`CXXFLAGS` 环境变量。
- **特点**：
  - **侵入性**：**低**，无需修改 `CMakeLists.txt`。
  - **维护成本**：**高**，每次编译都需手动设置。
  - **适用场景**：**适用于快速测试**。
- **示例命令**：

```bash
export CFLAGS="-fsanitize=address -g -O0 -fno-omit-frame-pointer"
export CXXFLAGS="-fsanitize=address -g -O0 -fno-omit-frame-pointer"
cmake .. && make
```

#### CMakeLists.txt 配置法

- **实现方式**：在 `CMakeLists.txt` 中使用 `target_compile_options` 和 `target_link_options` 附加标志。
- **特点**：
  - **侵入性**：**中**，需要修改 CMake 脚本。
  - **维护成本**：**低**，一旦写入，配置可固定。
  - **适用场景**：**推荐用于开发分支**。
- **示例代码**：

```cmake
target_compile_options(your_target PRIVATE -fsanitize=address -g -O0 -fno-omit-frame-pointer)
target_link_options(your_target PRIVATE -fsanitize=address)
```

#### 命令行配置法 (CMake Presets)

- **实现方式**：使用 CMakePresets.json 或直接传递 `-DCMAKE_CXX_FLAGS` 参数。
- **特点**：
  - **侵入性**：**低**，通过命令行参数控制。
  - **维护成本**：**低**，便于集成到CI/CD流程。
  - **适用场景**：**适合 CI/CD**。
- **示例命令**：

```bash
cmake -B build -DCMAKE_CXX_FLAGS="-fsanitize=address -g -O0 -fno-omit-frame-pointer"
```

### 💡 关键编译与链接标志解读

集成 ASan 的核心是添加 `-fsanitize=address` 编译和链接标志，同时可以配合其他调试选项一起使用。以下是几个关键标志的作用：

- **`-fsanitize=address`**：启用 AddressSanitizer，是核心标志。
- **`-g`**：生成调试信息，让 ASan 报告能显示错误发生的准确代码行号，对调试至关重要。
- **`-O0`**：关闭所有编译优化，避免变量被优化掉，使 ASan 报告更易理解。
- **`-fno-omit-frame-pointer`**：保留帧指针，能显著提升 ASan 回溯堆栈的准确性。
- **`-static-libasan`**：静态链接 ASan 库，这对 macOS 或需要在旧版系统上运行的情况尤为有用。
- **`-fsanitize-recover=address`**：使 ASan 在遇到错误后继续运行，便于一次性发现多个问题。

### 🗺️ 构建与运行指南

集成之后，就可以通过以下步骤进行检测了：

1.  **清理并重新配置**：建议先清理旧的构建缓存，再运行 CMake，确保所有文件都用新的标志重新编译。
2.  **添加运行时选项**：通过 `ASAN_OPTIONS` 环境变量配置其行为。例如，启用内存泄漏检测和出错时终止程序：

```bash
export ASAN_OPTIONS="detect_leaks=1:halt_on_error=1:abort_on_error=1:print_stats=1"
```

**常用选项说明**:

- `detect_leaks=1`：在程序退出时检测内存泄漏（Linux 下默认开启，macOS 需显式开启）。
- `halt_on_error=1`：在遇到第一个错误时立即停止程序，便于快速定位首次发生的问题。
- `abort_on_error=1`：在出错时调用 `abort()`，以便生成 core dump 或使用 gdb 进行更细致的调试。
- `print_stats=1`：在程序结束时打印 ASan 的内存使用统计信息。

3.  **运行并观察**：像平常一样运行你的程序，ASan 会在检测到问题（如 double free）时，在终端输出详细的诊断报告。
