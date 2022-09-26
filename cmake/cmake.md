# CMake 随记

## [CMake Cookbook](https://www.bookset.io/read/CMake-Cookbook/SUMMARY.md)

CMake Cookbook 是系统性的书籍，翻译版。

## CMake 生成器表达式

生成器表达式类似于三元运算符 `if(true) ? "true" : "false"`，多用于条件判断。

[官方文档](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html)

[知乎这篇博客](https://zhuanlan.zhihu.com/p/437404485)讲解可以看下。

SigSlot 这个库中给出了一个 生成器表达式的典型用法：
[SigSlotUtils](https://github.com/palacaze/sigslot/blob/f8c275df5037bfb6d57c74168634b65fdbfb44f3/cmake/SigslotUtils.cmake)。
