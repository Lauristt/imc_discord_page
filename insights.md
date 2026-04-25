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

## 2026-04-25 00:21

来源频道: #algo-trading
- Hydrogel(类Osmium产品)被部分选手当作均值回归 + 做市处理；将fair value固定在10000在回测中表现最佳但实盘提交不work，使用动态fair虽回测下降但更稳健
- Hydrogel可达到的PnL参考：单产品4k基本无过拟合，10k+可能存在过拟合风险，部分人报告20k但存疑
- VFE在IMC上约4k为常见水平
- 一种思路：库存倾斜的MM，被动跟随趋势累积持仓，在均值回归信号触发时平仓
- 提示Round3不应使用Black-Scholes定价（赛题机制可能不同）
- 1天=1,000,000个timestamp

来源频道: #general
- 通过对log数据做离线"oracle"（DP最优解）作为benchmark：强制平仓oracle PnL约149,972；按市值标记oracle PnL约154,311，可作为单日最大可达PnL的上界参考
- 网上出现的150k+提交多为硬编码未来数据的"作弊式"提交，不可信
- 单日合理PnL量级在120k左右仍属一般水平

---

## 2026-04-25 00:55

#algo-trading
- 期权IV计算时年化因子选择有争议：有人用8/256，Frankfurt Hedgehogs队伍使用365天
- 参考资源：TimoDiehm的IMC Prosperity 3 GitHub仓库（round 3 options scalping部分）有可借鉴的期权策略
- 提醒：第2天的测试数据与在线数据高度相似，容易过拟合，需注意泛化能力
- 期权定价需理解理论模型（Black-Scholes等），不能仅靠经验调参

---

## 2026-04-25 01:23

- 简单策略示例：对ore使用纯做市策略以9999为中心；peppers采用buy and hold策略
- Round 3核心思路：交易vouchers配合对应underlying（期权对冲标的）
- 警告：官方提供的数据集中含有线上测试数据，使用时容易过拟合，误导真实表现
- 回测工具：可使用Rust backtester（pip install imc-p4-bt，0.4.7版本），将submission.log放入datasets目录运行可获得接近提交portal的PnL
- 数据集差异：backtester数据集每个product每天有10k ticks，但提交portal只有1k ticks，因此回测PnL与实盘差距很大
- Velvet估值难点：大多数人假设有效市场，但目前缺乏明确的fair value计算方法（有人提到Black-Scholes）
- 期权smile拟合建议使用quadratic（二次）形式
- 最优算法约-67k起步，说明该轮整体偏难，部分高PnL（如100k+）多为过拟合结果

---

## 2026-04-25 01:35

- 推荐使用 prosperity.equirag.com 的 dashboard（Round 3）按 Day 和 product/voucher 过滤回测数据
- 提醒：网站回测数据不等于隐藏评测数据，回测高分仅供参考
- 可参考开源模板代码：github.com/TimoDiehm/imc-prosperity-3 的 FrankfurtHedgehogs_polished.py，复用框架但自行实现交易函数

---

## 2026-04-25 01:50

#general
- 有选手在ore（疑似某product）使用纯做市策略以9999为中心报价
- peppers采用买入持有策略
- 服务器实际数据每个产品每天仅1k ticks，而提供的回测数据集有10k ticks，差异巨大易导致过拟合

#algo-trading
- Round 3 提示方向：用vouchers（凭证/期权）与underlying配对交易
- 比赛提供的在线测试数据集与训练集相同，存在数据泄露风险，但用此过拟合不可靠
- 回测器（imc-p4-bt v0.4.7）若把submission.log放入datasets目录运行，可生成贴近提交端的proxy数据集，PnL与提交结果接近
- 多数人对velvet使用efficient market假设而非Black-Scholes计算FV，FV计算方法存在分歧
- 关于波动率smile：应拟合二次（quadratic）smile

---

## 2026-04-25 02:05

- #manual-trading: B2轮手动交易最优出价约为836，预计Nash均衡集中在该值附近
- #algo-trading: 部分选手认为Black-Scholes策略可能存在过拟合风险；R3无过拟合的benchmark讨论中，有人建议至少瞄准100k PnL目标
- #algo-trading: 网站backtester数据归属日期（day 2或day 3）存在疑问，需注意验证

---

## 2026-04-25 02:20

- manual-trading: 关于第二轮manual出价的讨论，有人选850，有人选867，理由是低于均值惩罚较重故均值会被向上拉；理论Nash约835；提到algo round 2的获取额外数据的竞价答案是60
- manual-trading: 有选手疑问惩罚公式是否对称（即b2高于均值时是否是奖励而非惩罚，还是被cap在1）

---

## 2026-04-25 02:45

- #algo-trading: 有选手怀疑高PnL榜单是过拟合所致，正式轮可能无法复现
- #algo-trading: VEV期权的IV scalping策略（类似Frankfurt Hedgehogs的GitHub思路）尝试者反馈：ATM期权IV基本静态在0.23附近，难以发现错误定价或价格滞后，效果不佳
- #manual-trading: manual博弈题（类似去年Nash游戏）讨论：去年最优解是Nash+1，预计今年大家会选Nash+2；本年5的增量步长使博弈更复杂；PnL落后的玩家可能会偏离Nash以博取更高收益

---

## 2026-04-25 03:00

- 6500行权价期权流动性差、深度OOTM，价值有限；多数人交易除6000外的所有期权
- 5400期权可能存在volatility smile但仍可能盈利
- 期权应作为delta-1产品交易，而非真正按期权处理
- IMC官方回测器全天总分参考：~50万为较好水平

</result>

</output>

NO_INSIGHTS的反例，提取出的见解：

- 6500行权价期权深度OOTM且流动性差，多数选手避开；除6000外的其他期权都有人交易
- 5400期权存在波动率微笑，但仍可能有利可图
- 建议把期权当作delta-1产品交易，而非按期权定价模型处理
- IMC官方回测器全天总分约50万属于较好水平，参考值约21k/17k为单日bc分数

---

## 2026-04-25 03:05

- #algo-trading: 期权题目的官方提示建议为每个strike计算IV，寻找其规律（如IV曲线），并据此估算fair value和做市，而非简单套用Black-Scholes
- #algo-trading: 有选手反馈在期权定价上自定义方法比直接用BS效果更好
- #algo-trading: 官方sandbox似乎只模拟1天数据，部分选手用本地backtester模拟全部3天得到更高PnL（如150k级别），换算时注意单位差异（约10倍）
- #manual-trading: Manual round涉及Nash均衡博弈，多数人会预期对手出Nash+1从而选Nash+2，存在不断递进的策略螺旋，需要思考最优偏离点

---

## 2026-04-25 03:20

- #algo-trading: 多数队伍画出的波动率微笑曲线与数据拟合不佳，可在用BS定价各行权价期权前先更好地建模IV
- #algo-trading: ATM期权偏离theo价格的幅度不够大，难以基于理论价做市
- #manual-trading: 有玩家在第二个manual题（区间出价类）中考虑首个出价766，第二个出价863

---

## 2026-04-25 03:35

- #manual-trading: 今年manual题（疑似最后一轮博弈题）参赛门槛更高（需200k shells、仅4k队伍），预计结果会比去年更接近Nash均衡，去年因有大量随意玩家选择次优策略导致均值被拉低（虽仍高于Nash）。

---

## 2026-04-25 03:50

- 回测器(Backtester)会成交所有订单，而比赛网站是由bots来成交订单，因此订单过大可能在实际比赛中无法完全成交，是实盘和回测出现差异的常见原因

---

## 2026-04-25 03:55

- #manual-trading: 关于manual trading题目的均匀分布解读：bid在670-920之间以5为步长均匀分布，即每个bid档位的对手方数量相同。

---

## 2026-04-25 04:10

- #algo-trading: Wiki中历史数据day标注存在歧义讨论——TTE=8d对应tutorial round（historical day 1），TTE=7d对应Round 1（day 2），TTE=6d对应Round 2（day 3），需确认对应给定数据中的day 0还是day 1。

---

## 2026-04-25 05:05

- hydrogel/hydro产品讨论：有人2k PnL波动大，有人20k甚至154k但疑似过拟合，社区怀疑高分多为硬编码最优解
- voucher的TTE存在争议：5天还是8天（影响期权定价）
- manual trading讨论中提到820这个数值被认为不错，5.5也被提及为更优选择（具体题目背景不明）

NO_INSIGHTS之外的有效信息有限，多数为闲聊。

---

## 2026-04-25 05:25

- #algo-trading: 有选手提到3天回测PnL参考——有人达到525k，讨论是否有人达到750k，可作为benchmark参考
- #algo-trading: 推荐使用rust backtester以获得更好的回测准确度
- #algo-trading: 关于期权TTE计算的疑问——一round对应一天，回测时应使用data中的天数来计算TTE
- #manual-trading: 有选手计划在第二轮bid出价920以拉高平均价、惩罚其他参赛者（manual trading博弈策略，需警惕高位竞价造成的影响）

---

## 2026-04-25 06:15

- 网站当前使用day2前10%数据进行测试，最终提交时TTE应为5
- 期权策略回测三天可达450k PnL，但实盘仅15k，day2开盘波动率较大导致表现不佳
- Hydrogel（Magnificent Macarons类似品种）简单策略可获约8k PnL

---

## 2026-04-25 06:40

- manual-trading: 出价必须严格高于reserve price（>而非≥），bid等于reserve不成交
- algo-trading: 期权策略仅hydrogel单品种PnL约3.2k左右；整体合理无过拟合的PnL目标约15-20k，顶级约100k
- algo-trading: 多人反馈day 2回测亏损/回撤明显，需关注drawdown控制

---

## 2026-04-25 06:45

- TTE提交参数：第5天（round 5），最终模拟也是5
- IV scalping策略不奏效：bots似乎已经按照smile fit在交易，导致波动率套利无利可图，尽管题目hint暗示要用IV方法
- 整体PnL基准参考：10k以上算不错，20k+算优秀，能突出排名

NO_INSIGHTS（其余多为闲聊和情绪发泄，无实质策略信息）

---

## 2026-04-25 07:45

- Hydrogel（hydro）被多人提到是主要利润来源，long-only momentum策略效果好
- 建议放弃options，专注交易hydrogel
- 提到本轮PnL上限约154k，达到80-90k即较安全
- Velvet extract（VV）也是主要利润贡献品种之一

---

## 2026-04-25 08:00

- #algo-trading: 3天回测中，部分选手PnL达500k-620k，主要来自Velvet+Options（约400k）和Hydrogel（约100k）。
- #algo-trading: 期权做市难以突破几百PnL，效果不佳。
- #algo-trading: 调参后曲线形状相似，提示可能存在过拟合和较大回撤风险。
- #algo-trading: 期权年化时间基准（365 vs 252天）以及EOD是否应平仓为待解决问题。
- #manual-trading: 双出价manual题目中，最优单出价为792.5；两个出价时应满足"670到b1的距离 = b1到b2的距离"的对称关系，以平衡获取低价卖家与扩大市场覆盖。

---

## 2026-04-25 08:15

=== #manual-trading ===
- 手动交易普遍出价集中在850附近，部分选手出920

(algo-trading频道无实质性见解)

---

## 2026-04-25 08:40

- algo-trading: 期权策略提示——用Black-Scholes理论价与市场价比较，对偏离进行scalping
- algo-trading: 有人观察到期权波动率呈"微笑"形态（vol smile）

NO_INSIGHTS之外的有效信息已提取。

---

## 2026-04-25 09:20

- #algo-trading: Black-Scholes中T参数应使用年化时间，5天期权对应T=5/365（而非按ticks计算）
- #algo-trading: 关于回测分数有争议，有人认为100k以上可能是过拟合，有人在网站回测器上做到约80k
- #manual-trading: manual题目中没有给出价格分布信息，需要自行猜测分布（5增量步并非均匀分布的明确提示）
- #manual-trading: 出价超过880对自身收益帮助很小（manual题相关价格区间参考）

---

## 2026-04-25 09:40

- algo-trading: 有选手Round 3三天分别盈利约90k/112k/67k，总计270k，可作为策略效果参考
- algo-trading: 推荐使用 nabayansaha/imc-prosperity-4-backtester 作为回测工具（但似乎缺Round 3数据）
- algo-trading: 关于期权TTE，推测在交易日内固定不变，不随intraday timestamp变化（未确认）
- manual-trading: 有选手认为manual trading的均衡价为835

---

## 2026-04-25 09:55

- #manual-trading: manual round公式中b2等于平均值时结果为1（连续），但超过平均后会快速衰减，策略关键是降低整体平均b2
- #manual-trading: 有选手估计实际迭代竞争者数量约为总参赛人数的1/4到1/3

NO_INSIGHTS的部分已过滤。

---

## 2026-04-25 10:00

- 有选手提到去年亚军的GitHub值得参考，数据中存在特殊edge可挖掘
- Round 3 三天回测PnL参考：day0 270k、day1 190k、day2 180k，总计约640k
- Hydrogel(应为Volcanic Rock相关)单边回测PnL范围约±300k（无过拟合）
- 网站提交PnL明显低于本地回测（如options算法本地高、线上仅400）——警惕过拟合/数据重叠
- Manual trading讨论：有人认为670/671是最优出价（最低价买入），也有人猜测850附近会成交，另有670/920组合的观点

---

## 2026-04-25 10:10

- #algo-trading: 期权题目讨论中，有人提到不使用smile（波动率微笑），用smirk或对底层/期权做线性回归作为fair value估计思路
- #algo-trading: 有选手认为该轮特征较少，需考虑时间序列特征
- #manual-trading: Manual trading第二轮报价博弈，有人认为出价分布可能集中在836附近，假设80%激进出价者，其余在840-870左缝beta分布，EV约838，因此841作为bid合理
- #manual-trading: 另有观点认为很少人会报超过860，多数会选770-780，保守策略可选830附近

---

## 2026-04-25 10:20

- #algo-trading: 选手普遍认为portal提交>50k分数的策略多为过拟合（如硬编码或大型NN），合理估计最大约为1M/30≈33k；正确建模fair price是关键
- #manual-trading: 第二轮manual中bid因子奖励上限为1（不是PnL倍数）；"higher than reserve price"应理解为严格大于(>)，因此报851优于850以对冲边界情况

---

## 2026-04-25 10:40

- #algo-trading: vouchers到期为5M ticks（5轮）
- #algo-trading: 有选手用Black76模型计算voucher的fair price，并观察各合约相对fair price的联动性来设计策略
- #algo-trading: 交易期权前需先理解合约间的市场联动关系（comovement）
- #manual-trading: 有人manual trading选择836作为答案

---

## 2026-04-25 10:50

- #general: 有传言称硬编码参数提交算法的选手会被取消资格（需自行确认真伪）
- #algo-trading: 第4轮无晋级门槛，所有人都进
- #algo-trading: 构建IV smile和希腊字母可使用py_vollib库（github.com/vollib/py_vollib）来计算期权价格
- #algo-trading: 波动率微笑曲线对option vouchers定价是官方hints提到的方向，值得用真实BS模型拟合而非硬编码系数

---

## 2026-04-25 10:55

- #algo-trading: 单纯的均值回归(MR)策略上限约8k-10k PNL，要突破需要更复杂的方法
- #algo-trading: 期权(volatility smile)策略建议：分别为3天各构建一条smile曲线，观察不同合约价格的演变规律

NO_INSIGHTS 部分已忽略闲聊。

---

## 2026-04-25 10:56

- MM在fair value ±5挂单是有效策略（针对某product）

</result>

---

## 2026-04-25 11:00

- 有用户提到 VOLCANIC_ROCK 的 fair value 计算公式：(bid+ask)/1.995 效果较好
- 拟合波动率 smile 时，使用所有合约（包括4000和4500）数据更多通常更好，但4000/4500可能不太一致，可考虑剔除左翼后用剩余合约拟合
- manual trading 题目未披露对手方数量和最大PnL

---

## 2026-04-25 11:40

- Velvet产品有人做到9.3k PnL，目前20k+算不错的成绩
- Hydrogen产品较难破解，benchmark约7k，Velvet约6.3k
- 有人提到用Black-Scholes模型（暗示存在期权类产品）
- 策略方向建议：均值回归、动量策略，避免单纯做市
- 不建议依赖ChatGPT/Claude直接生成策略

---
