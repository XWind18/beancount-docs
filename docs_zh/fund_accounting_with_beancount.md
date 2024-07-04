# 使用 Beancount 进行基金会计

[Martin Blais](http://plus.google.com/+MartinBlais)，Carl Hauser，2014年8月

[http://furius.ca/beancount/doc/proposal-funds](http://furius.ca/beancount/doc/proposal-funds)

*讨论如何在 Beancount 中进行基金会计，各种方法、解决方案和可能的扩展。*

## 动机

多名用户尝试使用命令行会计系统解决基金会计问题，部分原因是这种会计发生在预算有限且更倾向于使用免费软件的非营利组织的背景下，部分原因是这种灵活性和自定义需求似乎与命令行簿记系统非常匹配。

## 什么是基金会计？

例如，请参见[这个讨论](https://groups.google.com/d/msg/ledger-cli/N8Slh2t45K0/nu1ZACCueQYJ)：

> “我计算的另一个宗教义务是有效的什一税（我们称之为 Huqúqu'lláh，计算方法不同，但这是另一个故事）。为了计算应付的什一税，我将每笔存款的 19% 计入一个虚拟账户，然后从该账户中扣除每笔必要开支的 19%。
> 年底剩余的总额就是我应付的什一税。这个什一税账户不是一个真实的账户，因为它不存在于任何金融机构中；但作为个人义务，它足够真实。通过使用虚拟账户，我可以跟踪这个“侧带”负债，然后在需要时从资产账户中支付。如果我用 --real 报告，我只会看到我支付了多少这项负债；如果我不用 --real 报告，我会看到目前应付的 Huqúqu'lláh 总额。”
> — John Wiegley

这是另一位用户的评论描述：

> “基本上，想法是将你的财务生活分成几个称为“基金”的独立部分。每个基金都有自己的账户图表（在一定程度上）并且每个基金都遵守资产+负债+权益=0 的原则。这在非营利组织中经常需要，因为用于特定目的的钱需要单独核算。唯一不分离的领域是实际资产账户（如银行账户）和实际负债账户（如信用卡）可能代表多个基金持有或欠款，因此你不能完全使用独立的账本文件。在我们的教堂里，我们使用一个名为 PowerChurchPlus 的程序，这非常好用。我的妻子现在是一个社区音乐组织的财务主管，面临同样的问题，但规模很小，不值得花钱购买商业程序。”
> — Carl Hauser

根据 PowerChurchPlus 11.5 手册（PowerChurch, Inc. 2013）：

> “在会计中，基金这个术语有非常具体的含义。会计基金是一个自平衡的账户图表。 \[...\] 在会计中，我们需要将基金视为你的教堂的一个分部或子组。每个会计基金都有自己的资产、负债、权益、收入、转账和费用账户。
> 那么，什么时候使用额外的基金呢？如果你的教堂有一个需要独立报告的领域，你需要使用额外的基金。例如，如果你的教堂经营一个学前班或游戏班，你可能希望设置一个额外的基金来将他们的财务与教堂的财务分开。根据你的需要，你可能希望为男士或女士小组设置一个单独的基金。你甚至可能设置一个新的基金来单独跟踪所有固定资产，而不涉及日常运营账户。”

这是一个有趣且显然常见的问题。我们将在本节的其余部分描述用例。

### 共同账户管理

我个人使用这种“基金会计”理念来管理我和前妻的共同账户，我们都是在职的专业人士，并按需向共同账户注资。本节描述了我如何做这件事。

共同账户的会计记录在一个单独的文件中。创建了两个子账户来持有我们的“份额”：

```plaintext
2010-01-01 open Assets:BofA:Joint         
2010-01-01 open Assets:BofA:Joint:Martin  
2010-01-01 open Assets:BofA:Joint:Judie   
```

向共同账户的转账直接记录在两个子账户之一：

```plaintext
2012-09-07 * "Online Xfer Transfer from CK 345"
  Assets:BofA:Joint:Judie          1000.00 USD
  Income:Contributions:Judie
```

当我们发生费用时，我们会减少资产账户中的两个子账户。我们经常以 2:1 的比例记录它们，以解释收入差异，或者我只是将许多交易记录在自己名下（精确记录并不意味着我们在这方面不慷慨）：

```plaintext
2013-04-27 * "Urban Vets for Grumpy"
  Expenses:Medical:Cat           100.00 USD
  Assets:BofA:Joint:Martin          -50 USD
  Assets:BofA:Joint:Judie           -50 USD

2013-05-30 * "Takahachi"  "Dinner"
  Expenses:Food:Restaurant        65.80 USD
  Assets:BofA:Joint:Judie        -25.00 USD
  Assets:BofA:Joint:Martin
```

方便的是省略其中一个金额，因为我们并不非常精确。

### 处理多个基金

*(由 Carl Hauser 贡献)*

这是上面提到的 PowerChurchPlus 系统中使用的模型（用 Beancount 风格的名称替换其使用的账户编号）。“基金”名称*前缀*到账户名称。

```plaintext
Operations:Assets:Bank:...
Endowment:Assets:Bank:...
Operations:Liabilities:CreditCard:...
Endowment:Liabilities:CreditCard:...
Operations:Income:Pledges:2014
Operations:Expenses:Salaries:...
Operations:Expenses:BuildingImprovement:...
Endowment:Income:Investments:...
Endowment:Expenses:BuildingImprovement:...
```

任何交易都必须在它使用的每个基金中平衡。例如，我们的捐赠基金经常帮助支付超出当前捐款收入范围的事情。

```plaintext
2014-07-25 * "Bill’s Audio" "Sound system upgrade"
   Endowment:Assets:Bank1:Checking        800.00 USD
   Operations:Assets:Bank1:Checking       200.00 USD
   Endowment:Expenses:BuildingImprovement:Sound -800.00 USD
   Operations:Expenses:BuildingImprovement:Sound -200.00 USD
```

这代表了一张支付给 Bill’s Audio 的支票，资金来自捐赠和运营基金的资产账户 `Assets:Bank1:Checking`。

**注意 1：** 可以允许空基金名称并省略以下“:”，实际上，对于不想使用这些功能的人来说，这可能是默认选项（即，如果你不使用这些功能，什么都不会改变）。空字符串作为基金名称的基金当然与所有其他基金不同。

**注意 2：** `balance` 和 `pad` 指令对参与多个基金的账户不太有用。它们的使用需要知道账户在不同基金之间的分配，而外部持有者（如银行）的账户对账单不会有这些信息。允许类似于

```plaintext
2014-07-31 balance *:Assets:Bank1:Checking     579.39 USD
```

的检查可能很有用，但自动用 `pad` 条目纠正似乎不可能。

可以对任何基金或任何基金组合运行资产负债表报告，它将平衡。你可以轻松跟踪每个不同目的的所有物品。基金之间的转账作为一个基金的资产减少和费用记录，另一个基金的资产增加和收入记录。用于转账的收入和费用账户可以是通用的（`Operations:Income:Transfer`），也可以是为特定收入或费用类型设置的账户（`Endowment:Expense:BuildingImprovement:Sound`）作为转账交易的一个分录。

这里的账户名称语法只是可能的工作方式之一，并依赖于 Beancount 使用的五个特定命名的顶级账户。第一个账户名左侧的任何内容都可以视为基金名称，或者可以使用不同的分隔符将基金部分与账户名称部分分开。同样，我只展示了单级基金名称，但它们也可能是分层的。我不确定这有什么价值，但请注意，如果交易在叶级基金上平衡，它们也会在层次结构中的较高基金上平衡，并且可能会有一些利用价值。

对于 John W. 的 *Huqúqu'lláh* 例子，可以设置一个基金，其负债是“道德义务”而非法律义务（这似乎是仅在普通负债账户中跟踪什一税的反对理由）。当收入进入时（例如，直接存入真实银行的支票账户），将其 19% 计入“道德义务”基金的支票账户，并匹配一个负债。当进行必要开支时，从“道德义务”基金的支票账户中取回 19%，并减少负债。没有虚拟的过账或交易 —— 一切都必须平衡。如果我们有一个 `HisRetirement` 基金和一个 `HerRetirement` 基金，这样做会非常有用 —— 为遗产规划目的单独设置，但更常见的是我们希望知道我们的联合退休情况，这可以定义为一个等于 `HisRetirement` 和 `HerRetirement` 之和的*虚拟*基金 `OurRetirement`。请注意，这仅在创建报告时重要：只需正常的双重记账即可，在每个*真实*基金中平衡交易。当我说两个基金的“总和”时，我指的是取包含账户名称的并集，然后对公共账户的余额求和并保持其他账户的余额。

### 资产负债表

对于报告，想要按基金和跨基金的余额能力。我还使用一个报告，显示一部分基金，每列一个基金，对应的账户名称水平对齐，右侧有一个“总和”列。当所有基金都包括在此后续报告中时，你会获得一个完整的图景，了解你拥有和欠下的东西，以及为不同目的设置的资金或有不同限制的资金。以下是教会部分基金资产负债表的小样本。术语略有不同：Beancount 称为权益的在这里叫净资产。而且，由于在 PowerChurchPlus 中使用大量基金并不容易，净资产实际上按不同目的分类——这是我们在这里探索的想法真正闪耀的地方：如果基金真的很容易创建和合并报告，那么 PCP 中用于在基金内划分资产的一些机制就变得不必要了。

### 收入声明

对于收入和费用报告，通常我只关心一个基金中的费用，但如果需要，添加两个基金也完全有意义。

对我来说，这种基金会计方法很有吸引力，因为它依赖并保留了双重记账的基本原则：当交易总和为 0 时，资产负债表方程始终成立。通过这种方式，我们自动获得了组合*任何一组基金*的能力（我们不需要在输入交易或组织账户的深层结构时做任何特殊操作），并且它至少在算术上是有意义的，我们不依赖于任何与重命名或标记相关的“魔法”。我看不出通过将“基金”的想法推到账户层次结构中可以如此轻松或整齐地实现：基金属于*五个根账户之上*（资产、负债、权益、收入和费用），而不是它们之下。

## 实现的想法

**一些随机的想法。这需要更多的工作。**

- 如果需要多个冗余过账，可以使用[插件](beancount_scripting_plugins.md)自动生成这些过账。例如，如果使用类似于[镜像会计](http://furius.ca/beancount/doc/mirror)的技术，以“将相同的美元发送到多个账户”，至少用户不必手动执行此操作，这既乏味又容易出错。

- 可以使用在解析时重命名账户的过程，以将多个文件合并为一个文件。（允许用户安装这样的映射是我一直以来的想法，但从未实现，尽管它可以通过插件过滤器实现。）

- 我们可以依靠子账户的交易可以在父账户中合并和求和的事实（尽管在这方面的报告目前滞后。最终会实现）。

- 构建在前面的关于为基金做类似于标签堆栈的评论基础上。如果扩展当前的标签架构以允许标签具有值，`#fund=Operations` 或 `#fund=Endowment`。称它们为值标签。还需要允许过账有标签。值标签的语义是它们在任何过账中只能出现一次，明确在过账上的标签覆盖从交易继承的值标签，交易上的明确标签覆盖从标签堆栈继承的值，并且仅应用标签堆栈中最后一个值标签（具有特定键）到交易。这使得基金比前面的建议稍微不那么重要，即将它们放在账户名称前，但解决了前面提到的解析困难。这表明在基金中开立账户不是必要的，而前面的语法表明这是必要的。每个交易中严格的基金平衡规则仍然可以作为插件实现。报告基金（或基金总和）看起来像：

    - 选择任何过账匹配所需基金（或基金）的交易

    - 收集（必要时以明显的方式求和）与所报告基金（或基金）相关的过账（A）

    - 收集（以明显的方式求和）从所选交易中不与所需基金相关的过账（B）

    - 以（A）作为所需基金或基金的信息，（B）作为其他格式化报告。其他是确保报告平衡所需的，但可以选择省略。

从实现的角度来看，这似乎更正交于当前的现状，所需的现有代码更少。它添加了一个新功能——值标签，可以被插件和新报告使用来实现我们想要的基金会计。

## 示例

（Carl）这是我可能尝试处理与我的工资单相关的事情的示例，包括递延补偿（403(b) —— 与 401(k) 极为相似）和灵活支出账户（类似于 HSA，在 ledger-cli 小组中之前讨论过）。

### 没有基金

（Carl）首先，没有基金（通过 bean-check）：

这是我可能设置工资单的方式，包括健康保险、FSA 和 403b 供款，而没有使用基金。一个问题是它直接将总收入分配到 403b 和 FSA 账户，即使它认识到健康保险供款是工资减少，而 403b 和 FSA 也应该如此。因此，跟踪这两项的供款变得更加困难，以及跟踪应税收入。

通过认真思考我们可以解决这个问题——我们会提出代表供款的收入和费用类型账户，但它们看起来有点滑稽（在我看来），因为它们完全是内部的会计系统。

请参阅下一个示例，了解使用基金的方式。如果你伸出舌头，揉肚子并倒立，你会看到基于基金的解决方案在其交易复杂性方面相当于我们在上段中提出的复杂性——需要相同数量的行。主要优势是心理上的——更容易看出如何一致和有用。

```plaintext
option “title” “Paystub - no funds”

2014-07-15 open Assets:Bank:Checking
2014-07-15 open Assets:CreditUnion:Saving
2014-07-15 open Assets:FedIncTaxDeposits
2014-07-15 open Assets:Deferred:R-403b
2014-07-15 open Expenses:OASI
2014-07-15 open Expenses:Medicare
2014-07-15 open Expenses:MedicalAid
2014-07-15 open Expenses:SalReduction:HealthInsurance
2014-07-15 open Income:Gross:Emp1
2014-07-15 open Income:EmplContrib:Emp1:Retirement
2014-01-01 open Assets:FSA

; 这种设置 FSA 的方式看起来不错。它认识到年度指定金额是立即可用的（在资产账户中），我们有义务在一年中逐步供款（负债账户）。
2014-01-01 open Liabilities:FSA
2014-01-01 ! "Set up FSA for 2014"
  Assets:FSA                                                  2000 USD
  Liabilities:FSA                                             -2000 USD

2014-07-15 ! "Emp1 Paystub"
  Income:Gross:Emp1                                          -6000 USD
  Assets:Bank:Checking                                        3000 USD
  Assets:CreditUnion:Saving                                   1000 USD
  Assets:FedIncTaxDeposits                                     750 USD
  Expenses:OASI                                                375 USD
  Expenses:Medicare                                            100 USD
  Expenses:MedicalAid                                           10 USD
  Assets:Deferred:R-403b                                       600 USD
  Liabilities:FSA                                               75 USD
  Expenses:SalReduction:HealthInsurance                         90 USD
  Income:EmplContrib:Emp1:Retirement                          -600 USD
  Assets:Deferred:R-403b

2014-01-01 open Expenses:Medical
2014-07-20 ! "Direct expense from FSA"
  Expenses:Medical                                              25 USD
  Assets:FSA

2014-07-20 ! "Medical expense from checking"
  Expenses:Medical                                              25 USD
  Assets:Bank:Checking

2014-07-20 ! "Medical expense reimbursed from FSA"
  Assets:Bank:Checking                                          25 USD
  Assets:FSA
```

### 使用基金

（Carl）现在使用基金（使用建议的功能，因此无法通过 bean-check 检查）：

这是我可能设置工资单的方式，包括健康保险、FSA 和 403b 供款，使用基金。我可以直截了当地安排这些供款，以便它们被识别为工资减少以便所得税目的。而且我可以轻松看到我为 403b 供款了多少以及我的雇主供款了多少。

请参阅上一个示例，了解不使用基金的方式，不那么准确。

这没有做的是跟踪最终应税的 403b 资金。我认为这是一个人决定是否使用现金基础或权责发生制会计的问题。如果是现金基础，这些税款不是当前负债，不能报告为此类。如果是权责发生制，它们是当前负债，需要在收入记账时记录。

对于现金基础账簿，我们希望能够报告好像欠税一样的状态，但这是报告而不是记账的问题。我们需要确保有足够的可识别信息来自动创建这些报告。我相信采用基于基金的会计视角可以轻松做到这一点。

未解决的问题：如果你的基础对于联邦和州目的不同，甚至对于联邦和多个不同州目的不同。天哪！

我使用了基金名称在根账户名前的约定。请注意，通过适当的倒立和枢轴规则，你可以将基金名称放在任何地方。将其放在首位强调它标识了一组必须平衡的账户，并使事务处理器容易保证这一属性。没有基金名称的账户属于名称为空字符串的基金。

```plaintext
option "title" "Paystub - no Funds"

2014-07-15 open Assets:Bank:Checking
2014-07-15 open Assets:CreditUnion:Saving
2014-07-15 open Assets:FedIncTaxDeposits
2014-07-15 open Expenses:OASI
2014-07-15 open Expenses:Medicare
2014-07-15 open Expenses:MedicalAid
2014-07-15 open Expenses:SalReduction:HealthInsurance
2014-07-15 open Expenses:SalReduction:FSA
2014-07-15 open Expenses:SalReduction:R-403b
2014-07-15 open Income:Gross:Emp1
2014-07-15 open Income:EmplContrib:Emp1:Retirement
2014-01-01 open FSA:Assets                    ; FSA 基金账户
2014-01-01 open FSA:Income:Contributions
2014-01-01 open FSA:Expenses:Medical
2014-01-01 open FSA:Expenses:ReimburseMedical
2014-01-01 open FSA:Liabilities
2014-07-15 open Retirement403b:Assets:CREF    ; 退休基金账户
2014-07-15 open Retirement403b:Income:EmployeeContrib
2014-07-15 open Retirement403b:Income:EmployerContrib
2014-07-15 open Retirement403b:Income:EarningsGainsAndLosses

; 这实现了与上面对 FSA 的相同想法，通过平衡开盘时的资产和负债，但现在使用了一个单独的基金。
2014-01-01 ! "Set up FSA for 2014"
  FSA:Assets                                                  2000 USD
  FSA:Liabilities                                             -2000 USD

2014-07-15 ! "Emp1 Paystub"
  Income:Gross:Emp1                                          -6000 USD
  Assets:Bank:Checking                                        3000 USD
  Assets:CreditUnion:Saving                                   1000 USD
  Assets:FedIncTaxDeposits                                     750 USD
  Expenses:OASI                                                375 USD
  Expenses:Medicare                                            100 USD
  Expenses:MedicalAid                                           10 USD
  Expenses:SalReduction:R-403b                                 600 USD
  Retirement403b:Income:EmployeeContrib                        -600 USD
  Retirement403b:Assets:CREF                                   600 USD
  Expenses:SalReduction:FSA                                     75 USD
  FSA:Income:Contributions                                     -75 USD
  FSA:Liabilities                                             
  Expenses:SalReduction:HealthInsurance                       
  Retirement403b:Income:EmployerContrib                        -600 USD
  Retirement403b:Assets:CREF

2014-01-01 open Expenses:Medical
2014-07-20 ! "Direct expense from FSA"
  FSA:Expenses:Medical                                          25 USD
  FSA:Assets

2014-07-20 ! "Medical expense from checking"
  Expenses:Medical                                              25 USD
  Assets:Bank:Checking

2014-07-20 ! "Medical expense reimbursed from FSA"
  Assets:Bank:Checking                                          25 USD
  Income:ReimburseMedical
  FSA:Assets                                                   -25 USD
  FSA:Expenses:ReimburseMedical
```

## 转账账户提案

*由 Carl Hauser 撰写*

我在使用基金方法时遇到的一个问题是，在基金之间转账时很容易出错，例如上面最后的交易。形式化转账账户的想法可以帮助解决这个问题。最常见的错误是让两个账户中的资产都朝同一方向移动——都增加或都减少，如以下错误版本的交易所示：

```plaintext
2014-07-20 ! "Medical expense reimbursed from FSA - with mistake"
 Assets:Bank:Checking                            25 USD
 Income:ReimburseMedical
 FSA:Assets                                      25 USD
 FSA:Expenses:ReimburseMedical
```

这平衡了但不是我们想要的。假设我们添加了转账账户的概念。它们位于层次结构中的同一位置，如收入和费用一样，是非资产负债表账户。但它们实际上只在涉及多个基金的交易中发挥作用。转账账户交易有一个附加规则：转账的总和也必须为零（附加的意思是，每个基金内交易平衡的规则仍然存在）。因此，要使用它，我们需要稍微不同的设置：

```plaintext
2014-07-15 open FSA:Transfer:Incoming:Contribution
2014-07-15 open FSA:Transfer:Outgoing:ReimburseMedical
2014-07-15 open Transfer:Outgoing:FSAContribution
2014-07-15 open Transfer:Incoming:ReimburseMedical
```

错误的交易现在被标记，因为转账的总和是 `-50 USD`，而不是零。

```plaintext
2014-07-20 ! "Medical expense reimbursed from FSA - with mistake"
  Assets:Bank:Checking                                    25 USD
  Transfer:Incoming:ReimburseMedical
  FSA:Assets                                              25 USD
  FSA:Transfer:Outgoing:ReimburseMedical
```

使用转账账户处理 FSA 和退休账户金额的工资单交易可能如下所示（当然需要适当的 `open`）：

```plaintext
2014-07-15 ! "Emp1 Paystub - using transfer accounts"
  Income:Gross:Emp1                                      -6000 USD
  Assets:Bank:Checking                                    3000 USD
  Assets:CreditUnion:Saving                               1000 USD
  Assets:FedIncTaxDeposits                                 750 USD
  Expenses:OASI                                            375 USD
  Expenses:Medicare                                        100 USD
  Expenses:MedicalAid                                       10 USD
  Transfer:Outgoing:Retirement403b:SalReduction          600 USD
  Retirement403b:Transfer:Incoming:EmployeeContrib
  Retirement403b:Assets:CREF                               600 USD
  Transfer:Outgoing:FSA:SalReduction                        75 USD
  FSA:Transfer:Incoming:Contributions
  FSA:Liabilities                                                   
  Expenses:SalReduction:HealthInsurance                              
  Retirement403b:Income:EmployerContrib                   -600 USD
  Retirement403b:Assets:CREF
```

有些人可能认为这太复杂。无需更改转账账户的概念或规则，你可以简化过账，只为每个基金设置一个账户 `Fund:Transfer`，减少报告的精确度，但不丧失检查转账交易正确性的能力。

## 账户别名

Simon Michael 提到这与 HLedger [账户别名](http://hledger.org/how-to-use-account-aliases) 相关：

> “我认为这与希望查看实体财务情况分开和合并相关。账户别名可能是另一种方式，可以近似于这一点，如[http://hledger.org/how-to-use-account-aliases](http://hledger.org/how-to-use-account-aliases)。”

[^1]: 如果你发现自己在我们的现代生活方式中遇到文化挑战，也许你可以想象室友的情况，尽管我不喜欢这种关联带来的还原主义观点。
