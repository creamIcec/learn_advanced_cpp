> *以下过程在`Windows`上进行, 后续将会尝试`Linux`。*
> 开始下面的内容前，确保我们已经执行过[准备工作](<准备工作-Windows上进行C++开发.md>)。


> 本篇参考了多个来源:
> https://medium.com/csmadeeasy/opencv-c-installation-on-windows-with-mingw-c0fc1499f39
> https://answers.opencv.org/question/189548/how-can-i-link-the-opencv-libraries-correctly/


## 第一步: 明确架构和平台
#### `OpenCV`官网有预构建包, 我们可以用吗?

根据官方的说法[Install OpenCV on Windows](https://docs.opencv.org/4.x/d3/d52/tutorial_windows_install.html), `OpenCV`预构建包仅用于`VC`。如果我们想要在现有工具链(`MingW`+`CMake`+`g++`)中使用, 就必须**从源代码构建**。

## 第二步: ~~使用预构建包~~/从源代码构建

#### 从官方发布下载`opencv`的`windows`源文件

[OpenCV Release](https://github.com/opencv/opencv/releases/tag/4.11.0)

下载完成后, 运行程序(这个程序是一个自解压程序), 指定一个路径进行解压。
#### 使用`cmake`构建源代码

进入到解压的目录(也可以是任何位置), 新建一个空文件夹(下文在`opencv`的解压目录中建立了一个`out`文件夹, 与解压得到的`sources`和`build`并列), 进入该文件夹, 输入下面的指令:

```bash
 cmake -DENABLE_PRECOMPILED_HEADERS=OFF -DCPU_DISPATCH="" -G "MinGW Makefiles" ../sources
```

#### 使用`mingw32-make`构建二进制

在刚刚的`out`文件夹(即`cmake`刚刚运行的文件夹)中, 运行指令:

```bash
mingw32-make install
```

这一步可能会花费较长的时间, 我们耐心等待, 吃个饭什么的(写到这里中午12:30了???)
😵

## 第三步: 在项目`cmake`文件中导入

下面是一个最小化的可以运行的配置:

*注: <>括起来的表示需要根据情况改写的实际路径*

```c++
cmake_minimum_required(VERSION 3.23)  
project(CPPRevolutionLibTests)  
set(CMAKE_CXX_STANDARD 14)  
  
set(OpenCV_Include_DIR "<上一个mingw32-make指令输出的安装位置>/include")  
include_directories(${OpenCV_Include_DIR})  
  
add_executable(CPPRevolutionLibTests main.cpp)  
  
find_package(OpenCV REQUIRED PATHS "<上一个mingw32-make指令输出的安装位置>" NO_DEFAULT_PATH COMPONENTS core imgproc highgui)  
target_link_libraries(CPPRevolutionLibTests opencv_core opencv_imgproc opencv_highgui)
```

简单概括，这些配置指明了查找头文件(`include_directories`)和链接库文件(`target_link_libraries`)的位置。
为了简化路径的编写，我们还使用了`find_package`命令从指定的路径自动查找到头文件名称等，在这里体现为用`core`等用户友好的名称来代替手动编写的路径。

## 第四步: 测试

经过以上的各种配置，我们终于可以尝试编写一个简单的`OpenCV`程序了(激动)。

下面是部分修改过的官方示例代码:

```c++
#include <iostream>  
#include <opencv2/core.hpp>  
#include <opencv2/highgui.hpp>  
#include <opencv2/imgcodecs.hpp>  
  
int main() {  
  
    using namespace cv;  //可能是我们第一次使用非std的命名空间
  
    std::string image_path = samples::findFile("starry_night.jpg");  
    Mat img = imread(image_path, IMREAD_COLOR);  
  
    if(img.empty()){  
        std::cout << "Unable to read image: " << image_path << std::endl;  
    }  
  
    imshow("Display window", img);  
    int k = waitKey(0);  //等待按键  
  
    if(k == 'q'){  
        destroyAllWindows();  
    }  
  
    std::cout << "OpenCV installed successfully." << std::endl;  
    return 0;  
}
```

这和`python`中的如下程序等价:

```python
import cv2 as cv
import sys

image_path = cv.samples.findFile("starry_night.jpg")
img = cv.imread()

if img is None:
    sys.exit("Unable to read image: " + image_path)

cv.imshow("Display window", img)
k = cv.waitKey(0)

if k == ord("q"):
	destroyAllWindows()

print("OpenCV installed successfully.")
```

**注意，官方的这个示例涉及到使用官方提供的示例图像文件，因此运行前，需要先将示例图像的路径添加到环境变量中, 变量名为`OPENCV_SAMPLES_DATA_PATH`, 路径为:
`<最开始解压opencv的路径>/sources/samples`。**

注意到，引用头文件的路径格式:

```c++
#include <opencv2/core.hpp>
```

这和安装路径`include`文件夹中的路径结构相同, 指向:

`<安装路径>/include/opencv2/core/core.hpp`

接下来，使用`MSYS2`工具链(和[准备工作](<准备工作-Windows上进行C++开发.md>)中的工具链相同)编译，链接并输出目标，得到`exe`程序。

**重要注意: 由于我们使用的方式是动态链接，我们需要指定程序运行时查找动态链接库(dll)的位置。为此，需要将dll所在路径添加到环境变量PATH中。**

接下来，运行程序，成功!

![Exit with Code 0xC0000094](../images/opencv_cpp_introduction.png)


上面就是我们第一次使用一个第三方库的整个流程。流程中充满了各种tricky的地方，很高兴我们一路过关斩将解决了所有问题！同时日后可能也会遇到更多问题，将在我们不断探索的过程中一点点完善对于第三方库使用的整体认知:)