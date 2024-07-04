# Beancount 示例和教程<a id="title"></a>

Martin Blais，2014年10月

[<u>http://furius.ca/beancount/doc/example</u>](http://furius.ca/beancount/doc/example)

> [<u>示例文件生成器</u>](#example-file-generator)
>
> [<u>转换为 Ledger 输入</u>](#converting-to-ledger-input)
>
> [<u>示例用户概况</u>](#profile-of-example-user)
>
> [<u>未来添加</u>](#future-additions)
>
> [<u>教程</u>](#tutorial)
>
> [<u>生成示例文件</u>](#generate-an-example-file)
>
> [<u>生成报告</u>](#generating-reports)
>
> [<u>生成余额</u>](#generating-balances)
>
> [<u>生成资产负债表和收益表</u>](#generating-a-balance-sheet-and-income-statement)
>
> [<u>日记账</u>](#journals)
>
> [<u>持有量</u>](#holdings)
>
> [<u>其他报告</u>](#other-reports)
>
> [<u>其他格式</u>](#other-formats)
>
> [<u>通过网络界面查看报告</u>](#viewing-reports-through-the-web-interface)
>
> [<u>Beancount 报告的未来</u>](#the-future-of-beancount-reports)

## 示例文件生成器<a id="example-file-generator"></a>

Beancount 有一个命令可以生成几年的真实用户的历史条目：

```shell
bean-example > example.beancount
```

脚本会检查输出是否在 Beancount 中无法正常处理（如果发生这种情况，请尝试重新运行或联系作者 [<u>blais@furius.ca</u>](mailto:blais@furius.ca)）。请注意，有一个选项可以固定用于生成条目的随机生成器种子。

您可以为此用户生成多年的分类账数据；请参阅以下命令了解可用选项：

```shell
bean-example --help
```

该脚本的目的是：

-   提供一个真实的匿名数据集，供用户试验、生成报告等，以测试 Beancount 的功能。
-   与 Ledger 生成的报告进行比较。
-   作为本教程中使用的示例文件。
-   作为 Beancount 压力测试的输入。

### 转换为 Ledger 输入<a id="converting-to-ledger-input"></a>

可以将脚本生成的示例文件转换为 Ledger 语法，如下所示：

```shell
bean-report example.beancount ledger > example.lgr
```

如果在转换过程中遇到任何问题，请告诉我。虽然我有一个自动化测试来检查转换是否成功，但我主要关注 Beancount 本身，除了偶尔进行比较或在邮件列表上说明问题外，我并不经常使用 Ledger。

## 示例用户概况<a id="profile-of-example-user"></a>

以下是此用户的高层次概况。在接下来的文本中，为了简洁，我将使用男性代词“他”。

-   他住在美国某个地方，使用单一的操作货币：“USD”。他租住一间公寓。

-   **收入**：他是一名软件工程师（对不起，我忍不住），从一家软件公司获得一份适度的工资收入。
    -   他每两周领取一次工资。
    -   他最大限度地向公司的 401k 计划缴款。公司会为他的缴款提供匹配。
    -   他的工资以直接存款的形式支付到他的支票账户。
    -   他享受雇主提供的健康保险计划并支付保费。
    -   他明确跟踪他的假期小时数。

-   **支出**：
    -   **支票账户**：他通过支票支付公寓租金。他还从支票账户直接支付电费、互联网 ISP 和手机账单。他的银行也会收取月度银行费用。
    -   **信用卡**：他还有其他定期支出，如杂货和与朋友外出就餐，这些费用通过信用卡支付。

-   **资产**：
    -   **银行**：他在一家主要的美国银行有一个账户。
    -   **退休账户**：他的 401k 账户持有在 Fidelity 或 Vanguard，全部分配到两只基金中，混合股票和债券。此账户从未有任何销售。此账户按平均成本持有。
    -   **其他投资**：他能够储蓄的额外资金从支票账户转移到应税投资账户，在该账户中购买股票和 ETF 的组合，并偶尔进行销售和实现利润。

-   **负债**：
    -   **信用卡**：他有一到两张由另一家银行发行的信用卡。

-   **事件/标签**：他每年会短暂度假一到两次，到有趣的地方旅行。

### 未来添加<a id="future-additions"></a>

请注意，随着我加强示例生成器脚本，此概况会变得更加复杂。如果您希望在示例文件中看到 [<u>烹饪书</u>](command_line_accounting_cookbook.md) 或其他情况，请告诉我（提交补丁是通知我的最佳方式）。

-   生成虚假的 PDF 文件（可选）以及适合这些文件的 document 指令，大部分是隐式的，但有些是显式的。
-   生成更多支出：
    -   一些现实的现金支出。
    -   添加一辆汽车：汽车维修、加油等。移除电车示例，它过于城市化。
    -   添加慈善捐款。
    -   添加第二张信用卡并变换支出的来源。
-   添加一些链接的实际用途，例如 401k 匹配，雇主的贡献应与员工的贡献链接。或其他用途。
-   生成贷款和学生贷款的还款。拥有大额负债是常见情况，我想表示这一点，即使不是抵押贷款。
-   将退休账户转换为按平均成本法记账，并添加相应的费用，否则无法通过明确的批次进行跟踪。
-   根据烹饪书添加医疗费用的跟踪。
-   添加其他货币的外国账户及该货币的交易和转账。

## 教程<a id="tutorial"></a>

本节提供了使用 Beancount 生成报告的基本示例，基于生成的典型示例文件。这里的报告列表并不详尽。

您可以按照教程生成自己的示例文件，或者查看 [<u>beancount/examples/tutorial</u>](http://github.com/beancount/beancount/tree/v2/examples/tutorial/) 中的文件。

### 生成示例文件<a id="generate-an-example-file"></a>

首先生成示例文件：

```shell
bean-example > example.beancount
```

打开 `example.beancount` 并检查它。

接下来，在生成报告之前，先验证文件是否加载没有任何错误：

```shell
bean-check example.beancount
```

如果一切正常，它将静默返回，不会输出任何内容（bean-check 仅在存在错误时写入错误信息，成功时不写入任何内容）。

### 生成报告<a id="generating-reports"></a>

让我们生成所有账户的最终余额报告 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/balances.output)：

```shell
bean-report example.beancount balances
```

正如您所看到的，bean-report 脚本有各种生成报告的子命令。要列出可用报告，请使用 --help-reports [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/help-reports.output)：

```shell
bean-report --help-reports
```

要列出特定报告的可用选项，请使用 --help [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/help-subcmd.output)：

```shell
bean-report example.beancount balances --help
```

每种报告类型的特定选项各不相同。

您还可以列出脚本的全局选项 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/help-global.output)：

```shell
bean-report --help
```

全局选项允许您指定报告的所需输出格式。要查看可用报告的输出格式，请使用 --help-formats [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/help-formats.output)：

```shell
bean-report --help-formats
```

### 生成余额<a id="generating-balances"></a>

我们已经知道如何生成所有账户的余额报告。这是一个非常详细的账户列表。让我们将输出限制在我们感兴趣的账户上 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/balances-restrict.output)：

```shell
bean-report example.beancount balances -e ETrade
```

在这里，您可以查看每个投资子账户中持有的单位数量。要渲染成本，请使用 --cost 选项 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/balances-restrict-cost.output)：

```shell
bean-report example.beancount balances -e ETrade --cost
```

### 格式化工具<a id="formatting-tools"></a>

有时将账户的层次列表渲染为树状结构很有用。您可以使用 Beancount 提供的 “treeify” 工具来实现这一点：

```shell
bean-report example.beancount balances | treeify
```

此工具适用于任何看起来像账户名称列的数据列（您也可以将其配置为处理文件名或其他模式）。

### 生成资产负债表和收益表<a id="generating-a-balance-sheet-and-income-statement"></a>

让我们生成资产负债表：

```shell
bean-report example.beancount balsheet
```

不幸的是，目前唯一支持的输出格式是 HTML。此外，不支持从命令行过滤资产负债表条目。生成文件并在浏览器中打开 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/balsheet.output)：

```shell
bean-report example.beancount balsheet > /tmp/balsheet.html
```

您可以对收益表执行相同操作：

```shell
bean-report example.beancount income > /tmp/income.html
```

### 日记账<a id="journals"></a>

您还可以生成日记账（在 Ledger 术语中，这些是“注册表”）。例如，让我们看看支票账户的过账 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/journal.output)：

```shell
bean-report example.beancount journal -a Assets:US:BofA:Checking
```

要渲染运行余额列，请添加 --balance 选项 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/journal-with-balance.output)：

```shell
bean-report example.beancount journal -a Assets:US:BofA:Checking --balance
```

库存以持有单位数量渲染 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/invest.output)：

```shell
bean-report example.beancount journal -a Assets:US:ETrade:GLD
```

如果您想按成本渲染值，请使用 --cost 选项 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/invest-with-cost.output)：

```shell
bean-report example.beancount journal -a Assets:US:ETrade:GLD --cost
```

要渲染日记账，您通常会限制要查看的过账集。如果不这样做，过账将不会显示变化，因为交易的所有部分都会应用于运行余额，总额为零。但请尝试如下所示 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/journal-unrestricted.output)：

```shell
bean-report example.beancount journal --balance
```

### 持有量<a id="holdings"></a>

有多种方法可以获取持有量的汇总列表。列出详细持有量 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/holdings.output)：

```shell
bean-report example.beancount holdings
```

按账户汇总持有量，如下所示 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/holdings-by-account.output)：

```shell
bean-report example.beancount holdings --by=account
```

因为在实际账户中创建和使用专用 Beancount 子账户名称是常见做法，我们可以进一步细化，汇总到这些子账户的父级（“根”）账户 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/holdings-by-root-account.output)：

```shell
bean-report example.beancount holdings --by=root-account
```

我们可以按商品汇总持有量 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/holdings-by-commodity.output)：

```shell
bean-report example.beancount holdings --by=commodity
```

或者按商品报价的货币汇总持有量，这让我们一瞥我们的货币敞口 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/holdings-by-currency.output)：

```shell
bean-report example.beancount holdings --by=currency
```

最后，我们可以汇总所有持有量以获得净资产 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/networth.output)：

```shell
bean-report example.beancount networth
```

有一些将金额转换为通用货币的附加选项。请参阅帮助了解详情。

### 其他报告<a id="other-reports"></a>

还有许多其他杂项报告可用。试试其中一些。

列出所有账户 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/accounts.output)：

```shell
bean-report example.beancount accounts
```

列出事件 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/events.output)：

```shell
bean-report example.beancount events
```

按类型列出指令数量 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/stats-directives.output)：

```shell
bean-report example.beancount stats-directives
```

按类型列出过账数量 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/stats-postings.output)：

```shell
bean-report example.beancount stats-postings
```

### 其他格式<a id="other-formats"></a>

报告可能支持各种输出格式。您可以使用 --format (-f) 全局选项更改输出格式 [<u>\[output\]</u>](http://furius.ca/beancount/examples/tutorial/holdings-csv.output)：

```shell
bean-report -f csv example.beancount holdings
```

请注意，“beancount”和“ledger”是有效的输出格式：它们指的是 Beancount 和 Ledger 输入语法。

### 通过网络界面查看报告<a id="viewing-reports-through-the-web-interface"></a>

访问 Beancount 报告的原始方式是通过在本地计算机上提供的网络接口。像这样提供示例文件：

```shell
bean-web example.beancount
```

然后使用网络浏览器导航到 [<u>http://localhost:8080</u>](http://localhost:8080)。从那里，您可以点击任何数量的过滤视图并访问之前演示的一些报告。例如，点击年份视图；这将提供资产负债表、收益表和其他报告的子集。

bean-web 工具有许多用于限制提供内容的选项。（请注意，默认情况下服务器仅监听来自您计算机的连接；如果您想在网络服务器上托管并接受来自任何地方的连接，请使用 `--public`）。

*注意：有一个独立的项目提供了比 Beancount 自带的网络界面更好的网络界面：[<u>Fava</u>](https://github.com/aumayr/fava)。您可能想查看一下。*

## Beancount 报告的未来<a id="the-future-of-beancount-reports"></a>

从定制的角度来看，我发现上面的命令行报告相当不令人满意。它们让我很烦恼。这主要是因为我对 Beancount 的网络界面的功能感到满意。当我越来越多地使用命令行界面时，我发现自己渴望有一种更明确和定义良好的方式来应用我从网络界面获得的所有过滤器，甚至更多。出于这个原因，我[<u>目前正在实现类似 SQL 的查询语言</u>](http://furius.ca/beancount/doc/proposal-filtering)。这种语法将允许我删除所有自定义报告，并用单一一致的查询语法替代它们。这项工作正在进行中。

所有持有量报告甚至它们的内部对象都将由类似 SQL 的查询语法替代。持有量不需要单独处理：它们只是特定时间点的可用头寸列表，结合查询语法中的适当访问器和对其多个列的聚合支持，我们应该能够生成所有这些报告并简化代码。

*— Martin Blais，2014年10月*
