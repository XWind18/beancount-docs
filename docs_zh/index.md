# Beancount 用户手册<a id="title"></a>

*这是与Beancount相关的所有文档的顶级页面。*

[<u>http://furius.ca/beancount/doc/index</u>](http://furius.ca/beancount/doc/index)

## 用户文档<a id="documentation-for-users"></a>

[<u>上下文中的命令行记账</u>](command_line_accounting_in_context.md)：这是一个让你了解什么是命令行记账，为什么我要使用它以及我如何使用它的简单示例。阅读它作为入门介绍。

[<u>双入账计数方法</u>](the_double_entry_counting_method.md)：使用与命令行记账系统相兼容的假设和简化术语对双入账方法进行简单介绍。

[<u>安装Beancount</u>](installing_beancount.md)：下载和安装Beancount的说明，以及软件发布信息。

[<u>运行Beancount并生成报告</u>](running_beancount_and_generating_reports.md)：如何运行Beancount可执行文件并生成报告。这包含了对Beancount用户的技术信息。

[<u>入门指南</u>](getting_started_with_beancount.md)：解释如何创建、初始化和组织输入文件，以及声明账户和添加填充指令的基础知识。

[<u>Beancount语言语法</u>](beancount_language_syntax.md)：包含示例的语言语法的详细描述。这是Beancount语言的主要参考资料。

[<u>Beancount选项参考</u>](beancount_options_reference.md)：对所有可能选项值的描述和解释。

[<u>精度和容差</u>](precision_tolerances.md)：余额断言和交易容限适应上下文推断的误差金额。本文介绍了其工作原理。

[<u>Beancount查询语言</u>](beancount_query_language.md)：对bean-query命令行客户端工具的高级概述，该工具允许从Beancount分类账文件中提取各种数据表。

[<u>Beancount速查表</u>](beancount_cheat_sheet.md)：一张总结Beancount语法的单页“速查表”。

[<u>库存工作原理</u>](how_inventories_work.md)：解释库存及其表示形式以及如何将持有的仓位与库存中的仓位匹配的详细信息。这是交易文档的预备阅读材料。

[<u>导出您的投资组合</u>](exporting_your_portfolio.md)：如何将投资组合导出到外部投资组合跟踪网站。

[<u>教程和示例</u>](tutorial_example.md)：一个包含了几年一个虚拟Beancount用户的财务生活的现实例子记账文件。这可以用来了解Beancount的功能，看看报告或其Web界面是什么样子的。这可以让您快速了解使用Beancount可以得到什么。

[<u>Beancount邮件列表</u>](https://groups.google.com/forum/#!forum/beancount)：与Beancount相关的问题和讨论在这里进行。与此相关的还有[<u>Ledger-CLI论坛</u>](https://groups.google.com/forum/#!forum/ledger-cli)，其中包含了许多更一般的讨论，同时也充当了一个命令行记账主题的元列表。

[<u>Beancount历史与感谢</u>](beancount_history_and_credits.md)：对Beancount的开发历史的描述，并向其贡献者表达感谢。

[<u>Beancount和Ledger、HLedger的比较</u>](a_comparison_of_beancount_and_ledger_hledger.md)：CLI记账系统之间的定性特性比较。

[<u>在Beancount中获取价格</u>](fetching_prices_in_beancount.md)：如何使用Beancount的工具获取和维护您的资产的价格数据库。

[<u>导入外部数据</u>](importing_external_data.md)：描述从可以从金融机构下载的外部文件导入数据的过程，以及Beancount提供的用于自动化部分操作的工具和库。

## Cookbook和示例<a id="cookbook-examples"></a>

*这些文档是使用双入账方法执行特定任务的示例。使用这些文档来形成关于如何组织您的账户的直觉。*

[<u>命令行记账食谱</u>](command_line_accounting_cookbook.md)：多种不同类型的财务交易的记录示例。这无疑是了解如何最好地使用双入账方法的最佳方式。

[<u>使用Beancount进行交易</u>](trading_with_beancount.md)：解释交易盈亏（P/L）以及如何使用Beancount处理各种投资和交易情况的示例。这是对食谱的补充。

[<u>在Beancount中进行股票发放</u>](stock_vesting_in_beancount.md)：展示如何处理技术公司和初创公司常见的受限制的股票单位和发放事件的示例。

[<u>与Beancount共享费用</u>](sharing_expenses_with_beancount.md)：使用双入账方法实现与他人分享旅行或项目费用的真实而详细的示例。

[<u>我们如何分享费用</u>](how_we_share_expenses.md)：关于连续系统的更详细介绍，用于（a）在双方都有贡献时分担合伙人间的费用，以及（b）为拥有自己的账户的具体项目（我们的儿子）共同分担费用。这描述了我们的实际系统。

[<u>医疗支出</u>](health_care_expenses.md)（不完整）：一个不完全完成的文档，解释了在美国的网络和非网络提供者之间处理医疗支出的顺序。

[<u>跟踪医疗索赔</u>](tracking_medical_claims.md)：一个结构示例，跟踪与自费支付的网络外心理治疗会议，医疗索赔以及相关的HSA偿还有关的费用。

[<u>计算投资组合回报</u>](calculating_portolio_returns.md)：如何从Beancount分类账中计算投资组合回报。这描述了在一个实验性脚本中完成的工作以及为获取正确数据而涉及的过程。

## 开发人员文档<a id="documentation-for-developers"></a>

[<u>Beancount脚本和插件</u>](beancount_scripting_plugins.md)：编写脚本以内存中加载分类账内容并处理指令的指南，以及编写转换指令的插件的方法。

[<u>Beancount设计文档</u>](beancount_design_doc.md)：程序架构和设计选择的信息，代码约定，不变性和方法。如果您想更深入地了解，请阅读。

[<u>LedgerHub设计文档</u>](ledgerhub_design_doc.md)：导入工具和库的设计和架构，这些工具和库在[<u>beancount.ingest</u>](http://github.com/beancount/beancount/tree/master/beancount/ingest/)中实现。这曾经是一个名为LedgerHub（现已停用）的独立项目，其有用的部分最终被合并到Beancount中。这是一个有些过时的原始设计文档。

[<u>源代码</u>](https://github.com/beancount/beancount/)：Beancount源代码的官方存储库自2020年5月起位于[<u>Github</u>](http://github.com/beancount/beancount/)。（2008年至2020年5月之前，它托管在Bitbucket上）。

[<u>外部贡献</u>](external_contributions.md)：其他人基于Beancount库制作和共享的插件、进口商和其他代码的列表。

## 增强提案和讨论<a id="enhancement-proposals-discussions"></a>

*我有时会撰写增强提案，以便在继续进行前组织和总结我的思考，并征求其他用户的反馈意见。这是找出我计划添加的功能、详细了解事物如何工作或与其他类似软件（如Ledger或HLedger）进行比较的好素材。*

[<u>有关库存预订改进的提案</u>](a_proposal_for_an_improvement_on_inventory_booking.md)：在提案中，我概述了关于库存预订在Ledger和Beancount中的当前状况以及执行平均成本预订并计算不含佣金的资本增值的更好方法。

[<u>Beancount中的清算日期</u>](settlement_dates_in_beancount.md)：讨论清算或辅助日期如何在Beancount中使用以及它们在Ledger中的使用方法。

[<u>Beancount中的余额断言</u>](balance_assertions_in_beancount.md)：总结Beancount和Ledger中进行余额断言的不同语义，并提议扩展Beancount的语法以支持更复杂的断言。

[<u>在Beancount中进行基金会计</u>](fund_accounting_with_beancount.md)：讨论基金会计以及如何最好地使用Beancount在单个分类账中跟踪多个基金的方法。

[<u>Beancount中的舍入和精度</u>](rounding_precision_in_beancount.md)：讨论余额检查的重要舍入和精度问题，以及推导所需精度的更好方法的提案。（[<u>已实施</u>](precision_tolerances.md)）

## 外部链接<a id="external-links"></a>

*其他作者编写的关于Beancount的文档、链接、博客文章、论文等。*

[<u>Beancount源代码文档</u>](http://aumayr.github.io/beancount-docs-static/)（Dominik Aumayr）：Beancount代码库的Sphinx生成的源代码文档。生成此文档的代码位于[<u>此处</u>](https://github.com/aumayr/beancount-docs)。

[<u>Beancount ou la comptabilité pour les hackers</u>](http://blog.deguet.fr/beancount-comptabilite-pour-hackers/)（Cyril Deguet）：概述博客文章（法语）。

## V3文档<a id="v3-documentation"></a>

*自2020年夏季以来，正在进行向C++的重写。这些是与此相关的文档。*

[<u>Beancount V3</u>](beancount_v3.md)：目标与设计：对将Beancount重写为C++版本的动机、目标和设计的说明。

[<u>Beancount V3：与V2的改变</u>](https://docs.google.com/document/d/1Ia4zYmkB6I6IbWPRlcZYYuMS1ZI55T99dp9LiMJqXCE/)：一个当前在v3（“master”分支上）已完成但尚未列入v2文档集的更改列表。

[<u>安装Beancount（v3）</u>](installing_beancount_v3.md)：v3的安装过程。

[<u>Beancount V3：依赖项</u>](beancount_v3_dependencies.md)：构建第3版本所需的软件依赖项。

[<u>Beangulp</u>](beangulp.md)：v3的更新摄入框架。

## 关于本文档<a id="about-this-documentation"></a>

您可能已经注意到，我在使用[<u>Google Docs</u>](https://docs.google.com/)。我意识到这对于一个开源项目来说是不寻常的。如果您[<u>对此感到不满</u>](https://groups.google.com/d/msg/ledger-cli/u648SA1o-Ek/yom_P38FCAAJ)，那么需要知道的是，我也喜欢文本格式：在研究生院我广泛使用了[<u>LaTeX</u>](http://www.latex-project.org/)，在过去十年中我非常喜欢[<u>reStructuredText</u>](http://docutils.sourceforge.net)格式，甚至为其编写了Emacs支持。但是在2013年发生了一些事情：[<u>Google Docs</u>](https://docs.google.com/)变得足够好，可以编写可靠的技术文档，我开始广泛使用其修订和评论功能：我沉迷其中，我喜欢它。对用户来说，能够提出改进意见或在上下文中发表评论是一个非常有用的功能，它比邮件列表上的补丁或评论更能提高我的写作质量。它还为用户提供了指出需要改进的段落的机会。与我的其他项目相比，我已经收到了数量级更多的文档反馈；它很管用。

它看起来也非常*好*，这有助于激励我写得更多、写得更好。我想也许我又爱上所见即所得……别误会：LaTeX和无标记格式是很棒的，但世界在改变，对我来说，比起在我的存储库中慢慢变老的臃肿的TeX文件，拥有社区合作和动态文档更重要。我的目标是生成易于阅读、打印良好的高质量文本，尤其是现在，可以在移动设备或平板电脑上良好呈现，这样您就可以随时随地阅读。我也可以在外出时对其进行编辑，这对于记录想法或进行更正是很有趣的。最后，Google Docs还提供了一个API，可以访问文档，因此如果需要，我最终可以将所有元数据下载并转换为其他格式。并不是我喜欢其中的一切，但它提供的功能我确实很喜欢。

如果您要留下评论，我将不胜感激，如果您在阅读时可以**登录您的Google账户**，这样您的名字就会显示在您的评论旁边。谢谢！

- [Martin Blais](mailto:blais@furius.ca)
