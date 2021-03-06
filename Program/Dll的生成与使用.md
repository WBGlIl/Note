## 0x00 动态链接库(dll)概述

dll可执行文件，既不能直接运行，也不能接收消息，它们是一些独立的文件。其中包含能被可执行程序或其他dll调用来完成某项工作的函数，只有在其他模块调用dll中的函数时，dll才发挥作用。 
在实际编程中，我们可以把完成某项功能的函数放在一个动态链接库里，然后提供给其他程序调用。

### 静态库和动态库
* 静态库：函数和数据被编译进一个二进制文件（扩展名通常为.lib）,在使用静态库的情况下，在编译链接可执行文件时，链接器从静态库中复制这些函数和数据，并把它们和应用程序的其他模块组合起来创建最终的可执行文件（.exe）。当发布产品时，只需要发布这个可执行文件，并不需要发布被使用的静态库。
* 动态库：在使用动态库时，往往提供两个文件：一个引入库（.lib，非必须）和一个.dll文件。这里的引入库和静态库文件虽然扩展名都是.lib，但是有着本质上的区别，对于一个动态链接库来说，其引入库文件包含该动态库导出的函数和变量的符号名，而.dll文件包含该动态库实际的函数和数据。

### 使用动态链接库的好处

* 可以使用多种编程语言编写：比如我们可以用VC++编写dll，然后在VB编写的程序中调用它。
* 增强产品功能：可以通过开发新的dll取代产品原有的dll，达到增强产品性能的目的。比如我们看到很多产品踢动了界面插件功能，允许用户动态地更换程序的界面，这就可以通过更换界面dll来实现。
* 提供二次开发的平台：用户可以单独利用dll调用其中实现的功能，来完成其他应用，实现二次开发。
* 节省内存：如果多个应用程序使用同一个dll，该dll的页面只需要存入内存一次，所有的应用程序都可以共享它的页面，从而节省内存。
*

## 0x01 dll的创建

### 使用_declspec(dllexport)创建dll
为了让dll导出函数，需要在每一个需要被导出的函数前面加上标识符：__declspec(dllexport)。 
利用Build命令生成Dll1动态链接库，这时在Dll1/Debug目录下就会生成.dll文件和.lib文件，这两个文件即为所需的动态链接库的文件。

```c
__declspec(dllexport) int add(int a, int b){
    return a + b;
}

```
### 使用模块定义（.def）文件创建dll
使用def文件创建dll的话就不再需要__declspec(dllexport)，因此将代码写成最原始的样子:

```c
int add(int a, int b){
    return a + b;
}
```
同时为工程创建一个后缀名为.def的文件，并添加进工程，编辑其内容为：

```nxml
EXPORTS
add  @编号 
add  @编号 NONAME
```
EXPORTS语句用于表明dll将要导出的函数，@后接导出的序号，也可以隐藏函数名导出。 

## 0x02 dll的使用

### 隐式链接方式加载dll
将生成的lib文件放入程序根目录
在程序头文件加上`#pragma comment(lib,"Dll1.lib")`，然后在cpp中声明外部函数：`_declspec(dllimport) int add(int a, int b);`这样我们就可以使用这两个函数了。

### 显示加载方式加载dll
另一种是通过LoadLiabrary函数显示加载dll。代码如下。需要注意的是这时候我们不再需要注册.lib文件，也不需要声明外部函数。只要在需要使用的地方调用dll文件即可。

```c
HINSTANCE hInst = LoadLibrary("Dll1.dll");
typedef int(*ADD)(int a, int b);
ADD add = (ADD)GetProcAddress(hInst, "add");
```

### 两种加载方式对比

* 隐式链接方式实现简单，一开始就把dll加载进来，在需要调用的时候直接调用即可。但是如果程序要访问十多个dll，如果都采用隐式链接方式加载他们的话，在该程序启动时，这些dll都需要被加载到内存中，并映射到调用进程的地址空间，这样将加大程序的启动时间。而且一般来说，在程序运行过程中只是在某个条件满足的情况下才需要访问某个dll中的函数，如在上述例子中，我只有在点击按钮时才需要访问dll，其他情况下并不需要访问。这样如果所有dll都被加载到内存中，资源浪费是比较严重的。
* 显示加载的方法则可以解决上述问题，dll只有在需要用到的时候才会被加载到内存中。另外，其实采用隐式链接方式访问dll时，在程序启动时也是通过调用LoadLibrary函数加载该进程需要的动态链接库的。






















