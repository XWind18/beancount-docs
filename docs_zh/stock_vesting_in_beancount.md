# Beancount 中的股票归属<a id="title"></a>

[<u>马丁·布莱斯</u>](mailto:blais@furius.ca)，2015年6月

[<u>http://furius.ca/beancount/doc/vesting</u>](http://furius.ca/beancount/doc/vesting)

## 简介<a id="introduction"></a>

本文档通过一个示例解释了在 Beancount 中受限股票单位的归属。虽然这个示例可能与你的情况不完全一致，但提供了足够的细节，你应该能够根据自己的特定情况进行调整。

可以在[<u>这里</u>](http://github.com/beancount/beancount/tree/master/examples/vesting/vesting.beancount)找到一个工作示例文件，以便与本文一起使用。

## 受限股票补偿<a id="restricted-stock-compensation"></a>

许多科技公司向其员工提供以“授予”（或“奖励”）“受限股票单位”（RSU）的形式的激励补偿，这实际上是未来向你“释放”实际股票的承诺。股票是“受限”的，因为你无法访问它——你只能在它“归属”时收到它，并且这根据一个时间表进行。通常，你被承诺在 3 或 4 年的时间里每季度或每月归属一定数量的股票。如果你离开公司，你未归属的股票将会丢失。

一种看待这些 RSU 的方式是将其视为一种资产，一种定期到来的应收款。这些 RSU 本质上是以公司的股票本身计价的补偿。我们希望追踪这些未归属单位的展开，并正确核算它们转换为实际股票的成本基础以及归属时支付的任何税款。

## 跟踪奖励<a id="tracking-awards"></a>

### 商品<a id="commodities"></a>

首先我们要定义一些商品。在这个例子中，我为“Hooli 公司”工作，并且最终将获得该公司的股票（以美元计价）：

```plaintext
1990-12-02 commodity HOOL
  name: "Common shares of Hooli Inc."
  quote: USD
```

我们还要跟踪未归属股票的数量：

```plaintext
2013-01-28 commodity HOOL.UNVEST
  name: "Unvested shares of Hooli from awards."
```

### 奖励账户<a id="accounts-for-awards"></a>

收到的授予是收入。我使用“Income:US:Hooli”作为来自 Hooli 的所有收入账户的根账户，但特别定义了一个用于记录奖励的账户，其中包含未归属股票的单位：

```plaintext
2013-01-28 open Income:US:Hooli:Awards        HOOL.UNVEST
```

当股票归属时，我们需要将这部分收入记入某个地方，因此我们定义了一个费用账户，以记录一段时间内归属的股票数量：

```plaintext
2014-01-28 open Expenses:Hooli:Vested         HOOL.UNVEST
```

### 接收奖励<a id="receiving-awards"></a>

当你收到新的奖励时（例如，每年一次，有些人称之为“股票刷新”），你将其作为收入接收，并存入一个新的账户，用于跟踪这个特定奖励：

```plaintext
2014-04-02 * "Award S0012345"
  Income:US:Hooli:Awards                -1680 HOOL.UNVEST
  Assets:US:Hooli:Unvested:S0012345      1680 HOOL.UNVEST

2014-04-02 open Assets:US:Hooli:Unvested:S0012345
```

你可能会同时拥有多个活跃奖励。为每个奖励设置一个单独的账户是很有用的，因为它提供了一种自然的方法来列出其内容，并在奖励到期时关闭该账户——未归属奖励账户的列表提供了未归属和活跃归属奖励的列表。在这个例子中，我使用奖励编号（#S0012345）作为子账户名称。使用编号是有用的，因为声明中通常包含它。

我喜欢将所有奖励保存在一个专用的小节中。

## 归属事件<a id="vesting-events"></a>

然后我有一个不同的部分，包含所有跟随归属事件的交易。

### 账户<a id="accounts"></a>

首先，当我们归属股票时，这是一个应税收入事件。股票的现金价值需要一个收入账户：

```plaintext
2013-04-04 open Income:US:Hooli:RSU
```

支付的税款应记入你应已定义在其他地方的年度费用账户中，以核算该年度的税款（这在食谱的其他地方有详细说明）：

```plaintext
2015-01-01 open Expenses:Taxes:TY2015:US:StateNY
2015-01-01 open Expenses:Taxes:TY2015:US:Federal
2015-01-01 open Expenses:Taxes:TY2015:US:SocSec
2015-01-01 open Expenses:Taxes:TY2015:US:SDI
2015-01-01 open Expenses:Taxes:TY2015:US:Medicare
2015-01-01 open Expenses:Taxes:TY2015:US:CityNYC
```

在收到收入后的剩余现金存入一个临时账户，然后进行转换：

```plaintext
2013-01-28 open Assets:US:Hooli:RSURefund
```

此外，在另一个部分，我们应该有一个账户，用于托管和管理股票的经纪账户：

```plaintext
2013-04-04 open Assets:US:Schwab:HOOL
```

通常你无法选择这个经纪公司，因为你工作的公司通常会与外部公司安排，以管理受限股票计划。典型的公司包括 Morgan Stanley、Salomon Smith Barney、Schwab 甚至 E*Trade。在这个例子中，我们将使用 Schwab 账户。

我们还需要一个某种支票账户来接收分数股的现金。我在例子中假设了一个美国银行的账户：

```plaintext
2001-01-01 open Assets:US:BofA:Checking
```

### 归属<a id="vesting"></a>

首先是归属事件本身：

```plaintext
2015-05-27 * "Vesting Event - S0012345 - HOOL" #award-S0012345 ^392f97dd62d0
  doc: "2015-02-13.hooli.38745783.pdf"
  Income:US:Hooli:RSU                    -4597.95 USD
  Expenses:Taxes:TY2015:US:Medicare         66.68 USD
  Expenses:Taxes:TY2015:US:Federal        1149.48 USD
  Expenses:Taxes:TY2015:US:CityNYC         195.42 USD
  Expenses:Taxes:TY2015:US:SDI               0.00 USD
  Expenses:Taxes:TY2015:US:StateNY         442.32 USD
  Expenses:Taxes:TY2015:US:SocSec          285.08 USD
  Assets:US:Hooli:RSURefund               2458.97 USD
```

这对应于我每次归属事件时收到的工资单。由于 Hooli 的所有 RSU 奖励在每月的同一天归属，这意味着我有一组这样的交易，每个奖励对应一个（在示例文件中，我展示了两个奖励）。

需要注意一些事项：

- 收入分录的金额是归属股票数量乘以股票的公平市场价值（FMV）。在这种情况下，这是 35 股 ⨉ $131.37/股 = $4597.95。我只是写下工资单上提供的美元金额。
- 收到归属股票是一个应税收入事件，Hooli 自动代扣税款并代表员工支付给政府。这些是与你的 W-2 对应的 Expenses:Taxes 账户。
- 最后，我不直接将转换后的股票存入经纪账户；这是因为支付税款后的剩余现金不一定是整数股。因此，我使用了一个临时账户（Assets:US:Hooli:RSURefund），并将收到的股票与转换分离。

这样，每笔交易都对应一个工资单。这使得输入数据更容易。

还要注意，我使用了一个唯一链接（^392f97dd62d0）来分组一个特定归属事件的所有交易。如果你愿意，也可以使用标签。

### 转换为实际股票<a id="conversion-to-actual-stock"></a>

现在我们准备将剩余现金转换为股票单位。这发生在经纪公司，你应该在经纪账户中看到新股票。经纪公司通常会为每个归属事件每个奖励发布“股票释放报告”声明，提供进行转换所需的详细信息，即，从现金转换为股票的实际股数和成本基础（归属日的 FMV）：

```plaintext
2015-05-25 * "Conversion into shares" ^392f97dd62d0
  Assets:US:Schwab:HOOL                   18 HOOL {131.3700 USD}
  Assets:US:Hooli:RSURefund
  Assets:US:Hooli:Unvested:S0012345      -35 HOOL.UNVEST
  Expenses:Hooli:Vested                   35 HOOL.UNVEST
```

前两个分录存入股票，并从临时账户中减去归属交易留下的剩余现金金额。你应该确保使用声明提供的具体 FMV 价格，而不是大概价格，因为这对税务有影响。注意，你不能购买分数股，因此四舍五入后的股票数量（18 股）的成本将使临时账户中剩余一些现金。

最后两个分录从未归属股票余额中扣除，并“收到”一笔费用；该费用账户基本上计算了一段时间内归属了多少股票。

在这里，你可能会有一个这些转换中的每个股票授予。为了遵循一个好规则，我单独输入每个转换，这样一个声明匹配一个交易。

### 返还分数股现金<a id="refund-for-fractions"></a>

在所有转换事件将现金从临时账户中移出后，临时账户中剩余的是所有转换后的分数余额。在我的情况下，这个余额在归属后 3-4 周由 Hooli 单独以工资单的形式返还，甚至会分别列出每个余额。我也将其作为一个交易输入：

```plaintext
2015-06-13 * "HOOLI INC       PAYROLL" ^392f97dd62d0
  doc: "2015-02-13.hooli.38745783.pdf"
  Assets:US:Hooli:RSURefund            -94.31 USD
  Assets:US:Hooli:RSURefund             -2.88 USD ; (For second award in example)
  Assets:US:BofA:Checking               97.19 USD
```

在支付了分数后，临时账户应该为空。我使用余额断言来验证这一点：

```plaintext
2015-06-14 balance Assets:US:Hooli:RSURefund  0 USD
```

这给了我一些确定这些数字是正确的感觉。

### 组织你的输入<a id="organizing-your-input"></a>

我喜欢将所有归属事件放在我的输入文件中一起；这使得更新和核对它们变得更容易，特别是有多个奖励的情况下。例如，对于两个奖励，我将有多个这样的交易块，用 4-5 行空行分隔它们：

```plaintext
2015-05-27 * "Vesting Event - S0012345 - HOOL" #award-S0012345 ^392f97dd62d0
  doc: "2015-02-13.hooli.38745783.pdf"
  Income:US:Hooli:RSU                    -4597.95 USD
  Assets:US:Hooli:RSURefund               2458.97 USD
  Expenses:Taxes:TY2015:US:Medicare         66.68 USD
  Expenses:Taxes:TY2015:US:Federal        1149.48 USD
  Expenses:Taxes:TY2015:US:CityNYC         195.42 USD
  Expenses:Taxes:TY2015:US:SDI               0.00 USD
  Expenses:Taxes:TY2015:US:StateNY         442.32 USD
  Expenses:Taxes:TY2015:US:SocSec          285.08 USD

2015-05-27 * "Vesting Event - C123456 - HOOL" #award-C123456 ^392f97dd62d0
  doc: "2015-02-13.hooli.38745783.pdf"
  Income:US:Hooli:RSU                    -1970.55 USD
  Assets:US:Hooli:RSURefund               1053.84 USD
  Expenses:Taxes:TY2015:US:Medicare         28.58 USD
  Expenses:Taxes:TY2015:US:Federal         492.63 USD
  Expenses:Taxes:TY2015:US:CityNYC          83.75 USD
  Expenses:Taxes:TY2015:US:SDI               0.00 USD
  Expenses:Taxes:TY2015:US:StateNY         189.57 USD
  Expenses:Taxes:TY2015:US:SocSec          122.18 USD

2015-05-25 * "Conversion into shares" ^392f97dd62d0
  Assets:US:Schwab:HOOL                        18 HOOL {131.3700 USD}
  Assets:US:Hooli:RSURefund
  Assets:US:Hooli:Unvested:S0012345           -35 HOOL.UNVEST
  Expenses:Hooli:Vested                        35 HOOL.UNVEST

2015-05-25 * "Conversion into shares" ^392f97dd62d0
  Assets:US:Schwab:HOOL                         9 HOOL {131.3700 USD}
  Assets:US:Hooli:RSURefund
  Assets:US:Hooli:Unvested:C123456            -15 HOOL.UNVEST
  Expenses:Hooli:Vested                        15 HOOL.UNVEST

2015-06-13 * "HOOLI INC       PAYROLL" ^392f97dd62d0
  doc: "2015-02-13.hooli.38745783.pdf"
  Assets:US:Hooli:RSURefund                -94.30 USD
  Assets:US:Hooli:RSURefund                 -2.88 USD
  Assets:US:BofA:Checking                   97.18 USD

2015-02-14 balance Assets:US:Hooli:RSURefund    0 USD
```

## 未归属股票<a id="unvested-shares"></a>

### 断言未归属余额<a id="asserting-unvested-balances"></a>

最后，你可能偶尔想要断言未归属股票的数量。例如，我喜欢每半年这样做一次。负责 Hooli RSU 的经纪公司应该能够列出每个奖励的剩余未归属股票数量，因此只需在网站上查找即可：

```plaintext
2015-06-04 balance Assets:US:Hooli:Unvested:S0012345  1645 HOOL.UNVEST
2015-06-04 balance Assets:US:Hooli:Unvested:C123456    705 HOOL.UNVEST
```

### 为未归属股票定价<a id="pricing-unvested-shares"></a>

你还可以为未归属股票定价，以估算未归属的美元金额。你应该使用一个虚构的货币，因为我们希望避免生成包含这些未归属资产为常规美元的资产负债表：

```plaintext
2015-06-02 price HOOL.UNVEST               132.4300 USD.UNVEST
```

在撰写本文时，bean-web 界面不会转换这些单位，如果它们不是按成本持有，但使用 SQL 查询界面或编写自定义脚本，你应该能够生成这些数字：

```bash
$ bean-query examples/vesting/vesting.beancount

beancount> select account, sum(convert(position, 'USD.UNVEST')) as unvested 
           where account ~ 'Unvested' group by account;

             account                     unvested       
--------------------------------- ----------------------
Assets:US:Hooli:Unvested:S0012345 217847.3500 USD.UNVEST
Assets:US:Hooli:Unvested:C123456   93363.1500 USD.UNVEST
```

## 出售归属股票<a id="selling-vested-stock"></a>

在每个归属事件之后，股票留在你的经纪账户中。出售这些股票的过程与任何其他交易相同（详见[<u>使用 Beancount 进行交易</u>](trading_with_beancount.md)）。例如，出售示例中的股票如下所示：

```plaintext
2015-09-10 * "Selling shares"
  Assets:US:Schwab:HOOL        -26 HOOL {131.3700 USD} @ 138.23 USD
  Assets:US:Schwab:Cash    3593.98 USD
  Income:US:Schwab:Gains
```

这里你可以看到为什么在转换事件中使用正确的成本基础很重要：你将不得不为差额支付税款（在`Income:US:Schwab:Gains`中）。在这个例子中，应税差额是每股（138.23 - 131.37）美元。

我喜欢将所有经纪交易保存在文档的一个单独部分，其中包含与经纪相关的其他交易，例如费用、股息和转账。

## 结论<a id="conclusion"></a>

这是一个简单的示例，基于科技公司处理这种类型补偿的方式。它绝不是全面的，一些细节必然会因你的情况而有所不同。特别是，它没有解释如何处理期权（ISO）。我希望本文档中提供了足够的内容，使你能够推断并适应你的特定情况。如果遇到问题，请在[<u>邮件列表</u>](http://furius.ca/beancount/doc/mailing-list)上联系。
