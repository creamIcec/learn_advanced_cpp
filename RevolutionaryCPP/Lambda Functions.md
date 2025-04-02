C++中的 Lambda 概念和其他语言(Javascript, Java)类似，但表达方式上更加丰富。

### 基本用法

```c++
[捕获组](函数参数){函数体}
```

我们可以将一个 Lambda 函数传入可以使用的地方，例如`std::for_each`方法。
感觉上非常熟悉，是的，和其他语言类似, Lambda 一般就是一个简单的“处理器”，用于将传入的值进行处理并返回。它可以替代 for 循环。

C++中的 Lambda 和其他语言的主要区别是，多了一个`[捕获组]`。在捕获组中，我们给出的是参与这个函数的外部变量。比如，我们想设计一个 Lambda 函数，这个函数将一个`vector`中的每个元素乘以 2:

```c++
int multiplier = 2;

std::vector<int> v = {1,2,3,4,5};
std::for_each(v.begin(), v.end(), [multiplier](int x){
    std::cout << x << " multiple by " << multiplier << " is " << x * multiplier << std::endl;
});

return 0;
```

### 用法解释

这里面，由于`for_each`方法需要一个`lambda`作为"处理器"，我们可以这样传入: 指定捕获组包含`multiplier`，这表示将有作用域内的其他变量参与这个函数；指定形参`int x`，这是`for_each`将会传入的每个 vector 元素; 最后，在函数体中输出每个`x`乘以 2 的值即可。

除此以外，我们还可以让编译器自动推断捕获的变量，使用`=`或者`&`替代`multiplier`，变成下面的语句:

```c++
std::for_each(v.begin(), v.end(), [=](int x){/*....*/});
```

如果使用`=`, 表示函数内使用的外部变量是**按值传递的**, 如果使用`&`，则表示**按引用传递**。

这两种写法都有一个共同的好处，比其他语言提供了更大的灵活性：

> 我们可以人工指定函数内要用到的外部变量，虽然更加麻烦，不过可以避免不必要的内部修改。

同时需要明确的是，**只有在按引用传递**时，才能在函数内修改捕获的变量的值，否则不能修改。这一点也非常好理解，我们传入的是地址，也就是上下文无关了，函数内处于一个“全局访问地址”的状态，因此修改的值将直接作用到变量上；如果我们传入的是值，则表示我们只能接触到传入的那个数值，因此无法修改外部的变量。

### 进一步研究

#### 为什么需要多一个"捕获组"的概念?

Lambda可以作为一个对象在不同作用域之间传递。由于 C++的内存管理机制(在[异常处理(Exception Handling)](<异常处理(Exception%20Handling).md>)中也有所涉及)，在代码退出一个作用域之后，作用域中的栈上分配的变量将被释放，此时如果在其他作用域中使用Lambda，如果不适用捕获机制将变量"固定"，Lambda中对这些变量的访问将出现不确定的行为。这对于内存不安全的语言来说是致命的。

因此，使用捕获组将一个Lambda中需要用到的外部变量固定，就不会出现越界访问问题。

#### 可以用函数指针代替 Lambda 嘛?

当然可以，下面的都是**功能上等价**的:

```c++
int multiplier = 2;

void multiply(int x){
    std::cout << x << " multiple by " << multiplier << " is " << x * multiplier << std::endl;
};

int main(){
    std::vector<int> v = {1,2,3,4,5};
    void (*func1)(int x) = multiply;
    //可以这样写
    std::for_each(v.begin(), v.end(), multiply);
    //也可以这样写
    std::for_each(v.begin(), v.end(), func1);
    //也可以用lambda
    int _multiplier = 2;  //注意，由于作用域，我们不能捕获到全局的multiplier, 需要捕获同一个作用域中的变量，这里作为演示我们创建一个作用域内的变量。
    std::for_each(v.begin(), v.end(), [_multiplier](int x){
	    std::cout << x << " multiple by " << _multiplier << " is " << x * _multiplier << std::endl;
    });
    return 0;
}
```

其中, `std::for_each`中的`multiply`可以和`func1`互换。这也说明了一件事：

> _在 C++中，函数就是一个地址的代号，也就是一个指针。_

但是需要注意，功能等价的意思是: 只是功能上等价，本质上 Lambda 并不是一个"函数"。
关于 Lambda 的类型，下面这个 Stackoverflow 的回答说明了问题:

[What is the type of lambda when deduced with "auto" in C++11?](https://stackoverflow.com/questions/7951377/what-is-the-type-of-lambda-when-deduced-with-auto-in-c11)

> The type of a lambda expression is unspecified.
>
> But they are generally mere syntactic sugar for functors. A lambda is translated directly into a functor. Anything inside the `[]` are turned into constructor parameters and members of the functor object, and the parameters inside `()` are turned into parameters for the functor's `operator()`.
>
> A lambda which captures no variables (nothing inside the `[]`'s) _can be converted_ into a function pointer (MSVC2010 doesn't support this, if that's your compiler, but this conversion is part of the standard).
>
> But the actual type of the lambda isn't a function pointer. It's some unspecified functor type.

其中指出，Lambda 是一种 Functor，本质上是一个**仅包含一个 operator()函数的结构体**。这说明 Lambda 其实是*语法糖*。捕获组传入的参数将作为这个结构体的构造函数的参数，而 Lambda 函数的参数就是这个结构体中的`operator()`的参数。
