# pipx

`pipx` 是一个为安装和运行 Python **命令行工具** 而生的得力助手，它能像系统包管理器（如 `apt`, `brew`）一样，解决 `pip` 在全局安装时可能造成的依赖冲突等问题。

## 💡 它能解决什么问题？

简单来说，`pipx` 就是 `pip` 和 `venv` 的完美组合。它的核心价值在于：

- **隔离安装**：为每个工具创建独立的虚拟环境，避免不同工具间依赖冲突。
- **即装即用**：安装后即可在终端任意位置直接使用命令，无需手动激活环境。
- **按需尝试**：通过 `pipx run` 临时运行工具，用完即走，不污染系统。
- **清理无痕**：卸载时能彻底移除工具及其专属环境。

## 🚀 快速上手

1.  **安装 `pipx`**：
    - **macOS**: `brew install pipx && pipx ensurepath`
    - **Linux**: `python3 -m pip install --user pipx && python3 -m pipx ensurepath`
    - **Windows**: `scoop install pipx && pipx ensurepath`
      > 记得**重启终端**让环境变量生效。

2.  **常用命令速查**：
    - `pipx install <package>`：安装一个Python命令行工具
    - `pipx list`：列出所有通过pipx安装的工具
    - `pipx upgrade <package>`：升级指定工具
    - `pipx uninstall <package>`：卸载指定工具
    - `pipx run <package>`：临时运行一个工具而不安装

## ✨ 体验工作流

```bash
# 1. 安装代码格式化工具 black
pipx install black

# 2. 直接在任意目录下使用
black --version
black my_project/

# 3. 临时运行一个命令
pipx run pycowsay "Hello, pipx!"  # 运行完即销毁

# 4. 轻松升级
pipx upgrade black

# 5. 彻底卸载
pipx uninstall black
```

## 🔧 进阶技巧

- **管理工具依赖**：如果工具需要额外依赖，可用 `pipx inject` 注入。
- **指定Python版本**：使用 `--python` 参数为工具指定Python解释器。
- **自定义安装路径**：通过设置环境变量 `PIPX_HOME` 和 `PIPX_BIN_DIR` 来指定安装位置。
- **获取自动补全**：通过 `pipx completions` 生成命令补全脚本。
- **在CI中使用**：可直接用 `pipx run` 来运行工具，避免污染构建环境。

## ⚖️ 与其他工具的对比

- **`pipx` vs `pip`**：`pip` 安装**依赖库**和工具，`pipx` 只专注安装**终端工具**且自带隔离。
- **`pipx` vs `pipenv`/`Poetry`**：`pipenv`/`Poetry` 是给**项目**管理依赖的，`pipx` 是给整个系统安装**工具**的。
- **`pipx` vs `uv`**：`uv` 是 `pip`/`pipx` 等工具的超集，速度飞快，其 `uv tool` 功能对标 `pipx`。

## 🧩 管道与脚本中使用

在一些高级场景下，`pipx` 也能无缝接入管道和脚本流程，例如，你可以直接从标准输入读取内容进行处理：

```bash
echo "Hello from stdin" | pipx run pycowsay
```

这条命令会将 `echo` 的输出通过管道传递给 `pycowsay` 工具，由它在临时环境中处理并打印出有趣的奶牛图案，整个操作依然干净利落。

## 💎 总结：何时选用 pipx？

`s` 是一个功能聚焦的工具，它的使用场景很清晰：

| 使用场景                     | 推荐使用                           |
| :--------------------------- | :--------------------------------- |
| 安装**全局Python命令行工具** | **✅ pipx**                        |
| 隔离**项目**的开发依赖       | ❌ 使用 `pipenv` / `Poetry` / `uv` |
| 临时试用一个工具             | **✅ pipx run**                    |
| 安装一个供代码**导入**的库   | ❌ 使用 `pip` / `uv pip`           |
| 你需要一个包管理**全集**     | ❌ 考虑 `uv` 等新工具              |

希望这份指南能帮助你清晰理解并快速上手 `pipx`！如果对某个具体命令或与其他工具（比如 `uv`）的对比有更深入的疑问，也欢迎随时提出～
