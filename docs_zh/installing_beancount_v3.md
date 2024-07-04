# 安装 Beancount (C++ 版本)<a id="title"></a>

[<u>Martin Blais</u>](mailto:blais@furius.ca) - 2020年7月

[<u>http://furius.ca/beancount/doc/v3-install</u>](http://furius.ca/beancount/doc/v3-install)

*本文件介绍了如何在您的计算机上下载并运行 Beancount v3（开发中）。有关 v2 的安装说明，请参阅此文档：[Beancount - 安装 (v2)](installing_beancount.md)*

## 设置 Python<a id="setup-python"></a>

运行程序仍需要 Python 依赖项。

```bash
pip install -r requirements/dev.txt
```

## 使用 Bazel 构建<a id="building-with-bazel"></a>

*警告：这是一个实验性开发分支。不要期望所有内容都完美无缺。*

### Bazel 依赖项<a id="bazel-dependencies"></a>

Beancount v3 使用 Bazel 构建系统，这在很大程度上隔离了本地安装和您的计算机上的依赖项。

需要安装的依赖项有：

- **Bazel 本身。** 请按照 [<u>https://bazel.build/</u>](https://bazel.build/) 上的说明进行安装。

- **C++ 编译器。** g++ 或 clang 均可。我使用的是 clang-11。

- **Python 运行时**（版本 3.8 或更高）。请从您的发行版中安装。

Bazel 将自行下载并编译所需的所有库（甚至代码生成器，例如 Bison），并在其构建中指定的精确版本上构建，因此您不必担心它们。

### 构建与测试<a id="building-testing"></a>

只需运行以下命令：

```bash
bazel build //bin:all
```

目前没有安装脚本，您需要从源代码运行。您可以使用以下命令运行单个程序（例如 bean-check）：

```bash
bazel run //bin:bean_check -- /path/to/ledger.beancount
```

或者如果您不希望自动重建修改的代码，可以这样运行：

```bash
./bazel-bin/bin/bean_check /path/to/ledger.beancount
```

### 开发<a id="development"></a>

您可以运行所有单元测试，如下所示：

```bash
bazel test //...
```

由于 Bazel 详细记录了所有依赖项，在修改代码后重新运行此命令只会重新测试受影响的目标；这使得带有测试的迭代开发更加有趣。

另一个优点是，由于构建依赖的所有库都会下载和构建，尽管第一次构建可能很慢，但这使我们能够使用我们依赖的代码的最新发布版本。

目标在其目录下的 BUILD 文件中定义。所有构建规则都可以在 //third_party 下找到。

### 导入<a id="ingestion"></a>

导入代码涉及从存储库外部导入代码。Bazel 二进制文件是自包含的，将无法导入未声明为依赖项的模块，因此运行 `//bin:bean_extract` 目标可能无法工作。

这目前还不可行（除非将您的导入配置构建为显式链接到 Beancount 的 py_binary() 目标）。通过定义一个合适的 WORKSPACE 文件来获取其中的规则，可以在不编写太多 Bazel 代码的情况下实现这一点。我尚未提供这方面的示例（待定）。

作为解决方法，您可以设置您的 PYTHONPATH 以从源位置导入，并预先创建指向解析器 .so 文件的符号链接。您可以这样做：

```bash
make bazel-link
```

### 待办事项<a id="tbd"></a>

还有一些构建集成任务需要完成：

- pytype 尚未支持。
- pylint 也未集成到构建中。

## 使用 meson 进行开发安装<a id="installation-for-development-with-meson"></a>

***注意：**本节基于以下讨论更新：[<u>https://groups.google.com/g/beancount/c/7ppbyz_5B5w</u>](https://groups.google.com/g/beancount/c/7ppbyz_5B5w)*

Bazel 构建的 Beancount v3 构建了一些实验性的 C++ 代码，截至撰写时（2024年3月9日）尚未用于除技术演示以外的任何其他用途。在“生产”中，v3 使用 meson-python 构建扩展模块并将其打包到 Python 轮子中。这是 pip 在安装过程中使用的内容。

本节描述了用于开发 beancount v3 Python 代码的安装过程，而不是用于实验 C++ 代码。

### 在 Linux 上<a id="on-linux"></a>

在 Ubuntu 上测试过 python3.12

相关链接

[<u>https://mesonbuild.com/meson-python/how-to-guides/editable-installs.html</u>](https://mesonbuild.com/meson-python/how-to-guides/editable-installs.html#build-dependencies)

```bash
git clone https://github.com/beancount/beancount.git
```

创建并激活虚拟环境

```bash
python3.12 -m venv beancount/venv
. beancount/venv/bin/activate
cd beancount
```

安装所需的软件包。

```bash
python -m pip install meson-python meson ninja
```

在可编辑模式下安装 beancount，且不进行构建隔离

```bash
python -m pip install --no-build-isolation --editable .
```

**注意：** 有一种 [<u>观点</u>](https://groups.google.com/g/beancount/c/7ppbyz_5B5w/m/YlHiKhynFAAJ) 认为不需要 `--no-build-isolation` 选项，但也有人提到 [<u>安装未成功</u>](https://groups.google.com/g/beancount/c/7ppbyz_5B5w/m/nSxCzuutFAAJ) 没有该选项。此外，[<u>文档</u>](https://mesonbuild.com/meson-python/how-to-guides/editable-installs.html#editable-installs) 建议需要此选项。这可能取决于 Linux 环境的类型。

安装 pytest

```bash
python -m pip install pytest
```

运行测试，确保一切正常：

```bash
pytest --import-mode=importlib beancount/
```

### 在 Windows 上<a id="on-windows"></a>

在 64 位 Windows 10 专业版上测试过

**前提条件**

安装 Microsoft Visual Studio（测试版本 2022）

**步骤**

```bash
git clone https://github.com/beancount/beancount.git
cd beancount
```

如果运行的是 64 位 Windows，请启动“x64 本机工具命令提示符 for VS 20XX”。按下 Windows 键并键入 x64

<img src="installing_beancount_v3/media/9c5ea265a2ff61958e89a0965bd95bbd54854eb0.png" style="width:3.83854in;height:2.03255in" />

打开此提示符

转到安装 beancount 的目录

```bash
cd C:\_code\t\beancount
```

激活虚拟环境

```bash
venv\Scripts\Activate
(venv) C:\_code\t\beancount>
```

安装所需的软件包

```bash
(venv) C:\_code\t\beancount>py -m pip install meson-python meson ninja
```

在可编辑模式下安装 beancount。与在 Linux 上不同，Windows 上的 **`--no-build-isolation`** 会抛出一些错误，但似乎不需要。

```bash
(venv) C:\_code\t\beancount> py -m pip install --editable .
```

安装 pytest

```bash
(venv) C:\_code\t\beancount>py -m pip install pytest
```

运行测试

```bash
(venv) C:\_code\t\beancount>pytest --import-mode=importlib beancount
```

注意：Windows 上的一些测试失败，但这是由于一般的 [<u>可移植性问题</u>](https://github.com/beancount/beancount/issues?q=is%3Aopen+is%3Aissue+label%3Aportability)。# 安装 Beancount (C++ 版本)<a id="title"></a>

[<u>Martin Blais</u>](mailto:blais@furius.ca) - 2020年7月

[<u>http://furius.ca/beancount/doc/v3-install</u>](http://furius.ca/beancount/doc/v3-install)

*本文件介绍了如何在您的计算机上下载并运行 Beancount v3（开发中）。有关 v2 的安装说明，请参阅此文档：[Beancount - 安装 (v2)](installing_beancount.md)*

## 设置 Python<a id="setup-python"></a>

运行程序仍需要 Python 依赖项。

```bash
pip install -r requirements/dev.txt
```

## 使用 Bazel 构建<a id="building-with-bazel"></a>

*警告：这是一个实验性开发分支。不要期望所有内容都完美无缺。*

### Bazel 依赖项<a id="bazel-dependencies"></a>

Beancount v3 使用 Bazel 构建系统，这在很大程度上隔离了本地安装和您的计算机上的依赖项。

需要安装的依赖项有：

- **Bazel 本身。** 请按照 [<u>https://bazel.build/</u>](https://bazel.build/) 上的说明进行安装。

- **C++ 编译器。** g++ 或 clang 均可。我使用的是 clang-11。

- **Python 运行时**（版本 3.8 或更高）。请从您的发行版中安装。

Bazel 将自行下载并编译所需的所有库（甚至代码生成器，例如 Bison），并在其构建中指定的精确版本上构建，因此您不必担心它们。

### 构建与测试<a id="building-testing"></a>

只需运行以下命令：

```bash
bazel build //bin:all
```

目前没有安装脚本，您需要从源代码运行。您可以使用以下命令运行单个程序（例如 bean-check）：

```bash
bazel run //bin:bean_check -- /path/to/ledger.beancount
```

或者如果您不希望自动重建修改的代码，可以这样运行：

```bash
./bazel-bin/bin/bean_check /path/to/ledger.beancount
```

### 开发<a id="development"></a>

您可以运行所有单元测试，如下所示：

```bash
bazel test //...
```

由于 Bazel 详细记录了所有依赖项，在修改代码后重新运行此命令只会重新测试受影响的目标；这使得带有测试的迭代开发更加有趣。

另一个优点是，由于构建依赖的所有库都会下载和构建，尽管第一次构建可能很慢，但这使我们能够使用我们依赖的代码的最新发布版本。

目标在其目录下的 BUILD 文件中定义。所有构建规则都可以在 //third_party 下找到。

### 导入<a id="ingestion"></a>

导入代码涉及从存储库外部导入代码。Bazel 二进制文件是自包含的，将无法导入未声明为依赖项的模块，因此运行 `//bin:bean_extract` 目标可能无法工作。

这目前还不可行（除非将您的导入配置构建为显式链接到 Beancount 的 py_binary() 目标）。通过定义一个合适的 WORKSPACE 文件来获取其中的规则，可以在不编写太多 Bazel 代码的情况下实现这一点。我尚未提供这方面的示例（待定）。

作为解决方法，您可以设置您的 PYTHONPATH 以从源位置导入，并预先创建指向解析器 .so 文件的符号链接。您可以这样做：

```bash
make bazel-link
```

### 待办事项<a id="tbd"></a>

还有一些构建集成任务需要完成：

- pytype 尚未支持。
- pylint 也未集成到构建中。

## 使用 meson 进行开发安装<a id="installation-for-development-with-meson"></a>

***注意：**本节基于以下讨论更新：[<u>https://groups.google.com/g/beancount/c/7ppbyz_5B5w</u>](https://groups.google.com/g/beancount/c/7ppbyz_5B5w)*

Bazel 构建的 Beancount v3 构建了一些实验性的 C++ 代码，截至撰写时（2024年3月9日）尚未用于除技术演示以外的任何其他用途。在“生产”中，v3 使用 meson-python 构建扩展模块并将其打包到 Python 轮子中。这是 pip 在安装过程中使用的内容。

本节描述了用于开发 beancount v3 Python 代码的安装过程，而不是用于实验 C++ 代码。

### 在 Linux 上<a id="on-linux"></a>

在 Ubuntu 上测试过 python3.12

相关链接

[<u>https://mesonbuild.com/meson-python/how-to-guides/editable-installs.html</u>](https://mesonbuild.com/meson-python/how-to-guides/editable-installs.html#build-dependencies)

```bash
git clone https://github.com/beancount/beancount.git
```

创建并激活虚拟环境

```bash
python3.12 -m venv beancount/venv
. beancount/venv/bin/activate
cd beancount
```

安装所需的软件包。

```bash
python -m pip install meson-python meson ninja
```

在可编辑模式下安装 beancount，且不进行构建隔离

```bash
python -m pip install --no-build-isolation --editable .
```

**注意：** 有一种 [<u>观点</u>](https://groups.google.com/g/beancount/c/7ppbyz_5B5w/m/YlHiKhynFAAJ) 认为不需要 `--no-build-isolation` 选项，但也有人提到 [<u>安装未成功</u>](https://groups.google.com/g/beancount/c/7ppbyz_5B5w/m/nSxCzuutFAAJ) 没有该选项。此外，[<u>文档</u>](https://mesonbuild.com/meson-python/how-to-guides/editable-installs.html#editable-installs) 建议需要此选项。这可能取决于 Linux 环境的类型。

安装 pytest

```bash
python -m pip install pytest
```

运行测试，确保一切正常：

```bash
pytest --import-mode=importlib beancount/
```

### 在 Windows 上<a id="on-windows"></a>

在 64 位 Windows 10 专业版上测试过

**前提条件**

安装 Microsoft Visual Studio（测试版本 2022）

**步骤**

```bash
git clone https://github.com/beancount/beancount.git
cd beancount
```

如果运行的是 64 位 Windows，请启动“x64 本机工具命令提示符 for VS 20XX”。按下 Windows 键并键入 x64

<img src="installing_beancount_v3/media/9c5ea265a2ff61958e89a0965bd95bbd54854eb0.png" style="width:3.83854in;height:2.03255in" />

打开此提示符

转到安装 beancount 的目录

```bash
cd C:\_code\t\beancount
```

激活虚拟环境

```bash
venv\Scripts\Activate
(venv) C:\_code\t\beancount>
```

安装所需的软件包

```bash
(venv) C:\_code\t\beancount>py -m pip install meson-python meson ninja
```

在可编辑模式下安装 beancount。与在 Linux 上不同，Windows 上的 **`--no-build-isolation`** 会抛出一些错误，但似乎不需要。

```bash
(venv) C:\_code\t\beancount> py -m pip install --editable .
```

安装 pytest

```bash
(venv) C:\_code\t\beancount>py -m pip install pytest
```

运行测试

```bash
(venv) C:\_code\t\beancount>pytest --import-mode=importlib beancount
```

注意：Windows 上的一些测试失败，但这是由于一般的 [<u>可移植性问题</u>](https://github.com/beancount/beancount/issues?q=is%3Aopen+is%3Aissue+label%3Aportability)。
