# 在 Beancount 中跟踪网络外的医疗索赔<a id="title"></a>

作者：[马丁·布莱斯](mailto:blais@furius.ca) - 更新于：2023年11月

本文通过一个示例说明如何处理医疗治疗的保险和HSA（健康储蓄账户）索赔。我们以心理治疗为例，这种治疗是网络外的，需先自费支付，后期再报销。

## 记录治疗费用

当收到心理治疗时，将其记入应收和应付账款：

```plaintext
2023-09-06 * "Dr. Freud" "Session" #freud-2023-09
  Assets:AccountsReceivable:Psychotherapy     260.00 USD
  Liabilities:AccountsPayable:Psychotherapy  -260.00 USD

2023-09-11 * "Dr. Freud" "Session" #freud-2023-09
  Assets:AccountsReceivable:Psychotherapy     260.00 USD
  Liabilities:AccountsPayable:Psychotherapy  -260.00 USD
```

## 支付费用

稍后支付这些费用，清除应付账款：

```plaintext
2023-09-12 * "ZELLE SENT" #freud-2023-09
  Assets:US:BofA:Checking                     -520.00 USD
  Liabilities:AccountsPayable:Psychotherapy    520.00 USD
```

以此类推，直到月底：

```plaintext
2023-09-20 * "Dr. Freud" "Session" #freud-2023-09
  Assets:AccountsReceivable:Psychotherapy     260.00 USD
  Liabilities:AccountsPayable:Psychotherapy  -260.00 USD

2023-09-23 * "ZELLE SENT" #freud-2023-09
  Assets:US:BofA:Checking                     -260.00 USD
  Liabilities:AccountsPayable:Psychotherapy    260.00 USD

2023-09-27 * "Dr. Freud" "Session" #freud-2023-09
  Assets:AccountsReceivable:Psychotherapy     260.00 USD
  Liabilities:AccountsPayable:Psychotherapy  -260.00 USD

2023-09-28 * "ZELLE SENT" #freud-2023-09
  Assets:US:BofA:Checking                     -260.00 USD
  Liabilities:AccountsPayable:Psychotherapy    260.00 USD
```

## 提交保险索赔

月底时，治疗师会出具索赔单，我们向保险公司提交索赔，清除应收账款并将剩余部分转移到保险公司待发放的支票中：

```plaintext
2023-09-28 * "Claim for September filed with insurance" #freud-2023-09
  Assets:AccountsReceivable:Psychotherapy  -1040.00 USD
  Expenses:Mental:Psychotherapy              312.00 USD
  Assets:AccountsReceivable:Anthem           728.00 USD
```

## 收到保险支付

最终，保险公司会发出EOB（利益说明），确认覆盖的金额（例如，本例中覆盖70%，根据条款得知），然后收到并存入银行的支票：

```plaintext
2023-10-24 * "ATM CHECK DEPOSIT" #freud-2023-09
  Assets:US:BofA:Checking             728.00 USD
  Assets:AccountsReceivable:Anthem   -728.00 USD
```

## 提交HSA索赔

此时，我们知道剩余部分可从HSA支付，于是向HSA公司提交索赔，再次将其记为应收和应付账款：

```plaintext
2023-10-25 * "HealthEquity" "Filed for reimbursement" #freud-2023-09
  Liabilities:AccountsPayable:HealthEquity  -312.00 USD
  Assets:AccountsReceivable:HealthEquity     312.00 USD
```

HSA公司会将款项直接存入我们的支票账户：

```plaintext
2023-11-01 * "BofA bank (Claim ID:1234567-890); EFT to bank" #freud-2023-09
  Assets:US:HealthEquity:Cash               -312.00 USD
  Liabilities:AccountsPayable:HealthEquity   312.00 USD
```

在银行端导入这笔交易时，将其与应收账款对账：

```plaintext
2023-11-01 * "HEALTHEQUITY INC" #freud-2023-09
  Assets:US:BofA:Checking                    312.00 USD
  Assets:AccountsReceivable:HealthEquity    -312.00 USD
```

最终结果是HSA用于支付未保险覆盖的部分费用。

```plaintext
|-- Assets                       
|   |-- AccountsReceivable       
|   |   |-- Anthem                
|   |   |-- HealthEquity         
|   |   `-- Psychotherapy        
|   `-- US                       
|       |-- HealthEquity         
|       |   `-- Cash                  -312.00               USD
|       `-- BofA                   
|           `-- Checking         
|-- Expenses                     
|   `-- Mental                   
|       `-- Psychotherapy              312.00               USD
`-- Liabilities                  
    `-- AccountsPayable          
        |-- HealthEquity         
        `-- Psychotherapy        

Net Income: (-312.00 USD)
```

这种方法有一些缺陷：

- 保险覆盖的金额事先已知；在许多情况下，金额在保险公司发出EOB之前是不确定的（毕竟EOB的作用就是解释福利）。这需要修改上述方法。
