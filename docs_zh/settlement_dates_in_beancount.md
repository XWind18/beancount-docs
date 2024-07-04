# Beancount 中的结算日期与转账账户

作者：Martin Blais，2014年7月

[原文链接](http://furius.ca/beancount/doc/proposal-dates)

## 动机

当投资账户中的交易执行时，通常在交易执行日期（“交易日期”）和资金存入相关现金账户的日期（“结算日期”）之间存在延迟。这使得导入的余额断言有时需要调整其日期，有时甚至可能变得不可能。本文档提议在交易或分录中添加一个可选的“结算日期”，并介绍如何处理这一问题的相关语义。

## 提案描述

### 结算日期

在 Beancount 的最初实现中，我曾为交易附加了两个日期，但从未对此进行处理。备用日期被附加后就被忽略了。其含义是，它应该将交易分成两部分，带有某种转账账户，这可能有用，但我从未开发出来。

如下输入：

```plaintext
2014-06-23=2014-06-28 * "Sale"
   Assets:ScottTrade:GOOG    -10 GOOG {589.00 USD}
   S Assets:ScottTrade:Cash
```

其中，“S”分录标记表示“推迟到结算日期”。

或者，您可以将日期附加到分录：

```plaintext
2014-06-23 * "Sale"
   Assets:ScottTrade:GOOG    -10 GOOG {589.00 USD}
   2014-06-28 Assets:ScottTrade:Cash
```

以上两种语法提议都允许您指定哪些分录要推迟到结算。第二种方式更灵活，因为每个分录可能有不同的日期，但第一种方式的语法更简洁，产生的复杂性较少。

这两种方式都可以转换为多个交易，通过转账账户吸收待结算金额：

```plaintext
2014-06-23 * "Sale" ^settlement-3463873948
   Assets:ScottTrade:GOOG    -10 GOOG {589.00 USD}
   Assets:ScottTrade:Transfer

2014-06-28 * "Sale" ^settlement-3463873948
   Assets:ScottTrade:Transfer
   Assets:ScottTrade:Cash        5890.00 USD
```

到目前为止，我一直通过调整余额断言的日期来应对这种情况。我处理的销售量相对较少，所以这还不是一个大问题。我还不确定是否需要解决这个问题，也许最好的办法是记录如何在发生这种情况时处理问题。

也许有人可以说服我。

### 转账账户

在前一部分中，我们讨论了一种将资金在两个账户之间移动的单个分录包含两个日期并生成两个独立分录的风格。一个相关的辅助问题是如何执行反向操作，即如何*合并*两个分别记入公共转账账户（有时称为“[挂账账户](https://en.wikipedia.org/wiki/Suspense_account)”）的独立分录。

例如，用户可能希望分别输入交易的两边，例如通过在单独的输入文件上运行导入脚本，而不是手动对账和合并它们，我们希望通过识别这些转账账户中的匹配交易并创建它们之间的公共链接来明确支持这一点。

最重要的是，我们希望能够轻松识别哪些交易在另一边没有匹配，这表明数据缺失。

- 这个功能的一个原型在 [beancount.plugins.tag_pending](https://github.com/beancount/beancount/tree/v2/beancount/plugins/tag_pending.py) 中。
- 另见 redstreet0 的 “[zerosum](https://groups.google.com/d/msgid/beancount/8adbb83d-a7c7-476a-97ca-d600d110db20%40googlegroups.com?utm_medium=email&utm_source=footer)” 插件，来自 [这个帖子](https://groups.google.com/d/msgid/beancount/8adbb83d-a7c7-476a-97ca-d600d110db20%40googlegroups.com?utm_medium=email&utm_source=footer)。

## 仍待解决的问题

*我们如何确定适当的转账账户名称？子账户方法是否合理？如果用户希望有一个全局的挂账账户怎么办？*

TODO

*这个特性是否解决了在交易和结算之间进行余额断言的问题？*

TODO [写一个详细的例子]

*有任何缺点吗？*

TODO

*这对资产负债表和损益表有何影响，如果有的话？用户是否能明显看出这些挂账/转账账户中的金额？*

TODO

## 取消交易固定

一个更为大胆的想法是为交易-分录层次结构添加一个额外的层次，增加将多个部分交易分组的能力，并将平衡规则移动到该层次。基本上，两个分别输入的交易然后分组——通过某些规则，或者简单地自身分组——可以形成一个新的平衡规则单元。

这将对架构和 Beancount 设计提出更高的要求，但允许本地支持部分交易，保留它们的独立日期、描述等。也许这是一个更好的模型？考虑其优势。

## 以往工作

### Ledger 有效和辅助日期

Ledger 有“[辅助日期](http://ledger-cli.org/3.0/doc/ledger3.html#Auxiliary-dates)”的概念。这些工作的方式很简单：任何交易都可以有第二个日期，用户可以在运行时选择（使用 `--aux-date`）是使用主日期还是辅助日期。

我不清楚在存在余额断言的情况下如何实际使用这一点。如果没有余额断言，我能看到它的工作原理：你只需渲染所有结算日期。这可能只对特定报告有意义。

我更愿意保持一组解析交易的单一语义；交易的含义根据调用条件变化的想法在 Beancount 中将是一个先例，我更愿意避免实现这个解决方案。

辅助日期也称为“[有效日期](http://ledger-cli.org/3.0/doc/ledger3.html#Effective-Dates)”，可以与每个独立的分录相关联。辅助日期是次于“主日期”或“实际日期”的（即记录的记账日期）：

```plaintext
2008/10/16 * (2090) Bountiful Blessings Farm
    Expenses:Food:Groceries                  $ 37.50  ; [=2008/10/01]
    Expenses:Food:Groceries                  $ 37.50  ; [=2008/11/01]
    Expenses:Food:Groceries                  $ 37.50  ; [=2008/12/01]
    Expenses:Food:Groceries                  $ 37.50  ; [=2009/01/01]
    Expenses:Food:Groceries                  $ 37.50  ; [=2009/02/01]
    Expenses:Food:Groceries                  $ 37.50  ; [=2009/03/01]
    Assets:Checking
```

最初的动机是为了预算，允许将费用的记账移到邻近的预算期，以便将实际支付的金额结转到那些期间。在需要时，可以将一个月的银行金额与紧邻的前一个月或下一个月的预算进行比较。（备注：John Wiegley）

这类似于我上面建议的语法之一——让用户为每个分录指定一个日期——但其他分录不会分裂为独立交易。这些日期的使用类似地通过命令行选项（`--effective`）触发。我假设上面的支票账户分录在 2008/10/16 同时发生，而不考虑报告日期。让我们验证一下：

```bash
$ ledger -f settlement1.lgr reg checking --effective
08-Oct-16 Bountiful Blessings.. Assets:Checking           $ -225.00    $ -225.00
```

这正是我所想的。这行得通，但这种方法的问题是，在 2008/10/01（最早的有效日期）和 2009/03/01（最晚的有效日期）之间绘制的任何资产负债表都不会平衡。在这些日期之间，一些金额处于“悬空”状态，在这些日期之一绘制的资产负债表不会平衡。

这将打破 Beancount 中的不变性：我们要求你应该始终能够在任何时间点绘制资产负债表，任何交易子集应该平衡。我宁愿通过将这个示例交易拆分为多个交易来实现这一点，如上面的提案，将这些暂时处于悬空状态的金额移到一个显式的“悬空”或“转账”账户，每个交易都平衡。此外，这一步可以作为一个转换阶段实现，通过一个插件按需启用，替换具有有效日期的交易，变成多个具有不同于交易日期的有效日期的分录。

### GnuCash

TODO（blais）- GnuCash 和其他 GUI 系统如何处理这个问题？有标准账户方法吗？

## 参考文献

美国国税局 [要求你使用交易日期而非结算日期](http://www.irs.gov/publications/p17/ch14.html) 进行税务报告；摘自 IRS 出版物 17：

> **在已建立的市场上交易的证券。** 对于在已建立的证券市场上交易的证券，你的持有期从你购买证券的交易日之后开始，到你出售证券的交易日结束。
>
> 不要将交易日期与结算日期混淆，结算日期是必须交割股票和付款的日期。
>
> **示例。**
>
> 你是现金制的日历年纳税人。你在 2013 年 12 月 30 日以盈利卖出股票。根据证券交易所的规则，交易在交易日后 4 个交易日（2014 年 1 月 6 日）完成交割，你在同一天收到销售价款。尽管你在 2014 年收到付款，你仍应在 2013 年申报你的收益。收益是长期还是短期取决于你持有股票的时间是否超过一年。你的持有期在 12 月 30 日结束。如果你以亏损卖出股票，你也应在 2013 年申报。

### 讨论串

- [一个有趣的“巧合功能”](https://groups.google.com/d/msg/ledger-cli/ooxbPVRinSs/ymkRCerhxjcJ)
- [来自 Ledger 的第一印象](https://groups.google.com/d/msg/beancount/z9sPboW4U3c/UfJbIVzwmpMJ)
