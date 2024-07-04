# 安装 Beancount (v2) <a id="title"></a>

[<u>Martin Blais</u>](mailto:blais@furius.ca) - 更新日期：2015年6月

[<u>http://furius.ca/beancount/doc/install</u>](http://furius.ca/beancount/doc/install)

*本文档提供了在您的计算机上下载和安装 Beancount 的说明。*

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th><em>本文档是关于 Beancount v2 的；Beancount v3 正在开发中，使用了完全不同的构建和安装系统。有关构建 v3 的说明，请参阅 <a href="installing_beancount_v3.md"><u>本文档</u></a>。</em></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

## 发布版本 <a id="releases"></a>

Beancount 是一个成熟的项目：第一个版本写于 2008 年。目前的 Beancount 重写版本是稳定的。（从技术上讲，这是我称之为的 2.x 测试版）。

目前我每周末都在继续开发 Beancount，所以它处于积极开发和演变中，尽管绝大多数基本功能基本不变。我建立了一个广泛的测试套件，因此您可以将存储库的“默认”分支视为稳定分支。新功能在分支中开发，只有在完全稳定（所有测试通过无失败）时才合并到“默认”分支中。“默认”分支的更改会发布到 [<u>更改文件</u>](https://github.com/beancount/beancount/tree/v2/CHANGES) 中，并向 [<u>邮件列表</u>](https://groups.google.com/forum/#!forum/beancount) 发送相应的电子邮件。

可以在 [<u>PyPI</u>](https://pypi.python.org/pypi/beancount) 页面找到相关信息。

## 获取 Beancount <a id="where-to-get-it"></a>

以下是源代码的官方存储位置：

> [<u>https://github.com/beancount/beancount</u>](https://github.com/beancount/beancount)

通过 Git 克隆到您的机器上进行下载：

```bash
git clone https://github.com/beancount/beancount
```

## 如何安装 <a id="how-to-install"></a>

### 安装 Python <a id="installing-python"></a>

Beancount 使用 Python 3.5[^1] 或更高版本，这是一个相当新的 Python 版本（截至本文撰写时），并且需要一些常见的库依赖项。我尽量减少依赖项，但您确实需要安装一些。这非常简单。

首先，您应该有一个正常工作的 Python 安装。使用 [<u>python.org</u>](http://python.org) 下载的最新稳定版本 >=3.5 进行安装。确保还安装了开发头文件和库（例如，“Python.h”头文件）。例如，在 Debian/Ubuntu 系统上，您需要安装 **`python3-dev`** 包。

自 2016 年 2 月以来，Beancount 支持 setuptools，您需要安装依赖项。您需要安装“pip3”工具。它可能已经与 Python3 一起默认安装——通过调用“pip3”命令进行测试。无论如何，在 Debian/Ubuntu 系统下，您只需安装 **`python3-pip`** 包。

#### Python 依赖项 <a id="python-dependencies"></a>

请注意，为了从源代码构建一个完整的 Python 安装，您可能需要安装许多其他开发库及其相应的头文件，例如 libxml2、libxslt1、libgdbm、libmp、libssl 等。安装这些依赖项取决于您的特定发行版和/或操作系统。只需确保您的 Python 安装具有其默认配置的所有基本模块。

### 使用 pip 安装 Beancount <a id="installing-beancount-using-pip"></a>

这是安装 Beancount 的最简单方法。您只需使用以下命令安装 Beancount：

```bash
sudo -H python3 -m pip install beancount
```

这将自动下载并安装所有依赖项。

但是，请注意，这将安装推送到 [<u>PyPI 存储库</u>](https://pypi.python.org/pypi/beancount/) 的最新版本，而不是从源代码获取的最新版本。虽然发布到 PyPI 的版本相对频繁，但有时也会有一定滞后。如果您想了解发布日期之后包含哪些更改，请查阅 [<u>更改文件</u>](https://github.com/beancount/beancount/tree/v2/CHANGES)。

### 从存储库使用 pip 安装 Beancount <a id="installing-beancount-using-pip-from-repository"></a>

您还可以使用 pip 直接从其源代码存储库安装 Beancount：

```bash
sudo -H python3 -m pip install git+https://github.com/beancount/beancount#egg=beancount
```

### 从源代码安装 Beancount <a id="installing-beancount-from-source"></a>

从源代码安装的优势在于可以提供最新的稳定分支版本（“默认”）。默认分支与发布版本一样稳定。

#### 获取源代码 <a id="obtain-the-source-code"></a>

从官方存储库获取源代码：

```bash
git clone https://github.com/beancount/beancount
```

#### 安装第三方依赖项 <a id="install-third-party-dependencies"></a>

您可能需要安装一些非 Python 的库依赖项，例如 libxml2-dev、libxslt1-dev 等。尝试构建，缺少的部分会显而易见。如果在 Ubuntu 上安装，使用 apt-get 命令。

如果在 Windows 上安装，请参阅下面的 Windows 部分。

#### 使用 pip3 从源代码安装 Beancount <a id="install-beancount-from-source-using-pip3"></a>

然后您可以使用 pip 安装所有依赖项和 Beancount 本身：

```bash
cd beancount
sudo -H python3 -m pip install .
```

#### 使用 setup.py 从源代码安装 Beancount <a id="install-beancount-from-source-using-setup.py"></a>

首先，您需要安装依赖库。可以使用 pip 安装：

```bash
sudo -H python3 -m pip install python-dateutil bottle ply lxml python-magic beautifulsoup4
```

或者，您可以使用发行版中的工具，例如在 Ubuntu/Debian 上：

```bash
sudo apt-get install python3-dateutil python3-bottle python3-ply python3-lxml python3-bs4 …
```

然后，可以使用常规的 setup.py 命令将包安装到 Python 库中：

```bash
cd beancount
sudo python3 setup.py install
```

或者，您可以使用以下命令将包安装到用户本地 Python 库中：

```bash
sudo python3 setup.py install --user
```

记得将 `~/.local/bin` 添加到您的路径中，以访问本地安装。

#### 开发安装 <a id="installing-for-development"></a>

如果您想在进行更改时执行源代码中的内容，可以使用 setuptools 的“develop”命令指向它：

```bash
cd beancount
sudo python3 setup.py develop 
```

警告：这会修改您的 Python 安装中的 .pth 文件以指向您的克隆路径。您可能会也可能不希望这样做。

我个人使用的方式是*老派*的；我在本地构建它并设置我的环境以找到其库。您可以这样构建：

```bash
cd beancount
python3 setup.py build_ext -i
```

最后，您需要更新 PATH 和 PYTHONPATH 环境变量：

```bash
export PATH=$PATH:/path/to/beancount/bin
export PYTHONPATH=$PYTHONPATH:/path/to/beancount
```

### 从包安装 <a id="installing-from-packages"></a>

各种发行版可能会打包 Beancount。以下是已知存在的链接：

-   Arch：[<u>https://aur.archlinux.org/packages/beancount/</u>](https://aur.archlinux.org/packages/beancount/)

### Windows 安装 <a id="windows-installation"></a>

#### 原生安装 <a id="native"></a>

通过 pip 安装此包需要在安装过程中编译一些 C++ 代码，这只有在计算机上有合适的编译器时才可能，否则您将收到有关缺少 *vsvarsall.bat* 或 *cl.exe* 的错误消息。

要能够为 Python 编译 C++ 代码，您需要安装与 Python 安装所使用的 C++ 编译器相同的主要版本。通过在控制台中运行 *python* 并查找类似于 *\[MSC v.1900 64 bit (AMD64)\]* 的文本，您可以确定特定 Python 发行版所使用的编译器。在此示例中，它是 *v.1900*。

使用这个号码在 [<u>此处</u>](https://stackoverflow.com/questions/2676763/what-version-of-visual-studio-is-python-on-my-computer-compiled-with) 找到所需的 Visual C++ 版本。由于不同版本似乎兼容，只要前两位数字相同，您理论上可以使用任何 1900 到 1999 之间的 Visual C++ 编译器。

根据我的经验，Python 3.5 和 3.6 都是用 MSC v.1900 编译的，因此您可以执行以下任一操作来满足此要求：

-   安装独立的 [<u>Visual Studio 2017 构建工具</u>](https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=BuildTools&rel=15) 或

-   安装独立的 [<u>Visual C++ Build Tools 2015</u>](http://go.microsoft.com/fwlink/?LinkId=691126) 或

-   修改现有的 Visual Studio 2017 安装

    -   从*添加或删除程序*中启动 Visual Studio 2017 安装程序

    -   选择*个别组件*

    -   在*编译器、构建工具和运行时*下选择*VC++ 2017 版本 15.9 v14.16 最新 v141 工具*或更新版本

    -   安装

-   Visual Studio 2019

    -   添加 C++ 构建工具：C++ 核心功能、MSVC v142 构建工具

如果安装后 cl.exe 不在您的路径中，请运行 Visual Studio 的开发者命令提示符并在那里运行命令。

#### 使用 Cygwin <a id="with-cygwin"></a>

Windows 安装当然有点不同。如果您使用 Cygwin，这将非常简单。您只需先准备好您的计算机。以下是方法。

-   安装最新的 [<u>Cygwin</u>](https://www.cygwin.com/)。这可能需要一段时间（它会下载很多东西），但无论如何都非常值得。但是在启动安装之前，请确保在 setup.exe 提供的界面中手动启用以下包（它们默认未选择）：

    -   python3

    -   python3-devel

    -   python3-setuptools

    -   git

    -   make

    -   gcc-core

    -   flex

    -   bison

    -   lxml

    -   ply

-   启动新的 Cygwin bash shell（桌面上应该有一个新图标），并通过运行以下命令安装 pip3 安装工具：  
    **easy\_install-3.4 pip**  
    **确保您调用与您的 Python 版本匹配的 easy\_install 版本，例如如果安装了 Python 3.5，则调用 easy\_install-3.5，或更多版本。**

此时，您应该能够按照前面部分中的说明进行操作，从“使用 pip 安装 Beancount”开始。

#### 使用 WSL <a id="with-wsl"></a>

最新发布的 Windows 10 周年更新带来了 WSL “Windows Subsystem for Linux” 以及 [<u>Windows 上的 Ubuntu bash</u>](https://blogs.msdn.microsoft.com/wsl/2016/07/08/bash-on-ubuntu-on-windows-10-anniversary-update/)（安装说明及更多信息请参见 [<u>https://msdn.microsoft.com/commandline/wsl/about</u>](https://msdn.microsoft.com/commandline/wsl/about)）。

这使得 Beancount 安装变得简单，在 bash 中运行：

```bash
sudo apt-get install python3-pip
sudo apt-get install python3-lxml
sudo pip3 install m3-cdecimal
sudo pip3 install beancount --pre
```

这并不是完全的“Windows 兼容”，因为它运行在一个 pico 进程中，但提供了一种在 Windows 上获得 Linux 命令行体验的便捷方式。*(贡献者：willwray)*

#### 关于 lxml 的注意事项 <a id="notes-on-lxml"></a>

一些用户报告在安装 lxml 时遇到问题，解决方法是：在 Cygwin 下使用 pip 安装 lxml 时，可以尝试以下命令：

```bash
STATIC_DEPS=true pip install lxml
```

### 检查您的安装 <a id="checking-your-install"></a>

您应该能够运行 [<u>本文档</u>](running_beancount_and_generating_reports.md) 中的二进制文件。例如，运行 bean-check 应该会产生如下输出：

```bash
$ bean-check

usage: bean-check [-h] [-v] filename
bean-check: error: the following arguments are required: filename
```

如果这可以运行，您现在可以前往 [<u>教程</u>](tutorial_example.md) 并开始学习 Beancount 的工作原理。

### 报告问题 <a id="reporting-problems"></a>

如果您需要报告问题，可以发送电子邮件到邮件列表或在 Github 上 [<u>提交问题</u>](https://github.com/beancount/beancount/issues)。运行以下命令将列出您的计算机上安装的依赖项及其版本，您可以在错误报告中包括该命令的输出：

```bash
python3 -m beancount.scripts.deps
```

## 编辑器支持 <a id="editor-support"></a>

一些编辑器支持 Beancount：

-   Emacs 支持提供在 [<u>单独的仓库中</u>](https://github.com/beancount/beancount-mode)。安装说明请参见 [<u>入门指南</u>](getting_started_with_beancount.md) 文本。

-   [<u>Sublime 编辑器支持</u>](https://sublime.wbond.net/packages/Beancount) 由 [<u>Martin Andreas Andersen</u>](https://groups.google.com/d/msg/beancount/WvlhcCjNl-Q/s4wOBQnRVxYJ) 贡献。请参阅 [<u>他的 GitHub 仓库</u>](https://github.com/draug3n/sublime-beancount)。

-   Vim 编辑器支持由 [<u>Nathan Grigg</u>](https://github.com/nathangrigg) 贡献。请参阅 [<u>他的 GitHub 仓库</u>](https://github.com/nathangrigg/vim-beancount)。

-   Visual Studio Code 目前有两个可用的扩展。两者都在 Linux 上进行了测试。

    -   [<u>Beancount</u>](https://marketplace.visualstudio.com/items?itemName=Lencerf.beancount)，具有语法检查（bean-check）和账户、货币等支持。它不仅允许选择现有的开放账户，还显示它们的余额和其他元数据。非常有帮助！

    -   [<u>Beancount Formatter</u>](https://marketplace.visualstudio.com/items?itemName=dongfg.vscode-beancount-formatter)，可以格式化整个文档，排列数字等，使用 bean-format。

## 如果您遇到问题 <a id="if-you-have-problems"></a>

如果您在安装过程中遇到任何问题，请 [<u>提交问题</u>](https://github.com/beancount/beancount/issues/)，[<u>发送电子邮件给我</u>](mailto:blais@furius.ca)，或访问 [<u>邮件列表</u>](https://groups.google.com/forum/#!forum/beancount)。

## 安装后使用 <a id="post-installation-usage"></a>

通常，[<u>beancount/bin 下的脚本</u>](https://github.com/beancount/beancount/tree/v2/bin) 应该会自动安装到您的可执行路径中。例如，您应该能够在命令行中运行“bean-check”。

## 附录 <a id="appendix"></a>

**如果一切正常，您可以在这里停止阅读。** 这里我只是讨论了各种依赖项以及您需要它们的原因（或不需要它们的原因，一些是可选的）。这对开发人员感兴趣，部分信息可能有助于排除遇到的问题。

### 依赖项说明 <a id="notes-on-dependencies"></a>

#### Python 3.5 或更高版本 <a id="python-3.5-or-greater"></a>

Python 3.5 目前广泛可用，已经发布一年多。然而，我的开源发行经验告诉我，许多用户使用的是旧机器，暂时不会升级。因此，对于这些用户，您可能需要自己安装 Python……不用担心，手动安装 Python 非常简单。

在 Mac 上，可以使用各种包管理工具之一安装，例如 Brew，例如使用“`brew install python3`”。

在旧的 Linux 上，从 [<u>python.org</u>](https://www.python.org/downloads/) 下载源代码，然后按以下步骤构建：

```bash
tar zxcf Python-3.5.2.tgz
cd Python-3.5.2
./configure
make
sudo make install
```

这应该可以正常工作。我建议您安装最新的 3.x 版本（撰写本文时为 3.5.2）。

注意：要求至少版本 3.5 的原因是 cdecimal 库已经添加到标准库中，为 Beancount 提供其十进制表示的实现。我们需要这个库来表示所有数字。此外，Beancount 使用 3.5 中包含的“typing”类型注释（注意：如果使用旧版本，您可能能够显式安装“typing”；但不保证）。

一些用户报告某些发行版为 Python3 单独打包了 cdecimal 支持。这在 Arch 上是这样，我也在一些 Ubuntu 安装中见证了缺少 cdecimal 支持。2015 年 12 月 Beancount 插入了一个检查，如果您的 Python 安装没有快速的十进制数，将发出警告。如果您遇到这种情况，可以尝试显式安装 cdecimal，如下所示：

```bash
sudo pip3 install m3-cdecimal
```

然后 Beancount 将使用它。

#### Python 库 <a id="python-libraries"></a>

如果需要安装 Python 库，有几种不同的方法。首先是简单方法：有一个名为“pip3”的包管理工具（对于 Python 2 版本则是“pip”），您可以这样安装库：

```bash
sudo pip3 install <library-name>
```

从源代码安装库也非常简单：下载并解压源代码，然后从其根目录运行以下命令：

```bash
sudo python3 setup.py install
```

只需确保运行此命令时的“python3”可执行文件与您运行 Beancount 时使用的相同。

以下是 Beancount 依赖的库及其简短讨论。

##### python-dateutil <a id="python-dateutil"></a>

此库为 Beancount 提供解析各种格式日期的能力，方便用户在命令行和 SQL shell 中具有灵活选项。

##### bottle <a id="bottle"></a>

Beancount 的 Web 界面（通过 `bean-web` 调用）运行一个自包含的小型 Web 服务器，在本地机器上访问。这是使用一个使实现此类 Web 应用程序变得容易的小型库实现的：[<u>bottle.py</u>](http://bottlepy.org/)。

##### ply <a id="ply"></a>

用于从分类账文件中提取数据表的查询客户端（`bean-query`）依赖于一个解析器生成器：我使用 Dave Beazley 的流行库 [<u>PLY</u>](http://www.dabeaz.com/ply/)（版本 3.4 或更高），因为它使实现自定义 SQL 语言解析器变得非常容易。

##### lxml <a id="lxml"></a>

提供了一个工具，可以将 Web 界面的静态 HTML 版本烘焙到 zip 文件中（`bean-bake`）。这对于与没有安装您软件的会计师共享文件非常方便。用于执行此操作的 Web 抓取代码使用 lxml HTML 解析库。

#### 用于导出的 Python 库（可选） <a id="python-libraries-for-export-optional"></a>

一些我用来导出输出到 Google Drive 的脚本（以及用于维护和下载文档的脚本）已被检查到代码库中。如果您不导出到 Google Drive，您无需安装这些库；其他所有功能应正常运行。因此，它们是可选的。

如果您确实安装了此库，它需要近期（截至 2015 年 6 月）的以下安装：

-   [<u>google-api-python-client</u>](https://github.com/google/google-api-python-client)：用于 Google 的基于发现的 API 的 Python 客户端库。

-   [<u>oauth2client</u>](https://github.com/google/oauth2client)：用于访问受 OAuth 2.0 保护的资源的客户端库，由 Google API Python 客户端库使用。

-   [<u>httplib2</u>](https://github.com/jcgregorio/httplib2)：一个综合的 HTTP 客户端库。由 oauth2client 使用。

这些最好使用 pip3 工具安装。

更新：截至 2016 年，您应该可以通过以下命令安装上述所有内容：

```bash
pip3 install google-api-python-client
```

**重要提示：** 对 Python3 的支持相对较新，如果您有一个旧安装，可能会遇到一些失败，例如缺少依赖项，如缺少“gflags”导入。请从最新版本手动安装它们，您应该可以正常运行。

#### 用于导入的 Python 库（可选） <a id="python-libraries-for-imports-optional"></a>

支持从外部下载的文件中导入识别、提取和归档交易是 Beancount 的一部分。（这曾经是在 LedgerHub 项目中，现在已被弃用。）如果您不使用这些导入工具和库，您不需要这些导入库。

##### python-magic（可选） <a id="python-magic-optional"></a>

Beancount 尝试使用 stdlib 的“mimetypes”模块和一些本地启发式方法来识别文件类型，但此外，安装 libmagic 也是有用的，如果它不存在，Beancount 将使用它：

```bash
pip3 install python-magic
```

请注意，还有另一个较旧的库提供 libmagic 支持，名为“filemagic”。您需要的是“python-magic”而不是“filemagic”。更混乱的是，在 Debian 下，“python-magic”库被称为“filemagic”。

##### 其他工具 <a id="other-tools"></a>

预计用户将构建自己的导入器。但是，Beancount 将提供一些模块，帮助您调用各种外部工具。所需的依赖项取决于您配置导入器使用的工具。

#### Virtualenv 安装 <a id="virtualenv-installation"></a>

如果您想使用 virtualenv，可以尝试以下操作（由 Remy X 提供建议）。首先安装 Python 3.5 或更高版本[^2]：

```bash
sudo add-apt-repository ppa:fkrull/deadsnakes
sudo apt-get update
sudo apt-get install python3.5
sudo apt-get install python3.5-dev
sudo apt-get install libncurses5-dev
```

然后安装并激活 virtualenv：

```bash
sudo apt-get install virtualenv
virtualenv -p /usr/bin/python3.5 bean
source bean/bin/activate
pip install package-name
sudo apt-get install libz-dev  # 为了构建 lxml
pip install lxml
pip install beancount
```

### 开发设置 <a id="development-setup"></a>

#### 开发安装 <a id="installation-for-development"></a>

对于开发，您将希望避免在 Python 分发中安装 Beancount，而是只修改 `PYTHONPATH` 环境变量以从源代码运行它：

```bash
export PYTHONPATH=/path/to/install/of/beancount
```

Beancount 有一些编译的 C 扩展模块。这只是可移植的 C 代码，应该可以在任何地方工作。对于开发，您希望在源代码中就地编译 C 模块，并从那里加载它们。像这样在源代码中就地构建 C 扩展模块：

```bash
python3 setup.py build_ext -i
```

或者，同样的效果，根目录下有一个 Makefile，可以执行相同的操作，并在需要时重新构建词法分析器和解析器的 C 代码（您需要安装 flex 和 bison）：

```bash
make build
```

这样，您应该能够运行 `bin/` 子目录下的可执行文件。您可能还希望将其添加到您的 `PATH` 中。

#### 开发依赖项 <a id="dependencies-for-development"></a>

Beancount 需要一些额外的工具用于开发。如果您在阅读本文，您就是开发人员，所以我假设您知道如何安装软件包，略过细节，只列出您可能需要的工具：

-   [**<u>nosetests</u>**](https://pypi.python.org/pypi/nose/) (**nose**，或 **python3-nose**)：这是用于运行所有单元测试的测试驱动程序。您绝对需要这个。

-   [**<u>GNU flex</u>**](https://www.gnu.org/software/flex/)：如果您打算修改词法分析器，则需要此词法分析器生成器。它生成的 C 代码会将输入文件分解为解析器消耗的标记。

-   [**<u>GNU bison</u>**](http://www.gnu.org/software/bison/)：如果您打算修改语法，则需要此老式解析器生成器。它生成的 C 代码会解析由 flex 生成的标记。（我喜欢使用这样的老工具，因为它们的依赖性非常低，只是 C 代码。应该可以在任何地方工作。）

-   [**<u>GNU tar</u>**](http://www.gnu.org/software/tar/)：您需要这个来测试 bean-bake 的归档功能。

-   [**<u>InfoZIP zip</u>**](http://www.info-zip.org/)（附带 Ubuntu）：您需要这个来测试 bean-bake 的归档功能。

-   [**<u>lxml</u>**](http://lxml.de)：此 XML 解析库在 Web 测试中使用。（不幸的是，内置的库无法解析大型 XML 文件。）

-   [**<u>pylint</u>**](http://www.pylint.org/) >= 1.2：我正在对所有源代码运行这个 linter。如果您贡献的代码旨在集成和发布，它必须通过 linter 测试（否则我将不得不自己使其通过）。

-   [**<u>pyflakes</u>**](https://github.com/pyflakes/pyflakes)：我正在对所有源代码运行这个逻辑错误检测器。如果您贡献的代码旨在集成和发布，它必须通过这些测试（否则我将不得不自己使其通过）。

-   [**<u>snakefood</u>**](http://furius.ca/snakefood/)：我使用 snakefood 分析模块之间的依赖关系并强制某些顺序，例如核心不能从插件中导入。例如，根 Makefile 中有一个目标可以运行它。

-   [**<u>graphviz</u>**](http://www.graphviz.org/)：所有模块的依赖树由 snakefood 生成，但由 graphviz 工具绘制。如果您想查看依赖关系，则需要安装它。

我想大概就是这样了。您当然不需要上面提到的*所有*内容，但这是我使用的工具列表。如果您发现缺少任何内容，请发表评论，我可能遗漏了某些东西。

[^1]: 一些人[<u>报告</u>](https://www.google.com/url?q=https://groups.google.com/d/msgid/beancount/51fec791-1c38-46e8-b152-08c49e7686c3%2540googlegroups.com?utm_medium%3Demail%26utm_source%3Dfooter&sa=D&ust=1461188801128000&usg=AFQjCNH8StkxqPYKLKMhir6YAz6rXCOKsQ) 3.4 版本的 cdecimal 库有 bug。我建议实际安装 3.5 版本，似乎已经解决了该问题。从技术上讲，3.3 仍然可以运行，但我可能会在某个时候将其弃用为 3.5，可能是在 Ubuntu 和 Mac OS X 发行版默认安装时。

[^2]: 安装 py3.5：[http://www.jianshu.com/p/4f4b2ed568f4](http://www.jianshu.com/p/4f4b2ed568f4)
