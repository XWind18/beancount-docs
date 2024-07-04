# 在 Beancount 中进行交易<a id="title"></a>

[马丁·布莱斯](mailto:blais@furius.ca)，2014年7月

[http://furius.ca/beancount/doc/trading](http://furius.ca/beancount/doc/trading)

> [<u>简介</u>](#introduction)
>
> [<u>什么是利润和亏损？</u>](#what-is-profit-and-loss)
>
> [<u>已实现和未实现的利润和亏损</u>](#realized-and-unrealized-pl)
>
> [<u>交易批次</u>](#trade-lots)
>
> [<u>入账方法</u>](#booking-methods)
>
> [<u>带日期的批次</u>](#dated-lots)
>
> [<u>报告未实现的利润和亏损</u>](#reporting-unrealized-pl)
>
> [<u>佣金</u>](#commissions)
>
> [<u>股票拆分</u>](#stock-splits)
>
> [<u>成本基础调整</u>](#cost-basis-adjustment-and-return-of-capital)
>
> [<u>股息</u>](#dividends)
>
> [<u>平均成本入账</u>](#average-cost-booking)
>
> [<u>未来主题</u>](#future-topics)

## 简介<a id="introduction"></a>

本文是《命令行会计食谱》的配套文档，专门讨论 Beancount 中的交易和投资主题。在阅读本文之前，您可能应该先阅读《双重记账法简介》。

在讨论股票交易之前，我们需要先了解“利润和亏损”（P/L），也称为资本收益或损失。对于初学者来说，理解多个交易中的利润和亏损可能比较困难，甚至专业交易员在不同时间段内理解 P/L 时也可能缺乏技巧。值得花一些时间来解释这个问题，并且了解如何在双重记账系统中入账。

在讨论过程中，我们将通过详细示例展示如何在 Beancount 中记录这些交易。您可能还会对 Beancount 中改进入账方法的提案感兴趣。本篇不涉及免税账户的成本基础，但会在更通用的食谱中讨论。

## 什么是利润和亏损？<a id="what-is-profit-and-loss"></a>

假设您在 E*Trade 折扣经纪公司开设了一个账户，并购买了一些 IBM 公司的股票。如果您以每股 160 美元的价格购买 10 股 IBM 股票，这将花费您 1600 美元。这就是我们所说的“账面价值”或“成本”。这笔钱是您为了获得这些股票而花费的，也称为“头寸”。在 Beancount 中，您可以这样记录这笔交易：

```plaintext
2014-02-16 * "Buying some IBM"
  Assets:US:ETrade:IBM                 10 IBM {160.00 USD}
  Assets:US:ETrade:Cash          -1600.00 USD
```

实际上，您可能需要向 E*Trade 支付一些佣金，所以为了完整起见，我们将其加入：

```plaintext
2014-02-16 * "Buying some IBM"
  Assets:US:ETrade:IBM                 10 IBM {160.00 USD}
  Assets:US:ETrade:Cash          -1609.95 USD
  Expenses:Financial:Commissions     9.95 USD
```

这是告诉 Beancount 以“成本”存入一些单位的方法，在这种情况下是每股 160 美元的 IBM 单位。这个交易平衡，因为其所有部分的总和为零：10 x 160 + -1609.95 + 9.95 = 0。此外，我们选择使用一个专用于 IBM 股票的子账户；这不是必须的，但在例如资产负债表报告中会很方便，因为它会自然地将您每个头寸的所有股票聚合到它们自己的行中。拥有一个“现金”子账户还强调了您在那里没有投资的资金没有带来任何回报。

第二天，市场开盘，IBM 股票的价格为每股 170 美元。在这种情况下，我们将其称为“价格”。您的头寸的“市场价值”是您的股票数量乘以市场价格，即 10 股 x 170 美元/股 = 1700 美元。

这两个金额之间的差额就是我们所说的利润和亏损：

> 市场价值 - 账面价值 = 利润和亏损
>
> 10 x 170 美元 - 10 x 160 美元 = 1700 美元 - 1600 美元 = 100 美元（利润）

我们将正数称为“利润”，如果金额为负数，则称为“亏损”。

## 已实现和未实现的利润和亏损<a id="realized-and-unrealized-pl"></a>

上一节中的利润称为“未实现利润”。这是因为股票尚未实际卖出——这是一个假设的利润：如果我可以以市场价值卖出这些股票，我将获得多少钱。上一节中提到的 100 美元实际上是“未实现的利润和亏损”。

假设您喜欢这个未实现的利润，认为 IBM 涨价是暂时的运气。您决定以每股 170 美元的价格卖出这 10 股中的 3 股。现在，这些股票的利润将被“实现”：

> 市场价值 - 账面价值 = 利润和亏损
>
> 3 x 170 美元 - 3 x 160 美元 = 3 x (170 - 160) = 30 美元（利润）

这 30 美元是“已实现的利润和亏损”。您的头寸的剩余部分仍显示未实现利润，即价格可能会波动，直到您卖出它：

> 市场价值 - 账面价值 = 利润和亏损
>
> 7 x 170 美元 - 7 x 160 美元 = 70 美元

这就是您在 Beancount 中记录这部分头寸卖出的方式（再次包括佣金）：

```plaintext
2014-02-17 * "Selling some IBM"
  Assets:US:ETrade:IBM                 -3 IBM {160.00 USD}
  Assets:US:ETrade:Cash            500.05 USD
  Expenses:Financial:Commissions     9.95 USD
```

您是否注意到这里有些奇怪的地方？-3 x 160 = -480，-480 + 500.05 + 9.95 = 30……这个交易不平衡到零！问题是我们以 510 美元现金换取我们卖出的 3 股股票。这是因为我们实际以 170 美元的价格卖出了它们：3 x 170 = 510 美元。这就是我们需要通过添加另一条分录来记录利润，并方便地自动计算和跟踪我们的利润：

```plaintext
2014-02-17 * "Selling some IBM"
  Assets:US:ETrade:IBM                 -3 IBM {160.00 USD}
  Assets:US:ETrade:Cash            500.05 USD
  Expenses:Financial:Commissions     9.95 USD
  Income:US:ETrade:PnL
```

最后一条分录将由 Beancount 自动填充为 `-30 USD`，因为我们允许一个分录没有金额（请记住，在没有借方和贷方的双重记账系统中，利润是“收入”账户的负数）。这是政府对您的税收感兴趣的数字。

总结一下，您现在拥有：

> 7 股“账面价值为 160 美元的股票”= 1120 美元（账面价值）
>
> 30 美元的已实现利润和亏损
>
> 70 美元的未实现利润和亏损

## 交易批次<a id="trade-lots"></a>

实际上，交易的现实情况比这稍微复杂一些。您可能会多次购买 IBM，每次购买的价格可能不同。让我们通过另一个交易示例来看看这如何运作。考虑到您之前持有的 7 股账面价值为 160 美元的股票，第二天您看到价格进一步上涨，决定看好 IBM 并购买 5 股。这次您以每股 180 美元的价格购买：

```plaintext
2014-02-18 * "I put my chips on big blue!"
  Assets:US:ETrade:IBM                 5 IBM {180.00 USD}
  Assets:US:ETrade:Cash           -909.95 USD
  Expenses:Financial:Commissions     9.95 USD
```

现在，在 `Assets:US:ETrade:IBM` 账户中我们有什么？我们有两种东西：

-   7 股“账面价值为 160 美元的 IBM”，来自首次交易
-   5 股“账面价值为 180 美元的 IBM”，来自这次交易

我们将这些称为“批次”或“交易批次”。

实际上，如果您一个月后决定卖出整个头寸，您可以通过指定两个分录来合法地在 Beancount 中卖出它（即不产生错误）。假设此时的价格为每股 172 美元：

```plaintext
2014-03-18 * "Selling all my blue chips."
  Assets:US:ETrade:IBM          -7 IBM {160.00 USD} @ 172.00 USD
  Assets:US:ETrade:IBM          -5 IBM {180.00 USD}
  Assets:US:ETrade:Cash         2054.05 USD
  Expenses:Financial:Commissions   9.95 USD
  Income:US:ETrade:PnL
```

这样，您的 IBM 最终头寸为 0 股。

## 入账方法<a id="booking-methods"></a>

但是，如果您决定只卖出其中一些股票呢？假设您需要一些现金买礼物，想卖出 4 股。假设此时价格为每股 175 美元。

现在，您可以选择卖出较旧的股票，获得更大的利润：

```plaintext
2014-03-18 * "Selling my older blue chips."
  Assets:US:ETrade:IBM          -4 IBM {160.00 USD} @ 175.00 USD
  Assets:US:ETrade:Cash          690.05 USD
  Expenses:Financial:Commissions   9.95 USD
  Income:US:ETrade:PnL        ;; -60.00 USD (profit)
```

或者您可以选择卖出最近购买的股票，获得亏损：

```plaintext
2014-03-18 * "Selling my most recent blue chips."
  Assets:US:ETrade:IBM          -4 IBM {180.00 USD} @ 175.00 USD
  Assets:US:ETrade:Cash          690.05 USD
  Expenses:Financial:Commissions   9.95 USD
  Income:US:ETrade:PnL        ;;  20.00 USD (loss)
```

或者您可以选择两者混合卖出：只需使用两个分录。

注意，在实际操作中，这个选择将取决于多种因素：

-   您交易股票的司法管辖区的税法可能规定了如何入账的方法，您可能实际上没有选择。例如，他们可能规定您必须交易最早购买的批次，这种方法称为“先进先出”。

-   如果您有选择，您持有的各个批次可能具有不同的税收特征，因为您持有它们的时间不同。在美国，例如，持有超过一年的头寸享受较低的税率（“长期”资本收益税率）。

-   您可能有其他的收益或损失，您希望抵消这些，以尽量减少您的税务责任现金流需求。这有时称为“税收亏损收割”。

还有更多……但我不会在这里详细说明。我的目标是向您展示如何使用双重记账法记录这些交易。

## 带日期的批次<a id="dated-lots"></a>

我们几乎已经完成了整个工作原理的介绍。还有一个相当技术性的问题需要解决，这从一个问题开始：如果我以相同价格购买了多个批次的股票怎么办？

正如我们在上一节中提到的，您持有头寸的时间长短可能会对您的税收产生影响，即使利润和亏损的结果相同。我们如何区分这些批次？

嗯……我之前简化了一些内容，以便于理解。当我们将头寸放入库存时，我们在附加到我们放入的物品上的标签上也会记录下该批次的购买日期，如果您提供的话。这是您以这种方式记录进入头寸的方式：

```plaintext
2014-05-20 * "First trade"
  Assets:US:ETrade:IBM          5 IBM {180.00 USD, 2014-05-20}
  Assets:US:ETrade:Cash           -909.95 USD
  Expenses:Financial:Commissions     9.95 USD

2014-05-21 * "Second trade"
  Assets:US:ETrade:IBM          3 IBM {180.00 USD, 2014-05-21}
  Assets:US:ETrade:Cash           -549.95 USD
  Expenses:Financial:Commissions     9.95 USD
```

现在，当您卖出时，您可以做同样的操作，以明确您要减少哪个批次的头寸：

```plaintext
2014-08-04 * "Selling off first trade"
  Assets:US:ETrade:IBM         -5 IBM {180.00 USD, 2014-05-20}
  Assets:US:ETrade:Cash            815.05 USD
  Expenses:Financial:Commissions     9.95 USD
  Income:US:ETrade:PnL
```

## 报告未实现的利润和亏损<a id="reporting-unrealized-pl"></a>

好的，所以我们的账户余额持有每个单位的成本，为我们提供这些头寸的账面价值。不错。但市场价值呢？

头寸的市场价值只是这些工具的单位数量乘以我们感兴趣的时间点的市场价格。这个价格是波动的。所以我们需要价格。

Beancount 支持一种名为 `price` 的条目类型，允许您告诉它某个时间点的工具价格，例如：

```plaintext
2014-05-25 price IBM   182.27 USD
```

为了保持 Beancount 简单且依赖性少，软件不会自动获取这些价格（您可以查看 LedgerHub 来实现这一目的，或者编写自己的脚本，将最新价格插入您的输入文件中……有很多库可以在线获取价格）。它只知道所有这些价格条目的市场价格。使用这些，构建了一个内存中的历史价格数据库，并可以查询以获得最当前的值。

与其通过选项支持不同的报告模式，您可以通过启用一个插件来触发未实现收益的插入：

```plaintext
plugin "beancount.plugins.unrealized" "Unrealized"
```

这将在最后一个指令的日期创建一个合成交易，反映未实现的利润和亏损。它将一侧记录为收入，另一侧记录为资产的变化：

```plaintext
2014-05-25 U "Unrealized gain for 7 units of IBM (price:
              182.2700 USD as of 2014-05-25,
              average cost: 160.0000 USD)"
  Assets:US:ETrade:IBM:Unrealized          155.89 USD
  Income:US:ETrade:IBM:Unrealized         -155.89 USD
```

注意在这个例子中，我使用了一个选项来指定将未实现收益记入的子账户。未实现的利润和亏损在资产负债表中显示为单独的一行，父账户应显示市场价值在其余额中（包括其子账户的价值）。

## 佣金<a id="commissions"></a>

到目前为止，我们还没有讨论交易佣金。根据适用的税法，与交易相关的费用可能从我们之前计算的原始资本收益中扣除。这些费用被政府视为支出，通常情况下，您可以扣除这些交易佣金（这完全合理，因为这些钱并没有进入您的口袋）。

在上述例子中，资本收益和佣金支出被跟踪到两个独立的账户。例如，您可能最终报告的余额如下所示：

```plaintext
Income:US:ETrade:PnL                -645.02 USD
Expenses:Financial:Commissions        39.80 USD
```

（为了明确，这表示 645.02 美元的*利润*和 39.80 美元的支出。）您可以减去这些数字，以获得没有成本的 P/L 的*近似值*：645.02 - 39.80 = 605.22 美元。然而，这只是正确 P/L 值的近似值。为了理解原因，我们需要看看一个在报告期内部分股票卖出的例子。

假设我们有一个佣金率为每笔交易 10 美元的账户，2013 年购买了 100 股 ITOT，其中 40 股在同年卖出，其余 60 股在次年卖出，情景如下：

> 2013-09-01 以每股 80 美元购买 100 股 ITOT，**佣金 = 10 美元**
>
> 2013-11-01 以每股 82 美元卖出 40 股 ITOT，佣金 = 10 美元
>
> 2014-02-01 以每股 84 美元卖出 60 股 ITOT，佣金 = 10 美元

如果您在 2013 年底计算支付的佣金总额为 20 美元，并使用前面提到的近似方法，则在 2013 和 2014 年您将声明

> 2013：P/L 为 40 x（82 美元 - 80 美元）-（10 美元 + 10 美元）= 60 美元
>
> 2014：P/L 为 60 x（84 美元 - 80 美元）- 10 美元 = 230 美元

然而，严格来说，这是不正确的。用于*获取* 100 股的 10 美元佣金必须按卖出的股票数量按比例分配。这意味着在首次卖出 40 股时，只有 4 美元的佣金是可扣除的：10 美元 x（40 股 / 100 股），因此我们得到：

> 2013：P/L 为 40 x（82 美元 - 80 美元）-（4 美元 + 10 美元）= 66 美元
>
> 2014：P/L 为 60 x（84 美元 - 80 美元）-（6 美元 + 10 美元）= 224 美元

如您所见，每年的 P/L 声明不同，即使两年的 P/L 总和相同（290 美元）。

自动分配获取交易成本的方法是将获取交易成本添加到头寸的总账面价值中。在这个例子中，您可以说 100 股的头寸账面价值为 8010 美元，而不是 8000 美元：100 股 x 80 美元/股 + 10 美元，或者等效地，个别股票的账面价值为每股 80.10 美元。这将导致以下计算：

> 2013：P/L 为 40 x（82 美元 - 80.10 美元）- 10 美元 = 66 美元
>
> 2014：P/L 为 60 x（84 美元 - 80.10 美元）- 10 美元 = 224 美元

您甚至可以更进一步，将*销售*的佣金也合并到每股的价格中：

> 2013：P/L 为 40 x（81.75 美元 - 80.10 美元）= 66 美元
>
> 2014：P/L 为 60 x（83.8333 美元 - 80.10 美元）= 224 美元

这可能显得过头了，但想象一下，如果这些费用更高，例如在大宗商业交易中；细节对税务官员来说开始变得重要。准确的会计是重要的，我们需要开发一种更精确的方法来做到这一点。

## 股票拆分<a id="stock-splits"></a>

股票拆分目前通过清空账户的头寸并以不同价格重新创建头寸来处理：

```plaintext
2004-12-21 * "Autodesk stock splits"
  Assets:US:MSSB:ADSK          -100 ADSK {66.30 USD}
  Assets:US:MSSB:ADSK           200 ADSK {33.15 USD}
```

这些分录彼此平衡，因此规则得以遵守。正如您所见，这不需要特殊的语法功能。它还处理更一般的情况，例如 2014 年 4 月在 NASDAQ 交易所发生的 Google 公司股票的奇特拆分，拆分为两种不同类别的股票（具有投票权和无投票权的股票，分别为 50.08% 和 49.92%）：

```plaintext
2014-04-07 * "Stock splits into voting and non-voting shares"
  Assets:US:MSSB:GOOG        -25 GOOG {1212.51   USD} ; Old GOOG
  Assets:US:MSSB:GOOG         25 GOOG { 605.2850 USD} ; New GOOG
  Assets:US:MSSB:GOOGL        25 GOOG { 607.2250 USD}
```

最终，也许应该提供一个插件模块，以更轻松地创建这种股票拆分交易，因为涉及到一些冗余。我们需要找出最通用的方法来做到这一点。但目前上述方法可行。

这种方法的一个问题是批次的*连续性*丢失了，即每个批次的购买日期由于上述交易而被重置，无法自动确定交易的持续时间及其对税收的影响，即长期交易与短期交易。即使没有这个，利润仍然可以正确计算，但这是一个令人讨厌的细节。

处理这种情况的一种方法是使用带日期的批次（请参见本文档的相关部分）。这样，新的批次可以保留原始交易日期。这不仅提供了准确的时间信息，还提供了基于价格的资本增减。

另一种解决这个问题并轻松传播批次交易日期的方法[已被提议](a_proposal_for_an_improvement_on_inventory_booking.md)，并将在 Beancount 中实现。

当前实现的一个更重要的问题是，ADSK 股票拆分前后的单位含义不同。该商品单位的价格图将显示一个明显的断裂！这是一个更普遍的问题，尚未在 Beancount 和 Ledger 中解决。有关这个主题的讨论，请参阅 [商品定义更改](https://docs.google.com/document/d/1Y_h5sjUTJzdK1riRh-mrVQm9KCzqFlU65KMsMsrQgXk/) 文档。

## 成本基础调整和资本回报<a id="cost-basis-adjustment-and-return-of-capital"></a>

由于基金的内部交易活动，管理基金的成本基础可能会发生调整。这通常发生在免税账户中，这些账户的调整不会对税收产生影响，成本基础保持在每个头寸的平均成本上。

如果我们有具体的批次价格进行调整，可以按照我们处理股票拆分的方式进行入账：

```plaintext
2014-04-07 * "Cost basis adjustment for XSP"
  Assets:CA:RRSP:XSP           -100 ADSK {21.10 CAD}
  Assets:CA:RRSP:XSP            100 ADSK {23.40 CAD}
  Income:CA:RRSP:Gains      -230.00 CAD
```

然而，这种情况非常罕见。更常见的情况是使用平均成本入账的方法，我们目前没有办法处理这种情况。有一个[积极的提议](a_proposal_for_an_improvement_on_inventory_booking.md)正在进行，以使这成为可能。

成本基础调整通常出现在资本回报事件中。例如，当基金向股东返还资本时，这可能是由于停止运营而引起的。从税收角度来看，这些是非应税事件，影响基金中股权的成本基础。股票数量可能保持不变，但它们的成本基础需要调整，以便在未来出售时计算潜在的增减。

## 股息<a id="dividends"></a>

股息不会带来特别的问题。它们只是收入。可以以现金形式收到：

```plaintext
2014-02-01 * "Cash dividends received from mutual fund RBF1005"
  Assets:Investments:Cash            171.02 CAD
  Income:Investments:Dividends
```

也可以以股票形式收到：

```plaintext
2014-02-01 * "Stock dividends received in shares"
  Assets:Investments:RBF1005          7.234 RBF1005 {23.64 CAD}
  Income:Investments:Dividends
```

在收到股票股息的情况下，如同股票购买，您需要提供收到股息时的成本基础（这应该可以在您的对账单中找到）。如果账户按平均成本持有，这个分录将在需要执行平均成本入账时与其他分录合并。

## 平均成本入账<a id="average-cost-booking"></a>

目前，执行平均成本入账的唯一方法是痛苦的：您必须使用股票拆分部分中概述的方法来重新估值您的库存。然而，这在实际操作中是不可行的。有一个[积极的提议](a_proposal_for_an_improvement_on_inventory_booking.md)和相关语法，以全面解决这个问题。

一旦提案得到实施，它将如下所示：

```plaintext
2014-02-01 * "Selling 5 shares at market price 550 USD"
  Assets:Investments:Stock               -5 GOOG {*}
  Assets:Investments:Cash           2740.05 USD
  Expenses:Commissions                 9.95 USD
  Income:Investments:CapitalGains
```

任何在库存上使用成本为“\*”的分录将选择该货币（GOOG）的所有股票，将它们合并为一个按平均成本计算的股票，然后以新的平均成本减少该头寸。

## 未来主题<a id="future-topics"></a>

我将稍后处理以下主题：

-   **按市值计算**：处理第 1256 条款工具（即期货和期权）的年终按市值计算，通过重新评估成本基础。这类似于对所有这些类型的工具在每年年底应用的成本基础重新调整。

-   **卖空**：这需要一些改动。我们只需要允许按成本持有的单位数量为负数。目前，当按成本持有的单位数量变为负数时，我们会发出警告，以检测数据录入错误，但可以轻松扩展开放指令语法，以允许特定账户发生这种情况，这些账户可以持有卖空头寸，应显示为负数股票。否则，所有算术运算应该自然地工作。保证金的利息支付将显示为独立的交易。此外，当您卖空股票时，您不会为这些头寸收到股息，而是必须支付股息。您将有一个支出账户，例如 `Expenses:StockLoans:Dividends`。

-   **交易期权**：目前我不知道如何处理这个问题，但我想这些期权可以像股票一样持有，没有区别。我不预见任何困难。

-   **货币交易**：目前，我没有对我的外汇账户中的头寸进行会计处理，只是处理它们的 P/L 和利息支付。这带来了有趣的问题：

    -   外汇账户中的头寸不仅仅是股票的多头或空头：它们实际上是同时抵消两种商品。例如，持有多头头寸的 USD/CAD 应增加 USD 的敞口并减少 CAD 的敞口，可以看作同时持有 USD 的多头资产和 CAD 的空头资产。虽然可以将这些头寸视为独立的工具（例如，不考虑其组成部分的“USDCAD”单位），但对于长期持有的大头寸，特别是出于对冲目的，重要的是处理这一点，并以某种方式允许用户反映多种货币头寸的净货币敞口与其余资产和负债的对比。

    -   我们还需要处理这些头寸关闭时产生的收益：这些收益在账户货币中产生，经过转换为该货币后。例如，如果您持有以 USD 计价的货币账户，并且您做多 EUR/JPY，当您平仓时，您将获得 EUR 的收益，并在将 P/L 从 EUR 转换为等值的 USD（通过 EUR/USD）后，USD 的收益将存入您的账户。这意味着估算任何头寸的当前市场价值需要使用两个汇率：购买时的当前汇率差异和账户货币中的基准货币（例如 EUR）的汇率（例如 USD）。

其中一些涉及 Beancount 中的新功能，但有些不涉及。欢迎提供想法。
