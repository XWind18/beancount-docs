# Beancount C++ 版本：依赖项<a id="title"></a>

[<u>Martin Blais</u>](mailto:blais@furius.ca)，2020年6月

Beancount 将会用 C++ 重写，以下是我已经测试并且在长期维护中感到满意的一组依赖项：

## 基本环境<a id="base-environment"></a>

- [**<u>Bazel 构建</u>**](https://bazel.build/) ([<u>https://github.com/bazelbuild/bazel</u>](https://github.com/bazelbuild/bazel)): Google 的构建系统是我所知道的最稳定的构建方法，远优于 SCons，也肯定优于 CMake。它允许你通过显式引用其他存储库和特定提交修订的 git 仓库来固定一组特定的依赖项（包括未发布的），它有时带来的约束可能令人烦恼，但它能像其他构建系统所不能做到的那样，实现封闭的可重现构建。这最大限度地减少了意外情况，并希望减少平台依赖的可移植性问题。它还最大限度地减少了我们假设系统预先安装的软件包数量（例如，它会下载并编译自己的 Bison）。它运行快速，计算需要重新构建的最小测试和目标集，并且高度可配置。

> 选择 Bazel 的缺点与其他 Google 发行的开源项目相同：该产品的原始版本是内部使用的，因此有很多奇怪的特性需要处理（例如 //external，@bazel\_tools 仓库等），其中许多在公司外部文档不多，并且有大量未解决的问题。然而，在这个阶段，我已经设法 [<u>创建了一个工作构建</u>](https://github.com/beancount/beancount/tree/bazel)，包含本节描述的大部分依赖项。

- **[<u>C++14</u>](https://en.wikipedia.org/wiki/C%2B%2B14) 与 [<u>GCC</u>](https://gcc.gnu.org/) 和 [<u>Clang/LLVM</u>](https://clang.llvm.org/)**: 两个编译器都将得到支持。Clang 提供了更好的前端和标准库实现，但构建稍慢。GCC 更常见，但错误消息……好吧，我们都习惯了吧。尽管需要 C++14，我会避免使用语言的奇特功能（包括类）。可能会有关于 Windows 支持的问题。

- [**<u>Abseil-Cpp 基础库</u>**](https://abseil.io/) ([<u>https://github.com/abseil/abseil-cpp</u>](https://github.com/abseil/abseil-cpp)): 这个基础函数库来自 Google 自己的庞大代码库，经过无数次测试，非常稳定——这是 Google 产品运行的基础。它提供了一个非常稳定的 API（由于大量代码依赖于它，不太可能有大的变化），很好地补充了 stdc++，并且现有的接触面可能会保持静态。它比 Boost 更简单、更稳定，并且不提供我们不需要的众多库（另外，我喜欢 [<u>Titus'</u>](https://github.com/tituswinters) 的 C++ 方法）。它填补了许多在 Python 中免费提供但在 C++ 中渴望的基本字符串操作函数（例如 absl::StrCat）。

- **[<u>Google Test</u>](https://en.wikipedia.org/wiki/Google_Test)** ([<u>https://github.com/google/googletest</u>](https://github.com/google/googletest)): 这是我已经熟悉的广泛使用的 C++ 测试框架，支持匹配器和模拟。

## 数据表示<a id="data-representation"></a>

- [**<u>Protocol Buffers</u>**](https://developers.google.com/protocol-buffers) ([<u>https://github.com/protocolbuffers/protobuf</u>](https://github.com/protocolbuffers/protobuf)): 在这个 C++ 重写中，我将保持函数式风格，需要替换 Python 的 nametuples 来表示指令。这意味着创建许多简单的裸结构数据，需要在测试中动态创建（有一个很好的文本格式解析器），还需要序列化到磁盘，因为核心和查询语言之间的边界将成为协议缓冲消息文件。Protocol Buffers 提供了一个良好的层次数据结构，具有重复字段，并且支持多种语言（这为用例如 Go 编写插件打开了大门），并且可以为它们提供 Python 绑定。

    它还将成为 Beancount 核心和查询语言输入之间的接口。我们将使用 proto3，版本 >=3.12，以支持 [<u>可选存在字段</u>](https://www.google.com/url?q=https://github.com/protocolbuffers/protobuf/blob/master/docs/field_presence.md&sa=D&ust=1593922864811000&usg=AFQjCNH4AQawCGimgvXDz1vOL3CVzTyuZQ)（空值）。

- [**<u>Riegeli</u>**](https://github.com/google/riegeli): 一种用于将协议缓冲消息序列存储到文件的高效压缩二进制格式。我认为 Beancount 核心将输出这个格式；它紧凑且读取快速。这也是另一个应该受到更多关注的 Google 项目，支持 C++ 和 Python 以及协议缓冲。

- [**<u>mpdecimal</u>**](https://www.bytereef.org/mpdecimal/) ([<u>https://www.bytereef.org/mpdecimal/</u>](https://www.bytereef.org/mpdecimal/))**:** 这是 Python 实现的十进制数字使用的相同 C 级库。使用这个库可以轻松在 C++ 核心和 Python 运行时之间操作。我需要在 C++ 内存中表示十进制数字，具有最小功能和合理的小范围（BigNum 类通常超出我们需要的范围）。我们不需要太多十进制范围……主要是基本的算术运算和量化。

    还有其他库： [<u>GMP</u>](https://gmplib.org/)，[<u>decNumber</u>](http://speleotrove.com/decimal/decnumber.html)。有关更多信息，请参见 [<u>此线程</u>](https://stackoverflow.com/questions/14096026/c-decimal-data-types)。对于磁盘表示，我将需要定义一个协议缓冲消息，我在考虑定义一个字符串联合（易于读取但从字符串到十进制的转换很多），以及一些更有效的指数 + 尾数十进制等效值。

## 解析器<a id="parser"></a>

- **[<u>RE/flex 词法分析器</u>](https://www.genivia.com/doc/reflex/html/)** ([<u>https://github.com/Genivia/RE-flex</u>](https://github.com/Genivia/RE-flex)): 这个现代正则表达式基础的扫描生成器原生支持 Unicode，速度非常快，文档详细。它为老旧的 GNU flex 提供了一个很好的替代方案，GNU flex 在字符串文字外支持非 ASCII 字符方面存在困难（例如，账户名）。我在其他项目中使用它成功了。许多用户希望用母语书写账户名；这将使提供整个文件的 UTF-8 解析器变得容易。

- **[<u>GNU Bison</u>](https://www.gnu.org/software/bison/)** ([<u>https://git.savannah.gnu.org/git/bison.git</u>](https://git.savannah.gnu.org/git/bison.git)): 我们将继续使用 GNU Bison，但改用其支持的 C++ 完整模式。我对继续使用这个解析器生成器有所犹豫，因为它显示出老化迹象，但它非常稳定，我无法为升级到 ANTLR 的额外工作找到正当理由。

    我们将需要使用一些技巧来支持生成 C 代码用于 v2 和生成 C++ 代码用于下一个版本的相同语法；解析器代码可以提供一个函数调度表，这在 v2 中将是静态 C 函数，而在 C++ 版本中将是方法。一些生成参数（% 指令）会有所不同（请参见 [<u>此处</u>](https://github.com/blais/oblique/blob/master/oblique/parser.yxx#L8) 的示例）。

- **[<u>国际化组件（ICU）</u>](http://site.icu-project.org/home)** ([<u>https://github.com/unicode-org/icu.git</u>](https://github.com/unicode-org/icu.git)): 这是依赖于 Unicode 支持的标准库。我们的 C++ 将不会使用 std::wstring/std::wchar，而是使用 std::string 和必要时调用该库的函数。

## Python<a id="python"></a>

- [**<u>Python3</u>**](https://www.python.org/): 不多说了。我将继续使用最新版本。Python 是一种强大的扩展语言，没有计划改变这一点。

- **[<u>pybind11</u>](https://pybind11.readthedocs.io/en/stable/)** ([<u>https://github.com/pybind/pybind11</u>](https://github.com/pybind/pybind11)): 我希望提供一个几乎与 Beancount 目前相同的或更好的 Python API（即更简单）。我的一个要求是使传递一组用于指令的协议缓冲对象给 Python 回调变得廉价，而无需在 C++ 和 Python 之间复制（序列化和反序列化）——用于插件。我调查了多种库来在 Python 和 C++ 之间进行互操作：Cython、CLIF、SWIG 等，序列化是一个问题（参见 [<u>这个部分解决方案</u>](https://github.com/google/nucleus/blob/master/nucleus/util/proto_ptr.h)）。目前看起来最有前途的是 pybind11，一个纯头文件库，是 Boost::Python 的进化版，提供了对生成 API 的最大控制权。它还与用 fast\_cpp\_protos 构建的协议缓冲目标配合良好：只传递指针，因此插件可以传入和传出完整的指令列表。我也碰巧熟悉 Boost::Python，20 年前使用过它，它实际上非常相似（但不再使用 Boost 的其他部分）。

- **[<u>类型注解</u>](https://docs.python.org/3/library/typing.html), [<u>PyType</u>](https://github.com/google/pytype)**（或 [<u>MyPy</u>](http://mypy-lang.org/)?): 我已经遵循了自定义配置的 PyLint 进行合规，但代码库未使用日益普及的 [<u>类型注解</u>](https://docs.python.org/3/library/typing.html)。在将保留的代码子集重写中，我希望所有函数都带有注解，并用更自由的文档替换有时冗余的 Args/Returns 文档字符串（类型可能足够避免 Args/Returns 块的形式）。我将查看这如何影响 [<u>自动生成的文档</u>](https://github.com/beancount/docs)。

    一个重要的补充是，我希望开始不仅注解，还在构建的一部分自动运行一个类型检查器。我已经熟悉了 Google 的 pytype，但或许 mypy 是一个好的替代品。无论如何，唯一的障碍是为此编写 Bazel 规则，使其作为 py\_library() 和 py\_binary() 规则的一部分在整个代码库中自动调用。我还将尝试以同样方式（作为构建的一部分）运行 pylint，带有一个自定义标志，在开发过程中禁用它，而不是单独的 lint 目标。

- **Subpar** ([<u>https://github.com/google/subpar</u>](https://github.com/google/subpar)): 我尚不清楚如何为 Bazel 构建执行兼容 pip 的 setup.py，但肯定可以找到一种方法使用 Bazel 构建的二进制文件来构建 PyPI 的轮子。为了打包 Python + 扩展的自包含二进制文件，"subpar" Bazel 规则应该可以处理。然而，目前 [<u>不支持 C 扩展</u>](https://github.com/google/subpar/issues/59)。
