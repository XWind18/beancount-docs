# 在 Beancount 中共享费用

作者：Martin Blais，2015年5月

[原文链接](http://furius.ca/beancount/doc/shared)

## 简介

本文档介绍了一种在复杂的群体情况下，准确且简便地记账共享费用的方法。例如，与一群朋友一起旅行时，每个人为不同的事物付费。我们将展示如何处理应在群体中平均分摊的费用以及应分配给特定个人的群体费用。

该方法使用复式记账系统。其核心是将费用和相关支付的记账分开。这使得处理共享成本更加容易，因为使用这种方法，无论谁为什么支付，我们都能在最后精确核对每个人的欠款，并自动计算最终的校正转账。

本文围绕由两人平均分摊费用的一次简单旅行进行阐述。然而，该方法同样适用于任何类型的项目，其中的收入和费用需在多个参与者之间分摊，费用可平均或不平均分摊。

## 旅行示例

作者Martin和女友Caroline于2015年3月访问了墨西哥。我们前往科苏梅尔岛进行为期三天的潜水旅行，然后前往图卢姆放松两天并在塞诺特潜水。

我们的假设是：

- **混乱的支付方式**。由于生活忙碌，我们将提前和旅行期间无计划地支付各种费用，只要有需要安排这次旅行的费用，我们都会支付。例如，Caroline提前选择并预订了航班，而我支付了度假村费用并在出发前一周预订了租赁和潜水活动。
- **使用共享和个人资产**。我们都带了现金，并将在旅行期间从各自的钱包以及转换为当地货币（墨西哥比索）的共享现金池中支付，还将根据需要使用信用卡支付。
- **多种货币**。我们的某些费用以美元支付，某些以墨西哥比索支付。例如，航班费用以美元支付，当地餐费和住宿费用以比索支付，但当地潜水店收取美元。转换金额将来自现金和信用卡。
- **共享池中的个人费用**。虽然大多数费用应平均分摊，但有些费用只适用于我们其中一个人，我们希望明确记账。例如，Caroline参加了一个潜水认证课程（PADI开放水域潜水员），费用由她自己支付；同样，她不应支付我的昂贵船潜费用。复杂的是，潜水店在我们离开时给我们出了一张共同账单。

## 关于共享的说明

在讨论共享费用时，我认为应该提及一些事项。

**我们故意对每一分钱的支出进行详细记账**。本文档的目的是展示如何高效、简单地解开一组复杂的交易，精确记账每个参与者的费用，无论谁实际支付。我们不是吝啬鬼。

**我们假设费用平均分摊**。我们之间的“慷慨”不是本文档的相关话题。我们都是薪酬优厚的职业人士，假设我们已经同意平均分摊这次旅行的共同费用（50/50）。

该方法的一个属性是，选择如何分摊费用的决定可以与实际支付费用分开。例如，最终我们可以决定由我支付三分之二的旅行费用，但这可以精确决定，而不是随意地“哦，我记得我为这个支付过”那样。特别是在更大的人群中，这特别有用，因为如果费用未被准确跟踪，通常每个人都会觉得他们支付的比其他人多。一个群体中支付的*感知*费用总和总是大于 100%。

## 方法概述

本节简要概述了该方法。项目中包含一组常见的资产账户，所有个人费用和旅行转账都来自外部收入账户：

旅行期间，我们使用共同资产支付费用。大多数费用应最终由我们共享，但有些费用应分配给我们各自：

旅行结束后，将剩余资产（例如带回家的现金）分配给我们自己，以归零资产账户的余额，并通过对收入账户的反向分录记录：

最后，共享费用清单将在彼此之间分摊——使用一个插件分割每个分录，最终通过最终转账使每个人各自支付其应付费用，并达到平衡：

请注意，每个参与者的最终费用余额可能不同，这些是由于某些费用是分别分配的，或者我们决定不均分摊总费用。

## 如何跟踪费用

为了记录这次旅行的所有费用，我们将打开一个全新的 Beancount 输入文件。尽管费用来自每个人的个人账户，但最好将旅行视为一个特殊项目，就像一个仅在旅行期间存在的公司。我们为这次旅行编写的示例文件可以在[这里](https://raw.githubusercontent.com/beancount/beancount/v2/examples/sharing/cozumel2015.beancount)找到，有助于您跟随本文。

### 账户

输入文件中的账户集不必与您的个人 Beancount 文件的账户名称匹配。我们将使用的账户包括对应于 Martin 和 Caroline 的个人账户，但使用通用名称（例如 `Income:Martin:CreditCard` 而不是 `Liabilities:US:Chase`），费用账户可能与我个人 Beancount 文件中的常规费用不匹配，这并不重要。

作为惯例，任何特定旅行者的账户名称中应包含该人的姓名。例如，Caroline 的信用卡账户将命名为“`Income:Caroline:CreditCard`”。这很重要，因为我们将在后面使用这一点来分摊贡献和费用。

让我们看看执行此操作所需的不同类型的账户。

#### 外部收入账户

项目将从任一旅行者的个人账户收到转账形式的收入。这些账户我们认为是项目的外部账户，因此将其定义为收入账户：

```plaintext
;; Martin 的外部账户。
2015-02-01 open Income:Martin:Cash
2015-02-01 open Income:Martin:Cash:Foreign
2015-02-01 open Income:Martin:Wallet
2015-02-01 open Income:Martin:CreditCard

;; Caroline 的外部账户。
2015-02-01 open Income:Caroline:Cash
2015-02-01 open Income:Caroline:Wallet
2015-02-01 open Income:Caroline:MetroCard
2015-02-01 open Income:Caroline:CreditCard
```

这些账户的交易必须从您的个人 Beancount 文件中复制。显然，您必须小心包括所有与旅行相关的交易。我在个人文件中使用标签来做到这一点。

#### 资产和负债账户

在旅行期间，将有一些资产账户是活跃的，并且存在这些临时账户应在旅行结束时清零。一个例子是当地货币的零用现金池：

```plaintext
2015-02-01 open Assets:Cash:Pesos
  description: "A shared account to contain our pocket of pesos"
```

我们旅行时也带了现金，因此我为此创建了两个单独的账户：

```plaintext
2015-02-01 open Assets:Cash:Martin
  description: "Cash for the trip held by Martin"

2015-02-01 open Assets:Cash:Caroline
  description: "Cash for the trip held by Caroline"
```

请注意，尽管这些账户包含个人名称，但它们被认为是项目的一部分。将其分别追踪余额只是方便而已。

#### 费用账户

我们将定义各种账户来记录我们的费用。例如，“`Expenses:Flights`” 将包含与航班旅行相关的费用。为了方便起见，因为文件中有许多类型的费用，我们选择使用“自动账户”插件，让 Beancount 自动打开这些账户：

```plaintext
plugin "beancount.ops.auto_accounts"
```

这些账户大多数是我们共享的费用账户。例如，共享的潜水费用将记录在“`Expenses:Scuba`”账户中。

但是，对于应由我们其中一个人单独支付的费用，我们只需在账户名称中包含旅行者的姓名。例如，Martin 的额外船潜费用将记录在“`Expenses:Scuba:Martin`”账户中。

### 示例交易

让我们看看文件中存在的不同类型的交易。本节将带您了解一些有代表性的交易（我给这些起的名字是任意的）。

#### 贡献交易

对项目费用的贡献通常以外部账户支付的费用形式进行。例如，Caroline 用她的信用卡支付航班费用如下所示：

```plaintext
2015-02-01 * "Flights to Cancun"
   Income:Caroline:CreditCard        -976.00 USD
   Expenses:Flights
```

Martin 支付到机场的出租车费用如下所示：

```plaintext
2015-02-25 * "Taxi to airport" ^433f66ea0e4e
  Expenses:Transport:Taxi                   62.80 USD
  Income:Martin:CreditCard
```

#### 带现金

我们都带了一些现金，因为被告知在墨西哥可能很难使用信用卡。在我的个人 Beancount 文件中，现金账户是“`Assets:Cash`”，但在这里它必须作为外部贡献记账，带上我的名字：

```plaintext
;; 旅行时的初始现金。
2015-02-24 * "Getting cash for travel"
  Income:Martin:Cash                        -1200 USD
  Assets:Cash:Martin                         1200 USD
```

Caroline 的现金类似：

```plaintext
2015-02-24 * "Getting cash for travel"
  Income:Caroline:Cash                       -300 USD
  Assets:Cash:Caroline                        300 USD
```

再次注意，“`Assets:Cash:Martin`” 和“`Assets:Cash:Caroline`”账户被视为项目的一部分，这里只是在方便地分别记录我们在旅行期间的现金持有情况。

#### 转账

在旅行前，我非常忙碌。看起来 Caroline 将进行大部分安排和预付款，因此我通过 Google Wallet 向她转账以帮助她提前支付一些费用：

```plaintext
2015-02-01 * "Transfer Martin -> Caroline on Google Wallet"
  Income:Martin:Wallet                      -1000 USD
  Income:Caroline:Wallet                     1000 USD
```

#### 现金转换

我们通过在机场兑换少量现金获取当地货币（汇率非常差）：

```plaintext
2015-02-25 * "Exchanged cash at XIC at CUN airport"
  Assets:Cash:Caroline                    -100.00 USD @ 12.00 MXN
  Assets:Cash:Pesos                          1200 MXN
```

“`Assets:Cash:Pesos`”账户记录我们共同的本地货币池，用于各种小额费用。

#### 以美元支付的现金费用

某些本地费用需要以美元支付，例如这个例子中我从我的现金口袋中支付的费用：

```plaintext
2015-03-01 * "Motmot Diving" | "Deposit for cenote diving"
  Expenses:Scuba                            50.00 USD
  Assets:Cash:Martin
```

#### 以本地货币支付的现金费用

使用比索从我们的共享比索池支付现金费用如下所示：

```plaintext
2015-02-25 * "UltraMar Ferry across to Cozumel"
  Expenses:Transport:Bus                326 MXN
  Assets:Cash:Pesos
```

有时我们甚至不得不使用美元和比索混合支付。在这个例子中，我们用光了比索，所以我们不得不给他们美元和比索（图卢姆海滩区的所有餐厅和酒店都接受美元）：

```plaintext
2015-03-01 * "Hartwood" | "Dinner - ran out of pesos"
  Expenses:Restaurant                  1880 MXN
  Assets:Cash:Pesos                   -1400 MXN
  Assets:Cash:Martin                 -40.00 USD @ 12.00 MXN
```

我使用餐厅提供的美元兑换的不利汇率（当时市场汇率为 14.5）。

#### 个人费用

这是一个使用共享资金记录个人费用的例子。为了让我们能进行礁石潜水，我们必须每天向岛上支付 2.50 美元的“海洋公园”费用。这是一个短暂的旅行，我在那儿潜水了三天，Caroline 的费用包含在她的课程中，只需要支付一天：

```plaintext
2015-02-25 * "Marine Park (3 days Martin, 1 day Caroline)"
  Expenses:Scuba:ParkFees:Martin             7.50 USD
  Expenses:Scuba:ParkFees:Caroline           2.50 USD
  Assets:Cash:Martin
```

只需将这些费用记录到含有我们名字的费用账户中，最后将它们分开。

这是一个更复杂的例子：Scuba Club Cozumel 的潜水店在我们离开时向我们收取了一张包含所有租赁装备和额外潜水费用的账单。我在这里只是将详细的账单翻译成一个单独的交易，分别记录：

```plaintext
2015-03-01 * "Scuba Club Cozumel" | "Dive shop bill" ^69b409189b37
  Income:Martin:CreditCard           -381.64 USD
  Expenses:Scuba:Martin                        27 USD ;; Regulator w/ Gauge
  Expenses:Scuba:Caroline                       9 USD ;; Regulator w/ Gauge
  Expenses:Scuba:Martin                        27 USD ;; BCD
  Expenses:Scuba:Caroline                       9 USD ;; BCD
  Expenses:Scuba:Martin                         6 USD ;; Fins
  Expenses:Scuba:Martin                        24 USD ;; Wetsuit
  Expenses:Scuba:Caroline                       8 USD ;; Wetsuit
  Expenses:Scuba:Caroline                       9 USD ;; Dive computer
  Expenses:Scuba:Martin                         5 USD ;; U/W Light
  Expenses:Scuba:Caroline                      70 USD ;; Dive trip (2 tank)
  Expenses:Scuba:Martin                        45 USD ;; Wreck Dive w/ Lite
  Expenses:Scuba:Martin                        45 USD ;; Afternoon dive
  Expenses:Scuba:Caroline                      45 USD ;; Afternoon dive
  Expenses:Scuba:Martin                     28.64 USD ;; Taxes
  Expenses:Scuba:Caroline                   24.00 USD ;; Taxes
```

#### 最终余额

当然，您可以在旅行期间的任何时候使用余额断言。例如，离开坎昆机场前，我们知道我们不会再花费墨西哥比索，所以我数了数在 Caroline 决定在机场商店挥霍剩余的钱买昂贵的巧克力后我们剩下的钱：

```plaintext
2015-03-04 balance Assets:Cash:Pesos               65 MXN
```

理想情况下，簿记员应在每天或每隔几天的安静时刻做这件事，这样可以更容易地确认您可能忘记记录的费用（毕竟我们在度假，心情放松时容易忘记记录某些事情）。

#### 清零资产账户

在旅行结束时，您应通过将剩余资金转移到参与者（即，收入账户）来清零所有资产和负债账户的最终余额。这应使旅行期间的所有账户余额归零，并确保所有用于旅行费用的现金已转移给旅行者。

顺便说一下，谁保留这些钱并不重要，因为最终我们将进行一次校正转账，计算出创建均衡分摊所需的余额。您可以将其转移给任何人；最终结果将是相同的。

我们的清零交易如下所示：

```plaintext
2015-03-06 * "Final transfer to clear internal balances to external ones"
  Assets:Cash:Pesos                               -65 MXN
  Income:Martin:Cash:Foreign                       60 MXN
  Income:Caroline:Cash                              5 MXN
  Assets:Cash:Martin                             -330 USD
  Income:Martin:Cash                              330 USD
  Assets:Cash:Caroline                           -140 USD
  Income:Caroline:Cash                            140 USD

2015-03-07 balance Assets:Cash:Pesos                0 MXN
2015-03-07 balance Assets:Cash:Pesos                0 USD
2015-03-07 balance Assets:Cash:Martin               0 USD
2015-03-07 balance Assets:Cash:Caroline             0 USD
```

我们有三张20比索的钞票，我保留了这些钞票以备将来旅行使用。Caroline 保留了 5 比索的硬币（忘记作为小费给了）。我们转移了旅行期间携带的现金金额。

## 如何记录笔记

在这次旅行中，我没有带笔记本电脑——毕竟这是度假。我喜欢断开连接。我也没有带笔记本。相反，我只是在酒店晚上花几分钟记下笔记。这个过程每晚花费约 5-10 分钟，只是从记忆中回想并写下事情。

这些笔记如下所示：

每行都有：

- 交易的叙述（一个描述，便于我稍后选择费用账户）
- 谁支付的（钱来自哪个资产或收入账户）
- 金额（以美元或比索）

旅行结束后，我坐在电脑前输入对应的 Beancount 文件。如果我在度假期间带着电脑，我可能会随时输入。当然，由于错误，我不得不在这里那里做一些调整。

底线是：如果您组织得好，做这件事的开销是最小的。

## 对账费用

在旅行文件上运行 bean-web：

```bash
bean-web beancount/examples/sharing/cozumel2015.beancount
```

您可以在“所有交易”视图中查看余额（点击“所有交易”）。

资产负债表应显示资产账户的空余额：

资产账户的余额应反映旅行期间的总货币转换金额。您可以通过计算金额加权平均汇率来验证这一点：7539.00 / 559.88 ~= 13.465 USD/MXN（这是正确的）。

### 审查贡献

损益表应显示项目的所有费用和贡献摘要：

收入账户余额显示每个人的总贡献金额。请注意，在创建收入账户时，我特意为每个支付来源创建了一些特定账户，如“Caroline 的信用卡”等。

从这个视图中，我们可以看到为这次旅行贡献了总计 4254.28 美元（手头还剩下 65 比索）。考虑到货币兑换，费用方应匹配：3694.40 + 7474 / 13.465 ~= 4249 美元，差异很小（可以通过不同的货币兑换率来解释）。

如果要查看贡献支付列表和最终余额，请点击特定旅行者的根账户，例如“Income:Caroline”（点击“Caroline”），将带您进入该根账户的日记账：

此日记账包含子账户中的所有交易。底部的最终值应显示这些账户的总余额，即 Caroline 为这次旅行贡献的金额：415 美元，并保留 5 比索（硬币）。我们可以对 Martin 执行相同操作，找到最终余额为 3839.28 美元，并保留 60 比索（钞票）。

您也可以使用 bean-query 获取相同的金额：

```bash
~/p/.../examples/sharing$ bean-query cozumel2015.beancount

Input file: "Mexico Trip: Cozumel & Tulum SCUBA Diving"
Ready with 105 directives (160 postings in 45 transactions).

beancount> SELECT sum(position) WHERE account ~ '^Income:.*Caroline'

sum_positio
-----------
-415.00 USD
   5    MXN
```

### 分摊费用

损益表费用方显示费用细目。请注意，其中一些费用账户明确记录到各成员，账户名中有他们的名字，例如“`Expenses:Scuba:Martin`”。其他账户，例如“`Expenses:Groceries`”是打算分摊的。

我们将通过添加一个插件来实现这一点，该插件将转换所有交易，*实际*分摊打算共享的分录，即不包含成员名字的分录。例如，以下输入交易：

```plaintext
2015-02-28 * "Scuba Club Cozumel" | "Room bill"
  Income:Martin:CreditCard         -1380.40 USD
  Expenses:Scuba:Martin              178.50 USD
  Expenses:Accommodation            1201.90 USD
```

插件将其转换为：

```plaintext
2015-02-28 * "Scuba Club Cozumel" | "Room bill"
  Income:Martin:CreditCard         -1380.40 USD
  Expenses:Scuba:Martin              178.50 USD
  Expenses:Accommodation:Martin      600.95 USD
  Expenses:Accommodation:Caroline    600.95 USD
```

注意：

- 只有费用账户会分摊为多个分录。
- 名称中包含成员名字的账户按惯例已被分配给该成员。为了实现这一点，插件需要提供成员的名字。
- 所有结果收入和费用账户都包含成员名字。

插件启用如下：

```plaintext
plugin "beancount.plugins.split_expenses" "Martin Caroline"
```

从输入文件中取消注释这行代码后重新加载网页，并访问损益表页面，应显示按人分摊的费用账户的长列表。现在，为了生成每个人产生的总费用，我们必须生成所有费用账户的余额，这些账户名中包含成员名字，例如“`Expenses:.*Martin`”。

web 工具目前不提供这种过滤功能，但我们可以使用 bean-query 工具生成每个人的费用总额：

```plaintext
beancount> SELECT sum(position) WHERE account ~ '^Expenses:.*Martin'

sum_positio
-----------
2007.43 USD
3837.0  MXN

beancount> SELECT sum(position) WHERE account '^Expenses:.*Caroline'

sum_positio
-----------
1686.97 USD
3637.0  MXN
```

这意味着“Martin 产生了 2007.43 美元和 3837.0 比索的费用。”

您可以手动将其转换为美元金额：

```bash
yuzu:~$ dc -e '2007.43 3837.0 13.465 / +p'
2291.43 
```

或者您可以使用最近引入的 bean-query 的“`CONVERT`”函数：

```plaintext
beancount> SELECT convert(sum(position), 'USD') WHERE account ~ '^Expenses:.*Martin'

     convert_sum_position_c_     
---------------------------------
2288.528901098901098901098901 USD

beancount> SELECT convert(sum(position), 'USD') WHERE account ~ '^Expenses:.*Caroline'

     convert_sum_position_c_     
---------------------------------
1953.416886446886446886446886 USD
```

（2291.43 和 2288.53 金额之间的差异可归因于使用了略有不同的兑换率。）

类似地，您可以生成每个人的支付列表，例如：

```plaintext
beancount> SELECT sum(position) WHERE account ~ '^Income:.*Caroline'
```

## 最终转账

为了计算每个成员应付的总金额，过程类似：只需汇总分配给特定成员的所有账户的余额：

```plaintext
beancount> SELECT convert(sum(position), 'USD') WHERE account ~ 'Caroline'

     convert_sum_position_c_     
---------------------------------
1538.783186813186813186813187 USD

beancount> SELECT convert(sum(position), 'USD') WHERE account ~ 'Martin'

     convert_sum_position_c_      
----------------------------------
-1546.355494505494505494505494 USD
```

请注意，这包括该人的收入和费用账户。就像我们将两个单独的账簿合并为一个一样。（再次，少量差异可以归因于不同时间的汇率变化。）

我们现在可以进行最终转账，以便核算每个人的费用；我们同意将其舍入为 1500 美元：

```plaintext
2015-03-06 * "Final transfer from Caroline to Martin to pay for difference"
  Income:Caroline:Wallet                    -1500 USD
  Income:Martin:Wallet                       1500 USD
```

如果从输入文件中取消注释此交易（靠近末尾），您会发现修正后的余额：

```plaintext
beancount> SELECT convert(sum(position), 'USD') WHERE account ~ 'Martin'

     convert_sum_position_c_     
---------------------------------
-46.3554945054945054945054945 USD

beancount> SELECT convert(sum(position), 'USD') WHERE account ~ 'Caroline'

    convert_sum_position_c_     
--------------------------------
38.7831868131868131868131868 USD
```

## 更新您的个人账簿

太好了！现在我们已经为旅行核对了彼此的费用。但是您仍需在个人账簿中反映这些费用（如果有的话）。在本节中，我将解释如何在该文件中反映这些交易。

### 账户名称

首先，请注意，个人账簿中的账户名称不必与旅行账簿文件中使用的账户名称匹配。我从不一起处理这些文件。（不这样做的一个很好的理由是，每次旅行将包括其他人的账户名称，我不希望这些泄露到我的主要账簿文件中。）

### 使用标签

我喜欢使用标签报告与旅行相关的所有交易。在本例中，我使用了标签`#trip-cozumel2015`。

### 记录贡献

您对项目的贡献应记录到一个临时持有账户。我将其称为“`Assets:Travel:Pending`”。例如，旅行文件中的这个交易：

```plaintext
2015-02-25 * "Yellow Transfers" | "SuperShuttle to Playa del Carmen"
  Expenses:Transport:Bus                 656 MXN
  Income:Martin:CreditCard            -44.12 USD @ 14.86854 MXN
```

最终将被导入我的个人文件，并分类如下：

```plaintext
2015-02-25 * "YELLOW TRANSFER MX  CO" #trip-cozumel2015
  Liabilities:US:BofA:CreditCard      -44.12 USD
  Assets:Travel:Pending                44.12 USD
```

您可以将其概念化为向一个待定的现金池贡献资金，您最终将从中收到费用。

同样，我为旅行带的现金：

```plaintext
2015-02-24 * "Taking cash with me on the trip"
  Assets:Cash                             -1200.00 USD
  Assets:Travel:Pending                    1200.00 USD
```

请注意，所有与旅行相关的交易都应记录到那个账户，包括跨账户转账：

```plaintext
2015-03-06 * "Final transfer from Mexico trip" #trip-cozumel2015
  Assets:Google:Wallet                       1500 USD
  Assets:Travel:Pending                      -1500 USD
```

### 记录费用

旅行结束后，我们希望将待定账户中的余额转换为费用清单。要找到自己的费用清单，您可以发出类似这样的查询：

```plaintext
beancount> SELECT account, sum(position) WHERE account ~ '^Expenses:.*:Martin' 
           GROUP BY 1 ORDER BY 1

            account             sum_positio
------------------------------- -----------
Expenses:Accommodation:Martin    735.45 USD
Expenses:Alcohol:Martin          483    MXN
Expenses:Bicycles:Martin          69.5  MXN
Expenses:Flights:Martin          488.00 USD
Expenses:Groceries:Martin        197.0  MXN
Expenses:Museum:Martin            64    MXN
Expenses:Restaurant:Martin        22.28 USD
                                  1795.5  MXN
Expenses:Scuba:Martin            506.14 USD
Expenses:Scuba:ParkFees:Martin     7.50 USD
Expenses:Tips:Martin             225    MXN
                                 189.16 USD
Expenses:Transport:Bus:Martin    709    MXN
Expenses:Transport:Taxi:Martin    53.90 USD
                                 294    MXN
Expenses:Transport:Train:Martin    5.00 USD
```

我想您可以编写脚本以去掉账户名称中的成员名称，并让脚本输出已格式化为交易形式的余额，但由于账户名称不会与个人账簿文件一一对应，您仍需手动转换，所以我没有自动化这个过程。此外，您在旅行结束后只需进行一次这个操作，因此不值得花太多时间使这部分更简单[^2]。

以下是最终交易的样子；我的输入文件中有一节：

```plaintext
2015-02-25 event "location" "Cozumel, Mexico"    ;; (1)
pushtag #trip-mexico-cozumel-2015

… other transactions related to the trip… 

2015-03-01 event "location" "Tulum, Mexico"      ;; (1)

2015-03-07 * "Final reconciliation - Booking pending expenses for trip"
  Expenses:Travel:Accommodation         735.45 USD
  Expenses:Food:Alcohol                  483.0 MXN
  Expenses:Sports:Velo                    69.5 MXN
  Expenses:Transportation:Flights       488.00 USD
  Expenses:Food:Grocery                  197.0 MXN
  Expenses:Fun:Museum                     64.0 MXN
  Expenses:Food:Restaurant               22.28 USD
  Expenses:Food:Restaurant              1795.5 MXN
  Expenses:Scuba:Dives                  506.14 USD
  Expenses:Scuba:Fees                     7.50 USD
  Expenses:Scuba:Tips                    225.0 MXN
  Expenses:Scuba:Tips                   189.16 USD
  Expenses:Transportation:Bus            709.0 MXN
  Expenses:Transportation:Taxi           53.90 USD
  Expenses:Transportation:Taxi           294.0 MXN
  Expenses:Transportation:Train           5.00 USD
  Assets:Cash:Foreign                     60.0 MXN                ;; (2)
  Assets:Cash                           330.00 USD                ;; (3)
  Assets:Travel:Pending               -2337.43 USD                ;; (4)
  Assets:Travel:Pending                -288.67 USD @@ 3897.0 MXN  ;; (5)

2015-03-07 * "Writing off difference as gift to Caroline"     ;; (6)
  Assets:Travel:Pending                     -43.18 USD
  Expenses:Gifts

2015-03-14 balance Assets:Travel:Pending     0 USD   ;; (7)
2015-03-14 balance Assets:Travel:Pending     0 MXN

poptag #trip-mexico-cozumel-2015
```

观察：

1. 我使用了“事件”指令来跟踪我的位置。我这样做是因为我最终需要它用于移民目的（以及为了好玩，跟踪我在各地的天数）。
2. 我将我保留的额外现金转移到“外国现金口袋”账户：`Assets:Cash:Foreign`
3. 我“转移”了旅行后我带回的现金。
4. 这一步将美元费用从待定账户中移除。
5. 这一步将比索费用从待定账户中移除。我手动计算了金额（3897 / 13.5 ~= 288.67 USD），使用全额作为兑换率。
6. 我想在旅行结束后完全清零待定账户，因此我们同意不支付的超额金额。
7. 最后，我断言待定账户中没有美元和比索。

我通过运行 `bean-check` 或使用 `bean-doctor context` 在 Emacs 上进行不完整交易找到了缺失金额。

## 生成报告

如果您想为旅行中的每个参与者自动生成报告，有一个脚本可生成文本（以及最终的 CSV）报告，用于前述部分提到的查询。您可以使用此脚本在旅行或项目期间或之后提供费用状态。

该脚本位于 `split_expenses` 插件本身中，您可以这样调用它：

```bash
python3 -m beancount.plugins.split_expenses <beancount-filename> --text=<dir>
```

对于每个人，它将生成以下报告：

- 费用详细日记账
- 贡献详细日记账
- 每个类别的费用细分（账户类型）

最后，它还将生成每个人的最终余额，您可以使用它们发送最终的对账转账给彼此。现在尝试在提供的[示例文件](https://raw.githubusercontent.com/beancount/beancount/v2/examples/sharing/)之一上运行它。

## 其他示例

还有另一个示例文件，显示如何在三名参与者之间共享费用：[duxbury2015.beancount](https://raw.githubusercontent.com/beancount/beancount/v2/examples/sharing/duxbury2015.beancount)。随着时间的推移，将引入更多的示例文件。

## 结论

执行本文档中描述任务的方法不止一种。然而，我们展示的方法很好地扩展到更大的人群，允许我们处理某些费用在群体中属于个人的情况，并最终允许参与者之间的非均等分摊。这是非常通用的。

这也是记账一个长期项目的简短步骤。本文档中介绍的想法为复式记账方法在简单设置中的使用提供了一个很好的用例。希望通过这个用例帮助人们开发使用复式记账方法所需的直觉。

[^1]: 随着我们的 SQL-like 查询语言的发展，bean-web 最终将允许用户从表达式过滤的交易集中创建视图。这将会实现。
[^2]: 请注意，我们可以为 bean-query 工具实现一种特殊的“beancount”支持格式，以写出事务形式的余额。我不确定这有多大用处，但这是一个值得考虑的想法。
