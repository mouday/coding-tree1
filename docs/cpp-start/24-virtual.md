# 虚函数

示例

```shell
.
├── Person.cpp
├── Person.h
├── Student.cpp
├── Student.h
└── main.cpp
```

Person.h

```cpp
#include <string>

class Person
{
public:
    // 构造函数
    Person();

    // 析构函数
    virtual ~Person();

    // 虚函数
    virtual void eat();

    // 纯虚函数，给子类覆盖
    virtual void work() = 0;
};
```

Student.h

```cpp
#include "Person.h"

class Student : public Person
{
public:
    Student();
    ~Student();
    // 拷贝构造
    Student(const Student &other);
    void work();
    void eat();

private:
    int age;
    std::string name;
};

```

Person.cpp

```cpp
#include <cstdio>
#include "Person.h"

Person::Person()
{
    printf("Person::Person\n");
}

Person::~Person()
{
    printf("Person::~Person\n");
}

void Person::eat()
{
    printf("Person::eat\n");
}

```

Student.cpp

```cpp

#include "Student.h"
#include <cstdio>

Student::Student()
{
    printf("Student::Student\n");
}

Student::~Student()
{
    printf("Student::~Student\n");
}

Student::Student(const Student &other)
{
    printf("Student::Student copy\n");
    this->name = other.name;
    this->age = other.age;
}

void Student::work()
{
    printf("Student::work\n");
}

void Student::eat()
{
    printf("Student::eat\n");
}

```

main.cpp

多态示例

```cpp
#include "Student.h"

int main(int argc, char const *argv[])
{
    Person *person = new Student();
    person->work();
    person->eat();
    delete person;
    return 0;
}
```

输出

```shell
g++ main.cpp Person.cpp Student.cpp && ./a.out

Person::Person
Student::Student
Student::work
Student::eat
Student::~Student
Person::~Person
```

拷贝构造示例

```cpp
#include "Student.h"

int main(int argc, char const *argv[])
{
    Student student1;
    Student student2 = student1;
    return 0;
}
```

输出

```shell
g++ main.cpp Person.cpp Student.cpp && ./a.out
Person::Person
Student::Student
Person::Person
Student::Student copy
Student::~Student
Person::~Person
Student::~Student
Person::~Person
```
