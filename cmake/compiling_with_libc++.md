# 使用 LLVM/libc++ 作为 std 库进行编译链接

在 CMakefiles.txt 中，需要指定以下几项：

+ 首先，需要指定 CXX 的编译器为 Clang++
+ 其次，需要指定 CMAKE_CXX_FLAGS 中含有 `-stdlib=libc++`
+ 再次，需要指定 EXE/SHARED/STATIC/(MODULE) 等的 Linker Flags 为 `-stdlib=libc++ -lc++abi`

如果已经 CMAKE 生成过，需要删除生成文件重新生成。

```text
set(CMAKE_C_COMPILER /usr/bin/clang-14)
set(CMAKE_CXX_COMPILER /usr/bin/clang++-14)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++ -D_LIBCPP_VERSION")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -stdlib=libc++ -lc++abi")
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -stdlib=libc++ -lc++abi")
set(CMAKE_STATIC_LINKER_FLAGS "${CMAKE_STATIC_LINKER_FLAGS} -stdlib=libc++ -lc++abi")

```