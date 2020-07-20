# AutomaticMoneyMakingBot

自动赚钱机器人是通过网格交易跨期套利来实现低风险赚钱的项目。在进一步研究本项目前，你需要对数学、金融、数字货币和 Python 编程有基本的了解。尽管跨期套利的风险非常低，你仍需要认识到「币圈有风险，投资需谨慎」。本项目仅作为技术交流，本人不提供任何投资建议，也不允许任何个人或团体将本项目用于商业目的。

## 项目进度

- [ ] 理论
  - [X] OKEx 合约讲解
  - [X] OKEx 跨期套利
  - [ ] 网格交易讲解
  - [ ] OKEx 爆仓、保证金、风险准备金和分摊
  - [ ] 风险提示
- [ ] 程序
- [ ] 回测
- [ ] 实盘

## 理论

### OKEx 合约讲解

#### 参数设定

现货价格：P₁

初始投入数字货币数量：N

初始投入美金数量：N * P₁

杠杆数：L

开仓数字货币数量：N * L

合约面值：M

合约开仓价格：P₁

合约张数为：N * L * P₁ / M

合约平仓价格：P₂

#### 计算合约收益

做多赚得的数字货币 = 合约张数 * 合约面值 * ( 1 / P₁ - 1 / P₂ ) = [ N * L * P₁ / M ] * [ M ] * [ ( P₂ - P₁ ) / ( P₁ * P₂ ) ] = N * L * ( P₂ - P₁ ) / P₂

做空赚得的数字货币 = 合约张数 * 合约面值 * ( 1 / P₂ - 1 / P₁ ) = [ N * L * P₁ / M ] * [ M ] * [ ( P₁ - P₂ ) / ( P₁ * P₂ ) ] = N * L * ( P₁ - P₂ ) / P₂

做多后总共的数字货币 = 做多赚得的数字货币 + 初始投入数字货币数量 = N * L * ( P₂ - P₁ ) / P₂ + N

做空后总共的数字货币 = 做空赚得的数字货币 + 初始投入数字货币数量 = N * L * ( P₁ - P₂ ) / P₂ + N

做多后总共的美金 = 做多后总共的数字货币 * 合约平仓价格 = [ N * L * ( P₂ - P₁ ) / P₂ + N ] * P₂

做空后总共的美金 = 做空后总共的数字货币 * 合约平仓价格 = [ N * L * ( P₁ - P₂ ) / P₂ + N ] * P₂

做多美金收益率 = ( 做多后总共的美金 - 初始投入美金数量 ) / 初始投入美金数量 = { [ N * L * ( P₂ - P₁ ) / P₂ + N ] * P₂ - N * P₁ } / ( N * P₁ ) = { [ L * ( P₂ - P₁ ) / P₂ + 1] * P₂ - P₁ } / P₁ = [ L * ( P₂ - P₁ ) + P₂ - P₁ ] / P₁ = ( L + 1 ) * ( P₂ - P₁ ) / P₁

做空美金收益率 = ( 做空后总共的美金 - 初始投入美金数量 ) / 初始投入美金数量 = { [ N * L * ( P₁ - P₂ ) / P₂ + N ] * P₂ - N * P₁ } / ( N * P₁ ) = { [ L * ( P₁ - P₂ ) / P₂ + 1] * P₂ - P₁ } / P₁ = [ L * ( P₁ - P₂ ) + P₂ - P₁ ] / P₁ = ( L - 1 ) * ( P₁ - P₂ ) / P₁

#### 典型场景

Q1：如何保持数字货币市值不变？

A1：因为做空美金收益率 = ( L - 1 ) * ( P₁ - P₂ ) / P₁，所以要让 L - 1 = 0，即将杠杆倍数设为 1 做空。

Q2：如何 3 倍做多？即合约价格上涨 10% 时，账户折合美金资产上涨 30%。

A2：因为做多美金收益率 = ( L + 1 ) * ( P₂ - P₁ ) / P₁，所以要让 L + 1 = 3，即将杠杆倍数设为 2。

Q3：如何 2 倍做空？即合约价格下跌 10% 时，账户折合美金资产上涨 20%。

A3：因为做空美金收益率 = ( L - 1 ) * ( P₁ - P₂ ) / P₁，所以要让 L - 1 = 2，即将杠杆倍数设为 3。

### OKEx 跨期套利

#### 参数设定

现货价格：P₁

初始投入数字货币数量：N

初始投入美金数量：N * P₁

最大杠杆数：L

当周合约开仓价格：P₁

合约开仓价差：r₁

季度合约开仓价格：P₁ * ( 1 + r₁ )

当周合约平仓价格：P₂

合约平仓价差：r₂

季度合约平仓价格：P₂ * ( 1 + r₂ )

总共可开仓数字货币数量：N * L

#### 做空价差如何开仓

Q：当周合约 P₁ = 10，季度合约 P₂ = 10.5，r₁ = 5%，预测 r 会缩小。

A：两边开仓，做多当周，做空季度。

1. 做空当周合约数字货币数量（对冲保证金）：N

2. 做多当周合约数字货币数量：( N * L - N ) / 2

3. 做空季度合约数字货币数量：( N * L - N ) / 2

最终数字货币数量 = 对冲保证金赚取数字货币数量 + 做多赚取数字货币数量 + 做空赚取数字货币数量 + 原有数字货币数量 = { N * ( P₁ - P₂ ) / P₂ } + { ( L - 1 ) / 2 * N * ( P₂ - P₁ ) / P₂ } + { ( L - 1 ) / 2 * N * [ P₁ * ( 1 + r₁ ) - P₂ * ( 1 + r₂ ) ] / [ P₂ * ( 1 + r₂) ] } + { N }

最终美金数量 = 最终数字货币数量 * P₂ = N * ( P₁ - P₂ ) + ( L - 1 ) / 2 * N * ( P₂ - P₁ ) + ( L - 1 ) / 2 * N * [ P₁ * ( 1 + r₁ ) - P₂ * ( 1 + r₂ ) ] / ( 1 + r₂ ) + N * P₂

最终美金收益率 = ( 最终美金数量 - 初始投入美金数量 ) / 初始投入美金数量 = { N * ( P₁ - P₂ ) + ( L - 1 ) / 2 * N * ( P₂ - P₁ ) + ( L - 1 ) / 2 * N * [ P₁ * ( 1 + r₁ ) - P₂ * ( 1 + r₂ ) ] / ( 1 + r₂ ) + N * P₂ - N * P₁ } / ( N * P₁ ) = ( L - 1 ) / 2 * { ( P₂ - P₁ ) / P₁ + [ P₁ * ( 1 + r₁ ) - P₂ * ( 1 + r₂ ) ] / [ P₁ * ( 1 + r₂ ) ] } = ( L - 1 ) / 2 * [ P₂ / P₁ - 1 + ( 1 + r₁ ) / ( 1 + r₂ ) - P₂ / P₁ ] = ( L - 1 ) / 2 * [ ( 1 + r₁ ) / ( 1 + r₂ ) - 1 ] = ( L - 1 ) / 2 * ( r₁ - r₂ ) / ( 1 + r₂ )

也就是说，做空价差的最终美金收益率和开仓价格 P 无关，仅与杠杆数 L 和价差 r 有关。

#### 做多价差如何开仓

Q：当周合约 P₁ = 10，季度合约 P₂ = 9.5，r₁ = -5%，预测 r 会增大。

A：两边开仓，做空当周，做多季度。

1. 做空当周合约数字货币数量（对冲保证金）：N

2. 做空当周合约数字货币数量：( N * L - N ) / 2

3. 做多季度合约数字货币数量：( N * L - N ) / 2

最终数字货币数量 = 对冲保证金赚取数字货币数量 + 做空赚取数字货币数量 + 做多赚取数字货币数量 + 原有数字货币数量 = { N * ( P₁ - P₂ ) / P₂ } + { ( L - 1 ) / 2 * N * ( P₁ - P₂ ) / P₂ } + { ( L - 1 ) / 2 * N * [ P₂ * ( 1 + r₂ ) - P₁ * ( 1 + r₁ ) ] / [ P₂ * ( 1 + r₂) ] } + { N }

最终美金数量 = 最终数字货币数量 * P₂ = N * ( P₁ - P₂ ) + ( L - 1 ) / 2 * N * ( P₁ - P₂ ) + ( L - 1 ) / 2 * N * [ P₂ * ( 1 + r₂ ) - P₁ * ( 1 + r₁ ) ] / ( 1 + r₂ ) + N * P₂

最终美金收益率 = ( 最终美金数量 - 初始投入美金数量 ) / 初始投入美金数量 = { N * ( P₁ - P₂ ) + ( L - 1 ) / 2 * N * ( P₁ - P₂ ) + ( L - 1 ) / 2 * N * [ P₂ * ( 1 + r₂ ) - P₁ * ( 1 + r₁ ) ] / ( 1 + r₂ ) + N * P₂ - N * P₁ } / ( N * P₁ ) = ( L - 1 ) / 2 * { ( P₁ - P₂ ) / P₁ + [ P₂ * ( 1 + r₂ ) - P₁ * ( 1 + r₁ ) ] / [ P₁ * ( 1 + r₂ ) ] } = ( L - 1 ) / 2 * [ - P₂ / P₁ + 1 - ( 1 + r₁ ) / ( 1 + r₂ ) + P₂ / P₁ ] = ( L - 1 ) / 2 * [ 1 - ( 1 + r₁ ) / ( 1 + r₂ ) ] = ( L - 1 ) / 2 * ( r₂ - r₁ ) / ( 1 + r₂ )

也就是说，做多价差的最终美金收益率和开仓价格 P 无关，仅与杠杆数 L 和价差 r 有关。

#### 典型场景

在理论上，近期合约和远期合约的价差最终将收敛。因此，不论合约最终收敛于什么价格，只要价差收敛了，就可以盈利。下面这个表格展示出了价差收敛的三种情况。

|   都涨   | 开仓价格 | 平仓价格 | 方向 | 盈亏情况 |
| :------: | :------: | :------: | :--: | :------: |
| 当周合约 |    10    |    12    | 做多 |    2     |
| 季度合约 |    11    |    12    | 做空 |    -1    |
|  总盈亏  |          |          |      |    1     |

|   都跌   | 开仓价格 | 平仓价格 | 方向 | 盈亏情况 |
| :------: | :------: | :------: | :--: | :------: |
| 当周合约 |    10    |    9     | 做多 |    -1    |
| 季度合约 |    11    |    9     | 做空 |    2     |
|  总盈亏  |          |          |      |    1     |

| 一涨一跌 | 开仓价格 | 平仓价格 | 方向 | 盈亏情况 |
| :------: | :------: | :------: | :--: | :------: |
| 当周合约 |    10    |   10.5   | 做多 |   0.5    |
| 季度合约 |    11    |   10.5   | 做空 |   0.5   |
|  总盈亏  |          |          |      |    1     |

你可能注意到了，这三种情况的总盈亏都是相同的。这进一步验证了最终美金收益率推导的正确性。

### 网格交易讲解

由于价差总是围绕着 0% 上下波动，因此，我们可以在 0% 周围按指定的步长划分区间。对于每一个区间，都有一笔独立的资金。当升水行情时，价差达到区间的上沿时做空，价差达到区间的下沿时平仓。当贴水行情时，价差达到区间的下沿时做多，价差达到区间的上沿时平仓。

#### 为什么理论上价差一定会收敛

根据 OKEx K 线衔接规则：

> 1）次周合约交割时：老次周合约对接新当周合约，老当周合约对接新次周合约，季度合约不变；
>
> 2）季度合约交割时：老当周合约对接新季度合约，老季度合于对接新次周合约，老次周合约对接新当周合约；

说白了，就是当季合约迟早变成当周合约，当周合约迟早变成当季合约。在它们互相过渡的期间，总有一个时刻，两个合约的价值相等，此时理论价差为 0。

#### 典型场景

图中显示的是当季合约 EOS-USD-200925 和当周合约 EOS-USD-200724 在 2020 年 7 月 13 日至 2020 年 7 月 19 日间的价差。我们令区间步长为 0.2%，量化交易理论结果如下。

![EOS-USD-价差](/static/EOS-USD-价差.png)

| 单号 | 方向 | 开仓价 | 平仓价 | 盈亏 |
| :--: | :--: | :----: | :----: | :--: |
|  1   | 做空 | 1.2  |   1    | 0.2  |
|  2   | 做空 |  1   |  0.8   | 0.2  |
|  3   | 做空 | 0.8  |   -    |  -   |
|  4   | 做空 |  1   |  0.8   | 0.2  |
|  5   | 做空 |  1   |   -    |  -   |
|  6   | 做空 | 1.2  |   1    | 0.2  |

虽然单号 3 和单号 5 没有平仓，但是理论上价差一定会收敛，所以平仓只是时间问题。