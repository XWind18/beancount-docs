# 在 Beancount 中导入外部数据<a id="title"></a>

[<u>Martin Blais</u>](mailto:blais@furius.ca)，2016年3月

[<u>http://furius.ca/beancount/doc/ingest</u>](http://furius.ca/beancount/doc/ingest)

*本文档适用于 Beancount v2；Beancount v3 正在开发中，使用完全不同的构建和安装系统。有关 v3 的导入说明，请参阅[<u>此文档</u>](https://docs.google.com/document/d/1hBfsHZcoHgz5rvhCdP42g2FJ5ouycIMV4H1tfgXpwBU/)（Beangulp）。*

> [<u>介绍</u>](#introduction)
>
> [<u>导入过程</u>](#the-importing-process)
>
> [<u>自动化网络下载</u>](#automating-network-downloads)
>
> [<u>典型下载</u>](#typical-downloads)
>
> [<u>从 PDF 文件提取数据</u>](#extracting-data-from-pdf-files)
>
> [<u>工具</u>](#tools)
>
> [<u>调用</u>](#invocation)
>
> [<u>配置</u>](#configuration)
>
> [<u>从输入文件配置</u>](#configuring-from-an-input-file)
>
> [<u>编写导入器</u>](#writing-an-importer)
>
> [<u>回归测试导入器</u>](#regression-testing-your-importers)
>
> [<u>生成测试输入</u>](#generating-test-input)
>
> [<u>逐步改进</u>](#making-incremental-improvements)
>
> [<u>缓存数据</u>](#caching-data)
>
> [<u>内存缓存</u>](#in-memory-caching)
>
> [<u>磁盘缓存</u>](#on-disk-caching)
>
> [<u>组织文件</u>](#organizing-your-files)
>
> [<u>示例导入器</u>](#example-importers)
>
> [<u>清理</u>](#cleaning-up)
>
> [<u>自动分类</u>](#automatic-categorization)
>
> [<u>清理收款人</u>](#cleaning-up-payees)
>
> [<u>未来工作</u>](#future-work)
>
> [<u>相关讨论线程</u>](#related-discussion-threads)
>
> [<u>历史备注</u>](#historical-note)

## 介绍<a id="introduction"></a>

这是 Beancount 中的库和工具的用户手册，这些库和工具可以帮助您**自动导入外部交易数据**到您的 Beancount 输入文件中，并管理从金融机构网站下载的文档。

## 导入过程<a id="the-importing-process"></a>

人们经常想知道我们如何做到这一点，所以让我坦率地详细描述我们在这里讨论的内容。

任务的本质是将一个人所有账户中的交易转录到一个文本文件中：Beancount 输入文件。将所有交易导入一个系统是生成一个人财富和支出全面报告所需的步骤。有些人称之为“对账”。

我们可以通过从纸质对账单中手动输入交易来转录所有交易。然而，现今大多数金融机构都有网站，您可以在这些网站上下载历史交易的对账单，这些对账单有多种数据格式，您可以解析这些数据格式以输出 Beancount 语法。

从这些文档导入交易包括：

- **手动审查**交易的**正确性**甚至欺诈行为；

- 将新交易与从另一个账户导入的先前交易**合并**。例如，从银行账户支付信用卡账单的交易通常会从银行和信用卡账户分别导入。您必须手动合并相应的交易[^1]。

- 为支出交易分配正确的**类别**；

- **组织**文件，将生成的指令移动到文件中的正确位置；

- **验证余额**，可以通过视觉方式或插入一个余额指令来断言在插入新交易后账户的最终余额应该是什么。

如果我的导入器没有问题，这个过程通常需要 30-60 分钟来更新大多数活跃账户。较少活跃的账户每季度更新一次，或者我有空的时候更新一次。我通常每两周或有时每周的星期六上午进行这个操作。如果您维护一个组织良好的输入文件并包含大量断言，错配问题很容易发现，这个过程愉快而简单，完成后生成的更新后的资产负债表令人满意（我通常会重新导出到 Google Finance 投资组合中）。

### 自动化网络下载<a id="automating-network-downloads"></a>

文件下载不是我自动化的一部分，Beancount 也不提供连接网络并获取文件的工具。协议种类太多，难以提供有意义的解决方案[^2]。鉴于今天的安全网站和用于实现这些网站的 JavaScript 堡垒，实施自动化下载将是一个噩梦。网络爬虫可能过于复杂，无法成为可行的解决方案。

我**手动**登录到各个网站，使用用户名和密码点击正确的按钮生成所需的下载文件。这些文件由导入器自动识别，并使用本文档中描述的工具自动提取交易并将文档归档到组织良好的目录层次结构中。

虽然我没有脚本化下载，但我认为在某些网站上是可以实现的。这个工作留给您在认为值得的时候自己实现。

### 典型下载<a id="typical-downloads"></a>

以下是典型文件的描述；这是我的使用案例以及我所实现的内容。这应该能给您定性了解涉及的内容。

- **信用卡和银行**提供相当高质量的历史对账单下载，有 OFX 或 CSV 文件格式，但我需要手动分类这些交易的另一方，并将一些交易合并在一起。

- **投资账户**为我提供了高质量的可处理对账单，购买交易的提取完全自动化，但我需要手动编辑销售交易以关联正确的成本基础。一些机构为专门产品（例如 P2P 借贷）只提供 PDF 文件，这些文件需要手动转换。

- **工资单和归属事件**通常只提供 PDF 格式的文件，我不会尝试自动提取数据；我会手动转录这些数据，保持输入非常规范，并按对账单上的顺序记录过账。这使得更容易处理。

- **现金交易**：我必须手动输入这些交易。我只会单独记录非食品支出，对于食品支出，我大约每六个月会统计一次钱包余额，并为每个月插入一笔摘要交易，借记现金账户以平衡账户。如果您这样做，手动输入的交易会非常少，每周可能只有几笔（这取决于生活方式选择，对我来说这很有效）。当我外出时，我会在手机上使用 Google Keep 记录这些交易，最终在积累一定数量后进行转录。

### 从 PDF 文件提取数据<a id="extracting-data-from-pdf-files"></a>

我在将 PDF 文件中的数据转换方面取得了一些进展，这是一个常见需求，但并不完整；事实证明，完全自动化从 PDF 中提取表格数据在一般情况下并不容易。我有一些接近工作的代码，会在时机合适时发布。此外，我发现的最佳开源解决方案是一个名为 [<u>TabulaPDF</u>](http://tabula.technology/) 的工具，但您仍需要手动识别页面上的数据表位置；您可以使用其姊妹项目 [<u>tabula-java</u>](https://github.com/tabulapdf/tabula-java) 自动化一些抓取操作。

尽管如此，我的导入器通常能成功地从转换为文本的 PDF 对账单中识别出它们属于哪个机构并提取文档的发行日期。

最后，有许多不同的工具用于从 PDF 文档中提取文本，例如 [<u>PDFMiner</u>](https://pypi.python.org/pypi/pdfminer2)、[<u>LibreOffice</u>](https://www.libreoffice.org)、[<u>xpdf</u>](http://www.tutorialspoint.com/unix_commands/pdftotext.htm) 库、[<u>poppler</u>](https://poppler.freedesktop.org/) 库[^3] 等等……但没有一个工具能在所有输入文档上一致工作；您可能最终会安装多个工具，并依赖于不同工具处理不同的输入文件。出于这个原因，我不要求 Beancount 依赖 PDF 转换工具。您应该测试哪些工具适用于您的特定文档，并在导入器实现中调用这些工具。

## 工具<a id="tools"></a>

Beancount 提供了三个工具来协调导入过程的三个阶段：

1. [**<u>bean-identify</u>**](https://github.com/beancount/beancount/tree/v2/bin/bean-identify)：给定一组混乱的下载文件（例如在 ~/Downloads 目录中），自动识别哪些配置的导入器能够处理它们并打印出来。用于调试和检查配置是否正确关联适合的导入器处理每个下载文件；

2. [**<u>bean-extract</u>**](https://github.com/beancount/beancount/tree/v2/bin/bean-extract)：提取每个文件中的交易和对账日期（如果可能）。生成一些 Beancount 输入文本，以便移动到您的输入文件中；

3. [**<u>bean-file</u>**](https://github.com/beancount/beancount/tree/v2/bin/bean-file)：将下载文件归档到与账户结构相对应的目录层次结构中，以便保存，例如在个人 git 仓库中。文件名被清理，文件被移动，并为每个文件预先添加相关日期，以便 Beancount 可以生成相应的 Document 指令。

### 调用<a id="invocation"></a>

所有工具接受相同的输入参数：

```plaintext
bean-<tool> <config> <downloads-dir>
```

例如：

```plaintext
bean-extract blais.config ~/Downloads
```

文件工具接受一个额外的选项，允许用户决定将文件移动到哪个目录，例如：

```plaintext
bean-file -o ~/accounting/documents blais.config ~/Downloads
```

默认行为是将文件移动到配置文件所在的目录。

## 配置<a id="configuration"></a>

前面介绍的工具协调了导入过程，但它们并不会做很多具体的工作来处理单个下载文件。它们调用导入器对象的方法。您必须提供一个导入器列表；这个列表是导入过程的配置（没有它，这些工具不会做任何有用的事情）。

对于找到的每个文件，将调用每个导入器以断言它是否能够处理该文件。如果认为可以处理，可以调用方法生成交易列表，提取日期，或生成下载文件的清理后的文件名。

配置应该是一个 Python3 模块，在其中实例化导入器并将列表分配给模块级的 `CONFIG` 变量，如下所示：

```python
#!/usr/bin/env python3
from myimporters.bank import acmebank
from myimporters.bank import chase
…

CONFIG = [
  acmebank.Importer(),
  chase.Importer(),
  …
]
```

当然，由于您正在编写一个 Python 脚本，您可以在其中插入任何其他代码。所有重要的是这个 `CONFIG` 变量引用一个符合导入器协议的对象列表（将在下一节中描述）。它们的顺序无关紧要。

特别是，尽量将导入器编写得尽可能通用，并用您的输入文件中使用的特定账户名称参数化。这有助于保持代码独立于特定账户，并且我发现这有助于提高清晰度。

或者不这样做……归根结底，这些导入器代码存在于您个人的地方，而不是与 Beancount 一起。如果您愿意，可以让它们保持凌乱且不可共享。

### 从输入文件配置<a id="configuring-from-an-input-file"></a>

一个有趣的想法是使用 Beancount 输入文件推断导入器的配置。如果您想尝试并进行一些修改，可以从导入配置 Python 配置中加载输入文件，使用 API 的 `beancount.loader.load_file()` 函数。

## 编写导入器<a id="writing-an-importer"></a>

每个导入器必须遵循特定协议并至少实现其中一些方法。此协议的完整细节最好在源代码中找到：[<u>importer.py</u>](https://github.com/beancount/beancount/tree/v2/beancount/ingest/importer.py)。上述工具将负责查找下载文件并调用导入器对象上的适当方法。

以下是您需要实现或可能希望实现的方法的简要摘要：

- `name()`：此方法为每个导入器实例提供一个唯一标识。能够用唯一名称引用导入器很方便；例如，在识别过程中会打印出来。

- `identify()`：此方法仅在此导入器能够处理给定文件时返回 true。您必须实现此方法，所有工具都调用此方法以确定（文件，导入器）对的列表。

- `extract()`：此方法尝试从文件内容中提取一些 Beancount 指令。它必须通过实例化 `beancount.core.data` 中定义的对象来创建指令并返回它们。

- `file_account()`：此方法返回与此导入器关联的根账户。这是下载文件将由归档脚本移动到的位置。

- `file_date()`：如果可以从对账单内容中提取日期，请在此返回。这对带日期的 PDF 对账单非常有用……通常可以使用正则表达式从转换为文本的 PDF 中 grep 出日期。这允许归档脚本预先添加相关日期，而不是使用下载文件的日期（默认）。

- `file_name()`：最方便的是不需要重命名下载文件。通常，从银行生成的文件都有唯一的名称，当您下载多个文件并且名称冲突时，它们最终会被浏览器重命名。此函数用于导入器提供一个“漂亮”的名称以归档下载。

基本上，您在 `PYTHONPATH` 上的某个地方创建一个模块——任何您喜欢的地方，某个私人地方——并实现一个类，如下所示：

```python
from beancount.ingest import importer

class Importer(importer.ImporterProtocol):

    def identify(self, file):
        …

    # 重写其他方法…
```

通常，我在专用于每个导入器的目录中创建我的导入器模块文件，以便可以将示例输入文件都放在该目录中进行回归测试。

### 回归测试导入器<a id="regression-testing-your-importers"></a>

随着时间的推移，我发现回归测试对于保持导入器代码的正常工作至关重要。导入器通常是针对没有官方规范的文件格式编写的，经常会出现意想不到的惊喜。例如，我有一些 XML 文件，其中包含一些未转义的“&”字符，这需要针对该银行的自定义修复[^4]。我还看到一个折扣券经纪公司在 MM/DD/YY 和 DD/MM/YY 之间切换日期格式；那个导入器现在需要能够处理这两种类型。所以您做了必要的调整，最终发现其他东西坏了；这并不理想。特别是时间点很烦人：通常是在您尝试更新账本时出现问题：您有其他事情要做。

测试这些导入器的最简单、最懒惰且最相关的方法是使用一些**真实数据文件**并将导入器提取的内容与预期输出进行比较。为了使导入器至少具有某种可靠性，您确实需要能够在一些真实输入上重现提取结果。由于输入文件是如此不可预测且定义不完善，编写详尽的测试以涵盖所有可能性并不实际。在实践中，我每隔几个月就不得不对某些导入器进行一些修复，通过这个过程，只需花费大约半小时的时间：将导致问题的新下载文件添加到导入器目录中，局部运行以测试代码，修复代码。然后，我会在该目录中的*所有*先前下载的测试输入（旧的和新的）上运行测试，以确保我的导入器仍能按预期在旧文件上工作。

在 [<u>beancount.ingest.regression</u>](https://github.com/beancount/beancount/tree/v2/beancount/ingest/regression.py) 中对自动化这个过程提供了一些支持。我们想要的是一些例程，它将列出导入器的包目录，识别要用于测试的输入文件，并生成一个单元测试套件，将导入器方法生成的输出与放置在测试文件旁边的“预期文件”内容进行比较。

例如，给定一个包含导入器实现和两个示例输入文件的包：

```plaintext
/home/joe/importers/acmebank/__init__.py   <- 代码在这里
/home/joe/importers/acmebank/sample1.csv
/home/joe/importers/acmebank/sample2.csv
```

您可以将此代码放入 Python 模块（`__init__.py` 文件）中：

```python
from beancount.ingest import regression
…
def test():
    importer = Importer(...)
    yield from regression.compare_sample_files(importer)
```

如果您的导入器覆盖了 `extract()` 和 `file_date()` 方法，这将生成四个单元测试，这些测试将由 [<u>pytest</u>](https://docs.pytest.org/en/latest/) 自动运行：

1. 调用 `extract()` 在 `sample1.csv` 上，打印提取的条目为字符串，并将此字符串与 `sample1.csv.extract` 文件的内容进行比较。

2. 调用 `file_date()` 在 `sample1.csv` 上，并将日期与 `sample1.csv.file_date` 文件中的日期进行比较。

3. 类似于（1），但在 `sample2.csv` 上。

4. 类似于（2），但在 `sample2.csv` 上。

#### 生成测试输入<a id="generating-test-input"></a>

最初，包含预期输出的文件不存在。当缺少预期输出文件时，回归测试会自动从提取的输出生成这些文件。这将生成以下文件列表：

```plaintext
/home/joe/importers/acmebank/__init__.py   <- 代码在这里
/home/joe/importers/acmebank/sample1.csv
/home/joe/importers/acmebank/sample1.csv.extract
/home/joe/importers/acmebank/sample1.csv.file_date
/home/joe/importers/acmebank/sample2.csv
/home/joe/importers/acmebank/sample2.csv.extract
/home/joe/importers/acmebank/sample2.csv.file_date
```

您应该检查预期输出文件的内容，以视觉断言它们是否代表下载文件的内容。

如果再次运行测试，并且这些文件存在，预期输出文件将用作测试的输入。如果内容在将来有所不同，测试将失败并生成错误。（您现在可以通过手动编辑并插入一些意外数据来测试这一点。）

当您编辑源代码时，可以随时重新运行测试，以确保它仍能在这些旧文件上正常工作。当新下载的文件失败时，您可以重复上述过程：将其复制到该目录中，修复导入器，运行它，检查预期文件。就是这样[^5]。

#### 逐步改进<a id="making-incremental-improvements"></a>

有时我会对导入器进行改进，导致即使在旧文件中也会生成更多或更好的输出，因此所有旧测试现在都会失败。处理这种情况的一个好方法是将所有这些文件保存在版本控制中，本地删除所有预期文件，运行测试以重新生成新文件，然后与最近的提交进行比较，以检查更改是否符合预期。

### 缓存数据<a id="caching-data"></a>

某些二进制文件的数据转换可能成本高且速度慢。通常情况下，将 PDF 文件转换为文本就是这样[^6]。这尤其痛苦，因为在处理下载数据的过程中，我们通常会多次运行工具——如果一切顺利，至少运行两次：一次提取，一次归档——如果有问题，通常会多次运行。因此，我们需要缓存这些转换，以免必须两次运行痛苦的 40 秒 PDF 到文本转换。

Beancount 旨在为下载文件的转换提供两个级别的缓存：

1. 转换的内存缓存，以便多个导入器请求相同转换时只运行一次转换，以及

2. 转换的磁盘缓存，以便工具的多次调用可以重复使用。

#### 内存缓存<a id="in-memory-caching"></a>

内存缓存的工作方式如下：您的方法接收给定文件的包装对象，并调用包装对象的 `convert()` 方法，提供一个转换器 callable 函数。

```python
class MyImporter(ImporterProtocol):
    ...
    def extract(self, file):
        text = file.convert(slow_convert_pdf_to_text)
        match = re.search(..., text)
```

此转换自动记忆：如果两个导入器或两个不同的方法在文件上使用相同的转换器，则转换只运行一次。这是处理内存中冗余转换的简单方法。确保始终通过 `.convert()` 方法调用这些转换，并共享转换器函数以利用这一点。

#### 磁盘缓存<a id="on-disk-caching"></a>

目前，Beancount 仅实现了（1）。磁盘缓存将在以后实现。*跟踪此[<u>票据</u>](https://github.com/beancount/beancount/issues/113)以获取状态更新。*

## 组织文件<a id="organizing-your-files"></a>

本文档中描述的工具在允许您指定以下内容方面非常灵活：

- **导入配置**：提供导入器对象列表的 Python 文件；

- **导入器实现**：实现各个导入器及其回归测试文件的 Python 模块；

- **下载目录**：下载文件所在的目录；

- **归档目录**：下载文件的目标存放目录。

您可以从任何您想要的位置指定这些内容。尽管如此，有些人经常问如何组织他们的文件，所以我在 `beancount/examples/ingest/office` 下提供了一个模板示例，并在此描述。

我建议您创建一个 Git 或 Mercurial[^7] 版本控制的存储库，按照以下结构：

```plaintext
office
├── documents
│   ├── Assets
│   ├── Liabilities
│   ├── Income
│   └── Expenses
├── importers
│   ├── __init__.py
│   └── …
│       ├── __init__.py
│       ├── sample-download-1.csv
│       ├── sample-download-1.extract
│       ├── sample-download-1.file_date
│       └── sample-download-1.file_name
├── personal.beancount
└── personal.import
```

根目录“office”是您的存储库。它包含您的账本文件（`personal.beancount`）、您的导入配置（`personal.import`）、您的自定义导入器源代码（`importers/`）以及您的文档历史（`documents/`），这些文档应由 bean-file 组织良好。您总是在这个根目录中运行命令。

将文档存储在与导入器源代码相同的存储库中的一个优势是，您可以将回归测试文件符号链接到 `documents/` 目录下的某些文件。

您可以通过运行 identify 检查配置：

```plaintext
bean-identify example.import ~/Downloads
```

如果有效，您可以一次性从下载的文件中提取交易：

```plaintext
bean-extract -e example.beancount example.import ~/Downloads > tmp.beancount
```

然后打开 tmp.beancount 并将其内容移动到您的 personal.beancount 文件中。

完成后，可以将下载的文件存档，如下所示：

```plaintext
bean-file example.import ~/Downloads -o documents
```

如果我的导入器有效，我通常甚至不会打开那些文件。您可以使用 `--dry-run` 选项在执行之前测试移动目标。

要运行自定义导入器的回归测试，请使用以下命令：

```plaintext
pytest -v importers
```

个人而言，我在根目录中有一个 Makefile，其中包含这些目标以使生活更轻松。请注意，您需要安装 “pytest”，它是一个测试运行器；它通常打包为 “python3-pytest” 或 “pytest”。

## 示例导入器<a id="example-importers"></a>

除了上述文档外，我还为一个虚构的投资账户 CSV 文件格式编写了一个示例导入器。请参阅[<u>此目录</u>](https://github.com/beancount/beancount/tree/v2/examples/ingest/office/importers/utrade/)。

还有一个示例导入器使用外部工具（PDFMiner2）将 PDF 文件转换为文本，以识别它并从中提取对账单日期。请参阅[<u>此目录</u>](https://github.com/beancount/beancount/tree/v2/examples/ingest/office/importers/acme/)。

Beancount 还附带了一些非常基本的通用导入器。请参阅[<u>此目录</u>](https://github.com/beancount/beancount/tree/v2/beancount/ingest/importers/)。

- 有一个简单的 OFX 导入器，对我来说已经工作了很长时间。虽然它非常简单，但我已经使用了多年，足够从大多数信用卡账户中提取信息。

- 还有一些混合类，您可以将其混入导入器实现中以使其更方便；这些是 LedgerHub 项目的遗留物——您实际上不需要使用它们——可以帮助过渡。

最终，我计划在此框架中构建并提供一个通用的 CSV 文件解析器，以及一个 QIF 文件解析器，以允许从 Quicken 过渡到 Beancount。（我需要示例输入来完成此任务；如果您愿意分享您的文件，我可以使用它来构建这个，因为我没有任何真实的输入，我不使用 Quicken。）在某个时刻构建一个 GnuCash 的转换器也会很好；这也会放在这里。

## 清理<a id="cleaning-up"></a>

### 自动分类<a id="automatic-categorization"></a>

首次使用的用户经常问到的一个常见问题是“如何自动为仅有一方的导入交易分配类别？”例如，从信用卡账户导入交易通常只提供一个过账，如下所示：

```plaintext
2016-03-18 * "UNION MARKET"
  Liabilities:US:CreditCard    -12.99 USD
```

对于此类交易，您必须手动插入一个支出过账，如下所示：

```plaintext
2016-03-18 * "UNION MARKET"
  Liabilities:US:CreditCard    -12.99 USD
  Expenses:Food:Grocery
```

人们通常认为这样做很费时。

我的标准答案是，虽然这样做很有趣，但如果您有一个配置了账户名称自动完成的文本编辑器，手动执行此操作非常轻松，实际上不需要自动化。通过自动化这个过程，您不会节省太多时间。而且我个人喜欢逐个查看交易，检查它们是什么，有时添加评论（例如，我和谁一起吃晚餐，那个 Amazon 收费是为了什么，等等），然后分类。

这可以通过让用户提供一些简单的规则或使用过去交易的历史记录来为简单的学习分类器提供输入来解决。

*Beancount 目前不提供自动分类交易的机制。您可以将其构建到导入器代码中。我想提供一个钩子，允许用户注册一个完成函数，该函数可以在所有导入器中运行，您可以在其中挂接该代码。*

### 清理收款人<a id="cleaning-up-payees"></a>

下载文件中的收款人通常是丑陋的名称：

- 有时它们是企业的法定名称，通常与您去的地方的街道名称不一致，原因多种多样。例如，我最近在纽约一家名为“Lucky Bee”的餐馆吃饭，而 OFX 文件中的备忘录是“KING BEE”。

- 名称有时会缩写或包含一些杂质。在前面的示例中，实际备忘录是“KING BEE NEW YO”，其中“NEW YO”是一个被截断的位置字符串。

- 数据源之间的丑陋程度不一致。

最好能够在导入时规范化收款人名称。我认为可以通过一些简单的规则将正则表达式映射到用户提供的名称来完成大部分工作。实际上没有好的自动方法来获取收款人的“干净名称”。

*Beancount 目前不提供让您这样做的钩子。它最终会提供。您也可以构建一个插件，在加载账本时重命名这些账户。我也会构建它——这很简单，会产生更好的输出。*

## 未来工作<a id="future-work"></a>

我真的想添加的东西列表，除了巩固已经存在的内容之外：

- 一个可以实例化的通用可配置 CSV 导入器。我计划玩一下这个，构建一个嗅探器，可以自动确定每列的角色。

- 一个钩子，允许您注册一个跨所有导入器进行事务后处理的回调。

## 相关讨论线程<a id="related-discussion-threads"></a>

- [<u>入门；为银行 .csv 数据分配账户</u>](https://groups.google.com/d/msg/ledger-cli/u648SA1o-Ek/DzZmu8wVCAAJ)

- [<u>LedgerHub 的状态……如何开始？</u>](https://groups.google.com/d/msg/beancount/qFZvGBLuJos/WSaNY0sEc-wJ)

- [<u>Rekon 需要您的 CSV 文件</u>](https://groups.google.com/d/msg/ledger-cli/n_WNc-tZabU/sh09irl-C-kJ)

## 历史备注<a id="historical-note"></a>

曾经有一个实现本文档中描述的过程的第一个实现。该项目称为 LedgerHub，于 2016 年 2 月停用，重写后将生成的代码集成到 Beancount 本身的 [<u>beancount.ingest</u>](https://github.com/beancount/beancount/tree/v2/beancount/ingest/) 库中。原始项目旨在包括各种导入器的实现以与其他人共享，但这种共享并不成功，因此重写仅包括构建自己的导入器并调用它们的框架，以及非常有限数量的示例导入器实现。

关于 LedgerHub 的文档被保留，可以帮助您理解 Beancount 导入器支持的起源和设计选择。它们可以在这里找到：

- [<u>原始设计</u>](ledgerhub_design_doc.md)

- [<u>原始说明和最终状态</u>](http://furius.ca/beancount/doc/ledgerhub/manual)（本文档的旧版本）

- [<u>关于项目终止原因的分析</u>](http://furius.ca/beancount/doc/ledgerhub/postmortem)（事后分析）

[^1]: 本质上，有三种概念模式来输入此类交易：（1）用户手动编写单个交易，（2）用户将两个账户的两边作为单个交易输入，以及（3）两个单独的交易自动合并为单个交易。这些模式是相互对立的。故事中的一个曲折是相同的交易通常在其各自账户中发布的日期不同。Beancount 目前[2016 年 3 月] 不支持单个交易的多个日期，但正在讨论实现这些输入模式的支持。有关详细信息，请参阅[<u>本文档</u>](settlement_dates_in_beancount.md)。

[^2]: 您能在自由软件世界中找到的最接近通用下载器的是用于 OFX 文件的 [<u>ofxclient</u>](https://github.com/captin411/ofxclient)，而在商业世界中，[<u>Yodlee</u>](http://www.yodlee.com/) 提供了一项连接到许多金融机构的服务。

[^3]: poppler 中的 `pdftotext` 实用程序提供了有用的 `-layout` 标志，该标志输出不破坏表格的文本文件，这在每行一个交易的正常情况下非常有用。

[^4]: 在向他们发送了一些详细的电子邮件并没有得到回复或看到下载文件发生任何变化之后，我放弃了他们修复问题的希望。

[^5]: 正如您所看到的，这个过程部分是我不共享导入器代码的原因。为了保持导入器正常工作，存储过多的个人数据是必须的。

[^6]: 我不太明白为什么，因为打开查看几乎是瞬间的，但几乎所有将它们转换为其他格式的工具都非常慢。

[^7]: 我个人更喜欢 Mercurial，因为它的命令和输出更清晰，并且它的可扩展性更强，但 Git 的存储模型的一个优点是移动文件是免费的（不需要额外的副本）。在 Mercurial 存储库中移动文件会消耗一些存储空间。如果您重命名账户或更改文件组织方式，您可能最终会复制许多大文件。
