# Beancount 语言语法<a id="title"></a>

[<u>Martin Blais</u>](mailto:blais@furius.ca), 更新：2016年4月

[<u>http://furius.ca/beancount/doc/syntax</u>](http://furius.ca/beancount/doc/syntax)

## 目录

1. [介绍](#introduction)
2. [语法概述](#syntax-overview)
3. [指令](#directives-1)
   - [打开账户](#open)
   - [关闭账户](#close)
   - [商品](#commodity)
   - [交易](#transactions)
   - [元数据](#metadata)
   - [收款人和叙述](#payee-narration)
   - [成本和价格](#costs-and-prices)
   - [平衡规则 - 过账的“权重”](#balancing-rule---the-weight-of-postings)
   - [减少头寸](#reducing-positions)
   - [金额插值](#amount-interpolation)
   - [标签](#tags)
   - [标签栈](#the-tag-stack)
   - [链接](#links)
   - [余额断言](#balance-assertions)
   - [多种商品](#multiple-commodities)
   - [批次聚合](#lots-are-aggregated)
   - [检查父账户](#checks-on-parent-accounts)
   - [关闭前](#before-close)
   - [填充](#pad)
   - [未使用的填充指令](#unused-pad-directives)
   - [商品](#commodities)
   - [成本基础](#cost-basis)
   - [多重填充](#multiple-paddings)
   - [注释](#notes)
   - [文档](#documents)
   - [从目录中导入文档](#documents-from-a-directory)
   - [价格](#prices)
   - [从过账中获取价格](#prices-from-postings)
   - [同一天的价格](#prices-on-the-same-day)
   - [事件](#events)
   - [查询](#query)
   - [自定义](#custom)
4. [元数据](#metadata-1)
5. [选项](#options)
   - [操作货币](#operating-currencies)
6. [插件](#plugins)
7. [包含文件](#includes)
8. [接下来是什么？](#whats-next)

## 介绍<a id="introduction"></a>

这是 Beancount 语言的用户手册，Beancount 是一个命令行双重记账系统。Beancount 定义了一种计算机语言，允许您在文本文件中输入财务交易并从中提取各种报告。它是一个通用的计数工具，可以处理多种货币、按成本持有的商品（例如股票），甚至可以跟踪不寻常的东西，如度假时间、航空里程和奖励积分，以及任何您可能想要计数的东西，甚至是豆子。

本文档介绍了 Beancount 的语法以及理解其如何进行计算所需的一些技术细节。本文档*不*提供 [<u>双重记账方法的介绍</u>](the_double_entry_counting_method.md)、[<u>动机</u>](command_line_accounting_in_context.md)、输入文件中交易的 [<u>示例和指南</u>](command_line_accounting_cookbook.md)，以及如何 [<u>运行工具</u>](running_beancount_and_generating_reports.md)。这些主题都有 [<u>自己的专门文档</u>](index.md)，建议在深入阅读此用户手册之前先浏览这些文档。此手册涵盖使用 Beancount 的技术细节。

## 语法概述<a id="syntax-overview"></a>

### 指令<a id="directives"></a>

Beancount 是一种声明性语言。输入由一个文本文件组成，主要包含 **指令** 或 **条目**（我们在代码和文档中交替使用这些术语）；还有用于定义各种 **选项** 的语法。每个指令都以相关的 *日期* 开始，日期决定了指令生效的时间点，其 *类型* 定义了指令所代表的事件类型。所有指令都以以下语法开头：

    YYYY-MM-DD <type> …

其中 YYYY 是年份，MM 是月份的数字形式，DD 是日期的数字形式。所有数字都是必需的，例如，2007 年 5 月 7 日应为“2007-05-07”，包括其前导零。Beancount 支持国际 ISO 8601 标准日期格式，带有连字符（例如“2014-02-03”），或使用斜杠的相同顺序（例如“2014/02/03”）。

以下是一些示例指令，以便您了解其美学：

    2014-02-03 open Assets:US:BofA:Checking

    2014-04-10 note Assets:US:BofA:Checking "Called to confirm wire transfer."

    2014-05-02 balance Assets:US:BofA:Checking   154.20 USD

解析输入文件的最终产物是这些条目的简单列表，存储在数据结构中。Beancount 中的所有操作都是在这些条目上执行的。

每种特定指令类型在下面的章节中都有记录。

#### 指令顺序<a id="ordering-of-directives"></a>

指令声明的顺序并不重要。实际上，条目在解析后和处理前会按时间顺序重新排序。这是该语言的一个重要特性，因为它使您可以按任意方式组织输入文件，而无需担心影响指令的意义。

除交易外，每个指令都假定发生在每天的*开始*。例如，您可以在账户开户当天声明第一个交易：

    2014-02-03 open Assets:US:BofA:Checking

    2014-02-03 * "Initial deposit"
      Assets:US:BofA:Checking         100 USD
      Assets:Cash                    -100 USD

但是，如果您假设立即关闭该账户，您不能在同一天声明关闭，您必须通过声明关闭日期为 2/4 来调整日期：

    2014-02-04 close Assets:US:BofA:Checking

这也解释了为什么在同一天发生的任何交易之前会验证余额断言。这是为了保持一致性。

### 账户<a id="accounts"></a>

Beancount 在账户中累积商品。账户名无需在文件中预先声明，只需通过其语法即可识别为“账户”。账户名是一个以冒号分隔的单词列表，这些单词以字母开头，其第一个单词必须是以下五种账户类型之一：

    Assets Liabilities Equity Income Expenses

账户名的每个组成部分以大写字母或数字开头，后跟字母、数字或破折号 (-) 字符。所有其他字符都是不允许的。

以下是一些实际的账户名示例：

    Assets:US:BofA:Checking
    Liabilities:CA:RBC:CreditCard
    Equity:Retained-Earnings
    Income:US:Acme:Salary
    Expenses:Food:Groceries

在输入文件中看到的所有账户名集合隐含定义了一个账户层次结构（有时称为 **账户表**），类似于文件系统中的文件组织方式。例如，以下账户名：

    Assets:US:BofA:Checking
    Assets:US:BofA:Savings
    Assets:US:Vanguard:Cash
    Assets:US:Vanguard:RGAGX
    Assets:Receivables

隐含声明了如下结构的账户树：

    `-- Assets
        |-- Receivables
        `-- US
            |-- BofA
            |   |-- Checking
            |   `-- Savings
            `-- Vanguard
                |-- Cash
                `-- RGAGX

我们会说“`Assets:US:BofA`”是“`Assets:US:BofA:Checking`”的父账户，后者是前者的子账户。

### 商品 / 货币<a id="commodities-currencies"></a>

账户包含 **货币**，我们有时也称之为 **商品**（我们交替使用这两个术语）。与账户名一样，货币名通过其语法识别，但与账户名不同，货币名不需要预先声明。货币的语法是一个全大写的单词，如下所示：

    USD
    CAD
    EUR
    MSFT
    IBM
    AIRMILE

（从技术上讲，货币名最多可以有 24 个字符，必须以大写字母开头，以大写字母或数字结尾，其其他字符必须仅限于大写字母、数字或有限的标点字符：“'.\_-”（撇号、句号、下划线、破折号）。前三个可能会让您联想到现实世界的货币（美元、加元、欧元）；接下来的两个，股票代码（微软和 IBM）。最后一个：奖励积分（航空里程）。Beancount 并不识别这些东西；从它的角度来看，这些工具都是同样对待的。没有内置的任何预先存在的货币概念。这些货币名称只是可以放入账户并在这些账户中积累的“东西”的名称。

没有“特殊”货币单位这一点很优雅，所有商品都是同样对待的：Beancount 天生就是一个多货币系统。如果您像我们中的许多人一样是外籍人士，并且您的生活分布在两个或三个大洲，这一点您会非常欣赏。您可以轻松处理国际账户分类账。

您的货币使用可以非常有创意：您可以为您的房子创建一种货币，例如（例如 `MYLOFT`），为积累的度假时间创建一种货币（`VACHR`），或为您每年允许的退休账户的潜在贡献创建一种货币（`IRAUSD`）。实际上，您可以用这种方式解决许多问题。[<u>食谱</u>](command_line_accounting_cookbook.md) 描述了许多这样的具体示例。

Beancount 不支持美元符号语法，例如，“$120.00”。在输入文件中您应始终使用货币名称。这使得输入更加规范，这是一个设计选择。对于货币单位，我建议您使用标准的 [<u>ISO 4217 货币代码</u>](http://en.wikipedia.org/wiki/ISO_4217#Active_codes) 作为指南；这些代码很快会变得熟悉。然而，如上所述，您可以在货币名称中包含一些其他字符，如下划线（\_）、破折号（-）、句号（.）、或撇号（'），但不能包含空格。

最后，您会注意到有一个“`commodity`”指令可以用来声明货币。它完全是可选的：货币会在使用时自动生成。该指令的目的是仅仅为了附加元数据。

### 字符串<a id="strings"></a>

每当我们需要在条目中插入一些自由文本时，应将其放在双引号中。这主要适用于收款人和叙述字段；基本上，任何不是日期、数字、货币或账户名的内容都应如此。

字符串可以分成多行。（多行字符串将包括其换行符，并且在渲染时需要相应地处理。）

### 注释<a id="comments"></a>

Beancount 输入文件不仅仅包含您的指令：您可以自由地在文件中添加注释和标题来组织文件。任何在字符“;”之后的行上的文本都会被忽略，例如：

    ; I paid and left the taxi, forgot to take change, it was cold.
    2015-01-01 * "Taxi home from concert in Brooklyn"
      Assets:Cash      -20 USD  ; inline comment
      Expenses:Taxi

如果愿意，您可以使用一个或多个“;”字符。如果您想输入更大的注释文本，可以在所有行前添加“;”。如果您更喜欢在日志中解析并渲染注释文本，请参阅本文档中有关注释指令的章节。

任何不是以有效 Beancount 语法指令开头的行（例如，以日期开头）都会被默默忽略。这样，您可以插入标记来组织文件的各种大纲模式，例如 Emacs 中的 [<u>org-mode</u>](http://orgmode.org/)。例如，您可以按机构组织输入文件，并分别折叠和展开每个部分，如下所示：

    * Banking
    ** Bank of America

    2003-01-05 open Assets:US:BofA:Checking
    2003-01-05 open Assets:US:BofA:Savings

    ;; Transactions follow …

    ** TD Bank

    2006-03-15 open Assets:US:TD:Cash

    ;; More transactions follow …

不匹配的行将被忽略。

Ledger 用户请注意：在 Ledger 中，“;”用于标记注释和附加“Ledger 标签”（Beancount 元数据）到过账。这在 Beancount 中不是这种情况。在 Beancount 中，注释始终只是注释。元数据有自己单独的语法。

## 指令<a id="directives-1"></a>

快速参考和概述指令语法，请参阅 [<u>语法备忘单</u>](beancount_cheat_sheet.md)。

### 打开账户<a id="open"></a>

所有账户都需要声明“打开”才能接受过账的金额。您可以通过如下指令进行声明：

    2014-05-01 open Liabilities:CreditCard:CapitalOne     USD

打开指令的一般格式是：

    YYYY-MM-DD open Account [ConstraintCurrency,...] ["BookingMethod"]

可选的逗号分隔的约束货币列表强制所有过账金额使用声明的货币单位。建议指定货币约束：您提供给 Beancount 的约束越多，您输入错误的可能性就越小，因为如果您输入错误，它会警告您。

每个账户应在第一个交易发生之前的特定日期声明“打开”（或与第一个交易发生的日期相同）。需要明确的是，打开指令不必出现在交易之前，但打开指令的*日期*必须早于过账到该账户的日期。文件中的声明顺序并不重要。因此，例如，这是一个合法的输入文件：

    2014-05-05 * "Using my new credit card"
      Liabilities:CreditCard:CapitalOne         -37.45 USD
      Expenses:Restaurant

    2014-05-01 open Liabilities:CreditCard:CapitalOne     USD
    1990-01-01 open Expenses:Restaurant

打开账户的另一个可选声明是“记账方法”，即当减少批次导致无法确定匹配批次时（0、2 或更多批次匹配）将调用的算法。目前，它可以取以下值：

-   **STRICT**: 批次规格必须完全匹配一个批次。这是默认方法。如果调用此记账方法，它将引发错误。这确保了您的输入文件明确选择了所有匹配的批次。

-   **NONE**: 不执行批次匹配。任何价格的批次都将被接受。允许为相同货币混合正数和负数批次。（这类似于 Ledger 的处理方式……它忽略了批次匹配。）

### 关闭账户<a id="close"></a>

与打开指令类似，有一个关闭指令可以用来告诉 Beancount 某个账户已经变为非活动状态，例如：

    ; Closing credit card after fraud was detected.
    2016-11-28 close Liabilities:CreditCard:CapitalOne 

关闭指令的一般格式是：

    YYYY-MM-DD close Account

此指令有几种用法：

-   如果您在关闭日期之后向该账户过账金额，会引发错误消息（这是一种完整性检查）。这有助于避免数据输入错误。

-   它有助于报告代码确定哪些账户仍然处于活动状态，并过滤掉报告期间之外的已关闭账户。这尤其有用，因为您的分类账随着时间的推移会累积大量数据，其中会有一些账户停止存在，您不希望在其关闭后的几年内在报告中看到这些账户。

请注意，关闭指令当前不会生成隐式的零余额检查。您可能需要在关闭日期之前添加一个，以确保账户在关闭时内容为空。

目前，一旦账户关闭，您无法在该日期之后重新打开它。（当然，您可以删除或注释掉关闭指令。）最后，代码中有一些实用函数，可以让您确定特定日期是否有账户是打开的。我强烈建议您在账户实际关闭时将其关闭，这将使您的分类账更加整洁。

### 商品<a id="commodity"></a>

有一个“商品”指令可以用来声明货币、金融工具、商品（在 Beancount 中是同义词）：

    1867-07-01 commodity CAD

商品指令的一般格式是：

    YYYY-MM-DD commodity Currency

该指令是后来引入的，完全是可选的：您可以在不声明的情况下使用商品。该指令的目的是附加商品特定的元数据字段，以便以后插件可以收集这些元数据。例如，您可能希望为每种商品提供一个长描述名称，例如为商品“CHF”提供“瑞士法郎”，或为“HOOL”提供“Hooli 公司 C 类股票”，如下所示：

    1867-07-01 commodity CAD
      name: "Canadian Dollar"
      asset-class: "cash"

    2012-01-01 commodity HOOL
      name: "Hooli Corporation Class C Shares"
      asset-class: "stock"

例如，一个插件可以收集元数据属性名称并执行按资产类别的汇总。

您可以为商品使用任何日期……但相关的日期是其创建或引入的日期，例如，加元在 1867 年首次引入，ILS（新以色列谢克尔）自 1986-01-01 开始使用。对于公司，成立公司并创建股票的日期可能是个好日期。由于该指令的主要目的是收集每个商品的信息，您选择的特定日期并不重要。

重复声明相同商品是错误的。

### 交易<a id="transactions"></a>

交易是分类账中最常见的指令类型。它们与其他指令稍有不同，因为它们可以后跟过账列表。以下是一个示例：

    2014-05-05 txn "Cafe Mogador" "Lamb tagine with wine"
      Liabilities:CreditCard:CapitalOne         -37.45 USD
      Expenses:Restaurant

与所有其他指令一样，交易指令以 *YYYY-MM-DD* 格式的日期开头，后跟指令名称，在本例中为“`txn`”。然而，由于交易是我们双重记账系统的*存在理由*，因此在 Beancount 输入文件中应出现最多的指令类型，我们做了一个特例，允许用户省略“txn”关键字并使用标志代替：

    2014-05-05 * "Cafe Mogador" "Lamb tagine with wine"
      Liabilities:CreditCard:CapitalOne         -37.45 USD
      Expenses:Restaurant

标志用于指示交易的状态，其特定含义由您定义。我们建议对它们进行以下解释：

-   *: 已完成交易，已知金额，“这看起来正确。”

-   !: 未完成交易，需要确认或修订，“这看起来不正确。”

在使用“txn”将交易留为空白标志的第一个示例中，交易对象上将设置默认标志（“\*”）。我几乎总是使用“\*”变体，而从不使用关键字变体，主要是为了与所有其他指令格式保持一致。

您还可以将标志附加到过账本身，如果您想特别标记交易的一条腿：

    2014-05-05 * "Transfer from Savings account"
      Assets:MyBank:Checking            -400.00 USD
      ! Assets:MyBank:Savings

这在去重交易的中间阶段非常有用（有关详细信息，请参阅 [<u>入门指南</u>](getting_started_with_beancount.md) 文档）。

交易指令的一般格式是：

    YYYY-MM-DD [txn|Flag] [[Payee] Narration] [Flag] Account Amount [{Cost}] [@ Price] [Flag] Account Amount [{Cost}] [@ Price] ...

接下来的几行是“过账”。您可以为交易附加任意数量的过账。例如，工资条目可能如下所示：

    2014-03-19 * "Acme Corp" "Bi-monthly salary payment"
      Assets:MyBank:Checking             3062.68 USD     ; Direct deposit
      Income:AcmeCorp:Salary            -4615.38 USD     ; Gross salary
      Expenses:Taxes:TY2014:Federal       920.53 USD     ; Federal taxes
      Expenses:Taxes:TY2014:SocSec        286.15 USD     ; Social security
      Expenses:Taxes:TY2014:Medicare       66.92 USD     ; Medicare
      Expenses:Taxes:TY2014:StateNY       277.90 USD     ; New York taxes
      Expenses:Taxes:TY2014:SDI             1.20 USD     ; Disability insurance

过账中的金额也可以是使用 `( ) * / - +` 的算术表达式。例如，

    2014-10-05 * "Costco" "Shopping for birthday"
      Liabilities:CreditCard:CapitalOne         -45.00          USD
      Assets:AccountsReceivable:John            ((40.00/3) + 5) USD
      Assets:AccountsReceivable:Michael         40.00/3         USD
      Expenses:Shopping

过账的唯一约束是其余额金额的总和必须为零。详见下文。

#### 元数据<a id="metadata"></a>

还可以将元数据附加到交易及其任何过账，因此完整的一般格式是：

    YYYY-MM-DD [txn|Flag] [[Payee] Narration] [Key: Value] ... [Flag] Account Amount [{Cost}] [@ Price] [Key: Value] ... [Flag] Account Amount [{Cost}] [@ Price] [Key: Value] ... ...

请参阅下面有关元数据的专用章节。

#### 收款人和叙述<a id="payee-narration"></a>

交易可以有一个可选的“收款人”和/或“叙述”。在上面的第一个示例中，收款人是“Cafe Mogador”，叙述是“Lamb tagine with wine”。

收款人是表示参与交易的外部实体的字符串。收款人在过账到费用账户的交易中特别有用，因为账户积累了来自多个业务的费用类别。一个好的例子是“`Expenses:Restaurant`”，它将包含一个人可能访问的各种餐厅的所有过账。

叙述是您编写的交易描述。它可以是关于上下文的评论、陪伴您的人的名字、您购买的产品的备注……无论您喜欢什么。您可以插入任何您喜欢的内容。我喜欢在对账时插入注释，这很快，我可以以后参考我的注释，例如，回答问题“去年冬天和安德烈亚斯在西边那家很棒的小寿司店叫什么名字？”

如果您在交易行上放置一个字符串，它就变成了其叙述：

    2014-05-05 * "Lamb tagine with wine"
       … 

如果您只想设置收款人，请放一个空的叙述字符串：

    2014-05-05 * "Cafe Mogador" ""
       … 

由于历史原因，接受在这些字符串之间使用管道符号（“|”）（但这将在未来某个时间删除）：

    2014-05-05 * "Cafe Mogador" | ""
       …

您还可以省略任一（但必须提供标志）：

    2014-05-05 *
       …

***Ledger 用户请注意。*** Ledger 没有单独的叙述和收款人字段，只有一个字段，称为“收款人”元数据标签，叙述则以保存的注释（“持久注释”）形式出现。在 Beancount 中，交易对象简单地有两个字段：收款人和叙述，其中收款人很多时候只是空值。

有关如何以及何时使用收款人或不使用的更深入讨论，请参阅 [<u>收款人、子账户和资产</u>](http://furius.ca/beancount/doc/payee)。

#### 成本和价格<a id="costs-and-prices"></a>

过账表示存入或取出账户的单一金额。最简单的过账类型仅包含其金额：

    2012-11-03 * "Transfer to pay credit card"
      Assets:MyBank:Checking            -400.00 USD
      Liabilities:CreditCard             400.00 USD

如果您从另一种货币转换金额，您必须提供汇率以平衡交易（见下一节）。这可以通过在过账中附加“价格”来完成，即转换率：

    2012-11-03 * "Transfer to account in Canada"
      Assets:MyBank:Checking            -400.00 USD @ 1.09 CAD
      Assets:FR:SocGen:Checking          436.01 CAD

您还可以使用“@@”语法指定总成本：

    2012-11-03 * "Transfer to account in Canada"
      Assets:MyBank:Checking            -400.00 USD @@ 436.01 CAD
      Assets:FR:SocGen:Checking          436.01 CAD

Beancount 会自动计算每单位价格，即 1.090025 CAD（请注意，最后两个示例的精度会有所不同）。

交易后，我们不再关注存入账户的美元单位的汇率；这些“USD”单位只是存入账户。从某种意义上说，它们的转换汇率已被遗忘。

但是，您存入账户的一些商品必须“按成本持有”。当您希望跟踪这些商品的*成本基础*时，会发生这种情况。典型的用例是投资，例如，当您将股票存入账户时。在这种情况下，您希望存入商品的购置成本附加到单位上，这样，当您稍后移除这些单位时（出售时），您应该能够通过成本来识别要移除的单位，以控制税收影响（并避免错误）。您可以想象，每个账户中的单位都有一个附加的成本基础。这将允许我们稍后自动计算资本收益。

为了指定要按特定成本持有的账户过账，请在大括号中包括成本：

    2014-02-11 * "Bought shares of S&P 500"
      Assets:ETrade:IVV                10 IVV {183.07 USD}
      Assets:ETrade:Cash         -1830.70 USD

这是一个更深入讨论的主题。请参阅“[<u>库存如何工作</u>](how_inventories_work.md)”和“[<u>使用 Beancount 进行交易</u>](trading_with_beancount.md)”文档以详细讨论这些主题。

最后，您可以在过账中同时包含成本和价格：

    2014-07-11 * "Sold shares of S&P 500"
      Assets:ETrade:IVV               -10 IVV {183.07 USD} @ 197.90 USD
      Assets:ETrade:Cash          1979.90 USD
      Income:ETrade:CapitalGains

价格将仅用于在价格数据库中插入价格条目（参见下文中的价格部分）。这将在本文档的平衡过账部分详细讨论。

***重要提示。*** 指定为每股或总成本或价格的金额始终是无符号的。使用负号或负成本是错误的，如果您尝试这样做，Beancount 将引发错误。

#### 平衡规则 - 过账的“权重”<a id="balancing-rule---the-weight-of-postings"></a>

双重记账方法的一个重要方面是确保其过账金额的总和在所有货币中为零。这是产生会计等式的核心、不可协商的条件，使得可以过滤任何交易子集并绘制平衡为零的资产负债表。

但是，使用不同类型的单位、前面介绍的价格转换和按成本持有的单位，这一切是什么意思？

很简单：我们制定了一个简单而明确的规则，用于从每个过账中获取金额和货币，并用于平衡它们。我们称之为过账的“权重”，或平衡金额。以下是使用四种可能的成本/价格组合从过账中派生权重的简短示例：

    YYYY-MM-DD
      Account       10.00 USD                       -> 10.00 USD
      Account       10.00 CAD @ 1.01 USD            -> 10.10 USD
      Account       10 SOME {2.02 USD}              -> 20.20 USD
      Account       10 SOME {2.02 USD} @ 2.50 USD   -> 20.20 USD

以下是如何计算的解释：

1.  如果过账**只有金额**而没有成本，则余额金额只是过账上的金额和货币。使用前一节的第一个示例，金额为 -400.00 USD，与第二腿的 400.00 USD 平衡。

2.  如果过账**只有价格**，则价格乘以单位数，并使用价格货币。使用前一节的第二个示例，即 -400.00 USD x 1.09 CAD(/USD) = -436.00 CAD，与其他过账的 436.00 CAD 平衡。

3.  如果过账**有成本**，则成本乘以单位数，并使用成本货币。使用前一节的第三个示例，即 10 IVV x 183.08 USD(/IVV) = 1830.70 USD。这与 -1830.70 USD 的现金腿平衡，因此一切正常。

4.  最后，如果过账**同时具有成本和价格**，我们会简单地忽略价格。此可选价格稍后用于在内存价格数据库中生成条目，但不用于平衡。

有了这个规则，您应该能够轻松平衡所有交易。此外，该规则使 Beancount 能够自动为您计算资本收益（有关详细信息，请参阅 [<u>使用 Beancount 进行交易</u>](trading_with_beancount.md)）。

#### 减少头寸<a id="reducing-positions"></a>

当您将*减少*过账到账户中的头寸时，减少必须始终匹配现有批次。例如，如果账户中持有 3200 USD，而交易将 -1200 USD 的变化过账到该账户，1200 USD 将与现有的 3200 USD 匹配，结果是 2000 USD 的单一头寸。这也适用于负值。例如，如果账户有 -1300 USD 的余额，并且您过账 +2000 USD 的变化，您将获得 700 USD 的余额。

无论账户类型如何，过账到账户的变化都可能导致正余额或负余额；对于简单商品金额（即没有附加成本的金额），余额没有限制。例如，虽然资产账户*通常*有正余额，负债账户*通常*有负余额，您可以合法地将资产账户贷记为负余额，或将负债账户借记为正余额。这是因为在现实世界中确实会发生这种情况：您可能会多写一张支票，从银行的支票账户中获得临时信用（通常伴随着高额的“透支”费用），或错误地支付两次信用卡余额。

对于按成本持有的商品，过账的成本规格必须匹配应用交易之前库存中持有的一个批次。收集批次列表，并根据过账的 {...} 部分的规格进行匹配。例如，如果您提供成本，则仅保留成本匹配的批次。如果您提供日期，则仅保留日期匹配的批次。您还可以使用标签。如果您同时提供成本和日期，则将这两者与要减少的候选批次列表进行匹配。这本质上是对要减少的批次列表的过滤器。

如果过滤后的列表结果为单个批次，则选择该批次进行减少。如果列表结果为多个批次，但减少的总金额等于批次中的总金额，则所有这些批次将按该过账减少。

例如，如果过去有以下交易：

    2014-02-11 * "Bought shares of S&P 500"
      Assets:ETrade:IVV                20 IVV {183.07 USD, "ref-001"}
      … 

    2014-03-22 * "Bought shares of S&P 500"
      Assets:ETrade:IVV                15 IVV {187.12 USD}
      … 

每个以下的减少都是明确的：

    2014-05-01 * "Sold shares of S&P 500"
      Assets:ETrade:IVV               -20 IVV {183.07 USD}
      … 

    2014-05-01 * "Sold shares of S&P 500"
      Assets:ETrade:IVV               -20 IVV {2014-02-11}
      … 

    2014-05-01 * "Sold shares of S&P 500"
      Assets:ETrade:IVV               -20 IVV {"ref-001"}
      … 

    2014-05-01 * "Sold shares of S&P 500"
      Assets:ETrade:IVV               -35 IVV {}
      … 

然而，以下将是不明确的：

    2014-05-01 * "Sold shares of S&P 500"
      Assets:ETrade:IVV               -20 IVV {}
      … 

如果多个批次匹配减少过账并且数量不等于总数，我们就处于*匹配模糊*的情况。然后调用账户的*记账方法*。有多种记账方法，但默认情况下，所有账户都设置为使用“STRICT”记账方法。这种方法在模糊情况下会简单地引发错误。

您可以将账户的记账方法设置为“FIFO”，以指示 Beancount 选择最旧的批次。或将其设置为“LIFO”，以选择最新的批次。这将自动选择所有必要的匹配批次以完成减少。

请注意，未来的 Beancount 版本中将更合理地处理日期匹配要求。有关此即将进行的更改的详细信息，请参阅 [<u>库存记账改进建议</u>](a_proposal_for_an_improvement_on_inventory_booking.md)。

对于此类过账，通常不可能导致负单位数。Beancount 当前不允许持有负数量的按成本持有的商品。例如，仅包含此交易的输入将失败：

    2014-05-23 *
      Assets:Investments:MSFT        -10 MSFT {43.40 USD}
      Assets:Investments:Cash     434.00 USD

如果它没有失败，这将导致 -10 单位的 MSFT 余额。另一方面，如果账户在 5/23 有 12 单位的 MSFT 持有成本为 43.40 USD，则交易会正常过账，将现有的 12 单位减少到 2 个。最常见的错误是账户持有 10 个或更多单位的 MSFT，但成本不同，并且用户指定了错误的成本值。例如，如果账户有 20 MSFT {42.10 USD} 的正余额，上述交易仍会失败，因为没有 10 个或更多单位的 43.40 USD 的 MSFT 可以移除。

此约束强制执行的原因有几个：

-   股票单位的数据输入错误并不罕见，它们对分类账的正确性有重要负面影响——金额通常很大——并且它们通常会触发此约束的错误。因此，错误检查是一种检测此类错误的有用方法。

-   持有负数量的按成本持有的单位相当罕见。很可能您根本不需要它们。例外情况包括：股票的空头销售，期货合约的持有差价，以及取决于您的记账方式，货币交易中的空头头寸。

这就是默认启用此检查的原因。

未来的 Beancount 版本中，我们将适当放宽这一约束。如果账户中没有其他持有的商品单位，我们将允许账户持有负数量的商品单位。要么，我们将允许您将账户标记为没有任何此类约束。目的在于允许对商品的空头头寸进行记账。唯一的阻碍因素是此约束。

有关库存记账算法的更多详细信息，请参阅 [<u>库存如何工作</u>](how_inventories_work.md) 文档。

#### 金额插值<a id="amount-interpolation"></a>

Beancount 能够自动填写交易的一些细节。目前，您可以在交易中最多省略一个过账的金额：

    2012-11-03 * "Transfer to pay credit card"
      Assets:MyBank:Checking            -400.00 USD
      Liabilities:CreditCard             

在上述示例中，省略了信用卡过账的金额。Beancount 会自动计算该金额为 400.00 USD，以平衡交易。这也适用于多个过账，并且适用于有成本的过账：

    2014-07-11 * "Sold shares of S&P 500"
      Assets:ETrade:IVV                 -10 IVV {183.07 USD}
      Assets:ETrade:Cash            1979.90 USD
      Income:ETrade:CapitalGains

在这种情况下，IVV 的单位以高于购买价格（$197.90）而非购买价格（$183.07）的价格出售。现金第一过账的权重为 -10 x 183.07 = -1830.70，第二过账为简单的 $1979.90。最后的过账将被分配差额，即 $149.20 USD，这就是说，获得了 $149.20 的*收益*。

在计算要平衡的金额时，使用与检查交易是否平衡为零相同的余额金额来填写缺少的金额。例如，以下不会触发错误：

    2014-07-11 * "Sold shares of S&P 500"
      Assets:ETrade:IVV                 -10 IVV {183.07 USD} @ 197.90 USD
      Assets:ETrade:Cash           

现金账户将自动收到 1830.70 USD，因为 IVV 过账的余额金额为 -1830.70 USD（如果过账同时具有成本和价格，则始终使用成本基础并忽略可选价格）。虽然从平衡的角度来看这是接受的且正确的，但从会计的角度来看这是不完整的：销售的资本收益需要单独核算，此外，如果您像上面那样进行过账，现金账户中的存款将低于实际存款（1979.00 USD），并希望随后的现金账户余额断言会通过触发错误来指示这一疏忽。

最后，当余额包含多种商品时，这也适用：

    2014-07-12 * "Uncle Bob gave me his foreign currency collection!"
      Income:Gifts                 -117.00 ILS
      Income:Gifts                -3000.00 INR
      Income:Gifts                 -800.00 JPY
      Assets:ForeignCash

将插入多个过账（每种商品一个）以替换省略的过账。

#### 标签<a id="tags"></a>

交易可以使用任意字符串标记：

    2014-04-23 * "Flight to Berlin" #berlin-trip-2014
      Expenses:Flights              -1230.27 USD
      Liabilities:CreditCard

这类似于在 Twitter 等平台上流行的“标签”概念。这些标签本质上允许您标记交易的子集。然后可以将其用作过滤器来生成仅此子集交易的报告。它们有多种用途。我喜欢用它们来标记所有旅行。

也可以指定多个标签：

    2014-04-23 * "Flight to Berlin" #berlin-trip-2014 #germany
      Expenses:Flights              -1230.27 USD
      Liabilities:CreditCard

（如果您想在指令中存储键值对，请参阅下面关于元数据的部分。）

##### 标签栈<a id="the-tag-stack"></a>

通常，多笔交易涉及同一标签会连续输入到文件中。作为一种便利，解析器可以自动标记这些交易。其工作原理很简单：解析器有一个“当前标签”栈，它会在读取交易时将其应用于所有交易。您可以将标签推送到此栈并从中弹出，如下所示：

    pushtag #berlin-trip-2014

    2014-04-23 * "Flight to Berlin"
      Expenses:Flights              -1230.27 USD
      Liabilities:CreditCard

    poptag #berlin-trip-2014

这样，您也可以将多个标签推送到长连续的一组交易，而无需全部手动输入。

#### 链接<a id="links"></a>

交易也可以链接在一起。您可以将链接视为一种特殊的标签，用于将一组在时间上财务相关的交易分组在一起。例如，您可以使用链接将与特定发票相关的交易分组在一起。这使得能够跟踪与发票相关的付款（或核销）：

    2014-02-05 * "Invoice for January" ^invoice-pepe-studios-jan14
      Income:Clients:PepeStudios           -8450.00 USD
      Assets:AccountsReceivable

    2014-02-20 * "Check deposit - payment from Pepe" ^invoice-pepe-studios-jan14
      Assets:BofA:Checking                  8450.00 USD
      Assets:AccountsReceivable

或者跟踪与单一目的相关的多笔转账：

    2014-02-05 * "Moving money to Isle of Man" ^transfers-offshore-17
      Assets:WellsFargo:Savings          -40000.00 USD
      Assets:WellsFargo:Checking          40000.00 USD

    2014-02-09 * "Wire to FX broker" ^transfers-offshore-17
      Assets:WellsFargo:Checking         -40025.00 USD
      Expenses:Fees:WireTransfers            25.00 USD
      Assets:OANDA:USDollar               40000.00

    2014-03-16 * "Conversion to offshore beans" ^transfers-offshore-17
      Assets:OANDA:USDollar          -40000.00 USD
      Assets:OANDA:GBPounds           23391.81 GBP @ 1.71 USD

    2014-03-16 * "God save the Queen (and taxes)" ^transfers-offshore-17
      Assets:OANDA:GBPounds             -23391.81 GBP
      Expenses:Fees:WireTransfers           15.00 GBP
      Assets:Brittania:PrivateBanking    23376.81 GBP

链接交易可以通过 Web 界面在其专用日志中渲染，无论当前视图/过滤的交易集如何（链接列表是全局页面）。

### 余额断言<a id="balance-assertions"></a>

余额断言是一种将您的对账单余额输入到交易流中的方法。它告诉 Beancount 验证特定商品在某个账户中的单位数量在某个时间点应等于某个预期值。例如，这

    2014-12-26 balance Liabilities:US:CreditCard   -3492.02 USD

表示“检查 2014 年 12 月 26 日*早上*，账户“`Liabilities:US:CreditCard`”中 USD 单位数量是否为 -3492.02 USD。”在处理条目列表时，如果 Beancount 遇到不同的 USD 余额，将报告错误。

如果未报告错误，您应该对之前在该账户中的交易列表非常有信心。这在实践中很有用，因为在许多情况下，一些交易可以从每个过账账户单独导入（见 [<u>去重问题</u>](getting_started_with_beancount.md)）。这可能导致您不经意间重复记录同一交易，并且定期插入余额断言将每次捕捉到这个问题。

余额指令的一般格式是：

    YYYY-MM-DD balance Account Amount

请注意，与所有其他非交易指令一样，余额断言适用于其日期的**开始**（即，当日午夜）。只需想象余额检查在当天午夜之后立即进行。

余额断言只对资产负债表账户（资产和负债）有意义。因为收入和支出账户的过账仅因为其临时价值而有趣，即，对于这些账户，我们对一定时间段内的变化总和（而非绝对值）感兴趣，因此在收入账户上使用余额断言意义不大。

另外，请注意，每个账户在打开后有一个隐含的余额断言，即账户在打开日期之后为空余额。您不需要在开户时显式断言零余额。

#### 多种商品<a id="multiple-commodities"></a>

Beancount 账户可以包含多种商品（尽管在实践中，您会发现这种情况并不常见，创建专用子账户来持有每种商品是合理的，例如，持有股票投资组合）。余额断言仅适用于断言的商品；不检查账户中其他商品的余额。如果要检查多种商品，请使用多个余额断言，如下所示：

    ; Check cash balances from wallet
    2014-08-09 balance Assets:Cash     562.00 USD
    2014-08-09 balance Assets:Cash     210.00 CAD
    2014-08-09 balance Assets:Cash      60.00 EUR

目前没有办法全面检查账户中的所有商品。

如果全面检查对您非常重要，可以通过定义现金账户的子账户来分别隔离每种商品，例如 `Assets:Cash:USD`、`Assets:Cash:CAD`。

#### 批次聚合<a id="lots-are-aggregated"></a>

余额断言适用于特定商品的单位总数，而不考虑其成本。例如，如果一个账户中有三批同一商品，例如 5 HOOL {500 USD} 和 6 HOOL {510 USD}，以下余额检查应成功：

    2014-08-09 balance Assets:Investing:HOOL     11 HOOL

所有批次会被聚合在一起，您可以验证其单位数。

#### 检查父账户<a id="checks-on-parent-accounts"></a>

可以在父账户上进行余额断言，包括其和其子账户的余额：

    2014-01-01 open Assets:Investing
    2014-01-01 open Assets:Investing:Apple       AAPL
    2014-01-01 open Assets:Investing:Amazon      AMZN
    2014-01-01 open Assets:Investing:Microsoft   MSFT
    2014-01-01 open Equity:Opening-Balances

    2014-06-01 *
      Assets:Investing:Apple       5 AAPL {578.23 USD}
      Assets:Investing:Amazon      5 AMZN {346.20 USD}
      Assets:Investing:Microsoft   5 MSFT {42.09 USD}
      Equity:Opening-Balances

    2014-07-13 balance Assets:Investing 5 AAPL
    2014-07-13 balance Assets:Investing 5 AMZN
    2014-07-13 balance Assets:Investing 5 MSFT

请注意，这要求已声明父账户为打开状态，以便在余额断言指令中合法使用。

#### 关闭前<a id="before-close"></a>

在关闭账户之前插入余额断言以确保其内容为空是有用的。关闭指令不会自动插入这个断言（我们可能最终会为此构建一个插件）。

#### 局部容差<a id="local-tolerance"></a>

有时需要覆盖余额检查的容差，以放宽该余额断言的检查。可以使用余额金额的局部容差量来完成，如下所示：

    2013-09-20 balance Assets:Investing:Funds     319.020 ~ 0.002 RGAGX

### 填充<a id="pad"></a>

填充指令会自动插入一个交易，使得后续的余额断言成功。如果需要，它会插入满足余额断言所需的差额。（就像 LaTeX 中的“橡皮空间”，Pad 指令对 Beancount 中的余额起作用。）

请注意，这里所说的“后续”是指*日期顺序*，而不是文件中的声明顺序。这本质上是一个会自动扩展或收缩以填补两个余额断言之间差额的交易。如下所示：

    2014-06-01 pad Assets:BofA:Checking Equity:Opening-Balances

填充指令的一般格式是：

    YYYY-MM-DD pad Account AccountPad

第一个账户是要贷记的账户，以自动计算金额。这是应在其后跟随余额断言的账户（如果账户没有余额断言，则填充条目无效，无所作为）。

第二个账户是交易的另一条腿，即资金来源，这几乎总是某个股权账户。这是因为该指令通常用于初始化新账户的余额，以避免我们手动插入此类指令，或者避免输入使账户达到当前余额的完整历史交易。

以下是一个实际的使用示例：

    ; Account was opened way back in the past.
    2002-01-17 open Assets:US:BofA:Checking

    2002-01-17 pad Assets:US:BofA:Checking Equity:Opening-Balances

    2014-07-09 balance Assets:US:BofA:Checking  987.34 USD

这将导致在填充指令之后同一日期插入以下交易：

    2002-01-17 P "(Padding inserted for balance of 987.34 USD)"
      Assets:US:BofA:Checking        987.34 USD
      Equity:Opening-Balances       -987.34 USD

这是一个正常的交易——您会在渲染日志中看到它与其他交易一起出现。（请注意特殊的“P”标志，可以被脚本用于查找这些交易。）

注意，没有余额断言，填充指令没有意义。因此，像余额断言一样，它们通常只用于资产负债表账户（资产和负债）。

您还可以在余额断言之间插入填充条目，这也可以。例如：

    2014-07-09 balance Assets:US:BofA:Checking  987.34 USD
    … more transactions… 
    2014-08-08 pad Assets:US:BofA:Checking Equity:Opening-Balances

    2014-08-09 balance Assets:US:BofA:Checking  1137.23 USD

如果没有中间交易，这将插入以下填充交易：

    2014-08-08 P "(Padding inserted for balance of 1137.23 USD)"
      Assets:US:BofA:Checking        149.89 USD
      Equity:Opening-Balances       -149.89 USD

这可能不明显，149.89 USD 是 1137.23 USD 与 987.34 USD 之间的差额。如果有更多中间交易将金额过账到支票账户，金额会自动调整以使第二个断言通过。

#### 未使用的填充指令<a id="unused-pad-directives"></a>

当前，您不能在输入文件中留下未使用的填充指令。它们将触发错误：

    2014-01-01 open Assets:US:BofA:Checking

    2014-02-01 pad Assets:US:BofA:Checking Equity:Opening-Balances

    2014-06-01 * "Initializing account"
      Assets:US:BofA:Checking                   212.00 USD
      Equity:Opening-Balances

    2014-07-09 balance Assets:US:BofA:Checking  212.00 USD

（这是否严格是一个可以辩论的问题，并且最终可能会移动到可选插件。）

#### 商品<a id="commodities"></a>

请注意，填充指令不指定任何商品。账户中与余额断言对应的所有商品都会受到影响。例如，以下代码会插入一个包含 USD 和 CAD 的过账的填充指令：

    2002-01-17 open Assets:Cash

    2002-01-17 pad Assets:Cash Equity:Opening-Balances

    2014-07-09 balance Assets:Cash    987.34 USD
    2014-07-09 balance Assets:Cash    236.24 CAD  

如果账户中包含其他未进行余额断言的商品，则不会为这些商品插入过账。

#### 成本基础<a id="cost-basis"></a>

目前，填充指令不适用于持有按成本持有头寸的账户。该指令实际上仅对现金账户有用。（这主要是因为余额断言尚不支持断言成本基础。将来如果我们决定支持断言总成本基础，则可能会考虑支持带成本基础的填充。）

#### 多重填充<a id="multiple-paddings"></a>

当前，您不能为同一账户和商品插入多个填充条目：

    2002-01-17 open Assets:Cash

    2002-02-01 pad Assets:Cash Equity:Opening-Balances

    2002-03-01 pad Assets:Cash Equity:Opening-Balances

    2014-04-19 balance Assets:Cash    987.34 USD

### 注释<a id="notes"></a>

注释指令只是用于在特定账户的日志中附加日期注释，如下所示：

    2013-11-03 note Liabilities:CreditCard "Called about fraudulent card."

渲染日志时，注释应在上下文中渲染。这对于记录与财务事件相关的事实和声明非常有用。我经常使用它记录那些否则不会进入交易的零碎信息。

余额。

#### 多种商品<a id="multiple-commodities"></a>

Beancount 账户可以包含多种商品（尽管在实践中，您会发现这种情况并不常见，创建专用子账户来持有每种商品是合理的，例如，持有股票投资组合）。余额断言仅适用于断言的商品；不检查账户中其他商品的余额。如果要检查多种商品，请使用多个余额断言，如下所示：

    ; Check cash balances from wallet
    2014-08-09 balance Assets:Cash     562.00 USD
    2014-08-09 balance Assets:Cash     210.00 CAD
    2014-08-09 balance Assets:Cash      60.00 EUR

目前没有办法全面检查账户中的所有商品。

如果全面检查对您非常重要，可以通过定义现金账户的子账户来分别隔离每种商品，例如 `Assets:Cash:USD`、`Assets:Cash:CAD`。

#### 批次聚合<a id="lots-are-aggregated"></a>

余额断言适用于特定商品的单位总数，而不考虑其成本。例如，如果一个账户中有三批同一商品，例如 5 HOOL {500 USD} 和 6 HOOL {510 USD}，以下余额检查应成功：

    2014-08-09 balance Assets:Investing:HOOL     11 HOOL

所有批次会被聚合在一起，您可以验证其单位数。

#### 检查父账户<a id="checks-on-parent-accounts"></a>

可以在父账户上进行余额断言，包括其和其子账户的余额：

    2014-01-01 open Assets:Investing
    2014-01-01 open Assets:Investing:Apple       AAPL
    2014-01-01 open Assets:Investing:Amazon      AMZN
    2014-01-01 open Assets:Investing:Microsoft   MSFT
    2014-01-01 open Equity:Opening-Balances

    2014-06-01 *
      Assets:Investing:Apple       5 AAPL {578.23 USD}
      Assets:Investing:Amazon      5 AMZN {346.20 USD}
      Assets:Investing:Microsoft   5 MSFT {42.09 USD}
      Equity:Opening-Balances

    2014-07-13 balance Assets:Investing 5 AAPL
    2014-07-13 balance Assets:Investing 5 AMZN
    2014-07-13 balance Assets:Investing 5 MSFT

请注意，这要求已声明父账户为打开状态，以便在余额断言指令中合法使用。

#### 关闭前<a id="before-close"></a>

在关闭账户之前插入余额断言以确保其内容为空是有用的。关闭指令不会自动插入这个断言（我们可能最终会为此构建一个插件）。

#### 局部容差<a id="local-tolerance"></a>

有时需要覆盖余额检查的容差，以放宽该余额断言的检查。可以使用余额金额的局部容差量来完成，如下所示：

    2013-09-20 balance Assets:Investing:Funds     319.020 ~ 0.002 RGAGX

### 填充<a id="pad"></a>

填充指令会自动插入一个交易，使得后续的余额断言成功。如果需要，它会插入满足余额断言所需的差额。（就像 LaTeX 中的“橡皮空间”，Pad 指令对 Beancount 中的余额起作用。）

请注意，这里所说的“后续”是指*日期顺序*，而不是文件中的声明顺序。这本质上是一个会自动扩展或收缩以填补两个余额断言之间差额的交易。如下所示：

    2014-06-01 pad Assets:BofA:Checking Equity:Opening-Balances

填充指令的一般格式是：

    YYYY-MM-DD pad Account AccountPad

第一个账户是要贷记的账户，以自动计算金额。这是应在其后跟随余额断言的账户（如果账户没有余额断言，则填充条目无效，无所作为）。

第二个账户是交易的另一条腿，即资金来源，这几乎总是某个股权账户。这是因为该指令通常用于初始化新账户的余额，以避免我们手动插入此类指令，或者避免输入使账户达到当前余额的完整历史交易。

以下是一个实际的使用示例：

    ; Account was opened way back in the past.
    2002-01-17 open Assets:US:BofA:Checking

    2002-01-17 pad Assets:US:BofA:Checking Equity:Opening-Balances

    2014-07-09 balance Assets:US:BofA:Checking  987.34 USD

这将导致在填充指令之后同一日期插入以下交易：

    2002-01-17 P "(Padding inserted for balance of 987.34 USD)"
      Assets:US:BofA:Checking        987.34 USD
      Equity:Opening-Balances       -987.34 USD

这是一个正常的交易——您会在渲染日志中看到它与其他交易一起出现。（请注意特殊的“P”标志，可以被脚本用于查找这些交易。）

注意，没有余额断言，填充指令没有意义。因此，像余额断言一样，它们通常只用于资产负债表账户（资产和负债）。

您还可以在余额断言之间插入填充条目，这也可以。例如：

    2014-07-09 balance Assets:US:BofA:Checking  987.34 USD
    … more transactions… 
    2014-08-08 pad Assets:US:BofA:Checking Equity:Opening-Balances

    2014-08-09 balance Assets:US:BofA:Checking  1137.23 USD

如果没有中间交易，这将插入以下填充交易：

    2014-08-08 P "(Padding inserted for balance of 1137.23 USD)"
      Assets:US:BofA:Checking        149.89 USD
      Equity:Opening-Balances       -149.89 USD

这可能不明显，149.89 USD 是 1137.23 USD 与 987.34 USD 之间的差额。如果有更多中间交易将金额过账到支票账户，金额会自动调整以使第二个断言通过。

#### 未使用的填充指令<a id="unused-pad-directives"></a>

当前，您不能在输入文件中留下未使用的填充指令。它们将触发错误：

    2014-01-01 open Assets:US:BofA:Checking

    2014-02-01 pad Assets:US:BofA:Checking Equity:Opening-Balances

    2014-06-01 * "Initializing account"
      Assets:US:BofA:Checking                   212.00 USD
      Equity:Opening-Balances

    2014-07-09 balance Assets:US:BofA:Checking  212.00 USD

（这是否严格是一个可以辩论的问题，并且最终可能会移动到可选插件。）

#### 商品<a id="commodities"></a>

请注意，填充指令不指定任何商品。账户中与余额断言对应的所有商品都会受到影响。例如，以下代码会插入一个包含 USD 和 CAD 的过账的填充指令：

    2002-01-17 open Assets:Cash

    2002-01-17 pad Assets:Cash Equity:Opening-Balances

    2014-07-09 balance Assets:Cash    987.34 USD
    2014-07-09 balance Assets:Cash    236.24 CAD  

如果账户中包含其他未进行余额断言的商品，则不会为这些商品插入过账。

#### 成本基础<a id="cost-basis"></a>

目前，填充指令不适用于持有按成本持有头寸的账户。该指令实际上仅对现金账户有用。（这主要是因为余额断言尚不支持断言成本基础。将来如果我们决定支持断言总成本基础，则可能会考虑支持带成本基础的填充。）

#### 多重填充<a id="multiple-paddings"></a>

当前，您不能为同一账户和商品插入多个填充条目：

    2002-01-17 open Assets:Cash

    2002-02-01 pad Assets:Cash Equity:Opening-Balances

    2002-03-01 pad Assets:Cash Equity:Opening-Balances

    2014-04-19 balance Assets:Cash    987.34 USD

### 注释<a id="notes"></a>

注释指令只是用于在特定账户的日志中附加日期注释，如下所示：

    2013-11-03 note Liabilities:CreditCard "Called about fraudulent card."

渲染日志时，注释应在上下文中渲染。这对于记录与财务事件相关的事实和声明非常有用。我经常使用它记录那些否则不会进入交易的零碎信息。

注释指令的一般格式是：

    YYYY-MM-DD note Account "String"

注释字段可以是多行的。如果愿意，您也可以省略注释字符串并将注释文本放在接下来的行上：

    2013-11-03 note Liabilities:CreditCard
      Called about fraudulent card.
      Company had already flagged it.

注意，注释指令与元数据注释不同（后者是附加到交易的）。

### 文档<a id="documents"></a>

类似于注释，文档指令将外部文件附加到分类账中：

    2013-11-03 document Liabilities:CreditCard "statements/2013-10.pdf"

如果设置了文档根目录，文档将放置在相对路径中。

该指令的一般格式是：

    YYYY-MM-DD document Account "String"

字符串可以指向分类账文件相对的文件路径。日志中的文档应提供指向外部文档的链接。

#### 从目录中导入文档<a id="documents-from-a-directory"></a>

由于现代会计越来越多地涉及电子文档，因此手动为每个文件插入文档指令既繁琐又容易出错。文档插入工具将允许用户插入目录中的文档。以下是一个示例：

    bean-file --documents ~/Documents/Banking/mybankstatements -f journal.beancount

工具将扫描目录中的文档，并为缺少的文档插入文档指令。日期可以从文件名解析（如文件名中不包含日期，则使用文件时间戳）。

此外，工具可以通过获取一个 CSV 文件并插入该文件中的文档来进一步增强。CSV 文件中的每一行包含文档指令所需的所有参数。

### 价格<a id="prices"></a>

价格指令用于声明货币单位的价格。它们也有自己的日期和一般格式：

    2014-02-03 price HOOL 1521.78 USD

该指令的一般格式是：

    YYYY-MM-DD price Symbol Amount

价格指令是输入的主要用途之一，但并不是唯一的。您可以选择插入交易过账的价格（见下文）。价格通常由一个守护进程或 Web 界面生成，并且会自动维护更新。

#### 从过账中获取价格<a id="prices-from-postings"></a>

如前所述，可以从过账生成价格条目。例如，以下交易：

    2012-11-03 * "Transfer to account in Canada"
      Assets:MyBank:Checking            -400.00 USD @ 1.09 CAD
      Assets:FR:SocGen:Checking          436.00 CAD

如前所述，解析器会读取过账的金额，将其存储为金额，并且只会处理所需字段。将来，它会插入以下价格条目：

    2012-11-03 price CAD 0.917431 USD

在内部，会跟踪所有价格条目，允许在时间段内查询货币对价格。

注意：这仅适用于原始货币对，尚不支持批次的内联成本。如果需要查看库存单位的计算增量，请使用价格指令。

#### 同一天的价格<a id="prices-on-the-same-day"></a>

当前，Beancount 仅支持每天一次的价格条目。未来可能会考虑价格的更详细时间戳。

### 事件<a id="events"></a>

事件指令用于为分类账中记录的事件附加日期。它可以是多行的：

    2013-01-01 event "company/created" "Lending Club"

渲染日志时，事件应在上下文中渲染。注释和文档等附加条目不会引发账户检查。

事件指令的一般格式是：

    YYYY-MM-DD event "String" "String"

### 查询<a id="query"></a>

查询指令用于记录和执行 SQL 查询，允许从分类账中过滤信息。以下是一个示例：

    2014-04-03 query "SELECT account, balance WHERE account ~ 'Assets'"

### 自定义<a id="custom"></a>

自定义指令允许您定义自己的指令并扩展日志的可能性。以下是一个示例：

    2014-04-03 custom "color" "Assets:US:BofA:Checking" "blue"

自定义指令的一般格式是：

    YYYY-MM-DD custom "String" "String" "String"

## 元数据<a id="metadata-1"></a>

如前所述，您可以为交易及其过账附加元数据。交易的元数据键值对如下所示：

    2014-02-01 * "Transaction" "Narration"
      Liabilities:CreditCard       -100.00 USD
      Expenses:Restaurant

您可以附加任何键值对：

    2014-02-01 * "Transaction" "Narration"
      key: "value"
      another-key: "another value"
      Liabilities:CreditCard       -100.00 USD
      Expenses:Restaurant

元数据也可以应用于每个过账：

    2014-02-01 * "Transaction" "Narration"
      key: "value"
      another-key: "another value"
      Liabilities:CreditCard       -100.00 USD
        key: "value"
      Expenses:Restaurant
        another-key: "another value"

## 选项<a id="options"></a>

### 操作货币<a id="operating-currencies"></a>

操作货币是可选的，但强烈建议您定义它们。它们定义了 Beancount 进行报告和汇总的基础货币。以下是一个示例：

    option "operating_currency" "USD"
    option "operating_currency" "CAD"

操作货币的格式是：

    option "operating_currency" "Currency"

这使得 Beancount 知道进行报告和计算的货币。

## 插件<a id="plugins"></a>

插件用于增强 Beancount 的功能，允许您扩展其功能。插件的定义如下所示：

    plugin "name_of_plugin"

## 包含文件<a id="includes"></a>

包含文件指令允许您包含外部文件，从而分离日志的不同部分。以下是一个示例：

    include "path/to/file.beancount"

包含文件的格式是：

    include "path/to/file"

这允许您将大日志拆分为多个文件，并在需要时包含它们。

## 接下来是什么？<a id="whats-next"></a>

现在您已经了解了 Beancount 的基本语法，接下来可以参考更详细的文档，了解如何使用 Beancount 进行双重记账和生成报告。建议您阅读以下文档：

-   [双重记账方法介绍](the_double_entry_counting_method.md)
-   [动机](command_line_accounting_in_context.md)
-   [示例和指南](command_line_accounting_cookbook.md)
-   [运行工具](running_beancount_and_generating_reports.md)

这些文档将帮助您更深入地了解如何使用 Beancount 进行双重记账和生成报告。
