
### Type Traits: 探查类型的特征

#### Type Traits是什么?

这里的“特征”二字，比起“具有特殊的方面”而言，更接近机器学习中的“特征”。他是一个类型的各个特性:

- 这个类型是数字, 指针，函数或是其他?
- 这个类型有构造函数吗？可以复制吗？
- 这个类型会抛出错误吗？

Type Traits广泛用于**C++类型元编程**中，通常用于**判断类型**和**根据类型的特征不同调整编译过程**。

#### 判断类型

我们找到了一些简单的例子，比如下面的代码, 用于判断一个变量是否是指针:

```c++
#include <type_traits>  
#include <iostream>  
  
int main(){  
    int a = 10;  
    int* a_ptr = &a;  
  
    std::cout << "Is 'a' a pointer? " << std::boolalpha << std::is_pointer<decltype(a)>::value << std::endl;  
    std::cout << "Is 'a_ptr' a pointer? " << std::boolalpha << std::is_pointer<decltype(a_ptr)>::value << std::endl;  
  
    return 0;  
}
```

> `std::boolalpha`转换标志用于将`bool`类型的输出变成字面上的`true/false`，而不是直接输出`1/0`，便于我们阅读。相应地可以用`std::noboolalpha`关闭转换。

在上面的例子中，我们导入了头文件`type_traits`，用于类型特征获取，然后使用其中的`std::is_pointer<T>::value`用来判断这个类型是不是指针。**std::is_pointer\<T\>::value**中的`T`表示一个类型，在这里我们传入`decltype(type)`来获取目标的类型。

#### decltype是什么?

`decltype(v)`表示一个**和`v`一致的类型**。当我们不知道`v`的类型，但又想使用这个类型时，就可以使用`decltype(v)`。

`decltype`和`auto`的区别在于, `auto`用于创建一个指向已有变量的、自动推断类型的指针，而`decltype(v)`用于声明*另一个*和`v`类型一样的变量。

比如下面的例子，我们从一个库函数`lib_func`中得到了一个变量，但我们不知道它的类型或类型符号过于复杂，我们可以使用`auto`来指代:

```c++
auto result_ptr = lib_func();
```

接下来，我们想将这个`result_ptr`指向的对象/值**复制**一份到某个容器中备用(比如缓存), 我们就可以使用`decltype`(为了简化说明问题，这里假设`lib_func`返回的对象中有一个`copy`方法，可以取得这个对象的一个副本:

```c++
decltype(*resuplt_ptr) = result_ptr->copy();
```

这应该说明了`decltype`和`auto`的区别以及各自的用途。

#### 其他常用的Type Traits

除了判断是否是指针以外，还有其他常用的Type Traits, 具体可以查看下面的网页:

[cplusplus.com | Type Traits](https://cplusplus.com/reference/type_traits/)

下面列举一些常用的:

- `std::is_arithmetic`: 检查是否是算术类型。
> 关于什么是算数类型，下面的链接直白地说明了问题: [Arithmetic Types in C++](https://biowpn.github.io/bioweapon/2024/08/29/arithmetic-types-in-cpp.html)
- `std::is_function`: 检查是否是函数类型。
- `std::conditional<类型检查表达式, A, B>::value`: 如果给定的类型检查表达式结果为真(true), 则返回`A`类型，否则返回`B`类型。
- `std::enable_if<类型检查表达式>`: 同`template`的使用方式，在一个模板类/函数上使用，如果给定的类型检查表达式结果为真, 则使用配套的`template`指定的类型。大多数情况下用于启用对某个函数/类的编译:。例如:
```c++
#include <iostream>
#include <type_traits>

template <typename T>
typename std::enable_if<std::is_arithmetic<T>::value, T>::type find_max(T a, T b) {
    return a > b ? a : b;
}

int main() {
    int max = find_max(10, 20);
    std::cout << "Max: " << max << '\n';

    return 0;
}
```

我们拆开来理解这段代码:

`template <typename T>` 定义了一个模板类型`T`;
`typename`用于指定一个类型定义表达式作为`find_max`函数的返回类型;
`std::enable_if`是我们的Type Trait, 用于根据其中的类型条件表达式决定是否启用这个函数的编译;
`std::is_arithmetic<T>`是条件，表示`T`应该是一个算数类型;
`std::is_arithmetic<T>::value, T`是整个表达式，表示当`T`是一个算数类型时，启用`T`这个类型的`find_max`函数的编译;
后缀的`::type`表示获取到最终敲定的类型，作为`find_max`函数的返回类型。之后, `find_max`的形参类型`T`就被限制在算数类型范围内了。此时调用`find_max`函数并传入两个数字，则可以得到两个数中的较大者。

### 深入研究

#### 我们为什么需要Type Traits?

下面这篇帖子说明了一个常见的用法, 比较两个数是否相等，需要分成整数和浮点数的情况:

>In C++ traits are compile time things. I recently had to do some C#, so I had your problem in the other direction. A generic in C# is a function that gets created without knowing how it gets called. In C++, a template is a complete compile time thing that cannot exist without its caller.
>
>A good example to show this would be the equal function which can use a type trait:
```c++
template <typename T>
bool is_eq(T&&a, T&&b)
{
     if constexpr (std::is_floating_point_v<T>)
           return (a > b-eps && a < b+eps) || (a > b+eps && a < b-eps);
     else
           return a == b;
}
```

>This nicely works with all types, doubles, std::string, std::vector ... due to the constexpr in it. Though if you would remove the constexpr and make it a runtime if, you'll get a compilation failure when using this function with std::string or other types that don't support operator+ with eps.

以及下面的帖子, 解释了标准库中的`std::string`的原理，并表示我们可以利用Type Traits来自定义一个`string`类型:

>They allow for compile-time customisation of the behaviours of a class. Let's take a look at a common example - `std::string`. You may already know this, but `std::string` isn't a class in and of itself, it's actually an alias for type `std::basic_string<char, std::char_traits<char>>`. Note the second parameter there - a traits class to determine how to process characters. The `std::basic_string` template defers to its traits to determine how to do certain things, and this can be customised.
>
>The quintessential example of this is making a case-insensitive string. You can make your own `ci_traits` class fairly easily which exposes the same interface as `std::char_traits` but performs comparison in a case-insensitive manner (left as an exercise for the reader); and then all you need to do is `using ci_string = std::basic_string<char, ci_traits>` and suddenly you can create `ci_string` instances which have all the same interface as your `std::string` but which will perform case-insensitive comparison. This can extend even further to `std::string_view` which allow you to perform custom comparisons of views of strings even if the underlying string object is a different type. All this done by customising 4 simple functions in a traits class; not reinventing the string or string_view wheel.
>
>There are plenty of other situations, too. There are also the type traits, which allow you to make compile time decisions about the properties of a type (e.g. if it is integral do A, if floating point do B, if a pointer do C) and are the backbone of template metaprogramming.
>
>But, good trait handling is done at compile time. It's a tool to help the compiler route your code to the calls you want it to do. While technically there's nothing stopping you from making a traits-style class which contains pure runtime data; it's almost always not what you want to do in C++.

其中提到，`std::string`不是一个类，而是一个类型的简写别名，原形是
```c++
std::basic_string<char, std::char_traits<char>>
```

这里面的`std::char_traits`就是一个Type Traits。这里它类似于一个flag, 用于指示如何处理这个字符串: 使用符合`char`特性的方法。文章中提到，我们可以自定义一个特性, 比如*大小写不敏感(ci_traits)*, 并使用

```c++
using ci_string = std::basic_string<char, ci_traits>
```
来使用这个特性, `ci_string`类型的字符串在处理时将忽略大小写(例如比较两个字符串是否相同时)。

#### Type Traits会影响性能嘛?

不会影响运行时性能，反而会优化执行路径，从而提高性能。Type Traits所做的所有事情都是在**编译时**完成，在编译完成后得到的优化代码将帮助在运行时选择合适的执行路径，从而提高性能。例如，我们可以通过Type Traits判断一个对象/变量是否是可以直接用`memcpy`来复制的, 如果可以，则直接使用`memcpy`，而不是都调用复制构造函数，这样性能会得到提升。