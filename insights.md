# IMC Prosperity 策略笔记

> 自动从选手Discord频道收集整理

---

## 2026-04-24 17:08

- Manual trading（投标类题目）：第二个出价用于捕获第一个出价未覆盖的剩余储备，因此第一个出价越高，第二个出价能捕获的量就越少，需权衡两个出价的关系。

---

## 2026-04-24 17:24

- #algo-trading: 本轮Round 3比往届更难，流动性也被故意调低；信号是多层叠加的，不像之前几轮那样单一明显
- #algo-trading: 当前提交PNL中位数约1k，均值约2.6k，最低-700k，整体水平较低
- #algo-trading: Volcanic Rock（火山岩期权）的IV smile据反馈不呈现完美smile形状，不能直接套用去年算法
- #algo-trading: 历来流传的"150k PNL"几乎是用动态规划(DP)对价格路径做最优硬编码交易得到的"完美交易"上限，非DP纯策略实际更接近150k以下；100k算合理目标
- #algo-trading: Tomas暗示提示信息也藏在Wiki和官方"behind the scenes"视频里（prosperity.imc.com网页下方），值得查看
- #algo-trading: 评估流程遵循historical/test/eval标准模式，最终holdout数据集独立测试，参赛阶段无法看到holdout表现
- #manual-trading: Round 4 manual为双bid竞拍机制，卖家分布均匀，交易方数量未知；bid2大家会保守拉高，因此忽略bid2单独最优化bid1接近最优，但联合优化能挤出更多利润
- #manual-trading: 注意非理性参与者会拉低平均bid2，影响平均惩罚机制下的策略选择

---

## 2026-04-24 23:27

=== #general ===
- 本轮交易所流动性被故意降低（与往届不同），需注意
- Hydrogel产品当前最高约15.6k PnL（网页提交），代码据说很简单
- Velvet存在分歧：部分人认为均值回归，但累积和显示并非MR，可能动量特征更强
- Round 3/4/5的PnL是累加的，而非独立计分
- 参考资源：prosperity.equirag.com 可查看PnL分布（中位数793，均值2639，最低-700k）

=== #algo-trading ===
- 警告：网传150k PnL多为overfit，"oracle最优交易"全产品在1k tick上限约155k，真实alpha不可能接近此值
- 防过拟合建议：完全不看day 2数据，用day 0+1优化，day 2作为held-out验证集
- 参数稳健性方法：在多维参数空间做grid search，找PnL梯度平稳的高原区域，取该区域的中心/平均参数而非单点最优
- 用heatmap做2D可视化，或用有限差分计算PnL对各参数的梯度，寻找全局最大且稳定的区域
- 期权round合理回测PnL基准：3天约300k起步，上限大概300k–800k
- 仅用波动率smile建模整体波动率效果不佳，可能需要其他方法补充

=== #manual-trading ===
- 本轮报价分布：均匀分布于670–920之间，步长5（即670,675,...,920）
- 第二个bid的惩罚仅作用于第二bid成交的PnL部分，不是整体PnL（参考wiki）
- 双bid博弈结构：bid2会填补bid1未捕获的reserves，因此bid1越高，bid2能抓的越少；联合优化与逐个优化结果不同
- 策略思路：先单独最优化bid1，再优化bid2，结果更保守；联合优化潜在收益更高但风险更大
- 预测多数人会保守地把bid2设得偏高，因此最优bid1会更接近"忽略bid2"的单点最优解
- 部分玩家选择极端组合如(1000,0)、(670,671)等；920作为bid2是常见保守选择
- Round 2 manual有48支队伍并列第一（当时观察）

---
