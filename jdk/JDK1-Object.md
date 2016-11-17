# JDK源码分析1-Object
源码分析基于JDK1.8

## native 关键字

关于这个类没什么可以说的，因为其方法实现并不在这个类中。因为用native修饰的方法的实现其实是用别的语言实现的，一般是用c/c++实现的，实现方法就是JNI(Java native interface)。关于这个可以看看我们主线的salut-SDK模块，就是java来调用底层c实现的方法，c语言只需要提供动态库和头文件即可。
用native声明方法无非是为了java的跨平台。因为不同的操作系统对内存的拷贝、I/O访问，线程实现都是不同的。看看Object中的方法几乎都是与这些相关。
所以为了跨平台，那么就需要屏蔽这些差异，其实这些实现都是在JVM中。

现实native的方法的一般步骤是：
* 在Java中声明native()方法，然后编译；
* 用javah产生一个.h文件；
* 写一个.cpp文件实现native导出方法，其中需要包含第二步产生的.h文件（注意其中又包含了JDK带的jni.h文件）；
* 将第三步的.cpp文件编译成动态链接库文件；
* 在Java中用System.loadLibrary()方法加载第四步产生的动态链接库文件，这个native()方法就可以在Java中被访问了。
写了demo代码 [jni](https://github.com/anancds/learn-project/tree/master/jni-learn/src/main/java/com/cds/jni)
