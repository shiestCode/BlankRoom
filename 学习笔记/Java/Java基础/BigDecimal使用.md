Some Suggestion

- 避免使用Double来进行运算
- BigDecimal的初始化要使用String入参或者BigDecimal.valueOf()
- 浮点数的格式化建议使用BigDecimal
- 比较两个BigDecimal的value要使用compareTo



建议金额以最小货币单位且整型类型来进行存储,比如人民币的最小单位是分,那么就以分为单位进行存储