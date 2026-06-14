# tldr

命令行手册

https://tldr.inbrowser.app/

https://github.com/tldr-pages/tldr

https://github.com/tldr-pages/tldr-python-client

使用帮助

```shell
usage: tldr command [options]

Python command line client for tldr

positional arguments:
  command               command to lookup

options:
  -h, --help            show this help message and exit
  -v, --version         show program's version number and exit
  --search "KEYWORDS"   Search for a specific command from a query
  -u, --update, --update_cache
                        Update the local cache of pages and exit
  -k, --clear-cache     Delete the local cache of pages and exit
  -p PLATFORM, --platform PLATFORM
                        Override the operating system [android, freebsd, linux, netbsd, openbsd, osx, sunos, windows, common]
  -l, --list            List all available commands for operating system
  -s SOURCE, --source SOURCE
                        Override the default page source
  -c, --color           Override color stripping
  -r, --render          Render local markdown files
  -L LANGUAGE, --language LANGUAGE
                        Override the default language
  -m, --markdown        Just print the plain page file.
  --short-options       Display shortform options over longform
  --long-options        Display longform options over shortform
  --print-completion {bash,zsh,tcsh}
                        print shell completion script
```

环境变量配置

```shell
export TLDR_COLOR_NAME="cyan"
export TLDR_COLOR_DESCRIPTION="white"
export TLDR_COLOR_EXAMPLE="green"
export TLDR_COLOR_COMMAND="red"
export TLDR_COLOR_PARAMETER="white"
export TLDR_LANGUAGE="zh"
export TLDR_CACHE_ENABLED=1
export TLDR_CACHE_MAX_AGE=720
export TLDR_PAGES_SOURCE_LOCATION="https://raw.githubusercontent.com/tldr-pages/tldr/main/pages"
export TLDR_DOWNLOAD_CACHE_LOCATION="https://github.com/tldr-pages/tldr/releases/latest/download/tldr.zip"
export TLDR_OPTIONS=short
export TLDR_PLATFORM=linux
```

```shell
# 下载文档
tldr --update_cache

# 查看命令
tldr ls
```
