# Beancount 历史与致谢<a id="title"></a>

[<u>Martin Blais</u>](http://plus.google.com/+MartinBlais)，2014年7月

[<u>http://furius.ca/beancount/doc/history</u>](http://furius.ca/beancount/doc/history)

*本文记录了 Beancount 的发展历程及对贡献者的致谢。*

## Beancount 的历史<a id="history-of-beancount"></a>

John Wiegley 的 Ledger 激发了 Beancount 第一个版本的灵感。许多原始创意来源于 Ledger。当我第一次了解复式簿记并意识到它可以完美解决我公司计数各种分期付款的问题时，在对当时可用的解决方案（包括 GnuCash，它很容易被我搞崩）感到失望后，我很快找到了 Ledger 的网站。在那里，John 描述了他对基于文本系统的愿景，特别是去除借方和贷方仅使用符号的想法，以及一种方便的输入语法，允许省略一个过账的金额。我非常兴奋，并与他进行了多次关于 Ledger 的热烈讨论，以及我希望如何使用它。我们之间有一些想法的交流，John 对添加新功能的提议非常开放。

我对簿记充满了浓厚的兴趣，开始编写一个 Ledger 的 Python 接口。但最终，我发现自己用 Python 重写了整个系统——不是因为不喜欢 Ledger，而是因为它足够简单，我可以在短时间内完成大部分工作，并立即添加一些我认为有用的功能。这样做的一个原因是，我希望解析一次输入文件，然后通过网页请求从内存中的事务数据库中提供各种报告，因此我不需要处理速度，因此不需要为了性能使用 C++，我选择使用动态语言，使我可以快速添加许多功能。这成了 Beancount 版本 1，它独立存在并发展出自己的一套实验功能。

我的梦想是能够快速轻松地操作这些事务对象，以获得各种数据视图和细分。我认为第一个实现并没有推向极限；老实说，代码很差——我写得很快——修改系统变得很尴尬。特别是我最初实现资本收益的方式不够优雅，涉及一些手动计数。我对这点不满意，但它能工作。为了保持与 Ledger 语法的合理兼容性，我还使用了丑陋的特设解析器代码——我认为能够共享一些通用输入语法并使用任一系统进行验证，甚至可能在它们之间转换会很有趣——这使得我对修改语法以发展新功能感到犹豫，所以它稳定了几年，我对添加新功能失去了兴趣。

但它是正确的并且大部分工作正常，所以从 2008 年到 2012 年，我一直使用该系统来管理我的个人财务、公司的财务和与妻子的共同财产，这非常棒。在此期间，我学到了很多关于如何记账的知识（烹饪书文档旨在帮助您做到同样的事情）。2013 年夏天，我突然意识到如何正确且可泛化地实现资本收益，如何合并按成本持有的头寸和常规头寸的跟踪，以及一套简单的规则来合理地执行这些操作（库存工作的设计）。我还看到了更好的方法来分解内部数据结构，并决定打破与 Ledger 兼容性的限制，重新设计输入语法，使用 lex/yacc 生成器解析输入语言，这使我能够轻松地演进输入语法，而不必处理解析问题，并更容易地创建其他语言的端口。在这个过程中，出现了一种更简单和更一致的语法，在几次紧张的周末中，我重新从头开始重新实现整个系统，甚至没有看前一个版本，干净地重新实现。Beancount 版本 2 诞生了，比上一个版本更好，模块化，并且易于通过插件扩展。

结果是我认为优雅的设计，涉及一小组对象，这一设计可以很容易地成为其他计算机语言重新实现的基础。对于那些有兴趣尝试的人，伴随的设计文档中描述了这一点（这将受到欢迎，我期待这会发生）。虽然 Ledger 仍然是一个充满想法的有趣项目，表达簿记问题，但 Beancount 的第二个版本提出了一种更简单的设计，省略了不严格必要的功能，并通过一个简单的网络界面和非常少的命令行选项，旨在实现最大可用性。由于我有超过 5 年的实际使用经验，我为自己设定了一个目标，即去除所有我认为实际上没用并引入不必要复杂性的功能（如虚拟交易，允许不属于五种类型的账户等），并尽可能简化系统而不影响其可用性。结果是语言和软件更容易使用和理解，生成的数据结构更容易使用，处理更具“功能性”，Beancount 的内部非常模块化。我转换了所有 6 年的输入数据——感谢一些非常简单的 Python 脚本来操作文件——并开始专门使用新版本。它现在处于完全功能状态。

Ledger 的语法实现了许多触发大量隐含行为的功能；我发现这令人困惑，其中一些原因在许多改进提案中有记录。我不喜欢命令行选项。相比之下，Beancount 的设计提供了较少表达能力的低级语法，但它更接近生成的内存数据结构，希望在这方面更加明确。我认为这两个项目都有优点和缺点。尽管它很受欢迎，但在我看来，最新版本的 Ledger 仍有许多缺点。特别是，头寸的减少不是在发生时记账，而是似乎只是累积，并且仅在显示时使用不同的命令行选项进行匹配。因此，在 Ledger 中，可能会同时在一个账户中持有同一商品的正负批次。我认为这可能导致混淆甚至[<u>错误的会计</u>](https://groups.google.com/d/msg/ledger-cli/aQvbjTZa7HE/iMisMBkaI6UJ)交易批次。我认为 Ledger 仿真社区仍在解决这个问题，最终会趋同于我所采取的解决方案，或者甚至找到更好的解决方案。

在重新设计中，我分离出了用于导入的配置指令，并通过多次迭代，最终找到了一种优雅的方法，主要自动化导入并自动检测各种输入文件并将它们转换为我的输入语法。结果是 [<u>LedgerHub 设计文档</u>](ledgerhub_design_doc.md)。LedgerHub 目前已经实现，我专门使用它，但由于测试不足，我不能发布公共版本\[2014 年 7 月\]。欢迎您自行尝试。

2014 年 6 月和 7 月，我决定将七年来关于命令行会计的思考记录在一系列 Google 文档中，这些文档现在构成了 Beancount 当前文档的基础。我希望从这里逐步演进它。

## 编年史<a id="chronology"></a>

- Ledger 始于 2003 年 8 月  
  [<u>http://web.archive.org/web/\*/http://www.newartisans.com/ledger/</u>](http://web.archive.org/web/*/http://www.newartisans.com/ledger/)

- Beancount 始于 2008 年  
  [<u>http://web.archive.org/web/\*/furius.ca/beancount</u>](http://web.archive.org/web/*/furius.ca/beancount)

- HLedger 也始于 2008 年  
  [<u>https://github.com/simonmichael/hledger/graphs/contributors?from=2007-01-21&to=2014-09-08&type=c</u>](https://github.com/simonmichael/hledger/graphs/contributors?from=2007-01-21&to=2014-09-08&type=c)

## 致谢<a id="credits"></a>

到目前为止，我一直在为 Beancount 贡献所有代码。一些用户以其他方式做出了重要贡献：

- Daniel Clemente 一直在报告他在使用 Beancount 管理公司和个人财务时遇到的所有问题。他的不懈坚持和对细节的关注帮助我专注于修复 Beancount 的一些粗糙角落，我自己知道如何避免这些问题。

- 在多年的督促之后，我的老朋友 [<u>Filippo Tampieri</u>](http://plus.google.com/+FilippoTampieri) 终于决定将他的交易历史转换为 Beancount 格式。他对我的文档进行了许多深思熟虑的评论，并正在努力添加各种评估资产回报的方法。
