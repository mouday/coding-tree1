# 函数传值

代码实现

```cpp
#include <iostream>

class Data
{
public:
    Data();
    Data(const Data &data);
    ~Data();
};

Data::Data()
{
    std::cout << "Data::Data" << std::endl;
}

Data::Data(const Data &data)
{
    std::cout << "Data::Data Copy" << std::endl;
}

Data::~Data()
{
    std::cout << "Data::~Data" << std::endl;
}

// 传值
void foo1(Data data)
{
}

// 传指针
void foo2(Data *data)
{
}

// 传引用
void foo3(const Data &data)
{
}
```

传值

```cpp
int main(int argc, char const *argv[])
{
    Data data;
    foo1(data);
    return 0;
}
```

输出

```shell
Data::Data
Data::Data Copy
Data::~Data
Data::~Data
```

传指针

```cpp
int main(int argc, char const *argv[])
{
    Data data;
    foo2(&data);
    return 0;
}
```

输出

```shell
Data::Data
Data::~Data
```

传引用

```cpp
int main(int argc, char const *argv[])
{
    Data data;
    foo3(data);
    return 0;
}
```

输出

```shell
Data::Data
Data::~Data
```
