# vector

动态数组

## 常用成员函数

| 函数  | 说明
|-|-
| `push_back(const T& val)` |在末尾添加元素
| `pop_back()`  |删除末尾元素
| `at(size_t pos)` | 返回指定位置的元素，带边界检查
| `operator[]`  |返回指定位置的元素，不带边界检查
| `front()` |返回第一个元素
| `back()`  |返回最后一个元素
| `data()`  |返回指向底层数组的指针
| `size()`  |返回当前元素数量
| `capacity()`  |返回当前分配的容量
| `reserve(size_t n)`   |预留至少 n 个元素的存储空间
| `resize(size_t n)`   |将元素数量调整为 n
| `clear()` | 清空所有元素
| `insert(iterator pos, val)`  | 在指定位置插入元素
| `erase(iterator pos)` |删除指定位置的元素
| `begin()` |返回起始迭代器
| `end()` | 返回结束迭代器

## 容器对比

|特性 | `std::vector` | `std::array`| `std::list`
|-| - | - |-|
大小 |动态可变 |编译时固定 |动态可变
存储位置 |连续内存 |连续内存 |非连续内存
访问性能 |随机访问快速| 随机访问快速 |随机访问慢，适合顺序访问
插入和删除性能 |末尾操作性能高，其他位置较慢 |不支持 | 任意位置插入和删除较快
内存增长方式 |容量不足时成倍增长 |无 |无

实现接口

```cpp
template <class _Tp, class _Allocator /* = allocator<_Tp> */>
class vector
{
    vector();
    explicit vector(size_type n);
    explicit vector(size_type n, const value_type& x);
    vector(size_type n, const value_type& x, const allocator_type& a);
    vector(initializer_list<value_type> il);
    vector(vector&& x);

    ~vector();

    iterator begin() noexcept;
    size_type size() const;
    size_type capacity() const;
    bool empty() const;
    reference at(size_type n);
    reference operator[](size_type n)
    reference  front()
    reference  back()
    value_type*  data()
    reference emplace_back(Args&&... args);
    void pop_back();
    iterator insert(const_iterator position, value_type&& x);
    iterator emplace(const_iterator position, Args&&... args);
    void clear() noexcept;
    void push_back(const_reference x)
    void swap(vector&);
    iterator erase(const_iterator position);
    void resize(size_type sz);
}
```

初始化

```cpp
#include <vector>

std::vector<int> vec; // 创建一个空的整数向量
std::vector<int> vec(5); // 创建一个包含5个默认初始化整数的向量
std::vector<int> vec(5, 10); // 创建一个包含5个整数，值均为10的向量
std::vector<int> vec = {1, 2, 3, 4, 5}; // 使用初始化列表创建向量
```

示例

```cpp
#include <vector>
#include <iostream>

int main(int argc, char const *argv[])
{
    std::vector<int> vec = {1, 2, 3, 4, 5};

    // 遍历元素
    for (const auto &val : vec)
    {
        std::cout << val << " ";
    }
    std::cout << std::endl;

    // 添加元素
    vec.push_back(6);

    // 修改元素
    vec[0] = 10;

    // 移除元素
    vec.pop_back();

    // 获取大小
    std::cout << "Size: " << vec.size() << std::endl;

    // 检查是否为空
    std::cout << "Is empty: "
              << (vec.empty() ? "Yes" : "No")
              << std::endl;

    // 访问元素
    std::cout << "First element: " << vec.front() << std::endl;
    std::cout << "Last element: " << vec.back() << std::endl;

    // 获取容量
    std::cout << "Capacity: " << vec.capacity() << std::endl;

    // 清空向量
    vec.clear();

    // 检查清空后是否为空
    std::cout << "Is empty after clear: "
              << (vec.empty() ? "Yes " : "No")
              << std::endl;

    return 0;
}
```

输出

```shell
1 2 3 4 5 
Size: 5
Is empty: No
First element: 10
Last element: 5
Capacity: 10
Is empty after clear: Yes 
```

## 修改数据

### push_back

将元素添加到容器末尾

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> list;

    // 添加元素
    list.push_back(3);
    list.push_back(4);
    list.push_back(5);

    // 遍历
    for (int value: list) {
        std::cout << value << " ";
    }
    // 3 4 5 
}

```

### insert

插入元素到容器中的指定位置

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> list = {3, 4, 5};

    // 下标为2的位置插入数据
    list.insert(list.begin() + 2, 10);

    for (int value: list) {
        std::cout << value << " ";
    }
    // 3 4 10 5
}

```

插入多个数据

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> list = {3, 4, 5};

    // 下标为2的位置插入多个数据
    list.insert(list.begin() + 2, {10, 11, 12});

    for (int value: list) {
        std::cout << value << " ";
    }
    // 3 4 10 11 12 5 
}

```
### pop_back

移除容器的末元素

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> list = {3, 4, 5};

    list.pop_back();

    // 遍历
    for (int value: list) {
        std::cout << value << " ";
    }
    // 3 4 
}

```

### clear

清除内容

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> list = {3, 4, 5};

    // 清除内容
    list.clear();

    // 此时，没有任何输出
    for (int value: list) {
        std::cout << value << " ";
    }
}
```

### erase

从容器擦除指定的元素

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> list = {3, 4, 5};

    // 清除内容
    list.erase(list.begin() + 1);

    for (int value: list) {
        std::cout << value << " ";
    }
    // 3 5
}

```

范围移除

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> list = {3, 4, 5};

    // 清除内容 [0, 2)
    list.erase(list.begin(), list.begin() + 2);

    for (int value: list) {
        std::cout << value << " ";
    }
    // 5
}

```

## 访问数据

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> list = {3, 4, 5};

    // 访问下标为1的元素
    std::cout << list[1] << std::endl; // 4
    std::cout << list.at(1) << std::endl; //4

    // 访问第一个元素
    std::cout << list.front() << std::endl; // 3

    // 访问最后一个元素
    std::cout << list.back() << std::endl; // 5
}

```

## 容器信息

### size

返回元素数

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> list = {3, 4, 5};

    std::cout << list.size() << std::endl; // 3
}

```

### empty

检查容器是否为空

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> list = {3, 4, 5};

    std::cout << list.empty() << std::endl; // 0

    list.clear();

    std::cout << list.empty() << std::endl; // 1
}

```

## 迭代数据

### for-in

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> list = {3, 4, 5};

    // 遍历
    for (int value: list) {
        std::cout << value << " ";
    }
    // 3 4 5
}

```

### fori

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> list = {3, 4, 5};

    for (int i = 0; i < list.size(); i++) {
        std::cout << list.at(i) << " ";
    }
    // 3 4 5
}

```

### iterator

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> list = {3, 4, 5};

    for (std::vector<int>::iterator iter = list.begin(); iter != list.end(); iter++) {
        std::cout << *iter << " ";
    }
    // 3 4 5
}

```

## sort

默认从小到大排序

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> list = {2, 4, 3, 5, 1};

    std::sort(list.begin(), list.end());

    for (int value: list) {
        std::cout << value << " ";
    }
    // 1 2 3 4 5 
}
```

自定义从大到小排序

```cpp
#include <iostream>
#include <vector>

/**
* 自定义排序函数
* 返回true 交换位置
* 返回false 不交换位置
*/
bool compare(int a, int b) {
    return a > b;
}

int main() {
    std::vector<int> list = {2, 4, 3, 5, 1};

    std::sort(list.begin(), list.end(), compare);

    for (int value: list) {
        std::cout << value << " ";
    }
    // 5 4 3 2 1 
}
```

## reverse

反转vector

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> list = {2, 4, 3, 5, 1};

    std::reverse(list.begin(), list.end());

    for (int value: list) {
        std::cout << value << " ";
    }
    // 1 5 3 4 2 
}

```

## in

成员检查

1、`std::count`

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> list = {3, 4, 5};

    if(std::count(list.begin(), list.end(), 3)) {
        std::cout << "存在" << std::endl;
    } else {
        std::cout << "不存在" << std::endl;
    }

    // 输出: 存在
    return 0;

}

```


2、`std::find`

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> list = {3, 4, 5};

    if(std::find(list.begin(), list.end(), 3) != list.end()) {
        std::cout << "存在" << std::endl;
    } else {
        std::cout << "不存在" << std::endl;
    }

    // 输出: 存在
    return 0;

}

```

3、`std::find_if`

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> list = {3, 4, 5};

    if (std::find_if(list.begin(), list.end(), [](const int x) {
        return x == 3;
    }) != list.end()) {
        std::cout << "存在" << std::endl;
    } else {
        std::cout << "不存在" << std::endl;
    }

    // 输出: 存在
    return 0;
}

```

4、`std::any_of`

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> list = {3, 4, 5};

    if (std::any_of(list.begin(), list.end(), [](const int x) {
        return x == 3;
    })) {
        std::cout << "存在" << std::endl;
    } else {
        std::cout << "不存在" << std::endl;
    }

    // 输出: 存在
    return 0;
}

```

5、`std::none_of`

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> list = {3, 4, 5};

    if (!std::none_of(list.begin(), list.end(), [](const int x) {
        return x == 3;
    })) {
        std::cout << "存在" << std::endl;
    } else {
        std::cout << "不存在" << std::endl;
    }

    // 输出: 存在
    return 0;
}

```

6、`std::binary_search`

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> list = {3, 4, 5};

    if (std::binary_search(list.begin(), list.end(), 3)) {
        std::cout << "存在" << std::endl;
    } else {
        std::cout << "不存在" << std::endl;
    }
    
    // 输出: 存在
    return 0;
}

```
## ref

https://www.runoob.com/cplusplus/cpp-libs-vector.html

https://zh.cppreference.com/w/cpp/container/vector
