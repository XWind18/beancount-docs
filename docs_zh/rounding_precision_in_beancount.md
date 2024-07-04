# Beancount 中的舍入与精度提案<a id="title"></a>

[<u>Martin Blais</u>](http://plus.google.com/+MartinBlais)，2014年10月

*本文档描述了 Beancount 交易中的舍入误差问题以及它们的处理方法。还包含了更好处理 Beancount 中精度问题的提案。*

## 动机<a id="motivation"></a>

### 平衡精度<a id="balancing-precision"></a>

交易的平衡不能精确完成。这一点在 [<u>Ledger 邮件列表</u>](https://groups.google.com/d/msg/ledger-cli/m-TgILbfrwA/YjkmOM3LHXIJ) 中已经讨论过。必须允许用于平衡交易的金额有一定的容差。

当你考虑在文本文件中输入数字意味着有限的小数表示时，这一需求就很明显。例如，如果你要将单位数量和成本相乘，比如都写有 2 位小数，你可能会得到一个有 4 位小数的数字，然后你需要将该结果与通常只输入 2 位小数的现金金额进行比较，比如这个例子：

    2014-05-06 * "购买基金"
      Assets:Investments:RGXGX       4.27 RGAGX {53.21 USD} 
      Assets:Investments:Cash     -227.21 USD

如果你计算一下，第一个分录的精确余额金额是 227.2067 USD，而不是 227.21 USD。然而，管理投资账户的经纪公司会将现金取款金额四舍五入到最接近的分位，并且使用的是四舍五入后的正确金额。这个交易必须平衡；我们需要某种方式允许一些松散。

大多数涉及数学运算的情况都涉及将单位数量和价格或成本转换为相应的现金价值（例如，单位 x 成本 = 总成本）。我们在表示交易信息时的任务是重现大多数机构中的操作。这些操作总是涉及单位和货币的舍入（银行确实应用随机舍入），从这些机构的角度来看，正确的数字确实是舍入后的数字。这个问题不是数学上的纯粹性问题，而是实际问题，我们的系统应该与银行做的相同。因此，我认为我们应该始终将舍入后的数字发布到账户中。使用有理数在这方面并不是限制，但我们必须小心在需要的地方存储舍入后的数字。

### 自动舍入<a id="automatic-rounding"></a>

另一个相关问题是自动舍入插值数字的问题。让我们再看一个原始的例子：

    2014-05-06 * "购买基金"
      Assets:Investments:RGXGX       4.27 RGAGX {53.21 USD} 
      Assets:Investments:Cash     ;; 插值分录 = -227.2067 USD

这里第二个分录的金额是从第一个分录的余额金额插值出来的。理想情况下，我们应该找到一种方法将其舍入到 2 位小数的精度。

请注意，这也会影响插值价格和成本：

    2014-05-06 * "购买基金"
      Assets:Investments:RGXGX       4.27 RGAGX {USD} 
      Assets:Investments:Cash     -227.21 USD

在这里，成本意图是从现金金额中自动计算的：227.21 / 4.27 = 53.2107728337… USD。推断的正确成本也是舍入后的 53.21 USD。我们希望有一种机制允许我们推断所需的精度。

不幸的是，这种机制不能仅基于商品：不同的账户可能以不同的精度追踪货币。作为一个真实的例子，我有一个零售外汇交易账户，实际上使用 4 位精度来表示其价格和存款。

### 余额断言的精度<a id="precision-of-balance-assertions"></a>

余额断言的精度也受到这个问题的影响，例如：

    2014-04-01 balance Assets:Investments:Cash   4526.77 USD

用户并不打算让这个余额检查精确到 4526.77000000… USD。然而，如果这个现金账户之前收到了一个具有更高精度的存款，例如前一节的例子，那么我们就有问题了。现在现金金额包含了一些插值存款的碎片（0.0067 USD）。如果我们能够找到一个好的解决方案来自动舍入上一节中的分录，这就不会成为问题。但在此之前，我们必须找到一个解决方案。

Beancount 目前的方法是一个折中方案：它使用 [<u>用户可配置的容差</u>](http://github.com/beancount/beancount/tree/master/beancount/ops/balance.py#L23) 为 0.0150（以任何单位表示）。我们希望改变这一点，使得使用的容差能够依赖于商品、账户，甚至是使用的特定指令。

## 其他系统<a id="other-systems"></a>

其他命令行记账系统在选择容差方面有所不同：

- Ledger 尝试通过使用最近解析的上下文（按文件顺序）自动推导用于其余额检查的精度。使用的精度是最后为特定商品解析的值的精度。这可能会导致问题：它可能导致 [<u>难以调试的交易之间不必要的副作用</u>](https://groups.google.com/d/msg/ledger-cli/m-TgILbfrwA/cTHg2juqEJgJ)。

- HLedger 则使用全局精度设置。 [<u>整个文件首先被处理，然后从整个输入文件中看到的最精确的数字中推导出精度。</u>](https://groups.google.com/d/msg/ledger-cli/m-TgILbfrwA/SoGZDNhlDOkJ)

- 目前，Beancount 在其 [<u>余额检查算法</u>](http://github.com/beancount/beancount/tree/master/beancount/ops/validation.py) 中使用一个常数值作为容差（任何单位的 0.005）。这很薄弱，至少应该是商品相关的，如果不是账户中特定商品的容差。

## 提案<a id="proposal"></a>

### 自动推断容差<a id="automatically-inferring-tolerance"></a>

Beancount 应该使用一种完全**局部**于每个交易的方法推导其精度，可能还需要一个全局默认值。也就是说，对于每个交易，它将检查具有简单金额的分录（没有成本，没有价格），并推断容差的精度为用户在此交易中输入的最精确金额的一半。例如：

    2014-05-06 * "购买基金"
      Assets:Investments:RGXGX       4.278  RGAGX {53.21 USD} 
      Assets:Investments:Cash     -227.6324 USD
      Expenses:Commissions           9.95   USD

这里使用的精度位数是第二和第三分录的最大值，即 max(4, 2) = 4。第一个分录被忽略，因为其金额是数学运算的结果。容差值应该是最精确数字的一半，即 0.00005 USD。这应该允许用户通过在输入中插入更多的数字来使用任意精度。

只有小数位将用于推导精度… 整数应意味着精确匹配。用户可以指定一个单独的尾随句号来表示次美元精度。例如，以下交易应因计算金额为 999.999455 USD 而无法平衡：

    2014-05-06 * "购买基金"
      Assets:Investments:RGXGX       23.45 RGAGX {42.6439 USD} 
      Assets:Investments:Cash        -1000 USD

相反，用户应该明确允许使用一些容差：

    2014-05-06 * "购买基金"
      Assets:Investments:RGXGX       23.45 RGAGX {42.6439 USD} 
      Assets:Investments:Cash       -1000. USD

或者更好地，使用 1000.00 USD。这有一个缺点，即阻止用户指定更简单的整数金额。我不确定这是否是个大问题。

最后，不会对交易应用任何全局效应。任何交易都不应该影响其他交易的平衡上下文。

### 对成本持有金额的推断<a id="inference-on-amounts-held-at-cost"></a>

Matthew Harris 提出的一个想法是我们还可以使用单位数量乘以成本的小数最小值来建立平衡交易的容差值。例如，在以下交易中：

    2014-05-06 * "购买基金"
      Assets:Investments:RGXGX       23.45 RGAGX {42.6439 USD} 
      ...

可以推导出的容差值是：

0.01 RGAGX x 42.6439 USD = **0.426439 USD**

Matthew 提出的原始用例是一个不包含简单金额的交易，只包含以成本持有的转换：

      2011-01-25 * "资产转移，3467.90 USD"
        * Assets:RothIRA:Vanguard:VTIVX  250.752 VTIVX {18.35 USD} @ 13.83 USD  
        * Assets:RothIRA:DodgeCox:DODGX  -30.892 DODGX {148.93 USD} @ 112.26 USD

我实际上尝试实现了这个方法，结果要么是容差过大，要么是容差过小。在实践中效果不好，所以我放弃了这个想法。

### 自动舍入<a id="automated-rounding"></a>

对于自动计算的值，例如在自动分录中，剩余值是自动推导的，我们应该考虑舍入这些值。尚未对此进行任何工作；这些值目前未舍入。

### 修复余额断言<a id="fixing-balance-assertions"></a>

要修复余额断言，我们将通过查看余额金额中使用的小数位数，使用最精确的小数位数，并使用该位数值的一半来计算容差：

    2014-04-01 balance Assets:Investments:Cash   4526.7702 USD

这个余额检查意味着 0.00005 USD 的精度。

如果你使用整数数量的单位，不允许有任何容差。精确数量应匹配：

    2014-04-01 balance Assets:Investments:Cash   4526 USD

如果你想允许次美元差异，请使用单个逗号：

    2014-04-01 balance Assets:Investments:Cash   4526. USD

这个余额检查意味着 0.50 USD 的精度。

### 近似断言<a id="approximate-assertions"></a>

另一个想法是，在 [<u>Ledger 的这个票据中</u>](https://github.com/ledger/ledger/pull/329)，提出了一个明确的近似断言。

我们可以这样实现它（只是一个想法）：

    2014-04-01 balance Assets:Investments:Cash   4526.00 +/- 0.05 USD 

## 累积与报告剩余误差<a id="accumulating-reporting-residuals"></a>

为了明确呈现和监控分类账中发生的舍入误差的数量，我们应该将其 [<u>累积到一个权益账户</u>](https://groups.google.com/d/msg/ledger-cli/m-TgILbfrwA/YjkmOM3LHXIJ)，例如“Equity:Rounding”。这应该可以选择启用。应该允许用户指定一个账户来累积误差。每当一个交易未能完全平衡时，剩余或舍入误差将作为交易的一个分录插入到权益账户中。

默认情况下，这个累积功能应该关闭。目前还不清楚这些额外的分录是否会产生干扰（如果没有干扰，也许应该默认开启；实践会告诉我们）。

## 实现<a id="implementation"></a>

本文档的实现记录在 [<u>此处</u>](precision_tolerances.md)。
