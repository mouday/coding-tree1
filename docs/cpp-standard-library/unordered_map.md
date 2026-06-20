# unordered_map

基于哈希表的键值对容器

unordered_map 不保证元素的排序，但通常提供更快的查找速度

基本语法：

```cpp
#include <unordered_map>

// key_type 是键的类型。
// value_type 是值的类型。
std::unordered_map<key_type, value_type> map_name;
```

构造函数

```cpp
// 默认构造
std::unordered_map<int, std::string> myMap;

// 构造并初始化
std::unordered_map<int, std::string> myMap = {{1, "one"}, {2, "two"}};

// 构造并指定初始容量
std::unordered_map<int, std::string> myMap(10);

// 构造并复制另一个 unordered_map
std::unordered_map<int, std::string> anotherMap = myMap;
```

基本操作

```cpp
// 插入元素:
myMap.insert({3, "three"});

// 访问元素:
std::string value = myMap[1]; // 获取键为1的值

// 删除元素:
myMap.erase(1); // 删除键为1的元素

// 查找元素:
auto it = myMap.find(2); // 查找键为2的元素
if (it != myMap.end()) {
    std::cout << "Found: " << it->second << std::endl;
}
```

示例

```cpp

#include <unordered_map>
#include <iostream>

int main(int argc, char const *argv[])
{
    std::unordered_map<std::string, int> map;

    map["Alice"] = 30;
    map["Bob"] = 25;
    map["Charlie"] = 35;

    for (std::unordered_map<std::string, int>::iterator it = map.begin();
         it != map.end(); ++it)
    {
        std::cout << it->first << " => " << it->second << std::endl;
    }

    return 0;
}

```

输出

```shell
Charlie => 35
Bob => 25
Alice => 30
```

## insert

插入元素

```cpp
#include <iostream>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, std::string> map;

    map.insert({"red", "红色"});

    for(const std::pair<const std::string, std::string>& entity : map) {
        std::cout << entity.first << ": " << entity.second << std::endl;
    }
    // red: 红色
}

```

## erase

擦除元素

```cpp
#include <iostream>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, std::string> map= {
        {"red", "红色"},
        {"green", "绿色"},
        {"blue", "蓝色"},
    };

    map.erase("red");

    for(const std::pair<const std::string, std::string>& entity : map) {
        std::cout << entity.first << ": " << entity.second << std::endl;
    }
   // blue: 蓝色
   // green: 绿色
}

```

## clear

清除内容

```cpp
#include <iostream>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, std::string> map= {
        {"red", "红色"},
        {"green", "绿色"},
        {"blue", "蓝色"},
    };

    map.clear();

    for(const std::pair<const std::string, std::string>& entity : map) {
        std::cout << entity.first << ": " << entity.second << std::endl;
    }
}

```

## 取值

```cpp
#include <iostream>
#include <unordered_map>

int main() {
    std::unordered_map<std::string,std::string> map = {
        {"red", "红色"},
        {"green", "绿色"},
        {"blue", "蓝色"},
    };

    // 取值
    std::string value1 = map.at("red");
    std::cout << value1 << std::endl; // 红色

    // 取值
    std::string value2 = map["blue"];
    std::cout << value2 << std::endl; // 蓝色
}

```

## for-in

```cpp
#include <iostream>
#include <unordered_map>

int main() {
    std::unordered_map<std::string,std::string> map = {
        {"red", "红色"},
        {"green", "绿色"},
        {"blue", "蓝色"},
    };

    for(const std::pair<const std::string, std::string>& entity : map) {
        std::cout << entity.first << ": " << entity.second << std::endl;
    }
}

```

## find

寻找带有特定键的元素

```cpp
#include <iostream>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, std::string> map = {
        {"red", "红色"},
        {"green", "绿色"},
        {"blue", "蓝色"},
    };

    auto iterator = map.find("red");

    if (iterator != map.end()) {
        std::cout << iterator->first << ": " << iterator->second << std::endl;
    }
    // red: 红色
}

```

## count

`map.count` 成员检查

```cpp
#include <iostream>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, std::string> map= {
        {"red", "红色"},
        {"green", "绿色"},
        {"blue", "蓝色"},
    };

    // 查询 red
    if(map.count("red")) {
        std::cout << "存在" << std::endl;
    } else {
        std::cout << "不存在" << std::endl;
    }
    // 存在

    // 查询 yellow
    if(map.count("yellow")) {
        std::cout << "存在" << std::endl;
    } else {
        std::cout << "不存在" << std::endl;
    }
    // 不存在
}
```

## find

`map.find` 成员检查

```cpp
#include <iostream>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, std::string> map = {
        {"red", "红色"},
        {"green", "绿色"},
        {"blue", "蓝色"},
    };

    if (map.find("red") != map.end()) {
        std::cout << "存在" << std::endl;
    }
    // 输出: 存在

    return 0;
}
```
## ref

https://www.runoob.com/cplusplus/cpp-libs-unordered_map.html

https://zh.cppreference.com/w/cpp/container/unordered_map
