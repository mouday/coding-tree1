# 函数

传参示例

```cpp
#include <iostream>

using namespace std;

void foo1(int val)
{
    val = 11;
    cout << "val: " << addressof(val) << " " << val << endl;
}

void foo2(int &val)
{
    val = 22;
    cout << "val: " << addressof(val) << " " << val << endl;
}

void foo3(int *val)
{
    *val = 33;
    cout << "val: " << val << " " << *val << endl;
}

int main(int argc, char const *argv[])
{
    int val = 0;
    foo1(val);
    cout << "foo1 val: " << addressof(val) << " " << val << endl;

    foo2(val);
    cout << "foo2 val: " << addressof(val) << " " << val << endl;

    foo3(&val);
    cout << "foo3 val: " << addressof(val) << " " << val << endl;
    return 0;
}
```

输出结果

```shell
val: 0x7ff7b830e11c 11
foo1 val: 0x7ff7b830e13c 0
val: 0x7ff7b830e13c 22
foo2 val: 0x7ff7b830e13c 22
val: 0x7ff7b830e13c 33
foo3 val: 0x7ff7b830e13c 33
```
