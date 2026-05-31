# jemalloc

官网仓库：https://github.com/jemalloc/jemalloc

https://gitee.com/mirrors/jemalloc

https://jemalloc.net/

https://github.com/jemalloc/jemalloc/releases

https://jemalloc.net/jemalloc.3.html

## jemalloc简介

`jemalloc` 是个为多线程高并发场景设计的高性能内存分配器，也是不少明星项目（如 Redis、Firefox）的默认选择。相比于传统的 `glibc` 的 `ptmalloc`，它的核心优势在于能显著**减少多线程下的锁竞争**和**内存碎片**。

### 🔍 jemalloc 如何在多线程下“加速”？

`jemalloc` 的高性能主要源于其“分而治之”的设计哲学，通过以下几个层面的划分，极大避免了锁的争用：

1.  **Arena**：将内存划分为多个“竞技场”（Arena），每个新线程会被轮转分配到一个 Arena 上。每个 Arena 有自己独立的锁，这样多线程访问不同 Arena 时可以并行操作，极大地提高了并发度。
2.  **大小类 (Size Class)**：在 Arena 内部，内存请求会被划分到特定的大小类（如 8字节、16字节等），不同的大小类由不同的 `bin` 管理，进一步隔离了不同大小内存的分配请求。
3.  **线程本地缓存 (tcache)**：每个线程都拥有一个私有的缓存（tcache），用于存放小块内存。当线程分配/释放内存时，优先在 `tcache` 中完成，完全无锁，性能极高。

### 📦 如何在你的项目中安装与使用 `jemalloc`

这里有几种主流的使用方式，可以看看哪种更贴合你的场景。

#### 🐧 Linux 发行版安装

不同发行版下的安装命令。

#### ⛓️ 编译时静态链接

如果你的程序在编译阶段就需要确定使用 `jemalloc`，可以将它与你的代码静态链接。

- **编译命令**：
  ```bash
  gcc -o my_program my_program.c -ljemalloc -DJEMALLOC_NO_DEMANGLE
  ```
  使用时需要在源代码中包含头文件 `#include <jemalloc/jemalloc.h>`。
- **完整编译参数参考**：
  你也可以通过 `jemalloc-config` 命令获取完整的编译参数，确保链接无误:
  ```bash
  gcc your_program.c `jemalloc-config --cflags` -o your_program `jemalloc-config --libs`
  ```

#### 💊 运行时动态替换 (LD_PRELOAD) - **最推荐**

这种方式无需修改代码，只需在程序启动前预加载 `jemalloc` 的动态库即可生效。

- **设置环境变量**：
  ```bash
  export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so.2
  ```
  然后正常启动你的程序，它将自动使用 `jemalloc`。
- **手动指定动态库路径**：
  如果系统库路径不包含 `jemalloc`，你可以在编译时显式指定:
  ```bash
  gcc your_program.c -o your_program -I/path/to/jemalloc/include -L/path/to/jemalloc/lib -ljemalloc
  ```

> `-DJEMALLOC_NO_DANGLE` 这个编译选项是为了确保 `jemalloc` 特有的内存函数（如 `mallocx`, `sdallocx`）能被正确编译。

### ⚙️ 用 `MALLOC_CONF` 环境变量进行调优

在**运行时**通过环境变量 `MALLOC_CONF` 进行配置，可以随时调整策略和输出信息，无需重启应用。

最常用的参数包括这几个:

| 参数                | 说明                                                                               | 示例值         |
| :------------------ | :--------------------------------------------------------------------------------- | :------------- |
| `background_thread` | 开启后台线程，异步回收内存，避免阻塞应用线程，提升响应速度。                       | `true`         |
| `dirty_decay_ms`    | 控制脏页（dirty page）被释放回系统的等待时间，缩短它可让内存更快归还给OS。         | `10000` (10秒) |
| `narenas`           | 限制创建的Arena数量，可以减少内存占用。可尝试设置为 `$(nproc)` 或 `2*$(nproc)`。   | `8`            |
| `stats_print`       | 程序退出时，将统计信息（内存分配总量、碎片率等）打印到控制台，对初步分析很有帮助。 | `true`         |

**配置示例：**

```bash
# 开启后台线程，设置脏页回收时间为10秒，程序退出时打印统计信息
export MALLOC_CONF="background_thread:true,dirty_decay_ms:10000,stats_print:true"
./your_program
```

### 🛠️ 用好 `jeprof` 工具，揪出内存问题

`jeprof` 是分析 `jemalloc` 内存问题的核心工具，可以用来定位**内存泄漏**或分析**内存热点**。

**1. 开启 Profiling**
生成可供分析的堆文件（heap file）。

- **通过 `MALLOC_CONF` 开启**：

  ```bash
  export MALLOC_CONF="prof:true,lg_prof_sample:19,prof_prefix:jeprof.out"
  ./your_program
  ```

- **关键参数说明**:

| 参数              | 说明                                                                          |
| :---------------- | :---------------------------------------------------------------------------- |
| `prof:true`       | 开启 profiling 功能（会有5-10%的性能开销）。                                  |
| `lg_prof_sample`  | 设置采样间隔。`19` 表示约 `2^19 = 524KB` 采样一次，值越小越精确，但开销越大。 |
| `prof_prefix`     | 指定生成的堆转储文件前缀（如 `jeprof.out.12345.0.i0.heap`）。                 |
| `prof_final:true` | 在程序正常退出时，主动生成最终的堆转储文件。                                  |

**2. 使用 `jeprof` 分析堆文件**
`jeprof` 可生成多种格式的报告。

- **安装 `jeprof`**：
  通常随 `jemalloc` 源码一同编译安装，或通过包管理器（如 `apt install libjemalloc-dev`）获得。
- **生成文本报告**：

```bash
jeprof --text ./your_program jeprof.out.12345.0.i0.heap
```

- **生成可视化调用图（推荐）**：
  生成的 PDF 可以直观看到各函数的内存占用比例。

```bash
yum install graphviz ghostscript  # 确保系统已安装 graphviz 和 ghostscript
jeprof --pdf ./your_program jeprof.out.12345.0.i0.heap > memory_profile.pdf
```

#### jeprof 常用选项

| 选项     | 说明                                                           | 示例                                            |
| :------- | :------------------------------------------------------------- | :---------------------------------------------- |
| `--text` | 生成文本格式报告，默认按累计内存排序。                         | `jeprof --text ./program heap.prof`             |
| `--pdf`  | 生成向量化的 PDF 调用图，适合快速定位热点。                    | `jeprof --pdf ./program heap.prof > report.pdf` |
| `--base` | 与基线对比，用于观察增量变化或泄露情况（需先导出基线堆文件）。 | `jeprof --base=base.heap ./program leak.heap`   |
| `--cum`  | 查看某函数及其所有调用者的累积内存消耗。                       | `jeprof --cum --text ./program heap.prof`       |

如果你在调试某个具体的高并发服务时，想针对它的内存热点进行深度优化，可以告诉我你的具体场景，我来帮你分析 `jeprof` 生成的调用图可以怎么解读。

## jemalloc安装

安装 jemalloc 主要有两种方法，你可以根据需要选择：

- **包管理器安装**：这是最**推荐**的方法。它简单、快捷，方便后续的维护和卸载。
- **源码编译安装**：适用于需要特定版本、定制编译选项（如调试、分析）或在较旧系统上进行安装的场景。

下面会详细介绍这两类方法及在不同操作系统下的具体操作。

### 📦 使用包管理器快速安装

这是绝大多数情况下的首选安装方式，对于不同系统，对应的命令如下：

| 操作系统 / 发行版          | 安装命令                                                                                            | 说明                               |
| :------------------------- | :-------------------------------------------------------------------------------------------------- | :--------------------------------- |
| **Ubuntu / Debian**        | `sudo apt update && sudo apt install libjemalloc-dev`                                               | 安装开发包（包含库文件和头文件）。 |
| **RHEL / CentOS 7.x, 8.x** | `sudo yum install epel-release && sudo yum install jemalloc-devel`                                  | 需要先启用 EPEL 仓库。             |
| **Fedora**                 | `sudo dnf install jemalloc-devel`                                                                   | -                                  |
| **Alpine Linux**           | `apk add jemalloc`                                                                                  | -                                  |
| **macOS**                  | `brew install jemalloc`                                                                             | 通过 Homebrew 安装。               |
| **Docker**                 | `Dockerfile<br>FROM <your-base-image><br>RUN apt-get update && apt-get install -y libjemalloc-dev ` | 在 `Dockerfile` 中集成安装命令。   |

### ⚙️ 从源码编译安装

当你需要特定版本或自定义功能时，可以选择源码编译。以下是最新稳定版（5.3.0）的完整编译过程：

1. **获取源码**：从 GitHub 克隆特定版本的代码并进入目录。

```bash
git clone --branch 5.3.0 https://github.com/jemalloc/jemalloc.git
cd jemalloc
```

2.  **生成配置**：如果你是首次构建，请运行此脚本来生成 `configure` 文件。

```bash
./autogen.sh
```

3.  **配置和编译**：运行 `configure` 脚本检查环境并设置安装路径（`--prefix` 指定安装位置，建议加上），然后开始编译。

```bash
./configure --prefix=/usr/local
make
```

4.  **安装**：将编译好的文件安装到系统目录。

```bash
sudo make install
```

### 从release编译安装

源码编译‌：推荐方式，可自定义优化参数

```shell
wget https://github.com/jemalloc/jemalloc/releases/download/5.3.1/jemalloc-5.3.1.tar.bz2
tar -jxvf jemalloc-5.3.1.tar.bz2
cd jemalloc-5.2.1/

./configure --prefix=/usr/lib --enable-debug --enable-prof
make -j 8
make install
```

#### 常用编译配置选项

下表列出了源码编译时可用的配置选项，你可以根据需求按 `./configure <选项>` 的格式启用或禁用特定功能。

| 类别         | 配置选项                        | 说明                                                         |
| :----------- | :------------------------------ | :----------------------------------------------------------- |
| **核心设置** | `--prefix=/path`                | 指定安装路径，默认为 `/usr/local`。                          |
|              | `--with-jemalloc-prefix=myapp_` | 为所有公共 API 添加前缀，避免符号冲突。                      |
| **功能选项** | `--enable-debug`                | 启用断言和调试代码，开发时使用，会影响性能。                 |
|              | `--enable-prof`                 | **开启 Profiling 功能**，用于内存分析。                      |
|              | `--disable-fill`                | 禁用内存填充，可略微提升性能但会降低安全性。                 |
| **平台相关** | `--with-lg-page=<N>`            | 设置页面大小，N 表示 2^N 字节（如 12=4KB），交叉编译时常用。 |
|              | `--disable-zone-allocator`      | 在 macOS（Darwin）上构建时使用，避免成为默认分配器。         |
| **清理操作** | `make clean`                    | 清理编译产生的临时文件。                                     |
|              | `make distclean`                | 执行深度清理，包括 `Makefile` 等生成的文件。                 |
|              | `make uninstall`                | **卸载 jemalloc**。                                          |

### 🪟 Windows 平台安装

在 Windows 上构建 jemalloc 相对复杂，推荐两种方法：

- **使用 MSYS2**：**这是成功率较高且更灵活的方法**。通过 MSYS2 环境可以模拟 Unix，更容易完成编译。
  1.  安装 MSYS2。
  2.  在 MSYS2 终端中安装必要组件：`pacman -S autoconf automake make mingw-w64-x86_64-gcc`。
  3.  进入 jemalloc 源码目录，执行 `./configure && make && make install` 完成编译。
- **配合 Cygwin 和 Visual Studio**：先安装 Cygwin 和 `autoconf` 等工具，然后在 Visual Studio 开发者命令提示符中，设置 `CC=cl` 后运行 `./autogen.sh` 来生成 `Makefile`，最后在 Visual Studio 中打开生成的 `jemalloc_vc20xx.sln` 解决方案进行编译。

---

### 💎 总结

对于开发者而言，**如果只是想在项目中使用 jemalloc，那么通过操作系统的包管理器安装 `libjemalloc-dev` 或 `jemalloc-devel` 是最高效便捷的选择。**

**从源码编译安装则适用于以下场景**：

- 需要一个不被包管理器收录的特定版本。
- 需要启用 Profiling、Debug 等特定编译选项以进行性能分析或开发调试。
- 使用的 Linux 发行版太旧，官方源中没有 jemalloc 的软件包。

## jemalloc 使用

来看一些具体的 jemalloc 使用示例，这几个场景基本涵盖了从常规使用到故障排查的关键环节。

假设我们有一个非常简单的C程序 `test.c`，包含了标准的内存操作：

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    void *ptr = malloc(1024);   // 分配内存
    printf("内存已分配\n");
    free(ptr);                  // 释放内存
    return 0;
}
```

下面我们来看看针对这个程序，jemalloc 可以有哪些用法。

---

### 1. 📥 基础使用：链接与预加载

这是让程序使用 jemalloc 的两种最直接的方式。

#### 场景A：编译时静态链接

当你能控制程序的编译过程时，可以选择链接 jemalloc。

1. **包含头文件**：
    在代码中包含 jemalloc 的头文件，并编译。

```c
#include <jemalloc/jemalloc.h>
```

2. **编译链接**：

使用 `-ljemalloc` 参数链接库。

```bash
gcc -o test test.c -ljemalloc
```

#### 场景B：运行时动态加载（无需修改代码）

如果无法修改程序代码，使用 `LD_PRELOAD` 让 jemalloc 在程序启动时强行注入是最高效的方案。

```bash
export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so.2
./test
```

> **注意**: 请根据你的系统路径调整 `libjemalloc.so` 的路径。

---

### 2. ⚙️ 性能调优：通过MALLOC_CONF调整策略

通过环境变量 `MALLOC_CONF`，你可以在不修改代码的情况下动态调整 jemalloc 的行为。

```bash
export MALLOC_CONF="background_thread:true,dirty_decay_ms:5000"
./test
```

- `background_thread:true`: 启用后台线程异步回收内存，有助于平复性能毛刺。
- `dirty_decay_ms:5000`: 设置脏页在释放回操作系统前的等待时间为 5 秒。可根据服务的内存模式调整。

---

### 3. 🕵️‍♂️ 内存调试：检测与分析内存泄漏

#### 如何生成 Heap Profile？

在运行程序前设置 `MALLOC_CONF` 来启用 Profiling 功能。

```bash
export MALLOC_CONF="prof:true,prof_leak:true,lg_prof_sample:19,prof_prefix:jeprof.out"
./test
```

- `prof:true`: 启用内存分析。
- `prof_leak:true`: 指示 jemalloc 在程序退出时检测并报告泄漏。
- `lg_prof_sample:19`: 设置采样间隔，默认 `19` 即每 2^19 字节 (~512KB) 采样一次。
- `prof_prefix:jeprof.out`: 指定生成的 heap profile 文件前缀。

#### 如何使用 Jeprof 分析？

程序运行后会生成 `jeprof.out.*.heap` 文件。用 `jeprof` 工具分析。

1.  **查看泄漏摘要** (文本模式)：

```bash
jeprof --show_bytes --leaks ./test jeprof.out.*
```

    这个命令会直接列出可能的内存泄漏点。

2.  **生成可视化报告** (PDF)：

```bash
jeprof --pdf ./test jeprof.out.* > leak_report.pdf
```

    这个命令会生成带调用图的 PDF 文件，能更直观地看到是哪些函数占用了内存。
    > **注意**: `jeprof` 工具需要先通过 `--enable-prof` 编译 jemalloc 才会在 `bin` 目录下生成。

---

### 4. 🧵 高级用法：通过 `mallctl` API 动态控制

jmalloc 提供了 `mallctl` 接口，让你可以在程序运行时查询或修改其状态。

```c
#include <jemalloc/jemalloc.h>
#include <stdio.h>

int main() {
    ssize_t decay_ms;
    size_t sz = sizeof(ssize_t);

    // 获取当前的脏页回收时间
    int ret = mallctl("arenas.dirty_decay_ms", &decay_ms, &sz, NULL, 0);
    if (ret == 0) {
        printf("当前脏页回收时间: %zd ms\n", decay_ms);
    }

    // 在程序运行中调整策略
    decay_ms = 1000;
    ret = mallctl("arenas.dirty_decay_ms", NULL, NULL, &decay_ms, sz);
    if (ret == 0) {
        printf("脏页回收时间已调整为 1 秒\n");
    }

    return 0;
}
```

编译时需要链接 jemalloc：

```bash
gcc -o test test.c -ljemalloc
```

在实际使用中，请注意 Profiling 会有一定的性能开销。建议在开发调试阶段开启，而在生产环境中根据对性能影响极小的配置（如 `prof:true,prof_active:false`）做到即开即用。

希望这些从基础到深入的示例能帮到你。你可以先在自己的环境里跑一下这些命令，如果在某个步骤（比如 `jeprof` 的生成）上遇到问题，或者想具体看看怎么用它来排查一次真实的泄漏，随时可以告诉我。

## jemalloc 检测double free

最简单的快速诊断流程是：

使用调试版的 jemalloc (--enable-debug)。

通过环境变量强制禁用线程缓存 (tcache:false)。

```bash
# 启用该选项，并设置为一个较大的扫描深度（如 128）
LD_PRELOAD=/path/to/jemalloc/lib/libjemalloc.so.2 \
MALLOC_CONF="tcache:false,abort:true,debug_double_free_max_scan:128" \
./your_program
```
