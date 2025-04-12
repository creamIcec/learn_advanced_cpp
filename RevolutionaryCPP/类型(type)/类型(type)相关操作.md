**R**un**T**ime **T**ype **I**dentification (RTTI) - 运行时类型识别

### 获取类型

**`typeid()`**

这个方法包含在头文件`<typeinfo>`中, 用于获取一个类的**内部类型表示**。内部类型表示是指，
这个方法获取到的名称不是我们想象中的和类名完全一致，而是随编译器而不同。大致上和类名接近，但不同编译器会添加一些前缀/后缀,  例如g++将会在类名之前添加类名长度信息:

```c++
Derived2* derived2_real_ptr = new Derived2();  
  
std::cout << "Type of derived2_real_ptr is " << typeid(*derived2_real_ptr).name() << std::endl;
```
将会输出
```text
Type of derived2_real_ptr is 8Derived2
```
其中"8"就是类名字符串的长度。

由于不能保证编译器相同(甚至不同版本的同一个编译器都可能输出不一样的结果), 也不能保证编译器输出的内容，**我们不能依赖该方法(typeid)来取得类的信息，该方法仅供调试使用**。

此外，在C++标准中，也没有规定编译器的具体实现应该是什么。也就是说，规范上编译器就可以只随便返回一个字符串。

### 转换类型

**`static_cast()`**
一个类型转换运算符，用于转换一个类型。在编译时完成转换，如转换失败会导致未定义行为。相较于普通的强制转换(使用`(type)`进行转换的)更加安全，因为不成功的转换将被明示，而不会被隐式转换。

1. 从一个基元类型转换到另一个基元类型
```c++
int i = 42;
float f = static_cast<float>(i); // Converts integer i to float f
```
2. 多态转换(向下转换)
```c++
class Base { /* ... */ };
class Derived : public Base { /* ... */ };

Base *bPtr = new Derived;
Derived *dPtr = static_cast<Derived *>(bPtr); // Converts Base pointer bPtr to Derived pointer dPtr
```

**`dynamic_cast()`**

也是一个运算符，用于(尝试)转换一个 **指向类对象(而非基元)** 的类型，基于多态。我们可以将一个类型为父类的指针downcast到子类。和`static_cast`主要的不同点是，`dynamic_cast`在运行时完成类型转换，且遇到不兼容的多态转换时会抛出错误；此外, `dynamic_cast`不能用于基元类型的转换。

假设我们有两个类, 一个`Base`和一个`Derived1`, `Derived1`继承`Base`。`Derived1`重写了`Base`中的一个`dummy`方法, `dummy`用于输出自己是哪个类。已知行为:

1. 从子类`dynamic_cast`到父类
```c++
Derived1* derived1_new_ptr = new Derived1();  
Base* base_new_ptr = dynamic_cast<Base*>(derived1_new_ptr);  
  
if(base_new_ptr){  
    std::cout << "Upcast to Base successful\n";  
}else{  
    std::cout << "Upcast to Base failed\n";  
}  
  
base_new_ptr->dummy();
```
虽然输出successful, 但`dummy()`输出的仍然是子类的内容。

2. 从父类`dynamic_cast`到子类
```c++
Base* base_ptr = new Derived1();  
  
Derived1* derived1_ptr = dynamic_cast<Derived1*>(base_ptr);  
if(derived1_ptr){  
    std::cout << "Downcast to Derived1 successful\n";  
}else{  
    std::cout << "Downcast to Derived1 failed\n";  
}  
  
derived1_ptr->dummy();
```
输出successful, 也输出子类的`dummy()`。

### 进一步研究

问题来了: 这两种不同结果的行为背后的**机制是什么?**

机制就是**多态**。虽然我们使用了`dynamic_cast`将类型强制转换到了父类，但由于`dummy()`是一个虚函数，调用哪个版本将在**运行时**决定。在运行时，父类的`dummy()`已经被子类的`dummy()`重写了, 而`derived1_new_ptr`本质上指向的是子类, 因此将调用子类的版本。如果我们想明确调用父类的`dummy()`，可以用作用域解析运算符`::`来实现:
```c++
base_new_ptr->Base::dummy();
```
通过在`dummy()`之前明确指出使用`Base`类，就可以顺利调用到父类的`dummy()`方法了。