# RAII

重载赋值运算符

```cpp

#include <stdlib.h>
#include <string.h>

class Person
{
public:
    char *data = nullptr;
    Person();
    // 赋值运算符
    Person &operator=(const Person &other);
    ~Person();
};

Person::Person()
{
    this->data = (char *)malloc(sizeof(char) * 10);
}

Person::~Person()
{
    if (this->data != nullptr)
    {
        free(this->data);
        this->data = nullptr;
    }
}

Person &Person::operator=(const Person &other)
{
    memcpy(this->data, other.data, 10);
    return *this;
}

int main(int argc, char const *argv[])
{
    Person person1;
    Person person2;
    person2 = person1;
    return 0;
}
```
