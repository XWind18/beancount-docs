# 库存记账

改进命令行记账的提案

<a id="title"></a>

Martin Blais，2014年6月

[<u>http://furius.ca/beancount/doc/proposal-booking</u>](http://furius.ca/beancount/doc/proposal-booking)

> [<u>动机</u>](#motivation)
>
> [<u>问题描述</u>](#problem-description)
>
> [<u>批次日期</u>](#lot-date)
>
> [<u>平均成本基础</u>](#average-cost-basis)
>
> [<u>不含佣金的资本收益</u>](#capital-gains-sans-commissions)
>
> [<u>成本基础调整</u>](#cost-basis-adjustments)
>
> [<u>处理股票分割</u>](#dealing-with-stock-splits)
>
> [<u>以前的解决方案</u>](#previous-solutions)
>
> [<u>Ledger 的缺点</u>](#shortcomings-in-ledger)
>
> [<u>Beancount 的缺点</u>](#shortcomings-in-beancount)
>
> [<u>要求</u>](#requirements)
>
> [<u>调试工具</u>](#debugging-tools)
>
> [<u>显式与隐式记账减少</u>](#explicit-vs.-implicit-booking-reduction)
>
> [<u>设计提案</u>](#design-proposal)
>
> [<u>库存</u>](#inventory)
>
> [<u>输入语法和过滤</u>](#input-syntax-filtering)
>
> [<u>算法</u>](#algorithm)
>
> [<u>隐式记账方法</u>](#implicit-booking-methods)
>
> [<u>默认插入的日期</u>](#dates-inserted-by-default)
>
> [<u>没有信息的匹配</u>](#matching-with-no-information)
>
> [<u>减少多个批次</u>](#reducing-multiple-lots)
>
> [<u>示例</u>](#examples)
>
> [<u>没有冲突</u>](#no-conflict)
>
> [<u>按成本的显式选择</u>](#explicit-selection-by-cost)
>
> [<u>按日期的显式选择</u>](#explicit-selection-by-date)
>
> [<u>按标签的显式选择</u>](#explicit-selection-by-label)
>
> [<u>按组合的显式选择</u>](#explicit-selection-by-combination)
>
> [<u>单位不足</u>](#not-enough-units)
>
> [<u>相同批次的冗余选择</u>](#redundant-selection-of-same-lot)
>
> [<u>自动价格外推</u>](#automatic-price-extrapolation)
>
> [<u>平均成本记账</u>](#average-cost-booking)
>
> [<u>未来工作</u>](#future-work)
>
> [<u>实现说明</u>](#implementation-notes)
>
> [<u>结论</u>](#conclusion)

## 动机<a id="motivation"></a>

“库存记账”问题，即在关闭一个头寸时选择减少哪个库存批次，是一个棘手的问题。在命令行会计社区中，目前为止在支持处理许多常见的现实案例方面进展相对较少。本文讨论了当前的情况，描述了我们希望能够解决的常见用例，确定了一套改进的记账方法的要求，并提出了一种支持这些要求的语法和实现设计，以及对记账方法的明确定义语义。

## 问题描述<a id="problem-description"></a>

要解决的问题是，在试图减少特定账户中某一头寸的双重记账交易中，确定应该减少账户中哪个库存批次。这应该通过简单的数据输入语法来指定，尽量减少用户的负担。

例如，可以通过在不同时间点购买两批 HOOL 股票来持有该股票：

```plaintext
2014-02-01 * "First trade"
  Assets:Investments:Stock        10 HOOL {500 USD}
  Assets:Investments:Cash
  Expenses:Commissions          9.95 USD

2014-02-15 * "Second trade"
  Assets:Investments:Stock         8 HOOL {510 USD}
  Assets:Investments:Cash
  Expenses:Commissions          9.95 USD
```

我们将称这两批股票为“交易批次”，并假设用于计算的底层系统分别跟踪每个批次的成本和购买日期。

这里的问题是，如果我们要卖掉一些股票，我们应该选择哪些股票？是第一批还是第二批？这是一个重要的决定，因为它会影响资本收益，从而影响税收（关于资本收益的更多信息，请参阅[<u>Beancount 烹饪书中的“股票交易”部分</u>](trading_with_beancount.md)）。根据我们的财务状况、今年的交易历史和未实现的收益，有时我们可能希望选择一个批次而不是另一个批次。现在，大多数折扣经纪商甚至允许您在进行减少或关闭头寸的交易时选择特定的批次。

这里要强调的是库存记账的两个方向：

1. **增加头寸。** 这是创建一个新交易批次或添加到现有交易批次的过程（如果成本和其他属性相同）。这很简单，相当于在描述账户余额的某种映射中添加一条记录（我称之为“库存”）。

2. **减少头寸。** 这通常是记账问题发生的地方，问题本质上是确定从现有头寸中移除哪个批次的单位。

本文的大部分内容都致力于第二个方向。

### 批次日期<a id="lot-date"></a>

即使每个批次的成本相同，您打算关闭的头寸的购买日期也很重要，因为不同的税收规则可能适用。例如，在美国，持有超过 12 个月的头寸适用显著较低的税率（“长期资本收益税率”）。例如，如果您以以下方式建立头寸：

```plaintext
2012-05-01 * "First trade"
  Assets:Investments:Stock        10 HOOL {300 USD}
  Assets:Investments:Cash
  Expenses:Commissions          9.95 USD

2014-02-15 * "Second trade at same price"
  Assets:Investments:Stock         8 HOOL {300 USD}
  Assets:Investments:Cash
  Expenses:Commissions          9.95 USD
```

如果我们在 2014-03-01 进行交易，选择第一个批次将导致长期（低）收益，因为自购买以来已过两年（从 2012-05-01 持有至 2014-03-01），而选择第二个批次将导致短期（高）资本收益税。因此，这不仅仅是批次成本的问题。我们目前的系统尚未自动报税，但我们希望以一种最终允许进行此类报告的方式输入数据。

注意，这里的讨论假设您能够选择与哪个批次进行交易。可能适用的税收规则有多种，有些情况下，取决于您所在的国家，您可能没有选择权，政府可能会规定您如何报告收益。

在某些情况下，您甚至可能需要对同一账户应用多种记账方法（例如，如果一个账户需要多个国家的税收申报）。我们将忽略本文中的这个特殊情况，因为它非常罕见。

### 平均成本基础<a id="average-cost-basis"></a>

对于免税账户，事情变得更加复杂。因为在这些账户中关闭头寸没有税收影响，经纪商通常会选择将您的头寸的账面价值视为单个批次。也就是说，每股的成本使用持有头寸的加权平均成本计算。这可以通过将每个头寸的总成本相加，然后除以总股数来计算。继续我们的第一个示例：

> *10 HOOL x 500 USD/HOOL = 5000 USD*
>
> *8 HOOL x 510 USD/HOOL = 4080 USD*
>
> *成本基础 = 5000 USD + 4080 USD = 9080 USD*
>
> *总股数 = 10 HOOL + 8 HOOL = 18 HOOL*
>
> *每股成本基础 = 9080 USD / 18 HOOL = 504.44 USD/HOOL*

因此，如果您要关闭部分头寸并卖出一些股票，您将以每股 504.44 USD 的成本进行交易。您实现的收益将相对于此成本计算；例如，如果您以每股 520 USD 卖出 5 股，您的收益将按如下方式计算：

    (520 USD/HOOL - 504.44 USD/HOOL) x 5 HOOL = 77.78 USD

这种记账方法在这些类型的账户中可能会出现两种类型的财务事件：

1. 费用可能通过以当前市场价格卖出一些股票来扣除。如果您在美国持有 Vanguard 的退休账户，您会看到这些情况。经纪商每季度收取固定金额的费用（我的雇主计划为每季度 5 美元），这会以每个基金出售的一小部分股票的形式体现出来。此交易忽略了资本收益：他们只是计算需要卖出多少股票以支付费用，并不向您报告收益。您必须假设这无关紧要。大多数人不会在免税账户中跟踪他们的收益，但像我们这样的人可能仍然想计算回报。

2. 成本基础可能会自发调整。这很罕见，但我曾在加拿大的一个免税账户中看到过一次或两次：根据基金管理交易活动中的一些未公开细节，基金管理需要对成本基础进行调整，您需要将此应用到您的头寸中。您会收到一封电子邮件告知调整金额。99% 的人可能不会在意，不理解并忽略此消息……或许是对的，因为这不会影响他们的税收，但如果您想在该账户中计算您的交易回报，您确实需要计算它。所以我们需要能够增加或减少现有头寸的成本基础。

注意，您的头寸平均成本会在每次交易时发生变化，并且在每次增加现有头寸时需要重新计算。这种方法的一个问题是，如果您跟踪每股成本，这些多次重新计算可能会导致存储金额时出现一些错误（通常是十进制数）。一种更稳定的解决方法是仅跟踪总成本，而不是每股成本。还要注意，使用小数并不能充分解决这个问题：实际上，经纪商可能会报告一个特定的成本，该成本四舍五入到特定的小数位数，我们可能希望能够使用该报告的数字来进行减少交易，而不是保持理想的金额。这些问题在此暂且不提。

总结：我们需要能够支持以平均成本进行交易，并且需要能够调整现有头寸的成本基础。我一直在考虑一种解决方案，不会强制将账户标记为特殊状态。我认为我们只需要一个小的语法变更，让用户可以指示系统按照平均成本进行记账。

使用前面的第一个示例，收购股票的形式与之前相同，即在购买 10 和 8 股 HOOL 后，我们最终持有两个批次，一个是 10 HOOL {500 USD}，另一个是 8 HOOL {510 USD}。然而，当要卖出时，将使用以下语法（这是我一直打算实现的一个旧想法）：

```plaintext
2014-02-01 * "Selling 5 shares at market price 550 USD"
  Assets:Investments:Stock               -5 HOOL {*}
  Assets:Investments:Cash           2759.95 USD
  Expenses:Commissions                 9.95 USD
  Income:Investments:CapitalGains
```

当遇到“`*`”代替成本时，它将：

1. 合并所有批次并重新计算每股平均价格（504.44 USD）

2. 根据这个合并的库存进行记账，减少合并后的批次。

交易后，我们将拥有一个单一的头寸 13 HOOL {504.44 USD}。在不同价格下增加该头寸会创建一个新批次，下次以平均成本减少时这些批次会再次合并。除去积累的四舍五入误差问题，这种解决方案在数学上是正确的，并且保持了账户不被强制为任何特定记账方法的属性——每次减少时可以使用多种方法。这是一个很好的属性，即使不是绝对必要的，即使我们希望能够为每个账户指定一个“默认”记账方法并始终使用它。支持例外情况总是很好的，因为在现实生活中，它们有时会发生。

### 不含佣金的资本收益<a id="capital-gains-sans-commissions"></a>

我们希望能够支持自动计算不含佣金的资本收益。这个问题在[<u>交易文档的佣金部分</u>](trading_with_beancount.md)和[<u>邮件列表线程</u>](https://groups.google.com/d/msg/ledger-cli/zRSMle5AV3Q/ySUgsMVNg-kJ)中有详细描述。复杂之处在于，不能简单地从包含佣金的收益中减去报告期间发生的佣金，因为为了购买头寸而产生的佣金必须按出售的股份比例计算。最简单且最常见的实现方法是将购买头寸的成本包括在头寸的成本基础中，并在减少头寸时从市场价值中扣除销售成本。

无论我们提出什么新设计，必须允许我们计算这些调整后的收益，因为这是各个个人的必要需求。在 Beancount 中，我已经找到了一个解决方案，幸运的是，这仅涉及自动转换由其过账上存在特殊标志触发的交易……如果且仅如果我能找到一种方法来指定要记账的批次而不需要指定其成本，因为一旦成本折算到头寸中，调整后的成本不会出现在输入文件中，用户不得不手动计算，这是不合理的。在我提出的解决方案中，以下交易会自动转换，并由插件替换原始交易：

```plaintext
2014-02-10 * "Buy"
  Assets:US:Invest:Cash    -5009.95 USD
  C Expenses:Commissions       9.95 USD
  Assets:US:Invest:HOOL       10.00 HOOL {500 USD / aa2ba9695cc7}

2014-04-10 * "Sell #1" 
  Assets:US:Invest:HOOL       -4.00 HOOL {aa2ba9695cc7}
  C Expenses:Commissions       9.95 USD
  Assets:US:Invest:Cash     2110.05 USD
  Income:US:Invest:Gains

2014-05-10 * "Sell #2"
  Assets:US:Invest:HOOL       -6.00 HOOL {aa2ba9695cc7}
  C Expenses:Commissions       9.95 USD
  Assets:US:Invest:Cash     3230.05 USD
  Income:US:Invest:Gains
```

它们将由插件自动转换为以下内容，并替换上述原始交易：

```plaintext
2014-02-10 * "Buy"
  Assets:US:Invest:Cash        -5009.95 USD
  X Expenses:Commissions           9.95 USD
  X Income:US:Invest:Rebates      -9.95 USD
  X Assets:US:Invest:HOOL         10.00 HOOL {500.995 USD / aa2ba9695cc7}

2014-04-10 * "Sell #1" 
  X Assets:US:Invest:HOOL         -4.00 HOOL {aa2ba9695cc7}
  X Expenses:Commissions           9.95 USD
  X Income:US:Invest:Rebates      -9.95 USD
  Assets:US:Invest:Cash         2110.05 USD
  Income:US:Invest:Gains ; Should be (530-500)*4 - 9.95*(4/10) - 9.95 = 
                         ; 106.07 USD

2014-05-10 * "Sell #2"
  X Assets:US:Invest:HOOL         -6.00 HOOL {aa2ba9695cc7}
  X Expenses:Commissions           9.95 USD
  X Income:US:Invest:Rebates      -9.95 USD
  Assets:US:Invest:Cash         3230.05 USD
  Income:US:Invest:Gains ; Should be (540-500)*6 - 9.95*(6/10) - 9.95 = 
                         ; 224.08 USD
```

“X”标志仅标记已转换的过账，因此以后可以以特殊颜色或属性呈现。折扣账户作为单独的对账户仅用于使交易平衡。

我需要放宽交易批次消歧的原因是用户需要能够指定匹配腿而不必指定头寸的成本，因为在减少头寸（2014-04-10）时，没有办法知道原始批次的成本。在上述示例中，这意味着如果所有信息只有 4 HOOL 和 500 USD，没有办法反推出 500.995 USD 的成本来指定与开盘交易批次匹配，因为那发生在 10 HOOL 的情况下。

因此我们需要更加显式地进行记账。在上述示例中，我通过标签选择匹配批次，即用户可以在成本规格中提供唯一的批次标识符，之后可以用于消歧要记账的减少/销售批次。上述示例使用字符串“`aa2ba9695cc7`”进行此操作。

（此问题的替代解决方案是同时跟踪*原始成本（不含佣金）和实际成本（含佣金），然后根据前者找到批次，但使用后者使交易平衡。这个想法是允许用户继续使用头寸成本选择批次，但在各种库存变动情况下不确定是否可行。这需要进一步考虑。）

### 成本基础调整<a id="cost-basis-adjustments"></a>

为了调整成本基础，可以通过以下方式显式替换账户内容：

```plaintext
2014-03-15 * "Adjust cost basis from 500 USD to 510 USD"
  Assets:US:Invest:HOOL      -10.00 HOOL {500 USD}
  Assets:US:Invest:HOOL       10.00 HOOL {510 USD}
  Income:US:Invest:Gains
```

这种方法效果很好，并允许系统自动计算收益。但支持以价格单位对总成本的调整也很好，例如“将此头寸的成本基础增加 340.51 USD”。问题是用户现在需要计算调整后的每股成本……这很不方便。以下是一个解决此问题的方法：

```plaintext
2014-03-15 * "Adjust cost basis from 500 USD to 510 USD"
  Assets:US:Invest:HOOL      -10.00 HOOL {500 USD}
  Assets:US:Invest:HOOL       10.00 HOOL {}
  Income:US:Invest:Gains    -340.51 USD
```

如果所有过账都完全指定——它们有可计算的余额——我们允许省略单一价格。这很好地解决了这个问题。

### 处理股票分割<a id="dealing-with-stock-splits"></a>

股票分割的会计处理会在记账方面带来一些复杂性。一般来说，问题在于我们需要处理商品随时间变化的含义。例如，您可以在 Beancount 中进行以下操作，它可以正常工作：

```plaintext
2014-01-04 * "Buy some HOOL"
  Assets:Investments:Stock        10 HOOL {1000.00 USD}
  Assets:Investments:Cash

2014-04-17 * "2:1 split into Class A and Class B shares"
  Assets:Investments:Stock       -10 HOOL {1000.00 USD}
  Assets:Investments:Stock        10 HOOL  {500.00 USD}
  Assets:Investments:Stock        10 HOOLL {500.00 USD}
```

这种方式分割批次的问题在于，分割前后 HOOL 单位的价格和含义不同，但暂且假设我们对此没有问题（这可以通过*重新标记*商品名称来解决，最好以用户不可见的方式进行——我们在[<u>其他地方有一个待讨论的话题</u>](https://docs.google.com/document/d/1Y_h5sjUTJzdK1riRh-mrVQm9KCzqFlU65KMsMsrQgXk/)）。

与记账相关的问题是，当您出售该头寸时，使用哪个成本？库存将包含 500 USD 的头寸，因此应该使用这个成本，但对用户来说是否明显？假设可以通过某种方法解决。

如果我们自动附加交易日期，是否会在分割时重置，现在需要使用 2014-04-17？如果是这样，我们无法自动检查交易列表以确定这是长期交易还是短期交易。我们需要以某种方式保留最初增加该头寸的事件的一些属性，包括原始交易日期（而不是分割日期）和用户指定的原始标签（如果有）。

### 强制每个账户单一方法<a id="forcing-a-single-method-per-account"></a>

对于使用平均记账方法的账户，允许指定账户只能使用此记账方法可能很有用。目的是避免在我们知道账户只能使用此方法时输入数据错误。

一种确保这一点的方法是在增加头寸时自动聚合库存批次。这确保在任何时间点，每种商品只有一个批次。如果我们将每个账户的库存记账类型关联起来，可以定义一个特殊类型，在增加头寸时触发聚合。

另一种方法是不强制这些聚合，但提供一个插件，检查那些默认使用平均成本库存记账方法的账户，确保只发生这种记账。

## 以前的解决方案<a id="previous-solutions"></a>

本节回顾现有命令行会计系统中的记账方法实现，并突出特定缺点，说明为什么我们需要定义一种新的方法。

### Ledger 的缺点<a id="shortcomings-in-ledger"></a>

*（请注意：我不太了解 Ledger 内部工作的细节；我从构建测试示例和阅读文档中推断出其行为。如果我说错了，请通过留言告知。）*

Ledger 的记账方法非常自由。在内部，[<u>Ledger 不区分按成本持有的转换和常规价格转换</u>](http://www.google.com/url?q=http%3A%2F%2Fledger-cli.org%2F3.0%2Fdoc%2Fledger3.html%23Prices-versus-costs&sa=D&sntz=1&usg=AFQjCNGAZPEtX3TjQysbqsDnF48d5a29Hw)：所有转换都假定按成本持有，交易批次只在报告时出现（使用其 `--lots` 选项）。

其“`{}`”成本语法[<u>用于在减少头寸时消歧批次</u>](http://ledger-cli.org/3.0/doc/ledger3.html#Explicit-posting-costs)，而不是在增加头寸时使用。增加头寸的成本使用“`@`”价格表示法指定：

```plaintext
2014/05/01 * Buy some stock (Ledger syntax)
  Assets:Investments:Stock        10 HOOL @ 500 USD
  Assets:Investments:Cash
```

这将导致 10 HOOL {500 USD} 的库存，即成本始终存储在持有头寸的库存中：

```plaintext
$ ledger -f l1.lgr bal --lots
10 HOOL {USD500} [2014/05/01]
            USD-5000  Assets:Investments
            USD-5000    Cash
10 HOOL {USD500} [2014/05/01]    Stock
--------------------
10 HOOL {USD500} [2014/05/01]
            USD-5000
```

在转换持有成本和价格转换之间没有区别——所有转换都被视为“按成本持有”。在下一节中，我们将看到，这种行为与 Beancount 的语义不同，后者要求通过“`{}`”表示法来指定按成本持有的转换，并在增加和减少过账时都进行区分。

Ledger 方法的优点是输入机制更简单，但如果您在同一账户中累积多种货币的多次转换，结果可能会令人困惑。库存在其批次组成上容易变得支离破碎。例如，如果您在 5 种不同货币之间转换，可能会在 CAD、JPY、EUR 和 GBP 中持有 USD，并且可能会在这些货币之间相互转换。以下货币转换将保持其相对于转换货币的原始成本：

```plaintext
2014/05/01 Transfer from Canada
  Assets:CA:Checking             -500 CAD
  Assets:US:Checking              400 USD @ 1.25 CAD

2014/05/15 Transfers from Europe
  Assets:UK:Checking             -200 GBP
  Assets:US:Checking              500 USD @ 0.4 GBP

2014/05/21 Transfers from Europe
  Assets:DE:Checking             -100 EUR
  Assets:US:Checking              133.33 USD @ 0.75 EUR
```

这导致 `Assets:US:Checking` 账户中的库存如下：

```plaintext
$ ledger -f l2.lgr bal --lots US:Checking
500.00 USD {0.4 GBP} [2014/05/15]
133.33 USD {0.75 EUR} [2014/05/21]
400.00 USD {1.25 CAD} [2014/05/01]  Assets:US:Checking
```

货币被视为股票。但[<u>没有人会这样看待他们的货币余额</u>](https://groups.google.com/d/msg/ledger-cli/zRSMle5AV3Q/kBCYjVEB-gsJ)，人们通常会将账户中的货币余额视为单位本身，而不是相对于其他货币的价格。我认为这令人困惑。转换后，我只认为该账户的余额为 1033.33 USD。

对这种观察的可能反驳是，大多数时候用户不在意仅包含货币的账户的库存批次，因此他们不会打印出来。不见即忘。但没有办法区分何时渲染例程应该渲染批次。报告时，余额例程应仅为某些账户（例如，投资账户）渲染批次（其中单位成本很重要），而不是其他账户（例如，由于转换而存入的货币账户）。此外，当我渲染资产负债表时，对于按成本持有的单位，我们需要报告账面价值，而不是股票数量。但对于货币，总是需要报告货币单位。如果不区分这两种情况，如何知道应该渲染哪种视图？我认为答案是使用选项选择要渲染的视图。问题是没有一个选项能为两种商品提供单一的统一视图。用户不应需要指定选项来查看部分不正确的视图，应该自动根据我们是否在意这些单位的成本来确定。如果区分这两种转换类型，可以正确处理这一点。

但或许最重要的是，这种方法并未特别解决*转换问题*：通过以不同的汇率来回转换仍然可能创造出凭空的钱：

```plaintext
2014/05/01 Transfer to Canada
  Assets:US:Checking             -40000 USD
  Assets:CA:Checking              50000 CAD @ 0.80 USD

2014/05/15 Transfer from Canada
  Assets:CA:Checking             -50000 CAD @ 0.85 USD
  Assets:US:Checking              42500 USD
```

这导致试算平衡只有 2500 USD：

```plaintext
$ ledger -f l3.lgr bal 
            2500 USD  Assets:US:Checking
```

这是不理想的结果。我们应该要求试算平衡始终为零（虚拟交易除外）。毕竟，这是双重记账方法优雅性的集中体现：因为所有交易的总和为零，任何交易子集的总和也应为零……除非不是。在我看来，任何渲染的资产负债表都应符合会计等式，精确无误。

在与 Ledger 的探索中，我希望通过其库存跟踪方法，它能自动处理这些转换，从而提供跟踪所有单位成本的合理依据，但我发现同样的问题出现。转换问题在 Beancount 中也是一个棘手的问题，但它已得到解决，我认为可以将同样的解决方案应用于 Ledger，以调整这些总和，而不改变其当前的记账方法：在战略点自动插入一个特殊的转换条目——当渲染资产负债表时。在最终分析中，我没有发现跟踪每个交易的成本比忘记价格转换的成本更有优势。希望能听到更多支持这种设计选择的意见。

因为所有转换都保持成本，Ledger 需要在记账上非常自由，因为强迫用户在常见的货币转换情况下指定每种货币的原始汇率是不方便的。这对我们确实希望按成本持有的转换有不利影响：允许任意批次减少，即允许对不存在的批次进行减少。例如，以下交易在 Ledger 中不会触发错误：

```plaintext
2014/05/01 Enter a position
  Assets:Investments:Stock        10 HOOL @ 500 USD
  Assets:Investments:Cash

2014/05/15 Sale of an impossible lot
  Assets:Investments:Stock       -10 HOOL {505 USD} @ 510 USD
  Assets:Investments:Cash
```

这导致库存中有一个 10 HOOL 的长头寸和一个 10 HOOL 的短头寸（成本不同）：

```plaintext
$ ledger -f l4.lgr bal --lots stock
10 HOOL {USD500} [2014/05/01]
   -10 HOOL {USD505}  Assets:Investments:Stock
```

我认为不应允许减少 505 USD 的 HOOL 头寸；Beancount 选择将其视为错误。

Ledger 明显支持对现有库存记账，因此其意图应该是能够对现有头寸减少。例如，以下交易确实导致空库存：

```plaintext
2014/05/01 Enter a position
  Assets:Investments:Stock      10 HOOL @ 500 USD
  Assets:Investments:Cash

2014/05/15 Sale of matching lot
  Assets:Investments:Stock     -10 HOOL {500 USD} [2014/05/01]  @ 510 USD
  Assets:Investments:Cash
```

输出如下：

```plaintext
$ ledger -f l5.lgr bal --lots 
              USD100  Equity:Capital Gains
```

如您所见，HOOL 头寸已消除。（不要在意输出中自动插入的权益条目，我相信这将在[<u>待定补丁中修复</u>](http://bugs.ledger-cli.org/show_bug.cgi?id=712)）。然而，必须同时指定日期和成本才能使其工作。以下交易结果与前面提到的长+短头寸库存类似的问题：

```plaintext
2014/05/01 Enter a position
  Assets:Investments:Stock      10 HOOL @ 500 USD
  Assets:Investments:Cash

2014/05/15 Sale of matching lot?
  Assets:Investments:Stock     -10 HOOL {500 USD} @ 510 USD
  Assets:Investments:Cash
```

输出如下：

```plaintext
$ ledger -f l6.lgr bal --lots 
10 HOOL {USD500} [2014/05/01]
   -10 HOOL {USD500}  Assets:Investments:Stock
              USD100  Equity:Capital Gains
--------------------
10 HOOL {USD500} [2014/05/01]
   -10 HOOL {USD500}
              USD100
```

这有点令人惊讶，我本以为批次会相互记账。我怀疑这可能是一个未报告的错误，并非预期行为。

最后，“`{}`”成本语法也可以用于增加腿。文档[<u>指出这些方法是等效的</u>](http://ledger-cli.org/3.0/doc/ledger3.html#Prices-versus-costs)。这导致没有关联日期的库存批次，但另一个腿不会转换为成本：

```plaintext
2014/05/01 Enter a position
  Assets:Investments:Stock      10 HOOL {500 USD}
  Assets:Investments:Cash
```

输出如下：

```plaintext
$ ledger -f l7.lgr bal --lots
                   0  Assets:Investments
   -10 HOOL {USD500}    Cash
    10 HOOL {USD500}    Stock
--------------------
                   0
```

我可能不理解这应该如何使用；在 Beancount 中，自动计算的腿会将 -5000 USD 过账到现金账户，而不是股票。

我认为，Ledger（可能）在尝试使减少相匹配之前累积所有批次，然后在报告时（仅在那时）根据某些报告选项匹配批次：

- 通过成本/价格和日期分组批次，使用 --lots

- 仅通过成本/价格分组批次，使用 --lot-prices

- 仅通过日期分组批次，使用 --lot-dates

文档中不太明显，但行为与此一致。

（如果这是正确的，我认为这是其设计的问题：匹配减少到现有批次的机制不应该是报告功能。应该在处理报告之前进行，以便实现库存记账的单一和确定历史。如果支持不同的记账方法——我不认为这是个好主意——应该在报告阶段之前支持。在我看来，通过命令行选项改变库存记账策略是个坏主意，策略应该由语言本身完全定义。）

### Beancount 的缺点<a id="shortcomings-in-beancount"></a>

Beancount 也有各种缺点，但不同的是。

与 Ledger 相比，Beancount [<u>区分</u>](https://groups.google.com/d/msg/ledger-cli/zRSMle5AV3Q/kBCYjVEB-gsJ) 货币转换和“按成本持有”的转换：

```plaintext
2014-05-01 * "Convert some Loonies to Franklins"
  Assets:CA:Checking          -6000 CAD
  Assets:Investments:Cash      5000 USD @ 1.2 CAD   ; Currency conversion

2014-05-01 * "Buy some stock"
  Assets:Investments:Stock        10 HOOL {500 USD} ; Held at cost
  Assets:Investments:Cash
```

在第一笔交易中，`Assets:Investment:Cash` 账户结果为 5000 USD 的库存，没有关联的成本。然而，第二笔交易后，`Assets:Investment:Stock` 账户结果为 10 HOOL 的库存，关联成本为 500 USD。这可能有点复杂，需要用户更多知识：有两种转换，用户必须理解并区分这两种情况，对于新手来说并不明显。需要对用户进行一些教育（我承认如果我写更多文档会有帮助）。

Beancount 方法也不那么自由，要求严格的批次减少。即，如果输入一笔减少不存在批次的交易，它将输出错误。这样做的目的是让用户在数据输入中不容易犯错。任何减少库存中的头寸必须与一个批次匹配。

这种选择带来了一些缺点：

- 需要用户始终找到匹配批次的成本。这意味着自动导入交易需要用户手动干预，因为他必须在分类账中查找以插入匹配批次。（理论上导入代码可以加载分类账内容以列出其持有，并且如果不含糊，可以自行查找成本并插入输出。但这使导入代码依赖于用户的分类账，大多数导入代码不会这样做）。这是一个重要步骤，因为找到正确的成本对于正确计算资本收益是必要的，但需要更灵活的方法，让用户可以懒一点，不必在选择明显时输入现有头寸的成本，例如，当只有一个批次可匹配时。我希望稍微放松这一方法。

- 严格要求对现有批次减少也意味着同一账户中不能同时存在正负同一商品的“按成本持有”头寸。然而，这在实践中不是大问题，因为短头寸非常罕见——很少有人进行——用户总是可以创建单独账户持有短头寸。如果需要，通常为每种商品创建专门的子账户，这自然地组织了交易活动（否则报告有点混乱，看到按股票分组的总头寸价值很好，因为它们提供了不同市场特征的曝光）。不同账户应效果良好，只要我们正确处理减少中的符号，即购买股票抵消现有短头寸视为减少头寸（符号反转）。

Beancount 记账方法的另一个问题是如何匹配带日期的批次。例如，如果尝试区分相同价格的两个批次，最明显的输入将不起作用：

```plaintext
2012-05-01 * "First trade"
  Assets:Investments:Stock        10 HOOL {500 USD}
  Assets:Investments:Cash

2014-02-15 * "Second trade at very same price"
  Assets:Investments:Stock         8 HOOL {500 USD}
  Assets:Investments:Cash

2014-06-30 * "Trying to sell"
  Assets:Investments:Stock        -5 HOOL {500 USD / 2012-05-01}
  Assets:Investments:Cash
```

这将报告记账错误。这令人困惑。在这种情况下，Beancount 出于严格要求，严格匹配库存批次，尝试匹配 (HOOL, 500 USD, 2012-05-01) 与 (HOOL, 500 USD, *None*) 失败。注意，批次日期语法在 Beancount 的增加和减少腿上都接受，因此“解决方案”是用户必须在增加头寸时专门提供批次日期。例如，以下方式有效：

```plaintext
2012-05-01 * "First trade"
  Assets:Investments:Stock        10 HOOL {500 USD / 2012-05-01}
  Assets:Investments:Cash

2014-02-15 * "Second trade at very same price"
  Assets:Investments:Stock         8 HOOL {500 USD}
  Assets:Investments:Cash

2014-06-30 * "Trying to sell"
  Assets:Investments:Stock        -5 HOOL {500 USD / 2012-05-01}
  Assets:Investments:Cash
```

问题在于用户输入第一笔交易时无法预测未来会以相同价格进行另一笔交易，通常不会插入批次日期。更可能的是，用户必须回过头去重新访问该交易以进行消歧。我们可以做得更好。

这是其实现的缺点，反映了头寸减少最初被视为精确匹配问题，而不是模糊匹配问题——当时我不确定如何安全处理。这需要在本文提案后修复，我正在通过本文文件理清我对这种模糊匹配的需求，并确保提出一个明确的设计，便于数据输入。Ledger 总是自动将交易日期附加到库存批次上，我认为这是正确的做法（下面我们建议附加更多信息）。

## 背景<a id="context"></a>

*\[2014 年 11 月更新\]*

区分两种记账类型很重要：

1. **记账。** 当我们考虑整个交易列表时的严格记账过程。我们称之为“记账”过程，只应执行一次。此过程应严格：无法正确记账减少到现有批次时应触发致命错误。

2. **库存聚合。** 当我们考虑条目子集时，可能会移除增加某些批次的条目，并保留减少或移除这些批次的其他条目。这需要支持随时间对库存变化进行聚合。在支持任意交易子集聚合时，我们必须考虑这一点。聚合过程不应导致错误。

让我们以一个示例来证明为什么需要非记账汇总。假设我们有一笔股票买入和卖出：

```plaintext
2013-11-03 * "Buy"
  Assets:Investments:VEA      10 AAPL {37.45 USD} ;; (A)
  Assets:Investments:Cash     -370.45 USD

2014-08-09 * "Sell"
  Assets:Investments:VEA     -5 AAPL {37.45 USD}
  Assets:Investments:Cash     194.40 USD
  Income:Investments:PnL
```

如果我们按年筛选，只查看 2014 年发生的交易，并且不“关闭”前一年，即不创建期初余额条目在年初存入批次——这可能发生在按标签筛选或其他条件组合筛选时——则批次 (A) 不存在于库存中以进行扣除。我们必须以某种方式支持 -5 AAPL 的情况。报告时，用户可以决定如何汇总这些腿，例如按单位或成本汇总，或按平均成本转换为单个批次。

然而，记账过程不应有这种宽容。应严格触发错误，如果无法匹配批次。仅在交易总集上执行一次，永不在任何子集上执行。记账阶段的目的有三：

1. **匹配**现有批次，并在无法匹配时发出错误。使用用户指定的方法进行此操作。

2. **替换**所有部分指定的批次为完全指定的批次，即记账过程中匹配的批次。

3. **插入链接**在形成交易的交易上：买入和相应的卖出应链接在一起。这基本上标识了交易，之后可以用于报告。（平均成本交易需要特别考虑。）

我们可以将 Ledger 方法视为仅具有汇总方法，缺少记账阶段。选择汇总方法使用 --lots 或 --lot-dates 或 --lot-prices。这表明可以在不改变其总体结构的情况下在 Ledger 上附加一个“记账”阶段：可以作为预处理或后处理插入，但这需要 Ledger 按日期排序交易，当前它不这样做（它对数据执行一次遍历）。

Beancount 已有这两种方法，但尚未明确指出它们是独立操作——而是库存支持在无法记账时可选标志触发错误。此外，库存汇总未考虑太多自定义，旨在与记账过程统一。现在很明显，它们是两个独立算法，几种汇总方法可能相关（尽管它们不与 Ledger 的方法相同。注意：可以将库存汇总方法视为库存对象批次上的 GROUP BY：分组列是什么？这需要用例）。无论如何，完全指定的批次应默认相互匹配：我们需要支持这种简单货币的退化情况（不按成本持有）……我们不希望在库存中累积所有货币变化为单独批次，它们需要在应用时立即交叉。
