---
title: 在Windows平台下编译并运行TensorFlow C++项目（支持GPU）
date: 2018-09-11 23:48:52
tags: [TensorFlow,Windows,配置]
categories: [工欲善其事]
---

由于工作需要，最近我尝试了在Windows平台上编译并运行TensorFlow的C++项目。整个过程非常痛苦，因为网上相关的信息实在是少之又少，几乎所有关于C++的编译都是在Linux平台下通过baze实现的。非常感谢这篇[tutorial](https://joe-antognini.github.io/machine-learning/build-windows-tf) 以及 这篇 [tutorial](https://joe-antognini.github.io/machine-learning/windows-tf-project)。通过结合上面两篇博文以及TF的官方[CMake介绍](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/contrib/cmake/README.md)，我终于成功实现了在Windows平台下对TensorFlow项目的编译和运行。(趁热赶紧回馈社会...

<!-- more -->


##  环境介绍
应client的要求，我们需要在Windows 7 pro的环境下提供一个基于 TensorFlow 做 Inference 的 DLL，以供他们在VS2015中调用。

所以，本文介绍的编译及运行的方法仅保证在以下环境下能够成功：

+ OS Platform: Windows 7 Pro
+ Tensorflow 1.5
+ Visual Studio 2015

## Build TensorFlow
如果你有过bazel build的经验，你应该知道：要想构建C++项目，首先需要build TensorFlow。这里我们将用 vs-2015 来完成编译。

### Pre-requirement
首先，我们需要完成一些先期工作，确保以下已存在于Windows中：

+ tensorflow 1.5. 
	+ git clone https://github.com/tensorflow/tensorflow.git v1.5.0
	+ cd v1.5.0
	+ git checkout tags/v1.5.0
+ CMake
+ SWIG (3.0.12)
+ python3.5
+ Visual Studio 2015

为了方便，在具体操作过程中，我将TensorFlow和swig放入了`c:\%USERNAME%\bin\`文件夹下，以下都将使用该路径

### CMake 生成 Visual Studio project files

所有即将编译的TensorFlow library都在TensorFlow repository下的`cmake`目录下。首先我们要在`cmake`文件夹下创建一个separate的`build`文件夹。(以下所有命令行操作都是在Command Prompt下进行的)

```
C:\...> cd tensorflow\tensorflow\contrib\cmake
C:\...> mkdir build
C:\...> cd build
```

然后我们使用Command Prompt来运行CMake：（路径根据实际情况可能需要修改）

```
cmake .. -A x64 -DCMAKE_BUILD_TYPE=Release ^
-DSWIG_EXECUTABLE=C:\Users\%USERNAME%\bin\swig\swigwin-3.0.12\swig.exe ^
-DPYTHON_EXECUTABLE=C:\Users\%USERNAME%\Anaconda3\python.exe ^
-DPYTHON_LIBRARIES=C:\Users\%USERNAME%\Anaconda3\libs\python35.lib
```

如果你和我一样也是需要一个GPU support的program，需要在加上：（别忘记在前面的命令行后面加上`^`，cudnn路径根据实际情况调整）

```
-Dtensorflow_ENABLE_GPU=ON ^
-DCUDNN_HOME="C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.0"
```

为了节省时间，我们最后避免一些不必要的文件的生成，加入：

```
-Dtensorflow_BUILD_PYTHON_BINDINGS=OFF ^
-Dtensorflow_ENABLE_GRPC_SUPPORT=OFF
```

最重要的是，如果我们想得到shared library，**一定要加**:
```
-Dtensorflow_BUILD_SHARED_LIB=ON ^
-DCUDA_HOST_COMPILER="C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin\amd64\cl.exe"
```

这样在CMake成功之后，会生成很多Visual Studio project file

### Build TensorFlow in VIsual Studio
点击`tensorflow.sln`文件，会打开vs工程。很重要的是：一定要在Configuration Manager中（或者在上边栏）选择**64-bit build**以及**Release**。

然后在 solution 中选择`ALL_BUILD`进行build，（接下来能做的就只能双手合十祈祷不报错了

这一步一般要花费2-3小时左右（视机器而定），一定要保证有至少12GB的内存，不然会在编译一小时左右报一个 `out of space` 的 error

如果一切顺利，那么在编译成功后我们得到一系列的library，其中最重要的是在路径 `tensorflow\contrib\cmake\build\Release` 下的 `tensorflow.dll` 和 `tensorflow.lib`

## Build Projects Depending on TensorFlow
在成功编译得到 `tensorflow.dll` 和 `tensorflow.lib` 后，我们就可以构建基于TensorFlow的程序了！

首先要注意的我们的程序一定要是**x64**下的**Release**！（和编译TensorFlow时对应）

在构建程序的时候，有一点一定要记住，一定要将下列code写入你的头文件中：

```cpp
#define COMPILER_MSVC
#define NOMINMAX
```

否则会报一个 ` "You must define TF_LIB_GTL_ALIGNED_CHAR_ARRAY for your compiler." `的error

接来下需要修改下列设置:

### Additional Include Directories
在c++程序中，compiler需要所有必要的header files，所以在我们这个基于TensorFlow的程序中，需要额外引入directories以避免compile error：
```
C:\Users\%USERNAME%\bin\v1.5.0
C:\Users\%USERNAME%\bin\v1.5.0\tensorflow\contrib\cmake\build
C:\Users\%USERNAME%\bin\v1.5.0\tensorflow\contrib\cmake\build\external\eigen_archive
C:\Users\%USERNAME%\bin\v1.5.0\third_party\eigen3
C:\Users\%USERNAME%\bin\v1.5.0\tensorflow\contrib\cmake\build\protobuf\src\protobuf\src
C:\Users\%USERNAME%\bin\v1.5.0\tensorflow\contrib\cmake\build\nsync\src\nsync\public
```



### ADD tensorflow.lib 

将路径在`tensorflow\contrib\cmake\build\Release` 下的 `tensorflow.lib` 添加到当前项目下，确保不会有compile的linking error。


根据不同项目，这里可能需要额外的lib文件：我就添加了如下两个static library：

+ `tf_protos_cc.lib` in `tensorflow\contrib\cmake\build\Release` 
+ `libprotobuf.lib` in `tensorflow\contrib\cmake\build\protobuf\src\protobuf\Release`


### ADD tensorflow.dll
follow 上述步骤后，我们就可以成功编译项目了。但是先别高兴太早，好没结束呢，现在run生成的exe文件会直接报错显示缺失`tensorflow.dll`文件，这时就需要我们将路径在`tensorflow\contrib\cmake\build\Release` 的`tensorflow.dll`文件复制到和exe文件所在的文件夹中，即可成功运行

(可以用这篇 [tutorial](https://joe-antognini.github.io/machine-learning/windows-tf-project)中的示例代码进行验证)

Happy Coding!

（完）


