
我们在进行模板编程时，可以使用类似“重载”的概念对模板类/函数进行特化。特化是指将通用的模板函数分化出一个适用于特定参数类型的版本。

### 模板特化有什么用?

正如我们在[C++ Type Traits](<C++ Type Traits.md>)中最后一段提到的, 类似于`Type Traits`进行判断, 特化也可以类似地指示“这个函数只用于某些类型”，从而对这个类型进行专门处理，可以提升性能和代码可读性。

### 如何使用模板特化

模板特化使用非常接近于重载。

#### 函数的特化

要特化一个包含模板参数的函数，只需下面的步骤:

1. 创建一个函数名称和返回值和模板函数相同的函数
2. 在函数前添加`template <>`语句
3. 修改参数中的模板为特定的类型
4. (可选，但特化要起作用的话我们必须要重写吧?)重写函数内容

例如，如下是一个模板函数:

```c++
template <typename T>  
void printData(const T &data){  
    cout << "Data printed from general template method: " << data << endl;  
}
```

作为例子，我们对类型`const char*`，也就是指向字符串常量的指针类型进行特化:

```c++
template <>  
void printData(const char* const &data){  
    cout << "Data printed from char* specific method: " << data << endl;  
}
```

如上所说的，只需要修改参数为具体类型和函数体即可。

##### 多参数的函数可以用吗?

当然可以啦! 下面是一个例子:

```c++
template <typename U>  
typename std::enable_if<std::is_arithmetic<U>::value, U>::type  
dataAdd(const U &data, int add){  
    cout << "Data add by " << add << " in general dataAdd function" << endl;  
    return data + add;  
}  
  
template <>  
int dataAdd(const int &data, int add){  
    cout << "Data add by " << add << " in integer specific dataAdd function" << endl;  
    return data + add;  
}
```

这两个函数的作用是返回`data`与`add`两个数的和。在这里，我们使用了一些`Type Trait`，用来保证`U`类型是一个数字类型。(参考[C++ Type Traits](<C++ Type Traits.md>))。

定义好之后，使用下面的语句调用:

```c++
int num1 = 2;  
int add = 3;  
  
cout << dataAdd(num1, add) << endl;  
  
double num2 = 2.3;  
cout << dataAdd(num2, add) << endl;
```

运行得到输出:

```text
Data add by 3 in integer specific dataAdd function
5
Data add by 3 in general dataAdd function
5.3
```

可见，模板特化对于多参数的函数仍然是有效的，编译器会通过某些手段来确定哪个参数被特化了(可能就是通过参数名称? 因为一个函数中一个参数名称只能出现一次)。


#### 类的特化

类的特化和函数特化工作过程类似，下面以一个例子直接一以概之，不再赘述:

```c++
#include <iostream>

template <typename K, typename V>
class MyPair {
public:
    MyPair(K k, V v) : key(k), value(v) {}

    void print() const {
        std::cout << "General template: key = " << key << ", value = " << value << '\n';
    }

private:
    K key;
    V value;
};

template <typename T>
class MyPair<T, int> {
public:
    MyPair(T k, int v) : key(k), value(v) {}

    void print() const {
        std::cout << "Partial specialization for int values: key = " << key
                  << ", value = " << value << '\n';
    }

private:
    T key;
    int value;
};

int main() {
    MyPair<double, std::string> p1(3.2, "example");
    MyPair<char, int> p2('A', 65);
    p1.print(); // General template: key = 3.2, value = example
    p2.print(); // Partial specialization for int values: key = A, value = 65
}
```

需要注意的是，这里`MyPair`只特化了两个模板中的一个(`V`), 变成了`int`。

#### 有什么非常有用的例子吗?

我们稍后回答这个问题，先说一个相关的发现。经过一番研究(~~翻箱倒柜~~), 我们得到了一个结论:

> 一门编程语言中应该没有重复特性。本文截至目前为止，我们对于模板特化的理解好像和`Type Traits`没啥区别? 都是针对特定的参数类型进行特殊处理。但是，模板特化最有用的情况其实和**重载高度关联**。

想想我们平时使用函数重载的时候。当重载版本过多的时候，我们是不是自然而然就会想到模板？
(也就是说，大多数重载函数只是为了适应不同类型的参数，但处理逻辑完全相同)

所以，模板特化就是相反的这个过程: **当模板可以用于大多数类型，但少数类型需要特殊处理时，我们就会想到特化**。

> 重载版本过多 -> 使用模板
> 模板需要处理一些特殊情况 -> 使用特化

例如，我们需要编写一个针对不同类型打印信息的模板函数:

```c++

template <typename T>
void printInfo(const T& data) {
    std::cout << "Generic print: " << data << std::endl;
}

template <>
void printInfo(const std::string& data) {
    std::cout << "String specific print: Length = " << data.length() << ", Value = " << data << std::endl;
}

int main() {
    int number = 42;
    std::string text = "Hello, world!";

    printInfo(number);
    printInfo(text);

    return 0;
}
```

大多数情况，我们只需要将数据本身打印出来就好了(数值型等, 因为数字本身就是意义)；但如果对于复杂类型，我们需要打印额外的信息呢？例如字符串，打印其本身可能还不够，我们还可以直接给出长度信息，方便用户。于是乎，针对`std::string`类型进行特化，得到上面的第二个函数。

### 进一步研究

#### 特化时, 函数前的`template <>`是什么意思?

这就回到了`template`关键字的本质含义了。

以下来自[templates|Templates - cppreference.com](https://en.cppreference.com/w/cpp/language/templates)

>When template arguments are provided, or, for [function](https://en.cppreference.com/w/cpp/language/function_template#Template_argument_deduction "cpp/language/function template") and [class](https://en.cppreference.com/w/cpp/language/class_template_argument_deduction "cpp/language/class template argument deduction")(since C++17) templates only, deduced, they are substituted for the template parameters to obtain a _specialization_ of the template, that is, a specific type or a specific function lvalue.

其中提到，模板将被具体化，根据使用情况，编译生成对应的具体代码。例如，下面这篇([How does the compilation of templates work?](https://stackoverflow.com/questions/19798325/how-does-the-compilation-of-templates-work))StackOverflow的回答:

>The compiler _**generates**_ the code for the specific types given in the template class instantiation.
> If you have for instance a template class declaration as

```cpp
template<typename T>
class Foo
{
public:
     T& bar()
     {
         return subject; 
     }
private:
     T subject;
};
```

> as soon you have for example the following instantiations

```cpp
Foo<int> fooInt;
Foo<double> fooDouble;
```

> these will _**effectively generate**_ the same linkable code as you would have defined classes like

```cpp
class FooInt
{
public:
     int& bar()
     {
         return subject; 
     }
private:
     int subject;
}
```

> and

```cpp
class FooDouble
{
public:
     double& bar()
     {
         return subject; 
     }
private:
     double subject;
}
```

> and instantiate the variables like

```cpp
FooInt fooInt;
FooDouble fooDouble;
```

这些都说明了, `template`关键字拥有以下特征:

- 在编译时起作用
- 将通用的模板代码根据使用情况生成对应的特化代码
- `<>`内表示模板参数，名称遵守c++变量命名规范，个数表示需要用到的模板个数

也就是说, `<>`里面为空表示的是没有模板，所有参数都被特化了。如果将`template<>`语句在不同函数前的应用视为*平行*, 而不是先有`template<T>`再用`template<>`来特化这样的*顺序*关系，会更好理解一些:

> 这就表示，同一名称的多个函数前面如果使用了`template`模板，则每个函数处理*一种情况*, 这种情况可能是通用情况，也可以是特化情况，他们是*分工*关系，而不是*因果*关系。

我们顺便解决了一个问题，编译器通过**函数名称**来找到对应的模板函数的。


