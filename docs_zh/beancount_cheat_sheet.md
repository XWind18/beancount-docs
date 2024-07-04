# Beancount 语法备忘单<a id="title"></a>

## 账户类型<a id="account-types"></a>

| 账户类型   | 符号 |
| ---------- | ---- |
| Assets     | +    |
| Liabilities| -    |
| Income     | -    |
| Expenses   | +    |
| Equity     | -    |

*示例账户名称：Assets:US:BofA:Checking*

## 商品<a id="commodities"></a>

**商品名称必须全大写：**

- 货币：USD, EUR, CAD, AUD
- 股票代码：GOOG, AAPL, RBF1005
- 其他：HOME_MAYST, AIRMILES, HOURS

## 指令<a id="directives"></a>

**通用语法：**

`YYYY-MM-DD <directive> <arguments...>`

## 开户和销户<a id="opening-closing-accounts"></a>

```plaintext
2001-05-29 open Expenses:Restaurant
2001-05-29 open Assets:Checking USD,EUR  ; Currency constraints
2015-04-23 close Assets:Checking
```

## 声明商品<a id="declaring-commodities"></a>

*这是可选的；仅在您希望通过货币附加元数据时使用。*

```plaintext
1998-07-22 commodity AAPL
  name: "Apple Computer Inc."
```

## 价格<a id="prices"></a>

*多次使用以填充历史价格数据库：*

```plaintext
2015-04-30 price AAPL 125.15 USD
2015-05-30 price AAPL 130.28 USD
```

## 注释<a id="notes"></a>

```plaintext
2013-03-20 note Assets:Checking "Called to ask about rebate"
```

## 文档<a id="documents"></a>

```plaintext
2013-03-20 document Assets:Checking "path/to/statement.pdf"
```

## 交易<a id="transactions"></a>

```plaintext
2015-05-30 * "Some narration about this transaction"
  Liabilities:CreditCard -101.23 USD
  Expenses:Restaurant 101.23 USD

2015-05-30 ! "Cable Co" "Phone Bill" #tag ^link
  id: "TW378743437"  ; Meta-data
  Expenses:Home:Phone 87.45 USD
  Assets:Checking  ; You may leave one amount out
```

### 过账<a id="postings"></a>

```plaintext
... 123.45 USD  ; Simple
... 10 GOOG {502.12 USD}  ; With per-unit cost
... 10 GOOG {{5021.20 USD}}  ; With total cost
... 10 GOOG {502.12 # 9.95 USD}  ; With both costs
... 1000.00 USD @ 1.10 CAD  ; With per-unit price
... 10 GOOG {502.12 USD} @ 1.10 CAD  ; With cost & price
... 10 GOOG {502.12 USD, 2014-05-12}  ; With date
! ... 123.45 USD ...  ; With flag
```

## 余额断言和填充<a id="balance-assertions-and-padding"></a>

*断言仅适用于给定货币的金额：*

```plaintext
2015-06-01 balance Liabilities:CreditCard -634.30 USD
```

*自动插入交易以满足以下断言：*

```plaintext
YYYY-MM-DD pad Assets:Checking Equity:Opening-Balances
```

## 事件<a id="events"></a>

```plaintext
YYYY-MM-DD event "location" "New York, USA"
YYYY-MM-DD event "address" "123 May Street"
```

## 选项<a id="options"></a>

```plaintext
option "title" "My Personal Ledger"
```

*请参阅[<u>此文档</u>](beancount_options_reference.md)了解支持选项的完整列表。*

## 其他<a id="other"></a>

```plaintext
pushtag #trip-to-peru
...
poptag #trip-to-peru
; Comments begin with a semi-colon
```
