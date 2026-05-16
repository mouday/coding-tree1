# fstream

C++ 文件输入输出库

## 类继承关系

```cpp

class basic_fstream: public basic_iostream {
    basic_fstream();

    explicit basic_fstream(
        const string& filename, 
        ios_base::openmode mode = ios_base::in | ios_base::out
    );

    bool is_open() const;

    /**
     * 打开文件
     */
    void open(
        const string& filename, 
        ios_base::openmode mode = ios_base::in | ios_base::out
    );

    void close();
}

using fstream  = basic_fstream<char>;
```

## 常见mode模式

| mode模式 | 说明
| - | -
| `std::ios::in` | 以输入模式打开文件
| `std::ios::out` | 以输出模式打开文件
| `std::ios::app`| 以追加模式打开文件
| `std::ios::ate`| 打开文件并定位到文件末尾
| `std::ios::trunc`| 打开文件并截断文件，即清空文件内容

## 写入文本文件

```cpp
#include <fstream>

int main(int argc, char const *argv[])
{
    std::fstream fs;

    // 以输出模式打开文件
    fs.open("test.txt", std::ios::out);
    // 写入文本
    fs << "Hello, World!" << std::endl;

    // 关闭文件
    fs.close();

    return 0;
}
```

输出

```shell
cat test.txt 
Hello, World!
```

## 读取文本文件

```cpp
#include <fstream>
#include <iostream>

int main(int argc, char const *argv[])
{
    std::fstream fs;

    // 以读模式打开文件
    fs.open("test.txt", std::ios::in);

    // 逐行读取
    std::string text;
    while (getline(fs, text))
    {
        std::cout << text << std::endl;
    }

    // 关闭文件
    fs.close();

    return 0;
}
```

输出

```shell
g++ fstream_read.cpp -o fstream_read && ./fstream_read
Hello, World!
```

## tellg

获取文件大小

```cpp
#include <iostream>
#include <fstream>
#include <string>

using namespace std;

streamsize get_file_size(string &filename)
{
    fstream file(filename, ios::in | ios::binary | ios::ate);
    if (!file)
    {

        return -1;
    }
    streamsize size = file.tellg();
    file.close();
    return size;
}

int main(int argc, char const *argv[])
{
    string filename("test.txt");
    streamsize file_size = get_file_size(filename);
    cout << "file size: " << file_size << endl;

    if (file_size > -1)
    {
        cout << "success" << endl;
    }
    else
    {
        cout << "open failed" << endl;
    }

    return 0;
}
```

输出结果

```shell
$ g++ -std=c++11 filesize.cpp -o filesize && filesize
file size: 11
success
```
