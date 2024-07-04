# Beangulp<a id="title"></a>

[<u>Martin Blais</u>](mailto:blais@furius.ca), 2021年1月

最初，导入 Beancount 数据是通过 LedgerHub 项目以及一个导入库来支持的，然后重新集成为 Beancount 仓库中的一个纯框架库（带有一些示例），称为 beancount.ingest。现在，我们再次分离该仓库，并将导入框架分离到另一个仓库，以便更轻松地进行维护和发展。本文档列出了此新版本中的一些期望更改。

## 新仓库<a id="new-repo"></a>

新仓库将位于

> [<u>http://github.com/beancount/beangulp</u>](http://github.com/beancount/beangulp)

Beangulp 将仅兼容 Beancount v3 分支的最新版本。

预计 Beancount v3 在开始时将快速发展，因此，为了使早期采用者的生活更轻松，应谨慎使用版本号递增和版本化依赖项。理想情况下，Beangulp 应仅依赖 Beancount 的当前次版本，例如，如果 Beancount 3.0.0 发布，Beangulp 应声明：

```plaintext
install_requires: beancount >3.0, <3.1
```

参见 [<u>setuptools 文档</u>](https://setuptools.readthedocs.io/en/latest/userguide/dependency_management.html#declaring-required-dependency) 和 [<u>PEP-440</u>](https://www.python.org/dev/peps/pep-0440/#version-specifiers)。

## 状态<a id="status"></a>

截至 2022年1月，此提案的大部分已完成并实现。（感谢 Daniele Nicolodi 完成了大部分工作。）

## 更改<a id="changes"></a>

### 仅限库<a id="library-only"></a>

当前的实现允许在“配置文件”上使用以下工具，这些配置文件是已评估的 Python：

1.  bean-identify, bean-extract 和 bean-file 工具，或
2.  创建一个脚本并调用一个实现子命令的端点。

为了使这项工作正常运行，使用了一个非常复杂的跳板，将评估跳转到相同的代码。我承认，即使是我自己写的代码，每当我进去编辑时，这也难住了我。它还使用户添加自定义导入过程的前后定制变得不便。

下一版本将仅支持 (2)。bean-identify、bean-extract、bean-file 程序将全部移除。用户将需要编写自己的脚本。

### 单文件<a id="one-file"></a>

目前，每个导入器由两个文件组成：实现文件和相关的测试文件，例如：

```plaintext
soandso_bank.py
soandso_bank_test.py
```

测试文件很小，通常调用库函数查找模型文件和预期输出。由于几乎没有真正的测试代码，我们希望能够有一个包含测试调用的单个 Python 文件。

将在导入器实现中添加一个新端点来定义测试。

### 自运行<a id="self-running"></a>

此外，该新函数还应与用于组装配置脚本的函数相同。换句话说，导入器的 main() 函数应等同于使用用于测试的配置调用的单个配置导入器。

这将允许用户在特定文件集上“运行导入器”，而无需定义配置，使用测试配置，如下所示：

```plaintext
soandso_bank.py extract ~/Downloads/transactions.csv
```

拥有此选项使得人们可以共享单个文件并立即进行测试，而无需创建配置或主程序。

我认为没有简单干净的方法来实现这一点，除非在导入器定义文件中有类似以下内容：

```python
if __name__ == ‘__main__’:
    main = Ingest([SoAndSoBankImporter()])
    main()
```

这应该可以立即运行，无需更改。尽管现在导入器通常需要一些配置才能工作（我不确定，我不使用 Beancount 或广泛使用的导入器分发的任何导入器）。

### **测试**子命令和生成**<a id="test-subcommand-generate"></a>

由于导入器是可运行的，并定义了 pytest 运行的测试用例，我们还应添加一个子命令“test”来完成“identify”、“extract”和“file”。这是一个很好的地方来替换 --generate 选项，该选项需要注入 pytestconfig，而是自己实现它并添加第二个子命令：“genexpect”来生成测试的预期文件。

界面变为：

```plaintext
soandso_bank.py identify ~/Downloads/
soandso_bank.py extract ~/Downloads/
soandso_bank.py file ~/Downloads/ ~/documents
soandso_bank.py test ~/documents/testdocs
soandso_bank.py generate ~/Downloads/transactions.csv
```

通过这种方式，我们可以删除 pytestconfig 依赖注入，并简化单元测试的逻辑，单元测试必须处理检查与生成场景。这应该导致更简洁的代码。

### 单一预期文件<a id="one-expected-file"></a>

导入器的预期输出存储在多个文件中，后缀为 .extract, .file_name, .file_date 等。如果我们将所有输出存储在一个文件中，"genexpect" 子命令可以将所有内容生成到 stdout。这非常方便。

```plaintext
myimporter.py
myimporter.beancount
```

利用 Beancount 语法来存储 "file_name()"、"file_date()" 和其他输出的预期值。将这些存储在 Event 或 Custom 指令中，并让测试代码对它们进行断言。测试目录的新内容应为简单的文件对：(a) 原始下载文件和 (b) 预期输出，包含交易以及所有其他方法输出。

### 重复项识别<a id="duplicates-identification"></a>

这从未真正工作正常。我认为如果基于每个导入器分别实现——换句话说，让每个导入器定义如何识别重复项，例如，如果唯一交易 ID 可以作为链接插入以消除歧义——我们可以做得更干净。

理想情况下，每个导入器可以在导入器中指定重复 ID 检测。它可以调用一个更通用但不太可靠的方法，该代码应位于 Beangulp 中。

### CSV 实用工具<a id="csv-utils"></a>

我有很多用于切片和切割 CSV 文件的非常方便的实用工具，这些文件来自丑陋的 CSV 下载。CSV 下载通常用于包含多个表和数据类型，通常需要在这些文件上进行更高级别的处理。

我在各处都有这样的代码。这值得一个漂亮的库。此外，我有一个不错的表处理库，属于 Baskets 项目，尚未得到适当的质量处理。

清理我的小表库，并将其与所有 CSV 实用工具合并。将其放入 beangulp，甚至考虑将其作为自己的项目，包括一个 CSV 导入器。"CSV world."

*在发现 petl 之后，不确定这是否重要。我们可以依赖 petl。*

### 缓存<a id="caching"></a>

在大型文件上运行转换作业时，能够有一个缓存并避免多次运行转换会很好。可以缓存（小）转换后的数据并重新加载，以避免多次运行昂贵的转换。一个困难是，所需的转换取决于导入器配置，每个导入器都不了解其他导入器。

所有命令行参数和至少文件内容的头部应进行哈希处理。该库可以完全独立于 Beancount。

### API 更改<a id="api-changes"></a>

-   "file_date()" 不清楚；"get_filing_date()" 会更清楚。

-   extract() 上的额外参数与所有其他方法相比不规则。找到更好的方法？

-   我不完全确定我喜欢我的小记忆文件包装器（"cache"）与缓存。用仅磁盘缓存的替换它。

### 自动插入<a id="automatic-insertion"></a>

新代码应具备一个非常方便且易于构建的功能，即在文件中某个特殊字符串标记之前的点自动插入提取的输出到现有分类账。这可以作为库的一部分，作为存储导入器输出的替代方案，例如：

```plaintext
$ ./myimport.py extract | bean-insert ledger.beancount Amex
```

这也可以是“extract”的一个标志：

```plaintext
$ ./myimport.py extract -f ledger.beancount -F Amex
```

## Beangulp: Beancount 的灵活导入工具

Beangulp 项目旨在为 Beancount 提供一个灵活且强大的数据导入工具，满足不断变化的需求和简化用户操作的需求。通过优化和简化现有的导入流程，Beangulp 将使导入过程更加高效、直观和易于维护。
