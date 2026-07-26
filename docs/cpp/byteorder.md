# 字节序

判断电脑字节序

方式1：python

```shell
python3 -c "import sys; print(sys.byteorder)"
little
```

输出：

- 小端 little
- 大端 big

方式2：C/C++

```cpp
#include <stdio.h>

int main(int argc, char const *argv[])
{
    unsigned int num = 1; // 0x00000001
    unsigned char byte = (unsigned char)num;
    if (byte == 1)
    {
        printf("little\n");
    }
    else
    {
        printf("big\n");
    }
    return 0;
}
```

原理：

- 整数1，在内存中的16进制为`0x00000001`
- 如果读取的第一个字节是`01`（低位在前），则是小端
- 如果读取的第一个字节是`00`（高位在前），则是大端
