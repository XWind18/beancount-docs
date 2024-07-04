# Beancount外部贡献<a id="title"></a>

[<u>Martin Blais</u>](mailto:blais@furius.ca) - 更新日期：2024年6月

[<u>http://furius.ca/beancount/doc/contrib</u>](http://furius.ca/beancount/doc/contrib)

链接到由其他人编写的代码，这些代码建立在Beancount和/或Ledgerhub之上或与之相关。

## 索引<a id="indexes"></a>

本文档仅包含已在邮件列表中讨论或宣布的包。您可以在公共索引中找到其他包：

-   PyPI：您可以在[<u>PyPI</u>](https://pypi.org/search/?q=beancount&o=)上找到许多与Beancount相关的项目。

-   GitHub：截至2020年9月，搜索“beancount”在[<u>GitHub</u>](https://github.com/search?p=5&q=beancount&ref=opensearch&type=Repositories)上可以找到318个项目。

## 书籍和文章<a id="books-and-articles"></a>

- [<u>使用Python管理个人财务</u>](https://personalfinancespython.com/) (Siddhant Goel)：一本关于纯文本会计和Beancount的2020年书籍。

- [<u>五分钟分类账更新</u>](https://reds-rants.netlify.app/personal-finance/the-five-minute-ledger-update/) (RedStreet)：一系列文章，展示了如何自动下载机构（银行、信用卡、经纪商等）的数据，从而在五分钟内完成分类账更新。[<u>邮件列表线程</u>](https://groups.google.com/g/beancount/c/_NclCTXaExs/m/EFjqkqElAQAJ)。

- [<u>使用Beancount进行税收损失收割</u>](https://reds-rants.netlify.app/personal-finance/tax-loss-harvesting-with-beancount/) (RedStreet)：一篇关于美国视角下税收损失收割的文章，包含要求、洗售细节和安全买卖日期，以及与机器人顾问的比较。（相关：[<u>fava_investor TLH模块</u>](https://github.com/redstreet/fava_investor/tree/main/fava_investor/modules/tlh)。适用于fava和纯Beancount命令行版本）。

- [<u>共同基金净值的缩放估计</u>](https://reds-rants.netlify.app/personal-finance/scaled-estimates-of-mutual-fund-navs/) (RedStreet)：问题：美国的共同基金净值（NAV）每天仅更新一次，在交易窗口仍然开放时（如进行税收损失收割时），有时需要在尚未提供当天最终NAV的情况下做出财务决策，这时候一个简单的NAV估计非常有用，特别是在市场变化巨大的一天。

- [<u>从Fidelity抓取交易历史的快捷方式</u>](https://noisysignal.com/trade_hist_shortcut/) (David Avraamides)：我写了一篇文章，描述了如何使用快捷方式从Fidelity网站抓取交易历史，通过Python脚本将其转换为Beancount的分类账格式，然后保存到剪贴板，以便粘贴到分类账文件中。

## 插件<a id="plugins"></a>

- [<u>split_transactions</u>](https://www.google.com/url?q=https%3A%2F%2Fgist.github.com%2Fkljohann%2Faebac3f0146680fd9aa5&sa=D&sntz=1&usg=AFQjCNGn2AkL35onTeXgOQzLzkjVpvLcpg)：Johann Klähn [<u>编写了一个插件</u>](https://groups.google.com/d/msg/beancount/z9sPboW4U3c/1qIIzro4zFoJ)，可以将单个交易分成多个交易，与一个过渡账户对冲，如用于折旧。

- [<u>zerosum</u>](https://github.com/redstreet/beancount_reds_plugins)：Red S [<u>编写了一个插件</u>](https://groups.google.com/d/msg/beancount/MU6KozsmqGQ/sehD3dqZslEJ)，用来匹配那些相加应该等于零的交易，并将它们移到单独的账户。

- [<u>effective_dates</u>](https://github.com/redstreet/beancount_reds_plugins)：Red S 编写了一个插件，用于将交易的不同部分记录在不同的日期。

- [<u>beancount-plugins</u>](https://github.com/davidastephens/beancount-plugins)：Dave Stephens 创建了一个仓库，用于分享与折旧相关的各种插件。

- [<u>beancount-plugins-zack</u>](https://github.com/zacchiro/beancount-plugins-zack)：Stefano Zacchiroli 创建了这个仓库来分享他的插件。包含指令排序等功能。

- [<u>beancount-oneliner</u>](https://github.com/Akuukis/beancount-oneliner)：Akuukis 创建了一个插件，可以用一行编写条目（[<u>PyPi</u>](https://pypi.python.org/pypi/beancount-oneliner/1.0.0)）。

- [<u>beancount-interpolate</u>](https://github.com/Akuukis/beancount-interpolate)：Akuukis 创建了Beancount插件，用于插值交易（重复、分割、折旧、摊销）（[<u>PyPi</u>](https://pypi.python.org/pypi/beancount-interpolate)）。

- [<u>metadata-spray</u>](https://github.com/seltzered/beancount-plugins-metadata-spray)：通过正则表达式而不是显式条目在条目中添加元数据（由Vivek Gani编写）。

- [<u>Akuukis/beancount_share</u>](https://github.com/Akuukis/beancount_share)：一个Beancount插件，用于在一个分类账中分配多个人之间的费用。这个插件非常强大，可能可以满足您所有的共享需求。

- [<u>w1ndy/beancount_balexpr</u>](https://github.com/w1ndy/beancount_balexpr) (Di Weng)：一个插件，提供“余额表达式”，可以在Beancount条目中运行，作为自定义指令。见[<u>这个线程</u>](https://groups.google.com/d/msgid/beancount/cdcf2cc7-8061-4f69-ae6a-c82564463652n%40googlegroups.com?utm_medium=email&utm_source=footer)。

- [<u>autobean.narration</u>](https://git.io/autobean.narration) (Archimedes Smith)：允许通过内联注释填充每个过账的元数据，简洁地注释每个过账。

- [<u>autobean.sorted</u>](https://github.com/SEIAROTg/autobean/)：检查每个文件中的交易是否按非降序排列。帮助识别错放或日期错误的指令，警告那些不按文件中日期非降序排列的指令。

- [<u>hoostus/beancount-asset-transfer-plugin</u>](https://github.com/hoostus/beancount-asset-transfer-plugin)：一个插件，自动生成两个Beancount账户之间的实物转账，同时保留成本基础和获取日期。

- [<u>PhracturedBlue/fava-portfolio-summary</u>](https://github.com/PhracturedBlue/fava-portfolio-summary) (Phractured Blue)：Fava插件，用于显示投资组合摘要和回报率。

- [<u>rename_accounts</u>](https://github.com/redstreet/beancount_reds_plugins)：Red S 的插件，用于重命名账户。例如：将“Expenses:Taxes”重命名为“Income:Taxes”，有助于费用分析。[<u>更多信息</u>](https://github.com/redstreet/beancount_reds_plugins/tree/main/beancount_reds_plugins/rename_accounts#readme)。

- [<u>Long_short capital gains classifier</u>](https://github.com/redstreet/beancount_reds_plugins/tree/master/beancount_reds_plugins/capital_gains_classifier)：Red S 的插件，根据持有资产的时间长短，将资本收益分类为长期和短期，并根据价值分类为收益和损失。

- [<u>Autoclose_tree</u>](https://github.com/redstreet/beancount_reds_plugins/tree/main/beancount_reds_plugins/autoclose_tree)：自动关闭账户时关闭该账户的所有子账户。

## 工具<a id="tools"></a>

- [<u>alfred-beancount</u>](https://github.com/blaulan/alfred-beancount) (Yue Wu)：一个“Alfred” macOS工具的插件，可以快速输入Beancount文件中的交易。支持完整的账户名称和收款人匹配。

- [<u>bean-add</u>](https://github.com/simon-v/bean-add) (Simon Volpert)：一个Beancount交易输入助手。

- [<u>hoostus/fincen_114</u>](https://github.com/hoostus/fincen_114) (Justus Pendleton)：一个FBAR / FinCEN 114报告生成器。

- [<u>ghislainbourgeois/beancount_portfolio_allocation</u>](https://github.com/ghislainbourgeois/beancount_portfolio_allocation) ([<u>Ghislain Bourgeois</u>](https://groups.google.com/d/msgid/beancount/b36d9b67-8496-4021-98ea-0470e5f09e4b%40googlegroups.com?utm_medium=email&utm_source=footer))：一种快速了解不同投资组合中的资产分配的方法。

- [<u>hoostus/portfolio-returns</u>](https://github.com/hoostus/portfolio-returns) (Justus Pendleton)：投资组合回报率计算器。

- [<u>costflow/syntax</u>](https://github.com/costflow/syntax) (Leplay Li)：一个允许用户从他们最喜欢的消息应用程序中进行纯文本会计的产品。将一行消息转换为Beancount/\*ledger格式的语法。

- [<u>process control chart</u>](https://github.com/hoostus/beancount-control-chart) (Justus Pendleton)：相对于投资组合规模的支出。[<u>线程</u>](https://groups.google.com/d/msgid/beancount/0cd47f9a-37d6-444e-8516-25e247a9e0cd%40googlegroups.com?utm_medium=email&utm_source=footer)。

- [<u>Pinto</u>](https://pypi.org/project/pinto/) (Sean Leavey)：一个增强版的Beancount命令行界面。支持在分类账文件中自动插入交易。

- [<u>PhracturedBlue/fava-encrypt</u>](https://github.com/PhracturedBlue/fava-encrypt)：一种基于Docker的解决方案，用于保持Fava在线，同时保持Beancount数据的静态加密。参见[<u>这个线程</u>](https://groups.google.com/d/msgid/beancount/ece6f424-a86b-4e6d-8ecc-4e05c8e74373n%40googlegroups.com?utm_medium=email&utm_source=footer)了解上下文。

- [<u>kubauk/beancount-import-gmail</u>](https://github.com/kubauk/beancount-import-gmail)：beancount-import-gmail使用Gmail API和OAuth登录您的邮箱并下载订单详细信息，这些详细信息用于增强您的交易以便于分类。

- [<u>sulemankm/budget_report</u>](https://github.com/sulemankm/budget_report)：一个非常简单的命令行预算跟踪工具，适用于Beancount分类账文件。

- [<u>dyumnin/dyu_accounting</u>](https://github.com/dyumnin/dyu_accounting)：会计设置，用于自动生成符合印度政府要求的各种财务报表。

- [<u>Gains Minimizer</u>](https://github.com/redstreet/fava_investor/tree/main/fava_investor/modules/minimizegains) (RedStreet)：自动确定出售哪些批次以最小化资本收益税。[<u>在线示例</u>](http://favainvestor.pythonanywhere.com/example-beancount-file/extension/Investor/?module=minimizegains)。

- [<u>beanahead</u>](https://github.com/maread99/beanahead) (Marcus Read)：增加了包含未来交易的功能（自动生成定期交易，提供临时预期交易，预期交易与导入交易对账；所有功能都可以通过CLI访问）。

- [<u>autobean-format</u>](https://github.com/SEIAROTg/autobean-format) (Archimedes Smith)：另一个Beancount格式化程序，由早期项目autobean-refactor支持，是一个用于解析和以编程方式操作Beancount文件的库。基于适当的解析器，允许它格式化分类账的每个角落，包括算术表达式。

- [<u>akirak/flymake-bean-check</u>](https://github.com/akirak/flymake-bean-check) (Akira Komamura)：Emacs的flymake支持。

- [<u>bean-download</u>](https://reds-rants.netlify.app/personal-finance/direct-downloads/) (Red Street)：一个下载器，随beancount-reds-importers一起提供，您可以配置运行任意命令来下载您的账户对账单。现在有一个新功能：needs-update子命令。

- [<u>gerdemb/beanpost</u>](https://github.com/gerdemb/beanpost) (Ben Gerdemann)：Beanpost由一个PostgreSQL架构和导入/导出命令组成，允许您在Beancount文件和PostgreSQL数据库之间传输数据。Beancount的许多功能是使用自定义PostgreSQL函数实现的，允许复杂的查询和数据操作。这种设置提供了一个灵活的后端，可以与其他工具（如Web应用程序或报告系统）集成。

- [<u>LaunchPlatform/beanhub-cli</u>](https://github.com/LaunchPlatform/beanhub-cli) (Fang-Pen Lin)：BeanHub或Beancount用户的命令行工具。

- [<u>zacchiro/beangrep</u>](https://github.com/zacchiro/beangrep)：Beangrep是一个类似grep的过滤器，适用于Beancount纯文本会计系统。

## 替代解析器<a id="alternative-parsers"></a>

### Bison<a id="bison"></a>

Beancount v2解析器使用GNU flex + GNU bison（最大程度的可移植性）。

Beancount v3解析器使用[<u>RE/flex</u>](https://www.genivia.com/doc/reflex/html/) + GNU bison（支持Unicode和C++）。

### 使用Antlr<a id="using-antlr"></a>

[<u>jord1e/jbeancount</u>](https://github.com/jord1e/jbeancount) (Jordie Biemold) / 使用Antlr：一个用于Beancount输入语法的替代解析器，使用Antlr4解析器生成器编写。这提供了从JVM语言访问解析的Beancount数据的能力——没有插件的影响。详情见[<u>这篇文章</u>](https://groups.google.com/d/msgid/beancount/72ee8adb-a376-4e30-b6d4-ea8749f5f666n%40googlegroups.com?utm_medium=email&utm_source=footer)。

### 使用Tree-sitter<a id="using-tree-sitter"></a>

[<u>polarmutex/tree-sitter-beancount</u>](https://github.com/polarmutex/tree-sitter-beancount) (Bryan Ryall)：一个用于Beancount语法的Tree-sitter解析器。

[<u>https://github.com/dnicolodi/tree-sitter-beancount</u>](https://github.com/dnicolodi/tree-sitter-beancount) (Daniele Nicolodi)：另一个基于Tree-sitter的Beancount语法解析器。

### 使用Rust<a id="in-rust"></a>

[<u>jcornaz/beancount-parser</u>](https://github.com/jcornaz/beancount-parser) (Jonathan Cornaz)：一个用于Rust的Beancount文件解析器库。使用nom。

[<u>beancount_parser_lima</u>](https://docs.rs/beancount-parser-lima/latest/beancount_parser_lima/) (Simon Guest)：一个用于Beancount的零拷贝解析器。它旨在完整实现Beancount文件格式，除了那些被弃用的部分和其他文档中记录的功能。使用[<u>Logos</u>](https://docs.rs/logos/latest/logos/)、[<u>Chumsky</u>](https://docs.rs/chumsky/latest/chumsky/)和[<u>Ariadne</u>](https://docs.rs/ariadne/latest/ariadne/)。

## 导入器<a id="importers"></a>

- [<u>reds importers</u>](https://github.com/redstreet/beancount_reds_importers)：适用于[<u>多个</u>](https://github.com/redstreet/beancount_reds_importers/tree/main/beancount_reds_importers)美国机构和各种文件类型的简单导入器和工具。强调通过提供维护良好的银行、信用卡和投资机构的通用库以及各种文件类型，简化编写自己的导入器。这是**[<u>五分钟分类账更新</u>](https://reds-rants.netlify.app/personal-finance/the-five-minute-ledger-update/)**中表达的原则的参考实现。欢迎贡献。作者：RedStreet

- [<u>plaid2text</u>](https://github.com/madhat2r/plaid2text)：一个从[<u>Plaid</u>](http://www.plaid.com/)导入的工具，将交易存储到Mongo DB中，并能够将其呈现为Beancount语法。作者：Micah Duke。

- [<u>jbms/beancount-import</u>](https://github.com/jbms/beancount-import)：一个工具，用于半自动导入外部数据源的交易，支持与Beancount分类账中的现有交易合并和对账。用户界面是基于Web的。（[<u>公告</u>](https://github.com/jbms/beancount-import)，[<u>链接到先前版本</u>](https://groups.google.com/d/msg/beancount/YN3xL09QFsQ/qhL8U6JDCgAJ)）。作者：Jeremy Maitin-Shepard。

- [<u>awesome-beancount</u>](https://github.com/wzyboy/awesome-beancount)：一个收集中国银行导入器和技巧的项目。作者：[<u>Zhuoyun Wei</u>](https://github.com/wzyboy)。

- [<u>beansoup</u>](https://github.com/fxtlabs/beansoup)：Filippo Tampieri分享了一些他的Beancount导入器和自动完成工具。

- [<u>montaropdf/beancount-importers</u>](https://github.com/montaropdf/beancount-importers/)：一个从时间表格式中提取加班和休假以便客户开票的导入器。

- [<u>siddhantgoel/beancount-dkb</u>](https://github.com/siddhantgoel/beancount-dkb) (Siddhant Goel)：DKB CSV文件的导入器。

- [<u>prabusw/beancount-importer-zerodha</u>](https://github.com/prabusw/beancount-importer-zerodha)：印度经纪商Zerodha的导入器。

- [<u>swapi/beancount-utils</u>](https://github.com/swapi/beancount-utils)：另一个Zerodha的导入器。

- [<u>Dr-Nuke/drnuke-bean</u>](https://github.com/Dr-Nuke/drnuke-bean) (Dr Nuke)：基于灵活查询（API-like）的IBKR导入器，以及一个用于瑞士邮政金融的导入器。

- [<u>Beanborg</u>](https://github.com/luciano-fiandesio/beanborg) (Luciano Fiandesio)：Beanborg自动从外部CSV文件导入金融交易到Beancount簿记系统。

- [<u>szabootibor/beancount-degiro</u>](https://gitlab.com/szabootibor/beancount-degiro) ([<u>PyPI</u>](https://pypi.org/project/beancount-degiro))：荷兰经纪商Degiro交易账户的导入器。

- [<u>siddhantgoel/beancount-ing-diba</u>](https://github.com/siddhantgoel/beancount-ing-diba) ([<u>PyPI</u>](https://pypi.org/project/beancount-ing-diba/))：ING账户导入器（NL）。

- [<u>PaulsTek/csv2bean</u>](https://github.com/PaulsTek/csv2bean)：一个用Go编写的简单应用程序，用于使用Google Sheets预处理CSV文件。

- [<u>ericaltendorf/magicbeans</u>](https://github.com/ericaltendorf/magicbeans) (Eric Altendorf)：Beancount加密数据导入器。详细的批次跟踪和加密资产的资本收益/损失报告。“我写它是因为我对现有商业服务的准确性或透明度不满意。”

- [<u>OSadovy/uabean</u>](https://github.com/OSadovy/uabean/) (Oleksii Sadovyi)：一组用于流行乌克兰银行的Beancount导入器和脚本等。

- [<u>fdavies93/seneca</u>](https://github.com/fdavies93/seneca) (Frank Davies)：Wise的导入器。多货币转账。

- [<u>LaunchPlatform/beanhub-import</u>](https://github.com/LaunchPlatform/beanhub-import)：新的Beancount导入器，带有用户界面。

- [<u>rlan/beancount-multitool</u>](https://github.com/rlan/beancount-multitool) (Rick Lan)：Beancount Multitool是一个命令行界面（CLI）工具，用于将金融机构的金融数据转换为Beancount文件（支持：JA Bank JAネットバンク、Rakuten Card 楽天カード、Rakuten Bank 楽天銀行、SBI Shinsei Bank 新生銀行）。相关帖子：[<u>https://www.linkedin.com/feed/update/urn:li:activity:7198125470662500352/</u>](https://www.linkedin.com/feed/update/urn:li:activity:7198125470662500352/)

## 转换器<a id="converters"></a>

- [<u>plaid2text</u>](https://github.com/madhat2r/plaid2text)：Python脚本，用于导出Plaid交易并将其转换为Ledger或Beancount语法格式的文件。

- [<u>gnucash-to-beancount</u>](https://github.com/henriquebastos/gnucash-to-beancount/)：Henrique Bastos编写的脚本，用于将GNUcash SQLite数据库转换为等效的Beancount输入文件。

- [<u>debanjum/gnucash-to-beancount</u>](https://github.com/debanjum/gnucash-to-beancount)：上面的分支。

- [<u>andrewStein/gnucash-to-beancount</u>](https://github.com/AndrewStein/gnucash-to-beancount)：上面两个的进一步分支，修复了许多问题（见[<u>这个线程</u>](https://groups.google.com/d/msg/beancount/MaaASKR1SSI/GX5I8lOkBgAJ)）。

- [<u>hoostus/beancount-ynab</u>](https://github.com/hoostus/beancount-ynab)：一个从YNAB到Beancount的转换器。

- [<u>hoostus/beancount-ynab5</u>](https://github.com/hoostus/beancount-ynab5)：相同的转换器，适用于更近版本的YNAB 5。

- [<u>ledger2beancount</u>](https://github.com/zacchiro/ledger2beancount/)：一个将Ledger文件转换为Beancount的脚本，由Stefano Zacchiroli和Martin Michlmayr开发。

- [<u>smart_importer</u>](https://github.com/johannesjh/smart_importer)：一个智能导入器，适用于Beancount和Fava，具有智能账户名称建议功能。作者：Johannes Harms。

- [<u>beancount-export-patreon.js</u>](https://gist.github.com/riking/0f0dab2b7761d2f6895c5d58c0b62a66)：JavaScript脚本，用于导出您的Patreon交易，以便查看您具体给了哪些人钱。作者：kanepyork@gmail。

- [<u>alensiljak/pta-converters</u>](https://gitlab.com/alensiljak/pta-converters) (Alen Šiljak)：GNUcash到Beancount转换器（2019年）。

- [<u>grostim/Beancount-myTools</u>](https://github.com/grostim/Beancount-myTools) (Timothee Gros)：作者的个人导入工具，用于法国银行。

## 下载器<a id="downloaders"></a>

- [<u>bean-download</u>](https://github.com/redstreet/beancount_reds_importers) (RedStreet)：bean-download是一个方便下载交易的工具。您可以配置它与您的机构和任意命令下载它们，通常通过[<u>ofxget</u>](https://ofxtools.readthedocs.io/en/latest/)。它同时下载所有交易，适当命名并放置在您选择的目录中，您可以从中导入。该工具作为[<u>beancount-reds-importers</u>](https://github.com/redstreet/beancount_reds_importers)的一部分安装。参见[<u>相关文章</u>](https://reds-rants.netlify.app/personal-finance/direct-downloads/)。

- [<u>ofx-summarize</u>](https://github.com/redstreet/beancount_reds_importers) (RedStreet)：构建导入器时，查看您尝试导入的.ofx或.qfx文件内容非常有用。ofx-summarize命令正是做这个的。它随[<u>beancount-reds-importers</u>](https://github.com/redstreet/beancount_reds_importers)一起提供，只需调用该命令即可。对文件运行命令显示文件中的一些交易。非常有用的是能够通过Python调试器或解释器探索您的.ofx文件。

## 价格来源<a id="price-sources"></a>

- [<u>hoostus/beancount-price-sources</u>](https://github.com/hoostus/beancount-price-sources)：一个晨星价格抓取器，聚合了多个交易所，包括非美国的交易所。

- [<u>andyjscott/beancount-financequote</u>](https://github.com/andyjscott/beancount-financequote)：bean-price的Finance::Quote支持。

- [<u>aamerabbas/beancount-coinmarketcap</u>](https://github.com/aamerabbas/beancount-coinmarketcap)：coinmarketcap的价格抓取器（[<u>参见文章</u>](https://medium.com/@danielcimring/downloading-historical-data-from-coinmarketcap-41a2b0111baf)）。

- [<u>grostim/Beancount-myTools/.../iexcloud.py</u>](https://github.com/grostim/Beancount-myTools/blob/master/price/iexcloud.py)：Timothee Gros的iexcloud价格抓取器。

- [<u>xuhcc/beancount-cryptoassets</u>](https://github.com/xuhcc/beancount-cryptoassets) (Kirill Goncharov)：加密货币的价格来源。

- [<u>xuhcc/beancount-ethereum-importer</u>](https://github.com/xuhcc/beancount-ethereum-importer) (Kirill Goncharov)：用于Beancount的以太坊交易导入器。包括从Etherscan下载交易的脚本和一个导入器。

- [<u>xuhcc/beancount-exchangerates</u>](https://github.com/xuhcc/beancount-exchangerates) (Kirill Goncharov)：价格来源[<u>http://exchangeratesapi.io</u>](http://exchangeratesapi.io)。

- [<u>tarioch/beancounttools</u>](https://github.com/tarioch/beancounttools) (Patrick Ruckstuhl)：价格来源和导入器。

- [<u>https://gitlab.com/chrisberkhout/pricehist</u>](https://gitlab.com/chrisberkhout/pricehist) (Chris Berkhout)：一个命令行工具，可以从多个来源获取每日历史价格，并以多种格式输出。支持一些CoinDesk、欧洲中央银行、Alpha Vantage、CoinMarketCap的来源。用户可以请求特定的价格类型，如高、低、开盘、收盘或调整后收盘价。也可以通过bean-price使用。

## 开发<a id="development"></a>

- [<u>Py3k type annotations</u>](https://github.com/yegle/beancount-type-stubs)：Yuchen Ying正在为Beancount实现Python3类型注释。

- [<u>bryall/tree-sitter-beancount</u>](https://github.com/bryall/tree-sitter-beancount) (Bryan Ryall)：一个用于Beancount语法的tree-sitter解析器。

- [<u>jmgilman/beancount-stubs</u>](https://github.com/jmgilman/beancount-stubs)：Beancount源代码的Typing .pyi存根。

## 文档<a id="documentation"></a>

- [<u>Beancount文档</u>](https://beancount.github.io/docs/) ([<u>Kirill Goncharov</u>](http://github.com/xuhcc))：将Beancount文档从Google Docs源转换为Markdown和HTML的官方版本。包括大多数Google Docs文档，并由Kirill Goncharov维护在Beancount org仓库[<u>这里</u>](http://github.com/beancount/docs)。

- [<u>Beancount源代码文档</u>](http://aumayr.github.io/beancount-docs-static/) ([<u>Dominik Aumayr</u>](http://github.com/aumayr))：一个Sphinx生成的Beancount代码库源代码文档。生成此文档的代码位于[<u>这里</u>](https://github.com/aumayr/beancount-docs)。

- [<u>Beancount的SQL查询</u>](http://aumayr.github.io/beancount-sql-queries/) (Dominik Aumayr)：SQL查询示例。

- [<u>Beancount —— 命令行复式簿记</u>](https://wzyboy.im/post/1063.html) (Zhuoyun Wei)：一篇关于如何使用Beancount的中文教程（博客文章）。

- [<u>使用Beancount管理我的个人财务</u>](https://alexjj.com/blog/2016/2/managing-my-personal-finances-with-beancount/) (Alex Johnstone)

- [<u>用Beancount计算豆子——以及更多</u>](https://lwn.net/SubscriberLink/751874/a38128abb72e45c5/) (LWN)

## 界面 / Web<a id="interfaces-web"></a>

- [<u>fava: Beancount的Web界面</u>](https://github.com/aumayr/fava) (Dominik Aumayr, Jakob Schnitzer)：Beancount带有自己简单的Web前端（“bean-web”），仅作为调用和显示其报告的HTML版本的一个薄壳。“Fava”是一个替代的Web应用前端，具有更多和不同的功能，最初作为一个游乐场和概念验证，以探索一个更新、更好的呈现Beancount文件内容的设计。

- [<u>Fava Classy Portfolio</u>](https://github.com/seltzered/fava-classy-portfolio) (Vivek Gani)：Classy Portfolio是Fava的一个扩展，用于Beancount纯文本会计软件的Web界面。扩展显示不同投资组合（例如“应税”与“退休”）的列表，使用“资产类别”和“资产子类”元数据标签对商品进行细分。

- [<u>Fava Investor</u>](https://github.com/redstreet/fava_investor) 项目 (RedStreet)：Fava_investor旨在为Beancount和Fava提供一套全面的投资报告、分析和工具。它是一个模块集合，每个模块提供一个Fava插件、一个Beancount库和一个基于Beancount的CLI（命令行界面）。当前模块包括：视觉、按类的树状结构资产分配、按账户的资产分配、税收损失收割、现金拖累分析。

- [<u>Fava Miler</u>](https://github.com/redstreet/fava_miler) (RedStreet)：航空里程和奖励积分：到期和价值报告。

- [<u>Fava Envelope</u>](https://github.com/bryall/fava-envelope) (Brian Ryall)：一个Beancount Fava扩展，添加了信封预算功能到Fava和Beancount。它作为一个Fava插件和CLI开发。

- [<u>scauligi/refried</u>](https://github.com/scauligi/refried) (Sunjay Cauligi)：一个为Fava设计的信封预算插件，受YNAB启发：所有费用账户都变成单独的预算类别，通过这些账户的交易进行预算，插件自动将标签应用于所有重新预算的交易，以便轻松过滤。提供像YNAB一样的预算和账户视图。

- [<u>BeanHub.io</u>](https://beanhub.io/)：一个Beancount内容的Web前端。“*自从我开始使用Beancount以来，我一直梦想着让它完全自动化。几年来，我一直在构建工具以实现这个目标。连接到银行并直接从那里获取数据是我想实现的目标之一。我构建了这个功能，并一直在测试一段时间，用于我的会计簿。现在我的Beancount簿记80%完全自动化。我可以打开我的存储库，银行的交易将自动出现在一个新的提交中，而不需要我动一根手指。整个导入系统基于我们的开源beanhub-import和beanhub-extract。唯一专有的部分是Plaid集成。因此，假设您不信任我并仍然希望自动导入交易。只要您连接到Plaid并根据从Plaid获取的交易编写CSV文件，您应该能够拥有与使用BeanHub服务相同的自动交易导入系统。*”

博客文章：

> [<u>https://beanhub.io/blog/2024/06/24/introduction-of-beanhub-connect/</u>](https://beanhub.io/blog/2024/06/24/introduction-of-beanhub-connect/)
>
> [<u>https://beanhub.io/blog/2024/04/23/how-beanhub-works-part1-sandboxing/</u>](https://beanhub.io/blog/2024/04/23/how-beanhub-works-part1-sandboxing/)
>
> [<u>https://beanhub.io/blog/2024/06/26/how-beanhub-works-part2-layer-based-git-repos/</u>](https://beanhub.io/blog/2024/06/26/how-beanhub-works-part2-layer-based-git-repos/)

- [<u>jmgilman/bdantic</u>](https://github.com/jmgilman/bdantic)：一个用于扩展Beancount的包，使用[<u>pydantic</u>](https://pydantic-docs.helpmanual.io/)。使用这个包，您可以将分类账转换为JSON等。

- [<u>autobean/refactor</u>](https://github.com/SEIAROTg/autobean/tree/master/autobean/refactor) (Archimedes Smith)：用于以编程方式编辑分类账的工具，包括格式化、排序、重构、重新排列账户、通过插件优化、从v2迁移、在导入时插入交易等。

- [<u>seltzered/beancolage</u>](https://github.com/seltzered/beancolage) (Vivek Gani)：一个Eclipse Theia（供应商无关的vscode）应用程序，尝试捆绑现有的基于Beancount的包，如vscode-beancount和Fava。

- [<u>aaronstacy.com/personal-finances-dashboard</u>](http://aaronstacy.com/personal-finances-dashboard/)：一个HTML + D3.js可视化仪表盘，用于Beancount数据。

## 移动/手机数据输入<a id="mobilephone-data-entry"></a>

- [<u>Beancount Mobile</u>](https://play.google.com/store/apps/details?id=link.beancount.mobile) App (Kirill Goncharov)：一个Beancount的移动数据输入应用程序。（目前仅支持Android）。仓库：[<u>https://github.com/xuhcc/beancount-mobile</u>](https://github.com/xuhcc/beancount-mobile) ([<u>公告</u>](https://groups.google.com/d/msgid/beancount/014e0879-70e0-4cac-b884-82d8004e1b43%40googlegroups.com?utm_medium=email&utm_source=footer))。

- [<u>http://costflow.io</u>](http://costflow.io/)：在WeChat中进行纯文本会计。“*将消息发送到我们的机器人Telegram、Facebook Messenger、Whatsapp、LINE、WeChat等。Costflow会将您的消息神奇地转换为Beancount / Ledger / hledger格式的交易。将交易附加到您的Dropbox / Google Drive中的文件。借助这些应用程序，文件将同步到您的计算机。*”


