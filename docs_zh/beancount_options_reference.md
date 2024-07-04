# Beancount 选项参考<a id="title"></a>

本文档详细介绍了 Beancount 中可用的各种选项及其用法。这些选项可以在 Beancount 文件中设置，以自定义和控制分类账的行为。

## 基本选项<a id="basic-options"></a>

### 标题<a id="title-option"></a>

```plaintext
option "title" "Joe Smith's Personal Ledger"
```

设置分类账或输入文件的标题。此标题将显示在每页顶部。

### 账户根名称<a id="account-root-names"></a>

```plaintext
option "name_assets" "Assets"
option "name_liabilities" "Liabilities"
option "name_equity" "Equity"
option "name_income" "Income"
option "name_expenses" "Expenses"
```

每个账户类别的根名称。可以自定义这些类别名称，例如将 "Income" 改为 "Revenue" 或将 "Equity" 改为 "Capital"。输入文件中的账户名称必须匹配这些名称，解析器将进行验证。建议将这些选项放在文件开头，因为它们会影响解析器识别账户名称的方式。

### 前期余额账户<a id="account-previous-balances"></a>

```plaintext
option "account_previous_balances" "Opening-Balances"
```

用于总结前期交易为期初余额的权益账户的叶名称。

### 前期收益账户<a id="account-previous-earnings"></a>

```plaintext
option "account_previous_earnings" "Earnings:Previous"
```

用于将期初余额前的收入和费用转移到资产负债表中的权益账户的叶名称。

### 前期转换账户<a id="account-previous-conversions"></a>

```plaintext
option "account_previous_conversions" "Conversions:Previous"
```

用于插入转换以抵消期初日期之前转移的剩余金额的权益账户的叶名称。这将基本上修正由于定价转换引入的误差导致的基本会计等式。

### 当期收益账户<a id="account-current-earnings"></a>

```plaintext
option "account_current_earnings" "Earnings:Current"
```

用于将当前期间内的收入和费用转移到资产负债表中的权益账户的叶名称。通常称为“净收入”。

### 当期转换账户<a id="account-current-conversions"></a>

```plaintext
option "account_current_conversions" "Conversions:Current"
```

用于插入转换以抵消期间内转移的剩余金额的权益账户的叶名称。

### 舍入账户<a id="account-rounding"></a>

```plaintext
option "account_rounding" "Rounding"
```

用于过账和累计舍入误差的账户名称。默认情况下未设置此选项；设置此值为账户名称将自动启用对所有交易中残余金额的过账添加。

### 转换货币<a id="conversion-currency"></a>

```plaintext
option "conversion_currency" "NOTHING"
```

用于以零汇率转换所有单位的虚拟货币。这可以是分类账中未使用的任何货币名称。选择一个在您的语言中有意义且唯一的名称。

### 默认容差<a id="inferred-tolerance-default"></a>

```plaintext
option "inferred_tolerance_default" "CHF:0.01"
```

**此选项已弃用：** 该选项已重命名为“inferred_tolerance_default”。

用于在无法自动推断时使用的货币容差映射。容差用于验证以下内容的平衡：(1) 交易，(2) balance 指令的显式余额检查，以及 (3) pad 指令的精度。值必须是以下格式的字符串：`<currency>:<tolerance>`，例如 `USD:0.005`。默认情况下，对于没有推断值的货币，容差为零（表示无限精度）。作为特例，可以使用“*”货币覆盖所有未明确指定默认值的货币的容差，如下所示：`*:0.5`。单独使用此示例将为所有货币设置 `0.5` 的容差。

### 容差倍数<a id="inferred-tolerance-multiplier"></a>

```plaintext
option "inferred_tolerance_multiplier" "1.1"
```

用于推断容差值的倍数。当容差值未通过“inferred_tolerance_default”选项显式指定时，容差值会从输入文件中的数字推断。例如，如果交易的过账值为 `32.424 CAD`，则 CAD 的容差将推断为 `0.001` 乘以某个倍数。默认的倍数值为 `0.5`，即遇到的最小数字的一半。您可以通过更改此选项来自定义此倍数，通常扩展它以考虑略超过通常容差的金额。

### 从成本推断容差<a id="infer-tolerance-from-cost"></a>

```plaintext
option "infer_tolerance_from_cost" "True"
```

启用一个功能，扩展通过按成本持有或按价格转换的过账推断的成本货币的最大容差。这些过账可以通过将单位的最小数字乘以成本或价格值并取该值的一半来暗示容差值。例如，如果一个过账的金额为 `2.345 RGAGX {45.00 USD}`，则它暗示的容差为 `0.001 x 45.00 x M = 0.045 USD`（其中 M 是 `inferred_tolerance_multiplier`），并将此值加入到容差中，以扩大该交易中 USD 单位允许的容差。

### 总容差<a id="tolerance"></a>

```plaintext
option "tolerance" "0.015"
```

**此选项已弃用：** “tolerance”选项已被弃用且无效。

用于余额检查和填充指令的容差。在现实世界中，四舍五入会在各种地方发生，我们需要允许一个非常小的容差量来检查交易的平衡和要求自动插入填充条目。此值即为容差量，您可以覆盖此值。

### 使用传统固定容差<a id="use-legacy-fixed-tolerances"></a>

```plaintext
option "use_legacy_fixed_tolerances" "True"
```

恢复传统固定容差的处理方式。Balance 和 Pad 指令的固定容差为 `0.015` 单位，交易的平衡容差为 `0.005` 单位，适用于任何单位。这是为了方便那些希望将 Beancount 的行为恢复为旧容差逻辑的人。

### 文档目录<a id="documents"></a>

```plaintext
option "documents" "/path/to/your/documents/archive"
```

一个目录根列表，相对于当前工作目录，用于搜索文档文件。要自动找到文档文件，它们必须具有以下文件名格式：`YYYY-MM-DD.(.*)`。

### 经营货币<a id="operating-currency"></a>

```plaintext
option "operating_currency" "USD"
```

报告期间需要单独列出专栏的货币列表，用于指示您在现实生活中使用的主要货币。因为系统对输入文件中的任何单位定义是不可知的，所以我们使用此选项在表格单元格中显示这些值而不带单位字符串。

### 渲染逗号<a id="render-commas"></a>

```plaintext
option "render_commas" "TRUE"
```

布尔值，若为 `true`，则数字格式化例程将使用逗号作为千位分隔符。

### 插件处理模式<a id="plugin-processing-mode"></a>

```plaintext
option "plugin_processing_mode" "raw"
```

定义加载器运行的插件集的字符串：如果模式为 `default`，则一组预设插件在任何用户插件之前自动运行。如果模式为 `raw`，则不运行任何预设插件，仅运行用户插件。

### 插件<a id="plugin"></a>

```plaintext
option "plugin" "beancount.plugins.module_name"
```

**此选项已弃用：** “plugin”选项已弃用；应替换为 `plugin` 指令。

一个包含转换函数的 Python 模块列表，这些函数在解析后运行条目。解析器按原样读取条目，通过一组标准函数（如余额检查和插入填充条目）进行转换，然后将条目传递给这些插件以添加更多自动生成的内容。

### 多行字符串最大行数<a id="long-string-maxlines"></a>

```plaintext
option "long_string_maxlines" "64"
```

超过此行数的多行字符串将触发过长行警告。此警告旨在帮助检测意外长字符串的悬挂引号。

### 显式容差实验<a id="experiment-explicit-tolerances"></a>

```plaintext
option "experiment_explicit_tolerances" "True"
```

启用一个实验性功能，支持余额断言的显式容差值。如果启用，余额金额支持输入中的容差，语法为 `<number> ~ <tolerance> <currency>`，例如 `532.23 ~ 0.001 USD`。

### 预订方法<a id="booking-method"></a>

```plaintext
option "booking_method" "SIMPLE"
```

应用的预订方法，用于在交易发生时将可用批次匹配到库存。值可以是 `SIMPLE` 表示 Beancount 使用的原始方法，或 `FULL` 表示新的模糊匹配库存的方法。
