# 运行 Beancount 和生成报告

[马丁·布莱斯](mailto:blais@furius.ca)，2014年7月至9月

[http://furius.ca/beancount/doc/tools](http://furius.ca/beancount/doc/tools)

## 目录

1. [简介](#introduction)
2. [工具](#tools)
   - [bean-check](#bean-check)
   - [bean-report](#bean-report)
   - [bean-query](#bean-query)
   - [bean-web](#bean-web)
   - [全局页面](#global-pages)
   - [视图报告页面](#view-reports-pages)
   - [bean-bake](#bean-bake)
   - [bean-doctor](#bean-doctor)
   - [上下文](#context)
   - [bean-format](#bean-format)
   - [bean-example](#bean-example)
3. [过滤交易](#filtering-transactions)
4. [报告](#reports)
   - [余额报告](#balance-reports)
   - [试算平衡表](#trial-balance-balances)
   - [资产负债表](#balance-sheet-balsheet)
   - [期初余额](#opening-balances-openbal)
   - [损益表](#income-statement-income)
   - [日记账报告](#journal-reports)
   - [日记账](#journal-journal)
   - [按成本渲染](#rendering-at-cost)
   - [添加余额列](#adding-a-balance-column)
   - [字符宽度](#character-width)
   - [精度](#precision)
   - [紧凑、正常或详细](#compact-normal-or-verbose)
   - [摘要](#summary)
   - [等效 SQL 查询](#equivalent-sql-query)
   - [转换](#conversions-conversions)
   - [文档](#documents-documents)
   - [持有报告](#holdings-reports)
   - [持有与聚合](#holdings-aggregations-holdings)
   - [净资产](#net-worth-networth)
   - [其他报告类型](#other-report-types)
   - [现金](#cash)
   - [价格](#prices-prices)
   - [统计](#statistics-stats)
   - [更新活动](#update-activity-activity)

## 简介

本文档描述了处理 Beancount 输入文件的工具以及许多可从中生成的报告。语言的语法在[Beancount 语言语法](beancount_language_syntax.md)文档中描述。本手册仅涵盖从命令行使用 Beancount 的技术细节。

## 工具

### bean-check

**bean-check** 是一个用于验证输入语法和交易正确性的程序。它所做的就是加载输入文件并运行你在其中配置的各种插件，以及一些额外的验证检查。它报告错误（如果有），然后退出。你可以这样运行它：

```bash
bean-check /path/to/my/file.beancount
```

如果没有错误，应该没有输出，它会安静地退出。如果有错误，它们将打印到 stderr，带有文件名、行号和错误描述（Emacs 可以理解的格式，因此你可以使用 `next-error` 和 `previous-error` 来导航到文件）：

```plaintext
/home/user/myledger.beancount:44381:   Transaction does not balance: 34.46 USD

   2014-07-12 * "Black Iron Burger" ""
     Expenses:Food:Restaurant               17.23 USD
     Assets:Cash                            17.23 USD
```

你应该在生成报告之前修复所有错误。

### bean-report

这是用于提取专业报告到控制台的主要工具，可以生成文本或各种其他格式。你可以这样调用它：

```bash
bean-report /path/to/my/file.beancount <report-name>
```

例如：

```bash
bean-report /path/to/my/file.beancount balances
```

有许多可用的报告。有关主要报告的描述，请参阅报告部分。如果你想生成完整的报告列表，请使用以下命令获取帮助：

```bash
bean-report --help-reports
```

报告名称有时可以接受参数。目前，参数作为报告名称的一部分指定，通常用冒号 (:) 分隔，如下所示：

```bash
bean-report /path/to/my/file.beancount balances:Vanguard
```

有几个你应该知道的特殊报告：

- check 或 validate：这与运行 bean-check 命令相同。
- print：这只是打印 Beancount 解析的条目，以 Beancount 语法表示。这可以用来确认 Beancount 是否正确读取和解释了你的输入数据（如果你正在调试某些问题）。

其他报告都是你所期望的：它们打印出各种金额的汇总表。可以生成的报告在下面的专用部分中描述。

### bean-query

Beancount 的解析交易和分录列表就像一个内存数据库。**bean-query** 是一个命令行工具，充当该内存数据库的客户端，你可以在其中输入 SQL 变体查询。你可以这样调用它：

```bash
bean-query /path/to/my/file.beancount
```

然后你会看到类似这样的提示：

```plaintext
Input file: "/path/to/my/file.beancount"
Ready with 14212 directives (21284 postings in 8879 transactions).
beancount> _
```

更多详情请参阅[自己的文档](beancount_query_language.md)。

### bean-web

**bean-web** 在你的计算机上运行一个 web 服务器，提供所有报告。你可以这样运行它：

```bash
bean-web /path/to/my/file.beancount
```

它将在你的机器上运行一个端口 8080 的服务器。使用 web 浏览器导航到 [http://localhost:8080](http://localhost:8080)。你应该能够轻松点击浏览所有报告。

web 界面提供了一组全局页面和一组视图的报告页面。

#### 全局页面

目录页顶部提供链接到所有全局页面：

- 目录（你正在查看的页面）
- 你的账簿文件中发生的错误列表
- 账簿文件的源代码视图（用于引用输入文件中的位置的各种其他链接）。

目录提供了链接到所有常见视图的便捷列表，例如：

- “按年份查看”
- “按标签查看”
- “查看所有交易”

还有更多。

#### 视图报告页面

当你点击视图报告页面时，你会进入一组关于该视图中交易的页面。在这里可以查看各种关于这些交易的报告：

- 期初余额（视图开始时的资产负债表）
- 资产负债表（视图结束时的资产负债表）
- 损益表（视图期间的损益表）
- 各个账户的日记账（点击账户名称）
- 视图结束时的持有报告
- 视图条目中包含的文档和价格列表
- 关于视图数据的一些统计

……还有更多。应该有一个可用视图报告的索引。

### bean-bake

**bean-bake** 运行一个 **bean-web** 实例，并将所有页面烘焙到一个目录：

```bash
bean-bake /path/to/my/file.beancount myfinances
```

它还支持直接烘焙到一个归档文件：

```bash
bean-bake /path/to/my/file.beancount myfinances.zip
```

支持各种压缩方法，例如 .tar.gz。

这对于与没有能力运行 Beancount 的人（如你的会计师）共享 web 界面非常有用。页面链接已全部转换为相对链接，他们应该能够解压归档到一个目录，并以与你使用 **bean-web** 相同的方式浏览所有报告。

### bean-doctor

这是一个用于执行各种诊断和运行调试命令的调试工具，用于帮助报告错误。例如，它可以执行以下操作：

- 列出你计算机上安装的 Beancount 依赖项，以及缺失的依赖项。它可以告诉你缺少哪些应该安装的内容。
- 打印你的输入文件的解析器词法分析标记的转储。如果你报告解析错误，查看词法分析输出可能有用（如果你知道自己在做什么）。
- 它可以通过解析回合对你的输入文件进行测试，即打印文件并重新读取并进行比较。这是一个有用的解析器和打印机测试。
- 它可以检查目录层次结构是否对应于 Beancount 输入文件的账户图，并报告不符合的目录。如果你决定更改一些账户名称，并维护一个相应的文档存档，这会很有用。

#### 上下文

它可以列出应用事务的上下文，即在应用事务分录之前和之后每个账户的库存余额。你可以这样使用它：

```bash
$ bean-doctor context /home/blais/accounting/blais.beancount 28514
```

这将输出类似如下内容：

```plaintext
/home/blais/accounting/blais.beancount:28513:

;   Assets:US:ETrade:BND             100.00 BND {81.968730000000 USD}
;   Assets:US:ETrade:Cash           8143.97 USD

2014-07-25 * "(TRD) BOT +50 BND @82.10" ^273755872
 Assets:US:ETrade:BND                 50.00 BND {82.10 USD}  
 Assets:US:ETrade:Cash             -4105.00 USD         

;   Assets:US:ETrade:BND                    100.00 BND {81.968730000000 USD}

; ! Assets:US:ETrade:BND              50.00 BND {82.10 USD}
; ! Assets:US:ETrade:Cash           4038.97 USD
```

有一个相应的 Emacs 绑定（C-c x）可以在光标周围调用此功能。

### bean-format

这个纯文本处理工具将重新格式化 Beancount 输入，使所有数字在同一个最小列中右对齐。它使所有货币左对齐。它只修改空白。这个工具接受文件名作为参数，并在 stdout 上输出对齐的文件（类似于 UNIX 的 `cat`）。

### bean-example

这个程序生成一个 Beancount 示例输入文件。有关此示例文件内容的更多详细信息，请参阅[教程](tutorial_example.md)。

## 过滤交易

为了生成你的财务交易的不同视图，我们选择解析交易的完整列表的一个子集，例如“2013 年发生的所有交易”，然后使用这些数据生成 Beancount 提供的各种报告。

目前，仅在 web 界面中可用的视图预设列表如下。这些视图包括：

- 所有交易
- 某一年的交易
- 特定标签的交易
- 特定付款人的交易
- 至少包含一个特定名称部分的账户的交易

目前，要访问这些交易子集的报告，你需要使用 **bean-web** web 界面，并点击根全局页面中的相关关键字，这将带你进入一组视图的报告。

## 报告

最初将交易输入到单个输入文件的全部意义在于它允许你对数据的各种子集进行求和、过滤、聚合和排列，并生成各种常见报告。有三种不同的方法可以从 Beancount 生成报告：使用 **bean-web** 浏览到视图然后到特定报告（这是最简单的方法），使用 **bean-report** 并提供所需报告的名称（可能还有一些特定报告的参数），以及使用 **bean-query** 并通过指定 SQL 语句请求数据。

报告有时可以以不同的文件格式呈现。每种报告类型将支持以常见的几种格式呈现，例如控制台文本、HTML 和 CSV。一些报告以 Beancount 语法本身呈现，我们称这种格式为“beancount”。

有许多类型的报告可用，将来会有更多，因为路线图上的许多功能涉及新的输出类型。本节概述了最常见的报告类型。使用 bean-report 查询你的已安装版本支持的完整列表：

```bash
bean-report --help-reports
```

### 余额报告

所有余额报告都类似，它们生成某些账户及其关联余额的表格：[输出示例](http://furius.ca/beancount/examples/tutorial/balances.output)

```plaintext
|-- Assets                               
|   `-- US                               
|       |-- BofA                         
|       |   `-- Checking                       596.05 USD
|       |-- ETrade                       
|       |   |-- Cash                         5,120.50 USD
|       |   |-- GLD                             70.00 GLD
|       |   |-- ITOT                            17.00 ITOT
|       |   |-- VEA                             36.00 VEA
|       |   `-- VHT                            294.00 VHT
|       |-- Federal                      
|       |   `-- PreTax401k               
|       |-- Hoogle                       
|       |   `-- Vacation                       337.26 VACHR
|       `-- Vanguard                     
...
```

按成本持有的商品余额以其账面价值呈现。（未实现的收益，如果有，将由可选插件插入为单独的交易，如果在同一账户中呈现，则结果金额将与成本基础混合。）

如果账户的余额包含多种不同类型的货币（按成本持有的商品，例如美元、欧元、日元），每种货币将单独显示在一行上。这可能会使余额列过于繁忙，影响账户名称的垂直整齐度。在实际操作中，用户账簿中占大多数的是少数几种货币：本国货币单位。为此，可以使用“`operating_currency`”选项将常用货币的金额拆分为单独的列：

```plaintext
option "operating_currency" "USD"
option "operating_currency" "CAD"
```

如果你有多种货币，可以多次使用此选项（例如我是一个外籍人士，在我的东道国和祖国持有资产）。声明操作货币的另一个好处是无需渲染货币名称，因此可以更轻松地导入到电子表格中。

最后，如果某个账户没有被关闭，则认为它是“活跃的”。在过滤的交易集合中没有交易的已关闭账户将不被渲染。

#### 试算平衡表 (`balances`)

[试算平衡表](http://en.wikipedia.org/wiki/Trial_balance) 报告只是生成所有活跃账户的最终余额表，所有账户垂直渲染。

所有余额的总和报告在底部。与资产负债表不同，由于货币转换，可能并不总是平衡到零。（这相当于 Ledger 的 `bal` 报告。）

等效的 bean-query 命令是：

```sql
SELECT account, sum(position) 
GROUP BY account 
ORDER BY account;
```

#### 资产负债表 (`balsheet`)

[资产负债表](http://en.wikipedia.org/wiki/Balance_sheet) 是在特定时间点资产、负债和权益账户的余额快照。为了构建这样的报告，我们必须从其他账户中移动余额到资产负债表：

1. 计算该时间点的收入和费用账户余额，
2. 插入交易，将这些账户的余额清零，转移到一个权益账户（`Equity:Earnings:Current`），
3. 渲染资产账户余额树在左侧，
4. 渲染负债账户余额树在右侧，
5. 渲染权益账户余额树在负债账户下方。

参见[介绍文档](the_double_entry_counting_method.md)中的示例。

注意，权益账户包括从收入和费用账户报告的金额，也通常称为“净收入”。

实际上，我们通常会进行*两次*转移，因为我们通常会查看一个报告期，并希望区分报告期开始前转移的净收入金额（`Equity:Earnings:Previous`）和期间内的金额（`Equity:Earnings:Current`）。类似的转移对用于处理货币转换（这是一个有点复杂的话题，但有一个很好的解决方案；如果你想了解所有细节，请参考[专用文档](http://furius.ca/beancount/doc/conversions)）。

等效的 bean-query 命令是：

```sql
SELECT account, sum(position) 
FROM CLOSE ON 2016-01-01 
GROUP BY account ORDER BY account;
```

#### 期初余额 (`openbal`)

期初余额报告只是报告期开始时的资产负债表。此报告仅在代表时间段的过滤条目列表中有意义，例如“2014 年”。资产负债表使用仅在过滤交易时生成的汇总条目生成（参见[复式记账方法文档](the_double_entry_counting_method.md)）。

#### 损益表 (`income`)

[损益表](http://en.wikipedia.org/wiki/Income_statement) 列出收入和费用账户的最终余额。它代表这些账户内临时活动的摘要。如果资产负债表是在特定时间点的快照，损益表则是期间（在我们的例子中：过滤交易集合）的起点和终点之间的差异。活跃的收入账户余额在左侧渲染，活跃的费用账户余额在右侧。参见[介绍文档](the_double_entry_counting_method.md)中的示例。

收入和费用余额的差额即为净收入。

注意，收入和费用账户的初始余额应已被期初汇总交易清零，因为我们只关注这些账户的变化。

### 日记账报告

本节中的报告以线性方式呈现交易和其他指令列表。

#### 日记账 (`journal`)

此报告相当于机构的“账户对账单”，即账户中至少有一个分录的交易列表。这相当于 Ledger 的寄存器报告（`reg`）。

你可以这样生成日记账报告：

```bash
bean-report myfile.beancount journal …
```

默认情况下，这会渲染*所有*交易的日记账，这可能不是你想要的。选择一个特定账户来渲染，如下所示：

```bash
bean-report myfile.beancount journal -a Expenses:Restaurant
```

目前，“`-a`” 选项仅接受完整的账户名称或父账户的名称。我们将来会扩展它以处理表达式。

##### 按成本渲染

右侧的数字列显示选定账户的分录更改。注意，仅渲染影响给定账户分录的余额。

变更列渲染受影响的单位变化。例如，如果选择此分录：

```plaintext
Assets:Investments:Apple      2 AAPL {402.00 USD}
```

为变更报告的值将是“`2 AAPL`”。如果你想按成本渲染值，使用“`--at-cost`”或“`-c`”选项，在这种情况下将渲染“`804.00 USD`”。

没有“市值”选项。未实现的收益会自动由“`beancount.plugins.unrealized`”插件插入历史记录末尾。请参阅该插件的选项以插入未实现的收益。

注意，如果选定分录的总和为零，变更列中不会渲染任何金额。

##### 添加余额列

如果你想添加一个列来汇总报告变更的运行余额，使用“`--render-balance`”或“`-b`”选项。是否报告运行余额由你决定，因为并不总是有意义。

##### 字符宽度

默认情况下，报告的宽度与你的终端允许的一样宽。使用“`-w`”选项限制宽度到设定的字符数。

##### 精度

可以通过“`--precision`”或“`-k`”指定数字渲染的小数位数。

##### 紧凑、正常或详细

在正常操作中，Beancount 在交易之间渲染一个空行。这有助于区分多种货币影响的交易，因为它们在单独的行上渲染。如果你想更紧凑地渲染，使用“`--compact`”或“`-x`”选项。

另一方面，如果你想在交易行下渲染受影响的分录，使用“`--verbose`”或“`-X`”选项。

##### 摘要

这是其参数的摘要：

```plaintext
optional arguments:
  -h, --help            show this help message and exit
  -a ACCOUNT, --account ACCOUNT
                        Account to render
  -w WIDTH, --width WIDTH
                        The number of characters wide to render the report to
  -k PRECISION, --precision PRECISION
                        The number of digits to render after the period
  -b, --render-balance, --balance
                        If true, render a running balance
  -c, --at-cost, --cost
                        If true, render values at cost
  -x, --compact         Rendering compactly
  -X, --verbose         Rendering verbosely
```

##### 等效 SQL 查询

等效的 bean-query 命令是：

```sql
SELECT date, flag, description, account, cost(position), cost(balance);
```

#### 转换 (`conversions`)

此报告列出选定交易的货币转换总额。（大多数人不需要这个。）

#### 文档 (`documents`)

此报告生成 HTML 列表，列出账簿中找到的所有外部文档，或通过自动查找文档并将其添加到交易流中的插件生成。

### 持有报告

这些报告生成按成本持有的资产聚合。

#### 持有与聚合 (`holdings*`)

此报告生成账簿中找到的所有持有的详细列表。你可以使用“-g”选项按商品和账户生成聚合。

#### 净资产 (`networth`)

此报告生成账簿的净资产（权益）的简短摘要，按操作货币显示。

### 其他报告类型

#### 现金

此报告渲染按成本不持有的商品的余额，即现金：

```bash
bean-report example.beancount cash -c USD
```

输出示例：

```plaintext
Account                         Units  Currency  Cost Currency  Average Cost  Price  Book Value  Market Value

--------------------------  ---------  --------  -------------  ------------  -----  ----------  ------------

Assets:US:BofA:Checking        596.05       USD            USD                           596.05        596.05

Assets:US:ETrade:Cash        5,120.50       USD            USD                         5,120.50      5,120.50

Assets:US:Hoogle:Vacation      337.26     VACHR                                                              

Assets:US:Vanguard:Cash         -0.02       USD            USD                            -0.02         -0.02

Liabilities:US:Chase:Slate  -2,891.85       USD            USD                        -2,891.85     -2,891.85

--------------------------  ---------  --------  -------------  ------------  -----  ----------  ------------
```

报告允许你将所有货币转换为一种常用货币（在上例中，“全部转换为 USD”）。还有一个选项仅报告操作货币。我用这个来概述所有未投资的现金。

#### 价格 (`prices`)

此报告生成基础货币相对于报价货币的价格点列表。列表按日期排序。你也可以以 beancount 格式输出此表格。保存价格数据库到文件，然后可以将其与其他输入文件结合加载非常方便。

#### 统计 (`stats`)

此报告仅提供解析条目的各种统计数据。

#### 更新活动 (`activity`)

此表格为每个账户渲染上一次条目的日期。
