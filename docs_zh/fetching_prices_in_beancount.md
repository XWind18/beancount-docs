# Beancount 中的价格<a id="title"></a>

[Martin Blais](mailto:blais@furius.ca)，2015年12月

[<u>http://furius.ca/beancount/doc/prices</u>](http://furius.ca/beancount/doc/prices)

## 介绍<a id="introduction"></a>

处理 Beancount 文件时，其内容本身是固定的。特别是，商品的最新价格*从未*自动从互联网上获取。这是设计使然，以确保报告的每次运行都是确定的，并且可以离线运行。没有意外。

然而，我们确实需要访问价格信息以计算资产的市场价值。为此，Beancount 提供了 Price 指令，可以将这些价格点内联插入输入文件，以填充其内存价格数据库：

```plaintext
2015-11-20 price ITOT 95.46 USD  
2015-11-20 price LQD 115.63 USD  
2015-11-21 price USD 1.33495 CAD  
...
```

当然，您可以手动执行此操作，在线查找价格并自己编写指令。但对于公开交易的资产，您可以通过调用一些代码来自动化，下载价格并写出指令。

Beancount 提供了一些工具来帮助您完成此任务。本文档描述了这些工具。

## 问题<a id="the-problem"></a>

在维护 Beancount 文件的上下文中，我们需要解决几个特定需求。

首先，我们不能期望用户总是及时更新输入文件。这意味着我们不仅需要获取最新价格，还需要获取过去的历史价格。Beancount 提供了一个接口来提供这些价格和一些实现（从 Yahoo! Finance 和 Google Finance 获取）。

其次，我们不想为在特定时间点对我们不相关的持仓存储太多价格。文件中持有的资产列表会随时间变化。理想情况下，我们只希望包括这些资产在持有期间的价格列表。Beancount 价格维护工具能够确定在特定日期需要获取价格的商品。

第三，如果输入文件中已经存在相同的价格，我们不想再次获取。工具会检测到这一点并跳过这些价格。还有一个缓存机制可以避免冗余的网络调用。

最后，虽然我们提供了一些基本的价格来源实现，但我们不能为所有可能的在线来源和网站提供此类代码。这个问题类似于从各种机构导入和提取数据的问题。为了解决这个问题，我们提供了一种**扩展性**机制，您可以在 Python 模块中实现自己的价格来源获取器，并通过在输入文件中指定模块名称作为该商品的来源来指向它。

## “bean-price”工具<a id="the-bean-price-tool"></a>

Beancount 提供了一个“bean-price”命令行工具，它集成了上述思想。默认情况下，该脚本接受一组 Beancount 输入文件名，并获取计算账户中当前持仓的最新市场价值所需的价格：

```bash
bean-price /home/joe/finances/joe.beancount
```

还可以提供一组特定的价格获取任务运行，例如：

```bash
bean-price -e USD:google/TSE:XUS CAD:mysources.morningstar/RBF1005
```

这些任务是并发运行的，因此速度应该相当快。

### 来源字符串<a id="source-strings"></a>

每个“任务来源字符串”的一般格式是：

```plaintext
<quote-currency>:<module>/[^]<symbol>
```

例如：

```plaintext
USD:beancount.prices.sources.google/NASDAQ:AAPL
```

“quote-currency”是商品的报价货币。例如，Apple 的股票以美元报价。

“module”是包含 Source 类的 Python 模块的名称，该类可以实例化并连接到数据源以提取价格数据。这些模块会自动导入，并在其中实例化一个 Source 类，以从它们支持的特定在线来源提取价格。这允许您编写自己的获取器代码，而无需修改此脚本。您的代码可以放置在 Python 的 PYTHONPATH 中的任何位置，您无需为此修改 Beancount 本身。

“symbol”是传递给价格获取器以查找货币的字符串。例如，Apple 的股票在纳斯达克交易所交易，对应的 Google Finance 来源符号是“NASDAQ:AAPL”。其他价格来源可能有不同的符号系统，例如一些可能需要资产的 CUSIP。

提供了默认的价格来源实现；我们提供了 Yahoo! Finance 或 Google Finance 的获取器，涵盖了许多常见的公共投资类型（例如股票和一些共同基金）。为了方便起见，模块名称始终首先在 "`beancount.prices.sources`" 包下搜索，这些实现就在那里。因此，例如，为了使用提供的 Yahoo! Finance 数据获取器，您无需编写完整的 "`beancount.prices.sources.yahoo/AAPL`"，只需使用 "`yahoo/AAPL`"。

### 备用来源<a id="fallback-sources"></a>

在实践中，在线获取价格经常会失败。数据源通常只支持有限数量的资产，即使如此，支持的范围也可能因资产而异。例如，Google Finance 支持股票的历史价格，但不返回货币工具的历史价格（这些限制可能与它们与上游数据提供商之间的合同安排比与技术限制更相关）。

为此，可以为数据指定多个来源，使用逗号分隔。例如：

```plaintext
USD:google/CURRENCY:GBPUSD,yahoo/GBPUSD
```

每个来源依次尝试，如果一个来源未能返回有效价格，则尝试下一个来源作为备用。希望至少有一个指定的来源能成功。

### 反转价格<a id="inverted-prices"></a>

有时，价格仅以工具的倒数形式提供。这通常发生在货币上。例如，以加元报价的美元价格由 USD/CAD 市场提供，该市场给出了美元在加元中的价格（倒数）。为了使用这个，您可以在工具名称前添加 "`^`" 以指示工具计算获取价格的倒数：

```plaintext
USD:google/^CURRENCY:USDCAD
```

如果要反转价格来源，精度可能与获取的价格不同。例如，如果 USD/CAD 的价格为 1.32759，对于上面的指令定价“CAD”工具，它会输出：

```plaintext
2015-10-28 price CAD  0.753244601119 USD
```

默认情况下，反转的汇率将类似于其他 Price 指令的四舍五入方式四舍五入这些数字。

您可能知道，Beancount 的内存价格数据库在两个方向上都能工作（所有汇率的倒数会自动存储）。因此，如果您更喜欢用交换的货币输出价格条目而不是反转汇率数字本身，您可以使用 --swap-inverted 选项。在前面的 CAD 价格示例中，它会输出：

```plaintext
2015-10-28 price USD   1.32759 CAD
```

### 日期<a id="date"></a>

默认情况下，拉取资产的最新价格。您可以使用一个选项来获取过去特定日期的价格：

```bash
bean-price --date=2015-02-03 …
```

如果您使用输入文件指定要获取的价格列表，工具会计算出*当时*账簿中持有的资产列表，并仅获取这些资产的历史价格。

### 缓存<a id="caching"></a>

价格会自动缓存（如果是当前和最新的价格，价格只会缓存很短的一段时间，大约半小时）。这在您需要多次运行脚本进行故障排除时很方便。

您可以使用一个选项禁用缓存：

```bash
bean-price --no-cache
```

您还可以指示脚本在获取价格之前清除缓存：

```bash
bean-price --clear-cache
```

## 从 Beancount 输入文件获取价格<a id="prices-from-a-beancount-input-file"></a>

通常，使用 Beancount 输入文件指定要获取的货币列表。为此，您应该在输入文件中为每个要获取价格的货币添加 Commodity 指令，如下所示：

```plaintext
2007-07-20 commodity VEA
  price: "USD:google/NYSEARCA:VEA"
```

"price" 元数据应包含价格来源字符串列表。例如，股票产品可能如下所示：

```plaintext
2007-07-20 commodity CAD
  price: "USD:google/CURRENCY:USDCAD,yahoo/USDCAD"
```

而一种货币可能有多个目标货币需要转换：

```plaintext
1990-01-01 commodity GBP
  price: "USD:yahoo/GBPUSD CAD:yahoo/GBPCAD CHF:yahoo/GBPCHF"
```

### 获取哪些资产的价格<a id="which-assets-are-fetched"></a>

有很多方法可以从 Beancount 输入文件中计算出需要价格的商品列表：

1.  **Commodity 指令。** 文件中所有具有“price”元数据的 Commodity 指令列表。对于每个持仓，查阅指令并使用其 "`price`" 元数据字段指定从哪里获取价格。

2.  **按成本持有的商品。** 在特定日期按成本持有的所有持仓的价格。因为这些持仓按成本持有，我们可以假设它们的商品有相应的时间变化价格。

3.  **转换的商品。** 过去从其他商品转换过价格的持仓（例如，从另一种货币转换一些现金）。

默认情况下，要获取的代码列表仅包括这三个列表的交集。这是因为此脚本的最常见用法是在特定日期获取缺失的价格，并且仅需要的那些价格。

**非活动商品。** 您可以使用“`--inactive`”选项获取（1）中的整个价格列表，而不考虑（2）和（3）中确定的资产持仓。

**未声明的商品。** 默认情况下，缺少相应“Commodity”指令的商品将被忽略。要包含输入文件中看到的全部商品列表，请使用“`--undeclared`”选项。

**覆盖。** 默认情况下，排除相同数据的现有价格指令，因为价格已经在文件中。您可以使用“`--clobber`”忽略现有价格指令并避免过滤掉获取的内容。

最后，您可以使用“--all”包含非活动和未声明的商品，并允许覆盖现有的商品。除了测试外，您可能不想使用它。

如果您想进行一些故障排除并打印出看到的商品列表，请使用“`--verbose`”选项两次，即“`-vv`”。您也可以使用“`--dry-run`”选项仅打印要获取的价格列表，而不实际获取缺失的价格。

## 结论<a id="conclusion"></a>

### 编写自己的脚本<a id="writing-your-own-script"></a>

如果此工具定义的工作流程不适合您的需求，并且您想编写自己的脚本，您不必从头开始；您可以重用现有的价格获取代码来完成此任务。我希望在实验目录中提供一些此类脚本示例。例如，给定一个现有文件，每周五获取所有资产的价格，以填补缺失价格的历史。另一个示例是获取计算投资回报所需的所有价格指令，以便在存在贡献和分配的情况下正确计算投资回报。

### 贡献<a id="contributions"></a>

如果此工作流程适合您的需求，并且您希望为 Beancount 贡献一些价格来源获取器，请联系邮件列表。我愿意接受经过精心单元测试并使用了一段时间的非常通用的获取器。
