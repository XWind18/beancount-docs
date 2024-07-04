# 命令行会计烹饪书<a id="title"></a>

[<u>Martin Blais</u>](mailto:blais@furius.ca)，2014年7月

[<u>http://furius.ca/beancount/doc/cookbook</u>](http://furius.ca/beancount/doc/cookbook)

> [<u>介绍</u>](#introduction)
>
> [<u>注意事项</u>](#a-note-of-caution)
>
> [<u>账户命名约定</u>](#account-naming-conventions)
>
> [<u>选择账户类型</u>](#choosing-an-account-type)
>
> [<u>选择开户日期</u>](#choosing-opening-dates)
>
> [<u>如何处理现金</u>](#how-to-deal-with-cash)
>
> [<u>现金取款</u>](#cash-withdrawals)
>
> [<u>跟踪现金支出</u>](#tracking-cash-expenses)
>
> [<u>工资收入</u>](#salary-income)
>
> [<u>就业收入账户</u>](#employment-income-accounts)
>
> [<u>预订工资存款</u>](#booking-salary-deposits)
>
> [<u>假期小时</u>](#vacation-hours)
>
> [<u>401k 贡献</u>](#k-contributions)
>
> [<u>股票授予的归属</u>](#vesting-stock-grants)
>
> [<u>其他福利</u>](#other-benefits)
>
> [<u>积分</u>](#points)
>
> [<u>食物福利</u>](#food-benefits)
>
> [<u>货币转移和转换</u>](#currency-transfers-conversions)
>
> [<u>投资和交易</u>](#investing-and-trading)
>
> [<u>账户设置</u>](#accounts-setup)
>
> [<u>资金转移</u>](#funds-transfers)
>
> [<u>进行交易</u>](#making-a-trade)
>
> [<u>收到股息</u>](#receiving-dividends)
>
> [<u>结论</u>](#conclusion)

## 介绍<a id="introduction"></a>

学习复式记账法的最佳方式是查看实际案例。这种方法很优雅，但对于新手来说，如何通过不同的操作来记录各种财务事件的交易可能显得不太直观。这就是我写这本烹饪书的原因。它并不打算全面描述所有支持的功能，而是提供一套实用的指南，帮助你解决问题。我认为这可能是 Beancount 文档集中最有用的文档！

这里的所有例子适用于任何复式会计系统：Ledger、GnuCash，甚至商业系统。某些细节可能略有不同。本书使用 Beancount 软件的语法和计算方法编写。本书还假设你已经熟悉[<u>复式记账法的一般平衡概念</u>](the_double_entry_counting_method.md)和 Beancount 的一些语法，这些语法可以从其[<u>用户手册</u>](beancount_language_syntax.md)或其[<u>备忘单</u>](beancount_cheat_sheet.md)中获得。如果你还没有开始编写你的第一个文件，你可能想先阅读[<u>Beancount 入门</u>](getting_started_with_beancount.md)并先进行此操作。

命令行会计系统对它们可以计数的内容类型持中立态度，允许你创造性地发明各种单位来跟踪各种事务。例如，你可以计算“IRA 贡献美元”，这些不是实际美元，而是对应于“可能的实际美元贡献”，你可以获得资产、收入和支出类型的账户——这行得通。请注意，这些巧妙的技巧在更传统的会计系统中可能不可能。此外，这些系统中通常需要手动处理的某些操作可以为我们自动化，例如，“年结”完全由软件在任何时候进行，余额断言提供了一种保障，允许我们在风险较小的情况下更改过去交易的细节，因此不需要通过将过去冻结来“调节”。更大的灵活性在于手中。

最后，如果你有一个未在本文档中涵盖的交易输入问题，请在边缘留下评论，或将你的问题写在[<u>Ledger 邮件列表</u>](https://groups.google.com/forum/#!forum/ledger-cli)上。我希望本文档能够涵盖尽可能多的现实场景。

### 注意事项<a id="a-note-of-caution"></a>

阅读本文时，请注意作者是一个业余爱好者：我是一个计算机科学家，不是会计师。事实上，除了大学里上过的一般课程和完成 CFA 课程的第一年外，我没有任何实际的会计培训。尽管如此，我确实有一些实际经验，使用这种软件维护了三套账簿：我的个人账簿（8 年的完整财务数据记录所有账户），与配偶的联合账簿，以及我曾拥有的一家合同和咨询公司的账簿。我还用我的复式记账系统与我的会计师沟通了多年，并且他提出了建议。不过……我可能在这里或那里犯了一些基本错误，如果你发现任何可疑之处，请在边缘留下评论，我会很感激。

## 账户命名约定<a id="account-naming-conventions"></a>

你可以定义任何你喜欢的账户名，只要它以五类之一开头：资产、负债、收入、支出或权益（请注意，你可以使用选项自定义这些名称——有关详细信息，请参阅[<u>语言语法文档</u>](beancount_language_syntax.md)）。账户名称通常定义为具有多个名称*组件*，用冒号（:）分隔，暗示账户层次结构或“[<u>账户图表</u>](http://en.wikipedia.org/wiki/Chart_of_accounts)”：

```plaintext
Assets:Component1:Component2:Component3:...
```

随着时间的推移，我对账户名称的定义进行了许多迭代，并最终达到了以下约定，用于资产、负债和收入账户：

```plaintext
Type : Country : Institution : Account : SubAccount
```

我喜欢这种方式的原因是，当你渲染资产负债表时，生成的树首先按国家，然后按机构显示账户。

一些示例账户名称：

```plaintext
Assets:US:BofA:Savings       ; Bank of America “Savings” account
Assets:CA:RBC:Checking       ; Royal Bank of Canada “Checking” account
Liabilities:US:Amex:Platinum ; American Express Platinum credit card
Liabilities:CA:RBC:Mortgage  ; Mortgage loan account at RBC
Income:US:ETrade:Interest    ; Interest payments in E*Trade account
Income:US:Acme:Salary        ; Salary income from ACME corp.
```

有时，当有意义时，我会使用进一步的子账户。例如，Vanguard 内部会根据是员工贡献还是雇主的匹配金额来分别维护不同的账户：

```plaintext
Assets:US:Vanguard:Contrib401k:RGAGX  ; My contributions to this fund
Assets:US:Vanguard:Match401k:RGAGX    ; Employer contributions
```

对于投资账户，我倾向于通过将每种特定类型的股票存储在其自己的子账户中来组织它们的所有内容：

```plaintext
Assets:US:ETrade:Cash        ; The cash contents of the account
Assets:US:ETrade:FB          ; Shares of Facebook
Assets:US:ETrade:AAPL        ; Shares of Apple
Assets:US:ETrade:MSFT        ; Shares of Microsoft
…
```

这会自动按股票类型组织资产负债表，这真的很好。

我喜欢的另一个约定是，当我有不同类型的相关账户时，使用相同的机构组件名称。例如，上面的 E*Trade 资产账户有相关的收入流，将其记录在类似名称的账户下：

```plaintext
Income:US:ETrade:Interest    ; Interest income from cash deposits
Income:US:ETrade:Dividends   ; Dividends received in this account
Income:US:ETrade:PnL         ; Capital gains or losses from trades
…
```

对于“支出”账户，我发现通常没有相关的机构。对于这些账户，更有意义的是将它们视为类别，只需使用一个简单的层次结构来对应它们计算的支出类型，一些示例：

```plaintext
Expenses:Sports:Scuba        ; All matters of diving expenses
Expenses:Transport:Train     ; Train (mostly Amtrak, but not always)
Expenses:Transport:Bus       ; Various “chinese bus” companies
Expenses:Transport:Flights   ; Flights (various airlines)
…
```

我有*很多*这些，大约有 250 个或更多。由你决定定义多少以及如何细化或“分类”你的支出。但是，当然，你应该只在需要时定义它们；不要提前定义一个庞大的列表。添加新账户总是很容易的。

值得注意的是，机构不必是“真实”的机构。例如，我在一栋楼里拥有一个公寓单元，我使用 Loft4530 作为其所有相关账户的“机构”：

```plaintext
Assets:CA:Loft4530:Property
Assets:CA:Loft4530:Association
Income:CA:Loft4530:Rental
Expenses:Loft4530:Acquisition:Legal
Expenses:Loft4530:Acquisition:SaleAgent
Expenses:Loft4530:Loan-Interest
Expenses:Loft4530:Electricity
Expenses:Loft4530:Insurance
Expenses:Loft4530:Taxes:Municipal
Expenses:Loft4530:Taxes:School
```

如果你的所有业务都在一个国家进行，并且没有计划搬到其他地方，你可能会为了简洁省略国家组件。

最后，对于“权益”账户，嗯……通常你不会定义太多，因为这些账户主要是为了在资产负债表上报告前几年或当前期间的净收入和货币转换。通常，你至少需要一个，这个名称不太重要：

```plaintext
Equity:Opening-Balances           ; Balances used to open accounts
```

你可以自定义资产负债表上自动创建的其他权益账户的名称。

### 选择账户类型<a id="choosing-an-account-type"></a>

学习如何将交易记录到相应账户的一部分艺术是想出相关的账户名称，并设计一个资金在这些账户之间流动的方案，通过记下一些示例交易来实现。这有时会有些创意。当你在处理如何“计算”生活中的所有财务事件时，你经常会想知道为某些账户选择哪种账户类型。这应该是一个“资产”账户吗？还是“收入”账户？毕竟，除了创建报告之外，Beancount 并不对这些账户类型进行不同的处理……

但这并不意味着你可以随意使用任何类型。一个账户是否出现在资产负债表或损益表上确实很重要——通常有一个正确的选择。如果有疑问，这里有一些选择账户类型的指南。

> 首先，如果记录到账户的金额仅在*一段时间内*报告相关，它们应该是损益表账户之一：收入或支出。另一方面，如果金额*总是*需要包括在账户的总余额中，那么它应该是资产负债表账户：资产或负债。
>
> 其次，如果金额通常[^1]为正数，或“对你有利”，账户应该是资产或支出账户。如果金额通常为负数，或“对你不利”，账户应该是负债或收入账户。

基于这两个指标，你应该能够弄清任何情况。让我们通过一些示例来演练：

-   一顿餐馆餐表示你用一些资产（现金）或负债（用信用卡支付）换取的东西。没有人会关心“你从出生以来所有食物的总和”是多少。只有*过渡性*价值才重要：“我在餐馆花了多少钱*这个月*？”或者，*自年初以来*？或者，*在这次旅行中*？这显然指向一个支出账户。但你可能会想……这是一个正数，但这是我花的钱？是的，你从中支出的账户被扣减（借记），以换取你*收到*的支出。将支出账户中的数字视为你收到并立即消失的东西。这些餐点被消费……然后它们去了某个地方。好吧，我们在这里停止这个比喻。

-   你拥有一些债券股份，并收到一笔利息付款。这笔利息存入一个资产账户，例如一个交易账户。另一个要记录到的账户是什么？

### 选择开户日期<a id="choosing-opening-dates"></a>

你需要定义的一些账户不对应于现实世界的账户。`Expenses:Groceries` 账户表示自你开始记账以来的所有杂货支出总和。就我个人而言，我喜欢用我的*出生日期*作为这些账户的日期。这有一个理由：它汇总了你自来到这个世界以来花在杂货上的所有费用。

你可以在其他账户上使用这个理由。例如，与雇主相关的所有收入账户应该在你开始工作的日期开立，并在你离开的日期结束。这是有道理的。

## 如何处理现金<a id="how-to-deal-with-cash"></a>

让我们从现金开始。通常我会在出生日期定义两个账户：

```plaintext
1973-04-27 open Assets:Cash
1973-04-27 open Assets:ForeignCash
```

第一个账户用于实际使用，这代表我的钱包，通常只包含我的操作货币单位，即我通常认为是“现金”的商品。对我来说，它们是 USD 和 CAD 商品。

第二个账户用于存放我在世界各地旅行中积累的纸币，因此它们在另一个账户中不显眼，我不会在我的现金余额中看到它们。在旅行前，例如如果我返回日本，我会将 JPY 从 `Assets:ForeignCash` 转移到 `Assets:Cash`，并在旅行期间使用它们。

### 现金取款<a id="cash-withdrawals"></a>

从支票账户提取现金的 ATM 取款通常如下所示：

```plaintext
2014-06-28 * "DDA WITHDRAW 0609C"
  Assets:CA:BofA:Checking                     -700.00 USD
  Assets:Cash
```

你会在支票账户交易下载中看到这笔交易。

### 跟踪现金支出<a id="tracking-cash-expenses"></a>

当你告诉别人你在跟踪所有财务账户时，人们犯的一个错误是假设你必须将每笔小额的无关紧要的现金交易记录到笔记本中。事实并非如此！你可以自己决定记录*多少*这样的现金交易。

就我个人而言，我尽量减少更新账簿的手动工作量。我处理现金的规则是：

> *如果是食物或酒精，我不跟踪。*
>
> *如果是其他东西，我保留收据，稍后输入。*

这对我来说很有效，因为我的大多数现金支出往往是食物（或者我可能只是通过信用卡支付所有其他东西来做到这一点）。只有少数收据会在我的桌子上堆积几个月，然后我才懒得输入。

然而，你需要偶尔调整你的现金账户以反映这些支出。我并不经常这样做……大概每三个月一次，当我感觉需要时。我使用的方法是手动数一下我的钱包，并输入相应的余额断言：

```plaintext
2014-05-12 balance Assets:Cash       234.13 USD
```

每次这样做时，我还会添加一个现金分配以调整账户余额：

```plaintext
2014-06-19 * "Cash distribution"
  Expenses:Food:Restaurant           402.30 USD
  Expenses:Food:Alcohol              100.00 USD
  Assets:Cash                     ; -502.30 USD

2014-06-20 balance Assets:Cash       194.34 USD
```

如果你想知道现金账户中的金额为什么不加起来（234.13 -502.30 ≠ 194.34），那是因为在两个断言之间，我通过对支票账户进行一些 ATM 取款增加了现金账户的余额，这些取款出现在其他地方（在支票账户部分）。取款增加了现金账户的余额。如果我为 `Assets:Cash` 渲染一个日记帐，这些取款会显示出来。

我本可以使生活更简单，并使用 Pad 指令，如果我将所有内容记录为食物——pad 条目不仅在账户历史的开头有效，还可以在同一账户的两个余额断言之间使用——但我想将其中的 80% 记录为食物，20% 记录为酒精，以更准确地反映我对现金的实际使用情况[^2]。

最后，如果你在进行这一步骤的时间之间有很长一段时间，你可能希望通过手动添加多个现金分配[^3]来“分散”你的支出，这样如果你生成月度报告，大量现金支出不会在该月内或外显现为一个单一的块。

## 工资收入<a id="salary-income"></a>

记录你的工资是很有回报的：你将能够获得一年中收入的摘要以及各种扣款的明细，并且当你从雇主那里收到 W-2 表格（或如果你在加拿大，则为 T4），看到 Beancount 报告中的匹配数字时，你会感到满意。

我将与雇主相关的所有条目放在一个专用部分。我在开始工作的日期设置一个事件，例如，使用假设的公司“Hooli”（来自硅谷节目）：

```plaintext
2012-12-13 event "employer" "Hooli Inc."
```

这使我能够自动计算我在那里工作的天数。当我离开工作时，我会将其更改为新工作，或留空，如果我没有去其他工作：

```plaintext
2019-03-02 event "employer" ""
```

本节将做出一些假设。目标是向你展示正确记录收入的各种想法。你几乎肯定会根据具体情况调整这些想法。

### 就业收入账户<a id="employment-income-accounts"></a>

然后你为工资单定义账户。你需要确保有一个账户对应工资单的每一行。例如，以下是我为 Hooli Inc. 定义的一些收入账户：

```plaintext
2012-12-13 open Income:US:Hooli:Salary          USD ; "Regular Pay"
2012-12-13 open Income:US:Hooli:ReferralBonus   USD ; "Referral bonuses"
2012-12-13 open Income:US:Hooli:AnnualBonus     USD ; "Annual bonus"
2012-12-13 open Income:US:Hooli:Match401k       USD ; "Employer 401k match"
2012-12-13 open Income:US:Hooli:GroupTermLife   USD ; "Group Term Life"
```

这些对应于常规工资、员工推荐奖金、年度奖金、401k 收入（Hooli 在本例中会匹配你对退休账户的某个百分比的贡献）、人寿保险收入（同时作为收入和支出出现），以及你的健身房订阅福利支付。还有更多，但这是一个很好的例子。（在本例中，我将工资单上使用的名称写在注释中，但如果你喜欢，可以将其插入元数据中。）

你需要将源扣缴的税款记录到当年的账户中（有关详细信息，请参阅税收部分）：

```plaintext
2014-01-01 open Expenses:Taxes:TY2014:US:Medicare  USD
2014-01-01 open Expenses:Taxes:TY2014:US:Federal   USD
2014-01-01 open Expenses:Taxes:TY2014:US:StateNY   USD
2014-01-01 open Expenses:Taxes:TY2014:US:CityNYC   USD
2014-01-01 open Expenses:Taxes:TY2014:US:SDI       USD
2014-01-01 open Expenses:Taxes:TY2014:US:SocSec    USD
```

这些账户用于医疗保险税、联邦税、纽约州和纽约市税（是的，纽约市居民在州税的基础上还要额外征税）、州残疾保险（SDI）付款，最后是支付社会保障的税款。

你还需要在其他地方定义一些账户，用于自动从工资中支付的各种费用：

```plaintext
2012-12-13 open Expenses:Health:Life:GroupTermLife  USD ; "Life Ins."
2012-12-13 open Expenses:Health:Dental:Insurance    USD ; "Dental"
2012-12-13 open Expenses:Health:Medical:Insurance   USD ; "Medical"
2012-12-13 open Expenses:Health:Vision:Insurance    USD ; "Vision"
2012-12-13 open Expenses:Internet:Reimbursement     USD ; "Internet Reim"
2012-12-13 open Expenses:Transportation:PreTax      USD ; "Transit PreTax"
```

这些对应于典型的公司团体计划人寿保险支付、牙科、医疗和视力保险的保费、家庭互联网使用的报销以及公共交通的税前支付（纽约市允许你通过雇主使用税前资金支付你的 MetroCard）。

### 预订工资存款<a id="booking-salary-deposits"></a>

然后，当我通过直接存款将支付明细导入我的支票账户时，它会如下所示：

```plaintext
2014-02-28 * "HOOLI INC       PAYROLL"
  Assets:US:BofA:Checking               3364.67 USD
```

如果我还没有收到工资单，我可能会暂时将其记录到工资账户：

```plaintext
2014-02-28 * "HOOLI INC       PAYROLL"
  Assets:US:BofA:Checking               3364.67 USD
  ! Income:US:Hooli:Salary
```

当我收到或获取我的工资单时，我会删除此项并完成其他过账。对于总工资为 $140,000 的实际条目看起来大致如下：

```plaintext
2014-02-28 * "HOOLI INC       PAYROLL"
  Assets:US:BofA:Checking               3364.67 USD
  Income:US:Hooli:GroupTermLife          -25.38 USD
  Income:US:Hooli:Salary               -5384.62 USD
  Expenses:Health:Dental:Insurance         2.88 USD
  Expenses:Health:Life:GroupTermLife      25.38 USD
  Expenses:Internet:Reimbursement        -34.65 USD
  Expenses:Health:Medical:Insurance       36.33 USD
  Expenses:Transportation:PreTax          56.00 USD
  Expenses:Health:Vision:Insurance         0.69 USD
  Expenses:Taxes:TY2014:US:Medicare       78.08 USD
  Expenses:Taxes:TY2014:US:Federal      1135.91 USD
  Expenses:Taxes:TY2014:US:CityNYC        75.03 USD
  Expenses:Taxes:TY2014:US:SDI             1.20 USD
  Expenses:Taxes:TY2014:US:StateNY       340.06 USD
  Expenses:Taxes:TY2014:US:SocSec        328.42 USD
```

工资支付很少完全没有变化：工资处理器的四舍五入通常会导致一分钱的差异，社会保障支付将达到最大值，并且会不时发生各种扣款，例如应税福利收到的扣款。此外，401k 贡献将影响源扣缴的税款。因此，你最终需要逐个查看每张工资单以正确输入其信息。但这并不像听起来那么费时间！这里有一个技巧：如果你按工资单上的顺序列出过账，更新交易会容易得多。你只需复制粘贴前一个条目，从上到下阅读工资单并相应调整数字。每个条目只需一分钟。

需要注意的是前一个条目中的一些不寻常之处。团体人寿保险条目同时有 25.38 美元的收入部分和支出部分。这是因为 Hooli 支付了保费（工资单上就是这么写的）。Hooli 还报销了一些家庭互联网费用，因为我使用它处理生产问题。这显示为*负*过账，以减少我的 `Expenses:Internet` 账户的支出金额。

### 假期小时<a id="vacation-hours"></a>

我们的工资单还包括累积的假期和总假期余额，以假期小时为单位。你也可以在相同的交易中跟踪这些金额。你需要声明相应的账户：

```plaintext
2012-12-13 open Income:US:Hooli:Vacation           VACHR
2012-12-13 open Assets:US:Hooli:Vacation           VACHR
2012-12-13 open Expenses:Vacation:Hooli            VACHR
```

累积的假期是你收到的东西，因此以“VACHR”单位作为*收入*处理，并累积在一个*资产*账户中，该账户持有你当前可以“花费”作为休假的这些小时数。更新之前的工资收入交易条目：

```plaintext
2014-02-28 * "HOOLI INC       PAYROLL"
  Assets:US:BofA:Checking               3364.67 USD
  …
  Assets:US:Hooli:Vacation                 4.62 VACHR
  Income:US:Hooli:Vacation                -4.62 VACHR
```

每两周一次的工资单上的 4.62 VACHR 乘以一年 26 次大约等于 120 小时。按每天 8 小时计算，这是 15 个工作日，或 3 周，这对于本例中的新美国 Hooligans 来说是一个标准的假期套餐。

当你休假时，你会记录一个对你累积的假期时间的支出：

```plaintext
2014-06-17 * "Going to the beach today"
  Assets:US:Hooli:Vacation     -8 VACHR
  Expenses:Vacation:Hooli
```

*支出*账户跟踪你使用了多少假期。时不时地你可以检查工资单上报告的余额——你的雇主认为你剩余的假期量——是否与你记录的相同：

```plaintext
2014-02-29 balance Assets:US:Hooli:Vacation   112.3400 VACHR
```

你可以将假期小时单位按你的小时工资定价，这样你的假期*资产*账户会显示如果你决定辞职，公司会支付你多少。例如，按年薪 $140,000，工作 40 小时周和 50 周，每年 2000 小时，我们得到每小时 $70，你可以这样输入：

```plaintext
2012-12-13 price VACHR  70.00 USD
```

同样，如果你的假期小时过期或上限，你可以计算你通过工作太多而放弃假期时间所放弃的美元等值。你可以将一些 VACHR 从你的*资产*账户注销到一个收入账户（代表损失）。

### 401k 贡献<a id="k-contributions"></a>

401k 计划允许你使用税前美元对延期纳税的退休账户进行贡献。这是通过从你的工资中扣款进行的。要记录这些贡献，只需在交易中包括一个相应的退休账户贡献过账：

```plaintext
2014-02-28 * "HOOLI INC       PAYROLL"
  Assets:US:BofA:Checking               3364.67 USD
  …
  Assets:US:Vanguard:Cash               1000.00 USD
  …
```

如果你正在记录可用的贡献（请参阅本文档的税收部分），你会希望同时减少你的“401k 贡献”*资产*账户。你会在交易中添加两个过账：

```plaintext
2014-02-28 * "HOOLI INC       PAYROLL"
  Assets:US:BofA:Checking                       3364.67 USD
  …
  Assets:US:Vanguard:Cash                       1000.00 USD
  Assets:US:Federal:PreTax401k                 -1000.00 US401K
  Expenses:Taxes:TY2014:US:Federal:PreTax401k   1000.00 US401K
  …
```

如果你的雇主匹配你的贡献，这可能不会显示在你的工资单上。因为这些贡献是非应税的——它们直接存入一个延期纳税账户——你的雇主不必将其包含在扣缴声明中。你会看到它们直接在你的投资账户中作为存款出现。你可以这样记录到退休账户的税收余额：

```plaintext
2013-03-16 * "BUYMF - MATCH"
  Income:US:Hooli:Match401k           -1173.08 USD
  Assets:US:Vanguard:Cash              1173.08 USD
```

然后在你投资这笔现金时插入第二笔交易，或者如果你已经指定了资产分配并且由经纪人自动化，则直接从贡献中购买资产：

```plaintext
2013-03-16 * "BUYMF - MATCH"
  Income:US:Hooli:Match401k    -1173.08 USD
  Assets:US:Vanguard:VMBPX       106.741 VMBPX {10.99 USD}
```

请注意，管理你的 401k 账户的基金可能会在不同的桶中跟踪你的贡献和你雇主的贡献。你会声明子账户并进行相应的更改：

```plaintext
2012-12-13 open Assets:US:Vanguard:PreTax401k:VMBPX  VMBPX
2012-12-13 open Assets:US:Vanguard:Match401k:VMBPX   VMBPX
```

通常，他们这样做是为了分别跟踪每个来源的贡献，因为有一些限制取决于此。

### 股票授予的归属<a id="vesting-stock-grants"></a>

有关此主题的更多详细信息，请参阅[<u>专门文档</u>](stock_vesting_in_beancount.md)。

### 其他福利<a id="other-benefits"></a>

如果你愿意，你可以疯狂地跟踪福利。以下是一些疯狂的想法。

#### 积分<a id="points"></a>

如果你的雇主在现场提供一个赞助的按摩治疗计划，你可以在工资单上或通过某些内部网站（如果公司比较现代）预订按摩，并且可以使用某种内部积分系统支付，例如“HOOLI 积分”。你可以使用一种自创的货币，例如“MASSA”，并将其定价为 0.50 美元，即你可以购买它们的价格来跟踪这些积分：

```plaintext
2012-12-13 open Assets:US:Hooli:Massage    MASSA
2012-12-13 price MASSA  0.50 USD
```

当我购买新的按摩积分时，我会：

```plaintext
2013-03-15 * "Buying points for future massages"
  Liabilities:US:BofA:CreditCard    -45.00 USD
  Assets:US:Hooli:Massage              90 MASSA {0.50 USD}
```

如果你偶尔会获得一些这些积分，你可以在收入账户中跟踪。

#### 食物福利<a id="food-benefits"></a>

就像许多大技术公司一样，HOOLI 大概为其员工提供免费食物。这节省了时间，并鼓励人们吃得健康。这在当前的科技界是一种趋势。这个福利不会出现在任何地方，但如果你想将其定价为你的薪酬包的一部分，你可以使用一个*收入*账户来跟踪：

```plaintext
2012-12-13 open Income:US:Hooli:Food
```

根据你在工作中吃饭的频率，你可以估计每月的一些津贴：

```plaintext
2013-06-30 * "Distribution for food eaten at Hooli"
  Income:US:Hooli:Food		-350 USD
  Expenses:Food:Restaurant
```

## 货币转移和转换<a id="currency-transfers-conversions"></a>

如果你在货币之间进行转换，例如执行国际银行转账，你需要向 Beancount 提供汇率。这看起来像这样：

```plaintext
2014-03-03 * "Transfer from Swiss account"
  Assets:CH:UBS:Checking       -9000.00 CHF
  Assets:US:BofA:Checking      10000.00 USD @ 0.90 CHF
```

第二个过账的余额计算为 10000.00 美元 x 0.90 CHF/USD = 9000 CHF，交易平衡。根据你的喜好，你可以将汇率放在另一个过账上，如下所示：

```plaintext
2014-03-03 * "Transfer from Swiss account"
  Assets:CH:UBS:Checking       -9000.00 CHF @ 1.11111 USD
  Assets:US:BofA:Checking      10000.00 USD
```

第一个过账的余额计算为 -9000.00 CHF x 1.11111 USD/CHF = 10000.00 USD[^4]。通常我会选择报告给我的汇率，并将其放在相应的一侧。你可能还希望使用 F/X 市场用于交易汇率的方向，例如，瑞士法郎以 USD/CHF 交易，所以我更喜欢第一个交易。价格数据库在两个方向上转换汇率，因此这并不重要[^5]。

如果你使用电汇，这是这种资金转移的典型方式，你可能会产生费用：

```plaintext
2014-03-03 * "Transfer from Swiss account"
  Assets:CH:UBS:Checking       -9025.00 CHF
  Assets:US:BofA:Checking      10000.00 USD @ 0.90 CHF
  Expenses:Fees:Wires             25.00 CHF
```

如果你在旅游景点的那些看起来很阴暗的货币兑换处兑换现金，可能看起来像这样：

```plaintext
2014-03-03 * "Changed some cash at the airport in Lausanne"
  Assets:Cash             -400.00 USD @ 0.90 CHF
  Assets:Cash              355.00 CHF
  Expenses:Fees:Services     5.00 CHF
```

无论如何，你*永远不应该*使用成本基础语法转换货币单位，因为在存入单位后，原始转换率需要被遗忘，而不是保持与简单货币相关联。例如，这将是错误的用法：

```plaintext
2014-03-03 * "Transfer from Swiss account"
  Assets:CH:UBS:Checking       -9000.00 CHF
  Assets:US:BofA:Checking      10000.00 USD {0.90 CHF}  ; <-bad!
```

如果你误用了这一点，当你试图使用新存入的美元时，会产生错误：Beancount 会要求你指定这些“美元”的成本，以 CHF 计价，例如“从我以 0.90 USD/CHF 兑换的美元中借出”。现实世界中没有人这样做，当你表示你的交易时，你也不应该这样做：*一旦货币转换，它就只是不同货币的货币，没有关联的成本。*

最后，一个相当微妙的问题是，使用这些价格转换来回不同时间的不同汇率在某种程度上打破了会计等式：汇率变化可能会凭空创造出少量资金，所有余额最终不会加总为零。然而，这不是问题，因为 Beancount 实施了一个优雅的解决方案来自动纠正这个问题，因此你可以自由使用这些转换而不必担心：它在资产负债表上插入一个特殊的转换条目，以逆转报告的转换累计效果，并获得一个干净的零平衡。（讨论转换问题超出了本烹饪书的范围；如果你想了解更多，请参阅[<u>解决转换问题</u>](http://furius.ca/beancount/doc/conversions)。）

## 投资和交易<a id="investing-and-trading"></a>

跟踪交易及相关收益是一个相当复杂的话题。你将在[<u>使用 Beancount 进行交易</u>](trading_with_beancount.md)文档中找到对利润和损失的更完整介绍以及各种场景的详细讨论，该文档专门讨论此主题。这里我们将讨论如何设置你的账户并提供简单的示例交易以帮助你入门。

### 账户设置<a id="accounts-setup"></a>

你应该创建一个账户前缀来根各种与投资账户相关的子账户。假设你在 ETrade 有一个账户，这可能是“`Assets:US:ETrade`”。选择一个适当的机构名称。

你的投资账户将有一个现金部分。你应该创建一个专用的子账户来表示未投资的现金存款：

```plaintext
2013-02-01 open Assets:US:ETrade:Cash     USD
```

我建议你进一步定义一个子账户，用于你将投资的每种商品类型。尽管这不是严格必要的——Beancount 账户可以包含任意数量的商品——但这是一种汇总所有该商品头寸的好方法。假设你将购买 LQD 和 BND，两种流行的债券 ETF：

```plaintext
2013-02-01 open Assets:US:ETrade:LQD      LQD
2013-02-01 open Assets:US:ETrade:BND      BND
```

这也有助于生成更好的报告：余额通常显示为成本，看到按商品汇总的总成本有很多原因（即每种商品提供不同的市场特性）。使用专用的子账户来表示每个机构内持有的商品是一个好方法。除非你有特别的理由不这样做，否则我强烈建议默认坚持这一点（你可以随时通过重命名账户来更改）。

在你的账户上指定商品约束将有助于检测数据输入错误。股票交易往往比常规交易更复杂，这肯定会有所帮助。

然后，你希望在这个账户中收到两种形式的收入：资本收益，或“P&L”，和股息。我喜欢按机构记录这些收入，因为这就是它们必须为税收申报的方式。你可能还会收到利息收入。定义这些：

```plaintext
2013-02-01 open Income:US:ETrade:PnL          USD
2013-02-01 open Income:US:ETrade:Dividends    USD
2013-02-01 open Income:US:ETrade:Interest     USD
```

最后，为了记录交易费用和佣金，你需要一些通用账户来接收这些费用：

```plaintext
1973-04-27 open Expenses:Financial:Fees
1973-04-27 open Expenses:Financial:Commissions
```

### 资金转移<a id="funds-transfers"></a>

你通常会通过从外部账户（例如支票账户）转账到这个账户中添加一些初始资金：

```plaintext
2014-02-04 * "Transferring money for investing"
  Assets:US:BofA:Checking        -2000.00 USD
  Assets:US:ETrade:Cash           2000.00 USD
```

### 进行交易<a id="making-a-trade"></a>

购买股票的过账应将新商品存入商品的子账户，并相应地从现金账户扣除金额加上佣金：

```plaintext
2014-02-16 * "Buying some LQD"
  Assets:US:ETrade:LQD                 10 LQD {119.24 USD}
  Assets:US:ETrade:Cash          -1199.35 USD
  Expenses:Financial:Commissions     6.95 USD
```

注意，当你购买商品单位时，你是在账户的库存中建立一个新的交易批次，因此需要提供每单位的成本（在本例中，每股 LQD 119.24 美元）。这使我们能够正确地计算资本收益。

出售一些相同的股票的工作方式类似，除了添加一个额外的过账来吸收资本收益或损失：

```plaintext
2014-02-16 * "Selling some LQD"
  Assets:US:ETrade:LQD                 -5 LQD {119.24 USD} @ 123.40 USD
  Assets:US:ETrade:Cash            610.05 USD
  Expenses:Financial:Commissions     6.95 USD
  Income:US:Etrade:PnL
```

注意，从 `Assets:US:ETrade:LQD` 账户移除股票的过账是一个批次*减少*，你必须提供信息以标识你要减少的批次，在本例中，通过提供每股成本基础 119.24 美元。

通常我让 Beancount 为我计算资本收益或损失，这就是为什么我不在最后一个过账中指定它。Beancount 将自动通过设置该过账的金额为 -20.80 美元来平衡交易，这是一笔*收益* 20.80 美元（记住收入账户的符号是相反的）。指定 123.40 美元的销售价格是可选的，对于平衡交易没有影响，现金存款和佣金腿决定了利润。

### 收到股息<a id="receiving-dividends"></a>

收到股息有两种形式。首先，你可以收到现金股息，这将进入现金账户：

```plaintext
2014-02-16 * "Dividends from LQD"
  Income:US:ETrade:Dividends      -87.45 USD
  Assets:US:ETrade:Cash            87.45 USD
```

注意，这里没有指定股息的来源。你可以使用收入账户的子账户来单独计算。

或者你可以收到以股票再投资的股息：

```plaintext
2014-06-27 * "Dividends reinvested"
  Assets:US:Vanguard:VBMPX      1.77400 VBMPX {10.83 USD}
  Income:US:Vanguard:Dividends   -19.21 USD
```

这与股票购买类似，你还需要提供收到单位的成本基础。这通常发生在非应税的退休账户中。

有关更全面的讨论以及更多复杂的示例，请参阅[<u>使用 Beancount 进行交易</u>](trading_with_beancount.md)文档。

### 选择日期<a id="choosing-a-date"></a>

购买或出售单个股票批次通常涉及多个事件：交易被下达，交易被成交（通常在同一天），交易被结算。结算通常发生在交易成交后 2 或 3 个工作日。

为了简单起见，我建议使用交易日期作为交易日期。在美国，这是税务认可的日期，结算对你的账户没有影响（经纪人通常不会允许你在没有相应现金或保证金的情况下进行交易）。因此通常我不会费心为结算创建单独的条目，这不太有用。

可以设想更复杂的方案，例如，你可以将结算日期存储为元数据字段，然后在脚本中使用它，但这超出了本文档的范围。

## 结论<a id="conclusion"></a>

本文档尚不完整。我计划在完成时添加更多示例用例。我将在邮件列表上宣布这些内容。一些特别将讨论的主题包括：

-   医疗费用，例如保险费和回扣

-   税收

-   IRA、401k 和其他延期纳税账户

-   房地产

-   期权

[^1]: 这并不总是严格正确的：在公司会计中，某些账户类型因某些原因持有相反的值，通常用于抵消同类账户的值。这些被称为“对冲”账户。但作为个人，你不太可能有这些账户。如果你正在为公司设置账户图表，Beancount 实际上不关心余额是正号还是负号。你声明对冲账户就像声明常规账户一样。

[^2]: 我正在考虑支持扩展版的 Pad 指令，该指令可以接受一个百分比值，并使其可以仅填充全额的一部分，以实现自动化。

[^3]: 对 Beancount 的另一个扩展包括在两个余额断言之间支持多个 Pad 指令，并自动支持这种填充指令的分布。

[^4]: 如果你担心精度或四舍五入问题，请参阅[<u>本文档</u>](precision_tolerances.md)。

[^5]: 请注意，如果价格数据库需要反转日期，其计算结果可能是具有大量数字的价格。Beancount 使用 IEEE 十进制对象，Python 实现的默认上下文为 28 位数字，因此反转 0.9 将导致 1.111111….111 具有 28 位数字。
