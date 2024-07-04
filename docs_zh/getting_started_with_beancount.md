# Beancount 入门指南<a id="title"></a>

[<u>Martin Blais</u>](mailto:blais@furius.ca)，2014年7月

[<u>http://furius.ca/beancount/doc/getting-started</u>](http://furius.ca/beancount/doc/getting-started)

## 介绍<a id="introduction"></a>

本文档是一个关于创建您第一个 Beancount 文件的简明指南，介绍了一些选项的初始化、组织文件的指南以及声明账户和确保其初始余额不会引发错误的说明。如果您使用 Emacs 作为文本编辑器，这里也有一些配置的材料。

您可能需要先阅读一些[<u>用户手册</u>](beancount_language_syntax.md)以熟悉语法和可用指令的种类，或者如果您已经设置了文件或知道如何设置，可以继续阅读[<u>食谱</u>](command_line_accounting_cookbook.md)。如果您熟悉 Ledger，您可能想先了解一下[<u>Ledger 和 Beancount 的区别</u>](a_comparison_of_beancount_and_ledger_hledger.md)。

## 编辑器支持<a id="editor-support"></a>

Beancount 账本是简单的文本文件。您可以使用任何文本编辑器来编写您的输入文件。然而，一个理解足够 Beancount 语法以提供专门功能（如语法高亮、自动补全和自动缩进）的良好文本编辑器，可以极大地提高编写和维护账本的效率。

### Emacs<a id="emacs"></a>

用于在 Emacs 中编辑 Beancount 账本文件的支持传统上与 Beancount 一起分发。现在它作为一个独立项目存在于[<u>这个 Github 仓库</u>](https://github.com/beancount/beancount-mode/)中。

### Vim<a id="vim"></a>

由 Nathan Grigg 实现的在 Vim 中编辑 Beancount 账本文件的支持可在[<u>这个 Github 仓库</u>](https://github.com/nathangrigg/vim-beancount)中找到。

### Sublime<a id="sublime"></a>

Martin Andreas Andersen 贡献了对 Sublime 编辑的支持，可在[<u>这个 Github 仓库</u>](https://github.com/draug3n/sublime-beancount)中找到，或者作为 Sublime 包[<u>这里</u>](https://packagecontrol.io/packages/Beancount)。

### VSCode<a id="vscode"></a>

有许多用于处理 Beancount 文本文件的插件，包括 Lencerf 的[<u>VSCode-Beancount</u>](https://marketplace.visualstudio.com/items?itemName=Lencerf.beancount)。

## 创建您的第一个输入文件<a id="creating-your-first-input-file"></a>

让我们开始创建一个包含两个账户和一笔交易的最小输入文件。将以下输入内容输入或复制到一个文本文件中：

```plaintext
2014-01-01 open Assets:Checking
2014-01-01 open Equity:Opening-Balances

2014-01-02 * "Deposit"
  Assets:Checking           100.00 USD
  Equity:Opening-Balances
```

## 简要语法概述<a id="brief-syntax-overview"></a>

一些注释和 Beancount 语法的简要概述：

- 货币必须完全用大写字母（允许使用数字和一些特殊字符，如“\_”或“-”）。不支持货币符号（如 $ 或 €）。
- 账户名称不允许有空格（可以使用短划线），必须有至少两个组件，用冒号分隔。账户名称的每个组件必须以大写字母或数字开头。
- 描述字符串必须用引号括起来，例如："AMEX PMNT"。
- 日期仅解析为 ISO8601 格式，即 YYYY-MM-DD。
- 标签必须以 "#" 开头，链接以 "^" 开头。

有关语法的完整描述，请参阅[<u>用户手册</u>](beancount_language_syntax.md)。

## 验证您的文件<a id="validating-your-file"></a>

Beancount 的目的是从您的输入文件生成报告（无论是在控制台上还是通过其 Web 界面）。然而，有一个工具可以用来加载其内容并进行一些验证检查，以确保您的输入文件不包含错误。Beancount 可能非常严格；这是一个在您输入数据时使用的工具，以确保您的输入文件是合法的。这个工具叫做“bean-check”，您可以这样调用它：

```bash
bean-check /path/to/your/file.beancount
```

现在试试刚刚创建的文件。它应该没有输出。如果有错误，它们将打印在控制台上。错误以 Emacs 默认识别的格式打印，您可以使用 Emacs 的 `next-error` 和 `previous-error` 内置函数将光标移动到问题所在的位置。

## 查看 Web 界面<a id="viewing-the-web-interface"></a>

查看报告的一种方便方法是使用“bean-web”工具打开您的输入文件。试试：

```bash
bean-web /path/to/your/file.beancount
```

然后您可以将 Web 浏览器指向[<u>http://localhost:8080</u>](http://localhost:8080)并点击 Beancount 生成的各种报告。您可以修改输入文件并重新加载浏览器指向的网页——bean-web 将自动重新加载文件内容。

此时，您可能需要阅读一些[<u>语言语法</u>](beancount_language_syntax.md)文档。

## 如何组织您的文件<a id="how-to-organize-your-file"></a>

本节提供了如何组织文件的一般指南。这假设您已经阅读了[<u>语言语法</u>](beancount_language_syntax.md)文档。

### 输入文件的前言<a id="preamble-to-your-input-file"></a>

我建议您从一个文件开始[^1]。我的文件有一个头部，告诉 Emacs 用什么模式打开文件，接着是一些常用选项：

```plaintext
;; -*- mode: beancount; coding: utf-8; fill-column: 400; -*-
option "title" "My Personal Ledger"
option "operating_currency" "USD"
option "operating_currency" "CAD"
```

title 选项用于报告。“operating currencies”列表标识您最常用的作为“货币”的商品，并且在报告中值得在它们自己的专用列中呈现（此声明对任何计算的行为没有其他影响）。

### 部分和声明账户<a id="sections-declaring-accounts"></a>

我喜欢将我的输入文件组织成与每个实际账户对应的部分。每个部分通过使用 Open 指令定义与该实际账户相关的所有账户。例如，这是一个支票账户：

```plaintext
2007-02-01 open Assets:US:BofA:Savings              USD
2007-02-01 open Income:US:BofA:Savings:Interest     USD
```

我喜欢尽可能声明货币约束，以避免错误。此外，注意我如何声明与此账户相关的收入账户。这有助于在报告税务时分解收入，因为您可能会收到与该特定账户收入相关的税务文件（在美国，这将是您的银行生成的 1099-INT 表格）。

以下是投资账户的开盘账户示例：

```plaintext
2012-03-01 open Assets:US:Etrade:Main:Cash            USD
2012-03-01 open Assets:US:Etrade:Main:ITOT            ITOT
2012-03-01 open Assets:US:Etrade:Main:IXUS            IXUS
2012-03-01 open Assets:US:Etrade:Main:IEFA            IEFA
2012-03-01 open Income:US:Etrade:Main:Interest        USD
2012-03-01 open Income:US:Etrade:Main:PnL             USD
2012-03-01 open Income:US:Etrade:Main:Dividend        USD
2012-03-01 open Income:US:Etrade:Main:DividendNoTax   USD
```

重点是所有这些账户以某种方式相关。食谱的各个部分将描述为每个部分建议创建的账户集合。

并非所有部分都必须这样。例如，我有以下部分：

- **永久账户。** 我在顶部有一个部分，专门用于包含特殊和“永久”账户，例如应付账款和应收账款。
- **日记账。** 我在底部有一个“日记账”部分，其中包含所有现金支出，按时间顺序排列。
- **费用账户。** 我所有的费用账户（类别）都在它们自己的部分中定义。
- **雇主。** 对于每个雇主，我定义了一个部分，其中包含他们的直接存款条目，并跟踪假期、股票归属和其他工作相关的交易。
- **税务。** 我有一个税务部分，按纳税年度组织。

您可以根据自己的喜好组织，因为 Beancount 不关心声明的顺序。

### 关闭账户<a id="closing-accounts"></a>

如果一个实际账户已经关闭，或者将来不会有任何更多的交易记入其中，您可以使用 Close 指令在特定日期声明它“关闭”：

```plaintext
; Moving to another bank.
2013-06-13 close Assets:US:BofA:Savings
```

这告诉 Beancount 在不包含该账户活动日期的报告中不显示该账户。如果您在以后尝试记入它时，它还会触发错误，避免错误。

## 去重<a id="de-duping"></a>

一旦您设置了[<u>某种代码或过程</u>](importing_external_data.md)来自动从下载的文件中提取过账，您将经常遇到一个问题，即导入提供同一交易的两个单独部分的过账。一个例子是通过从支票账户转账支付信用卡余额。如果您下载支票账户的交易，您将提取如下内容：

```plaintext
2014-06-08 * "ONLINE PAYMENT - THANK YOU"
  Assets:CA:BofA:Checking           -923.24 USD
```

信用卡下载将产生以下内容：

```plaintext
2014-06-10 * "AMEX EPAYMENT    ACH PMT"
  Liabilities:US:Amex:Platinum       923.24 USD
```

很多时候，这些账户的交易需要记入费用账户，但在这种情况下，这些是同一交易的两个单独部分：一个转账。当您导入其中一个时，通常会寻找另一个部分并将它们合并在一起：

```plaintext
;2014-06-08 * "ONLINE PAYMENT - THANK YOU"
2014-06-10 * "AMEX EPAYMENT    ACH PMT"
  Liabilities:US:Amex:Platinum       923.24 USD
  Assets:CA:BofA:Checking           -923.24 USD
```

我经常将其中一行描述保留在注释中——这只是我的选择，Beancount 忽略它。还要注意，我必须选择其中一个日期。我只是选择我喜欢的日期，只要它不会破坏任何余额断言。

如果您忘记合并这些导入的交易，请不要担心！这就是余额断言的作用。定期在这些账户中的任意一个放置余额断言，例如每次导入时，您将在两次输入交易时获得一个很好的错误提示。这很常见，一段时间后解释该编译错误并快速修复它将成为第二天性。

最后，当我知道只导入这些的一部分时，我会手动选择另一个账户，并标记我知道稍后将导入的过账，告诉我还没有去重这个交易：

```plaintext
2014-06-10 * "AMEX EPAYMENT    ACH PMT"
  Liabilities:US:Amex:Platinum       923.24 USD
 
