# 导出您的投资组合<a id="title"></a>

[<u>Martin Blais</u>](mailto:blais@furius.ca)，2015年12月（第2版）

[<u>http://furius.ca/beancount/doc/export</u>](http://furius.ca/beancount/doc/export)

## 概述<a id="overview"></a>

本文档解释了如何将您的持仓从 Beancount 导出到 Google Finance 投资组合（最终到其他投资组合跟踪网站）。

*注意：这是本文档的第二版，重写于2015年12月，在极大简化导出投资组合的过程并完全分离价格下载的股票代码规范后。本新简化版本只使用一个元数据字段名称：“export”。以前的文档可以在[<u>这里</u>](https://docs.google.com/document/d/1eZIDRmQZxR6cmDyOJf7U3BnCm4PDMah2twxYFfKPJtM/)找到。*

## 投资组合跟踪工具<a id="portfolio-tracking-tools"></a>

互联网上有多个网站允许用户创建投资组合（或上传交易列表以创建此类投资组合），并报告因价格变动而导致的投资组合变化，显示您未实现的资本收益等。一个这样的例子是[<u>Google Finance</u>](http://finance.google.com)门户。另一个例子是[<u>Yahoo Finance</u>](http://finance.yahoo.com)。这些网站很方便，因为它们允许您在一天中的任何时候监控价格变动对整个投资组合资产的影响。

然而，这些网站都期望用户使用其界面和工作流程一个一个地手动输入每个持仓。使用 Beancount 的一个巨大优势是您不必手动输入此类信息；相反，您应该能够提取并上传到这些网站之一。您可以独立于使用的特定投资组合跟踪服务，并且能够在不丢失任何数据的情况下在它们之间切换；Beancount 可以作为您的持仓列表的纯净源，根据您的需求进行演变。

Google Finance 支持“导入”功能以创建投资组合数据，支持 Microsoft OFX 金融交换数据格式。在本文中，我们展示了如何构建一个 Beancount 报告，将持仓导出为 OFX 格式，以创建 Google Finance 投资组合。

## 导出到 Google Finance<a id="exporting-to-google-finance"></a>

### 将持仓导出为 OFX 格式<a id="exporting-your-holdings-to-ofx"></a>

首先，创建一个与您的 Beancount 持仓对应的 OFX 文件。您可以使用以下命令来执行此操作：

```bash
bean-report file.beancount export_portfolio > portfolio.ofx
```

请参阅报告自己的帮助以获取选项：

```bash
bean-report file.beancount export_portfolio --help
```

### 在 Google Finance 中导入 OFX 文件<a id="importing-the-ofx-file-in-google-finance"></a>

然后我们需要在基于 Web 的投资组合中导入该 OFX 文件。

1. 访问 [<u>http://finance.google.com</u>](http://finance.google.com) 并点击左侧的“投资组合”（或直接访问 [<u>https://www.google.com/finance/portfolio</u>](https://www.google.com/finance/portfolio)，截至2015年1月有效）

2. 如果您有一个现有的、以前导入的投资组合，请点击“删除投资组合”以删除它。

3. 点击“导入交易”，然后点击“选择文件”并选择您导出的 *portfolio.ofx* 文件，然后点击“预览导入”。

4. 您应该会看到一系列导入的批次，带有熟悉的股票代码和名称，类型为“买入”，有合理的股票数量和价格列。如果没有，请参阅下文中的说明。否则，滚动到页面底部并点击“导入”。

5. 您的投资组合现在应该出现了。完成。

您不应该通过网站直接更新此投资组合……相反，更新您的 Beancount 分类账文件，重新导出新的 OFX 文件，**删除**先前的投资组合，并重新导入一个新的投资组合。您的纯净源始终是您的 Beancount 文件，理想情况下，您不必担心外部网站上的投资组合数据被损坏或删除。

## 控制导出的商品<a id="controlling-exported-commodities"></a>

### 声明您的商品<a id="declaring-your-commodities"></a>

一般来说，我们建议您明确声明输入文件中使用的每种商品。这是附加有关这些商品的信息、元数据的好地方，您以后可以从 bean-query 或您制作的脚本中使用这些元数据。例如，您可以声明商品的可读描述和其他一些属性，如下所示：

```plaintext
2001-09-06 commodity XIN
  name: "iShares MSCI EAFE Index ETF (CAD-Hedged)"
  class: "Stock"
  type: "ETF"
  ...
```

Beancount 可以在有或没有这些声明的情况下工作（如果您没有提供它们，它会自动生成 Commodity 指令）。如果您喜欢严格并且有点纪律，您可以使用一个插件，要求每个商品必须声明，当出现未声明的商品时会发出错误：

```plaintext
plugin "beancount.plugins.check_commodity"
```

您可以为该 Commodity 指令使用任何日期。我建议使用商品的创建日期，或者可能是发行国首次引入该商品的日期，如果是货币的话。您可以在 Wikipedia 或发行者的网站上找到合适的日期。Google Finance 可能有这个日期。

### 默认情况下发生的情况<a id="what-happens-by-default"></a>

默认情况下，所有持仓都会以与您用于定义它们的 Beancount 商品相同的股票代码导出为持仓。如果您持有“AAPL”单位，它会为“AAPL”创建一个导出条目。导出代码默认尝试导出所有持仓。

然而，在任何除了最简单的明确情况外，这可能不足以生成一个正常工作的 Google Finance 投资组合。您在 Beancount 输入文件中使用的每种商品的名称可能与 Google Finance 数据库中的金融工具名称不对应；由于其数据库中支持的大量股票代码，仅指定股票代码通常是不明确的。Google Finance 尝试将一个模糊的符号字符串解析为其数据库中最可能的工具。可能它会解析为您未预期的不同金融工具。因此，即使您使用与交易所使用的相同基本股票代码，您通常仍需要通过指定其所在的交易所或符号系统来消除歧义。Google 提供了一个 [<u>这些符号空间的列表</u>](http://www.google.com/googlefinance/disclaimer/)。

这是一个真实的例子。iShares/Blackrock 发行的“[<u>CAD-Hedged MSCI EAFE Index</u>](http://www.blackrock.com/ca/individual/en/products/239624/ishares-msci-eafe-index-etf-cadhedged-fund)” ETF 产品的股票代码是多伦多证券交易所 (`TSE`) 上的“`XIN`”。如果您仅仅 [<u>在 Google Finance 上查找“XIN”</u>](https://www.google.com/finance?q=xin)，它会默认解析为更可能的“`NYSE:XIN`”股票代码（[<u>纽交所上的新源房地产公司</u>](https://www.google.com/finance?q=NYSE%3AXIN)）。因此，您需要通过指定该工具的 ETF 股票代码为“`TSE:XIN`”来消除歧义。

### 明确指定导出的股票代码<a id="explicitly-specifying-exported-symbols"></a>

您可以通过在每个商品指令中附加一个“`export`”元数据字段，指定用于导出商品的交易所特定代码，如下所示：

```plaintext
2001-09-06 commodity XIN
  ...
  export: "TSE:XIN"
```

“`export`”字段用于将您的商品名称映射到 Google Finance 系统中的相应工具。如果需要导出该商品的持仓，则使用此代码而不是 Beancount 货币名称。

Google Finance 使用的符号系统似乎遵循以下语法：

### *Exchange:Symbol*<a id="exchangesymbol"></a>

其中*Exchange*是股票交易的交易所代码，或其他金融数据源代码，例如“`MUTF`”表示“美国的共同基金”，[<u>还有更多</u>](http://www.google.com/googlefinance/disclaimer/)。*Symbol*是在该交易所内唯一的名称。我建议搜索每个金融工具在 Google Finance 上，确认该工具对应于您的工具（通过检查全名、描述和价格），并插入相应的代码，如下所示。

### 导出为现金等价物<a id="exporting-to-a-cash-equivalent"></a>

为了处理 Google Finance 不支持的持仓，导出报告可以将持仓转换为其现金等价值。这对于现金持仓也很有用（例如，存放在储蓄或支票账户中的现金）。

例如，我持有一个保险政策投资工具（这在加拿大很常见，例如，与伦敦人寿一起）。这是一个金融工具，但每个特定的保单发行都有其自己的关联价值——没有公开的数据源用于这些产品，我可以通过年度对账单获得其价值，但绝对不是在 Google Finance 中。但我仍然希望资产的价值反映在我的投资组合中。

您告诉导出代码进行此转换的方法是为“`export`”字段指定一个特殊值“CASH”，如下所示：

```plaintext
1878-01-01 commodity LDNLIFE
  export: "CASH"
```

这将把 `LDNLIFE` 商品的持仓转换为其相应的报价值再进行导出，使用最接近导出日期的价格。注意，尝试转换为现金的商品没有相应的成本或价格来确定其价值将生成错误。必须存在价格才能进行转换。

简单货币也应标记为现金以进行导出：

```plaintext
1999-01-01 commodity EUR
  name: "European Union Euro currency"
  export: "CASH"
```

最后，所有转换后的持仓都会聚合为单一现金持仓。没有必要将这些现金条目单独导出为 OFX 条目，因为 Google Finance 代码无论如何会将它们聚合为一个。

### 声明货币工具<a id="declaring-money-instruments"></a>

在这个现金转换故事中有一个小问题：Google Finance 导入器似乎不正确地理解导入器中的“现金”金额 OFX 持仓；我认为这可能只是 Google Finance 导入代码中的一个错误（或者我还没有找到使其工作的正确 OFX 字段值）。

相反，为了插入现金持仓，导出器使用一个现金等价商品，该商品总是以货币的 1 单位定价，例如，1.00 美元对于美元。例如，对于美元，我使用 [<u>VMMXX</u>](https://www.google.com/finance?q=MUTF:VMMXX) 这是一种 Vanguard Prime 货币市场基金，对于加拿大元，我使用 [<u>IGI806</u>](https://www.google.com/finance?q=MUTF_CA:IGI806)。一种好的商品类型是某种货币市场基金。重要的是找到一个定价非常接近 1 的商品，具体是哪一个并不重要。

如果您想包括现金商品，您需要为您账簿中的每种现金货币找到这样的商品，并告诉 Beancount 它们。通常这只会是一两个货币。

您通过附加特殊值“`MONEY`”到“`export`”字段来声明它们，指定该商品表示哪种货币，如下所示：

```plaintext
1900-01-01 commodity VMMXX
  export: "MUTF:VMMXX (MONEY:USD)"

1900-01-01 commodity IGI806
  export: "MUTF_CA:IGI806 (MONEY:CAD)"
```

### 忽略商品<a id="ignoring-commodities"></a>

最后，一些账簿中的商品应该被忽略。例如，在镜像会计中用于跟踪未归属股票的虚拟商品，或用于跟踪退休账户贡献的商品，如下所示：

```plaintext
1996-01-01 commodity RSPCAD
  name: "Canada Registered Savings Plan Contributions"
```

您告诉导出代码通过为“`export`”字段指定特殊值“`IGNORE`”来忽略一个商品，如下所示：

```plaintext
1996-01-01 commodity RSPCAD
  name: "Canada Registered Savings Plan Contributions"
  export: "IGNORE"
```

所有 `RSPCAD` 商品单位的持仓将因此不被导出。

是否应导出某些商品有时会提出有趣的选择。这里是一个示例：我在资产账户中跟踪我的累计假期时间。单位是“`VACHR`”。我将这种商品与大致等于我净时薪的价格关联。这让我粗略了解账簿上累计的假期时间金额，例如，如果我辞职，我会得到多少报酬。我是否希望它们包括在我的总净资产中？这些时间的价值是否应该反映在导出投资组合的价值中？我认为这在很大程度上取决于我计划在离职前用完这些假期时间，还是希望在资产负债表中显示这些累计价值。

## 与净资产比较<a id="comparing-with-net-worth"></a>

最终结果是，所有导出持仓的总和加上现金持仓的总和应该接近您的所有资产的价值，并且 Google Finance 网站计算的总价值应该非常接近此报告所报告的值：

```bash
bean-report file.beancount networth
```

作为对比，我自己的投资组合的价值通常在几百美元之内。

## OFX 导出的详细信息<a id="details-of-the-ofx-export"></a>

### 导入失败<a id="import-failures"></a>

导出一个包含 Google Finance 无法识别的股票代码的投资组合会**致命地**导致 Google 的导入功能失败。Google Finance 接下来无法识别您的**整个**文件。我建议您对所有导出的商品使用明确的 exchange:symbol 名称，以避免此问题，如本文档进一步描述的。

Google Finance 对您提供的特定 OFX 文件的格式也有些挑剔。`export_portfolio` 命令尝试避免会破坏它的 OFX 特性，但它很脆弱，您的投资组合内容的特定情况可能会触发无法导入的输出。如果是这样，在上述步骤（4）中，您将看到一长串看起来像 XML 标签的持仓（这就是失败的表现）。如果是这样，请发送电子邮件到邮件列表（最好您能隔离触发故障的持仓，并有能力对文件进行差异和一些故障排除）。

### 共同基金与股票<a id="mutual-funds-vs.-stocks"></a>

OFX 格式区分股票和共同基金。在实践中，Google Finance 导入器似乎不区分这两者（至少它似乎表现相同），所以这可能是一个无关紧要的实现细节。尽管如此，导出代码能够遵守 OFX 区分“BUYMF”和“BUYSTOCK” XML 元素的约定。

为此，导出代码尝试通过检查与商品关联的股票代码是否以字母“`MUTF`”开头并后跟冒号来分类哪些商品代表共同基金。例如，“`MUTF:RGAGX`”和“`MUTF_CA:RBF1005`”都将被检测为共同基金。

### 调试导出<a id="debugging-the-export"></a>

为了调试每个持仓的导出方式，请使用 `--debug` 标志，它将打印导出脚本如何处理每个持仓的详细信息到 **stderr**：

```bash
bean-report file.beancount export_portfolio --debug 2>&1 >/dev/null | more
```

脚本应打印导出持仓及其相应持仓的列表，然后打印转换持仓及其相应持仓的列表（通常许多现金持仓被聚合在一起），最后打印被忽略的持仓列表。这应该足以解释导出投资组合的全部内容。

### 购买日期<a id="purchase-dates"></a>

Beancount 当前没有所有批次的购买日期，因此购买日期导出为导出前一天。

最终，当 Beancount 中提供购买日期时（等待[<u>库存预订变更</u>](http://furius.ca/beancount/doc/booking-proposal)），可能会在导出格式中使用实际的批次购买日期。然而，尚不清楚使用正确日期是否合适，因为 Google Finance 可能会坚持插入自报告购买日期以来的股息现金……但 Beancount 已经负责插入一个特殊批次的现金，应该已经包括在内。我们将在到达时看到。

### 禁用股息<a id="disable-dividends"></a>

在“编辑投资组合”选项下有一个复选框，似乎可以禁用提供的股息计算。希望能找到一种方法在导入时自动禁用此复选框。

### 自动上传<a id="automate-upload"></a>

最好能用 Python 脚本自动替换投资组合。不幸的是，Google Finance API 已被弃用。也许有人可以编写一个屏幕抓取例程来完成此任务。

## 总结<a id="summary"></a>

每个持仓的导出可以通过其商品的处理方式进行控制，以下列方式之一：

1. **导出**为投资组合持仓。这是默认选项，但您应使用“`ticker`”或“`export`”元数据字段，采用“*ExchangeCode:Symbol*”格式指定股票代码。

2. **转换**为现金并导出为货币市场现金等价持仓，方法是将“`export`”元数据字段的值设置为特殊值“`CASH`”。

3. **忽略**通过为“`export`”元数据字段指定特殊值“`IGNORE`”。

4. 提供为**货币工具**，用于转换为现金并包含在投资组合中的每个持仓的现金等价值。通过在“`export`”元数据字段中指定特殊值“(`MONEY:<currency>)`”来标识这些。
