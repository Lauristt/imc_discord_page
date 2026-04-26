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

## 2026-04-25 11:45

- #algo-trading: 关于期权估值时间问题——TradingState中没有day字段，选手讨论如何在trader.py中确定当前是第几天（这对期权定价的剩余到期时间计算很重要）

</result>

NO_INSIGHTS

等等，上面已经提到了一个有价值的问题点。重新整理：

- #algo-trading: 期权估值需要知道当前比赛日（day），但TradingState中不包含day字段，需要自行解决（如通过timestamp推断或硬编码）以正确计算剩余到期时间

---

## 2026-04-25 12:06

- 期权策略关键：识别某些期权持续被错误定价，并用它们进行delta对冲（platyturtle提示）
- 有人认为期权的mispricing较为边际（marginal），盈利空间有限

---

## 2026-04-25 12:10

- #algo-trading: 有选手采用统计套利风格策略，基于定价模型给出的fair value交易偏离值
- #algo-trading: 期权定价使用py_vollib库计算greeks/价格/IV曲面，采用Black-Scholes模型
- #algo-trading: 关于隐含波动率维度的讨论——由于只有一个到期日，IV曲面实际上是2D（仅依赖moneyness/strike），而非3D/4D
- #algo-trading: 有选手通过观察图形直接硬编码参数获得不错PnL，但其他人质疑这种方法在完整数据集上可能失效

---

## 2026-04-25 12:25

- 统计套利思路：不要只看价格相关性，应该看spread之间的相关性
- HG和VEV价格相关性约0.3，但直接价格相关性未发现可交易机会（需转向spread相关性分析）

---

## 2026-04-25 12:52

- 第5轮算法基准（online）传闻约为20k PnL；本地backtester达到20k相对容易，线上较难

</result>

</invoke>

---

## 2026-04-25 13:00

- Hydrogel可尝试均值回归策略，类似osmium
- 每份option合约对应1单位velvets
- 不交易也是一种选择（无损失）

</result>

---

## 2026-04-25 13:12

- Velvet产品对PnL贡献较大，是稳定的收益来源
- Hydro产品被认为难以预测/管理，部分玩家算法在该产品上表现为零或亏损，被视为"赌博"性质
- 不同选手对Hydro看法分歧：有人认为简单算法即可获利，有人认为难以把握

=== #manual-trading ===
- Manual交易中每个counterparty只能成交一次
- 报价机制：bid1只与价格落在bid1和bid2之间的对手方成交（分层报价的成交规则）

---

## 2026-04-25 13:15

- VEV价格区间5000-5200对PnL最有利可图
- 部分高分玩家通过hardcode特定测试日来刷分，存在过拟合嫌疑
- 有选手放弃交易hydrogel，专注其他产品

---

## 2026-04-25 13:35

- Rust backtester和prosperity4btx结果不一致的问题（可能存在实现差异）
- 选手反馈：策略在backtester上40k/天，但sim中只有10k，sim数据可能与backtester存在偏差，策略易过拟合sim数据
- 现有backtester存在已知偏差，需注意
- Hydro相关策略在短期回测表现好，但3天完整回测可能巨亏(-667k)，需警惕短期过拟合

---

## 2026-04-25 13:37

- 模拟环境（sim）使用的是第2天CSV的前1000个tick数据
- 最终提交的实际模拟为100万个tick
- 部分选手反馈本地3天数据回测正收益但提交后变负，提示回测方法可能有误（如未考虑前瞻偏差或过拟合）

---

## 2026-04-25 14:00

- #general: 有选手提出策略思路：可能不应做市而应做主动taker；关注期权与标的的价格错配；利用均值回归；通过订单流不平衡和大成交量信号判断方向
- #manual-trading: 关于round 2 manual的优化提示——b1和b2与b2平均值挂钩，若看好b2则可超过756；有人给出报价(670, 920)

---

## 2026-04-25 14:20

- 某用户分享了VELVETFRUIT_EXTRACT及各行权价（4000-5500）VEV期权三天的成交量数据，可用于分析期权流动性分布（5000-5100行权价成交量最大，深度OTM的5500最少）
- 讨论指出期权策略本质上等价于在标的上做均值回归 + 交易5000-5500行权价的期权组合
- 多位选手反映IV scalping策略难以稳定盈利，存在过拟合风险

---

## 2026-04-25 14:35

- #algo-trading: 期权没有到期日（expiry），按fair value结算，可参考wiki
- #algo-trading: 选手反馈不使用vol smile拟合的简单策略反而表现更好，使用smile拟合需要做得很精细才有优势
- #algo-trading: 网站实盘和本地backtester表现差异较大，存在较大随机性

---

## 2026-04-25 14:38

- 期权策略：154K的高分疑似通过硬编码时间戳和价格实现"完美交易"，非通用策略
- 期权策略：使用smile（波动率微笑）拟合可达约31K PnL，正常无过拟合的预期上限约33K
- 期权IV参数：可使用mid IV作为基准
- 期权策略方向提示：mean reversion（均值回归）可能是有效思路
- 部分选手认为单纯依赖smile拟合并非最优策略，需要结合其他方法

---

## 2026-04-25 14:45

- Round 5无硬编码策略的回测PnL参考：约30k（一位选手）、25k（另一位，但hydro几乎无盈利，存在改进空间）、70k（另一位声称）

---

## 2026-04-25 14:48

- 仅对所有资产做market making (mm)，在网站上平均能达到约25k的成绩
- 有人怀疑高分（70k+/100k）是硬编码的结果，且可能在最终结算时崩盘

NO_INSIGHTS之外的有效信息已提取。

---

## 2026-04-25 15:00

- hydrogel采用均值回归策略可获得约8k收益
- 有选手考虑不交易velvet fruit extract以最大化利润
- 短期模拟为10万点、间距100；最终模拟为100万点、相同间距

</result>

NO_INSIGHTS 的反面，已提取上述见解。

---

## 2026-04-25 15:10

- Manual trading界面字段说明：网站UI中的"lowest bid"对应bid 1，"highest bid"对应bid 2

---

## 2026-04-25 15:48

- #algo-trading: 154k是对最优交易做DP（动态规划）能得到的精确收益上限值
- #algo-trading: 使用神经网络(NN)可以明显提升策略表现
- #manual-trading: manual trading出价建议(675, 920)，因为没人能在670清算，b1需要高于他们的清算价；如果第二出价选920，第一出价至少应为675，期望收益约24.5k

---

## 2026-04-25 15:55

- manual-trading: 提示在判断条件时注意 `>` 与 `>=` 的区别，b1=670 可能因为该边界条件处理而成为错误选择

</reasoning_effort>

- manual-trading: 提醒注意 `>` 与 `>=` 边界条件的区别，b1=670 可能因此判断失误（manual交易区间选择需谨慎处理临界值）

---

## 2026-04-25 16:09

- algo频道：VOLCANIC_ROCK_VOUCHER（velvetfruit extract options）是欧式期权，不能提前行权；有人用波动率smile策略获利，建议交易除6000和6500行权价之外的所有strike
- algo频道：简单方法在extract options上可获得约85k PnL；DP方法被认为是hardcode过拟合
- manual频道：B1拍卖中，安全低风险出价675/920可得24-25k；出价920意味着按920成交利润为0，需平衡保守与激进
- manual频道：B1出价逻辑——填充所有低于你出价的卖家，盈利约为(出价-真实价值)*下方卖家成交量

---

## 2026-04-25 16:19

- #algo-trading: 期权类衍生品不应简单当作均值回归资产处理，要思考真正会均值回归的是什么（如IV/价差），否则策略会突然失效
- #algo-trading: 有玩家提到改用gamma short替代gamma hedging
- #algo-trading: Round 3不评估pepper和osmium
- #manual-trading: 关于平均值估计区间讨论：790-830、840-880或870-885不等；cubic惩罚显著压低EV，需考虑博弈论因素（多数人会直接照搬Claude给出的答案）

---

## 2026-04-25 16:20

- Round相关讨论中提到，154k的高PnL多为过拟合结果，无过拟合的真实上限约为10k

---

## 2026-04-25 16:24

- Hydrogel: 真实合理收益约10-20k，40k+属于过拟合（有人无过拟合可达15k）
- 154K收益被认为是过拟合结果
- Manual trading的bid2 penalty被认为是对整体PnL一次性应用，而非每笔交易乘算

---

## 2026-04-25 16:25

- #algo-trading: Round相关讨论中提到HG（可能是某product）在官网回测约20k PnL，开源回测50-60k/天；过拟合上限约40k
- #algo-trading: 有人提议尝试gamma scalping策略（用于期权类product）

---

## 2026-04-25 16:35

- TTE（到期时间）在一天内递减，开盘为5，收盘为4

</example>

---

## 2026-04-25 16:39

### #algo-trading
- HG（hydrogel）策略思路：选择合适的anchor price可以稳定获得5k+利润
- 警告：若回测中HG呈下行趋势，仅靠持有负库存就能盈利，但实际sim中并非单向随机游走，此类策略可能过拟合
- TTE在回测中疑似固定为5（期权相关）

### #manual-trading
- manual题（猜数字类）预测：大量队伍会用AI得到约830的答案，部分会保守选880+，预计平均在850附近

---

## 2026-04-25 16:45

- Resin（推测，10k价位品种）历史回测中围绕10000做均值回归可获利，但实盘网站上价格在累积负库存后不再上穿10000，导致策略失效
- 该品种更适合用做市策略而非纯均值回归

---

## 2026-04-25 16:49

- 期权数据中部分合约在trades.csv里成交极少（1次或0次），导致rust回测的PnL与官网结果差异大；可用历史成交频率估算实测期权成交概率
- IV scalping利润低且不稳定，期权链中无明显套利机会，高PnL基本来自方向性押注（带赌博性质）
- 部分高分选手是通过硬编码timestamp在最佳点位买卖，实盘round 3不会使用day 0/1/2数据，硬编码者会失效
- MM挂单在期权上基本无法成交，不要期待大量被动成交

---

## 2026-04-25 16:54

- MM订单成交困难，本轮MM策略表现不稳定（特别是4500行权价期权几乎无成交）
- 一种简单策略：直接对所有期权和标的做多/做空（不涉及VOLCANIC_ROCK/gel），结果两极分化
- Day 3数据不会在比赛进行时显示，最终用最后一次成功提交在Day 3上跑分
- Manual trading出价规则：出价必须严格高于reserve price才被接受（如reserve 800则需出801）

---

## 2026-04-25 17:04

#algo-trading
- 本轮期权套利在考虑bid-ask spread后基本不存在
- Vega和Gamma策略效果不佳，仅Delta方向性策略相对可行
- 标的资产价格疑似为编码的随机游走
- Bot流量太少不足以做市
- 公允价值估计是关键难点：若开盘价格下跌5-10%，fair value估计会失效导致策略崩溃
- 期权链中的"edge"与期权本身性质无关
- 部分顶部成绩（~150k）疑似通过硬编码完美未来信息的方式获得，可能在最终结算被剔除
- 建议warmup阶段可以硬编码anchor_fair_prices

---

## 2026-04-25 17:15

- algo-trading: 期权4000 strike策略在回测(27k)和实盘portal(30k)表现不一致，存在过拟合风险
- algo-trading: 历史数据3天表现差异大(day1最高,day2最低)，因中间段数据特殊容易过拟合
- algo-trading: 本轮position limits较高，鼓励对小信号全仓押注，变成方向性赌博
- algo-trading: 有人尝试ARMA做hydrogel/macaron类产品，效果不佳；mean reversion也未达到他人宣称的水平
- algo-trading: 增大仓位上限(100→300)可线性放大PnL，但同时放大风险

---

## 2026-04-25 17:15

- algo-trading: 4000 strike期权回测盈利但在portal上表现较差(27k vs 30k)，可能存在过拟合风险
- algo-trading: 第3轮历史数据3天的PnL分布通常是Day1最高、Day2最低（中间那段导致）
- algo-trading: 本轮position limits较高，鼓励选手在轻微信号上重仓博弈，增加了过拟合诱惑
- algo-trading: 网站数据有一些异常dips，更容易过拟合
- algo-trading: hydrogel(volcanic rock相关)有人尝试ARMA无效，均值回归也未得到高分
- algo-trading: 高仓位限制被认为是为了 disincentivize full delta hedge

---

## 2026-04-25 17:40

- VEF（vouchers）的提示是隐含波动率（implied volatility），可用于定价
- 本轮（hydro/options相关）过拟合风险高，IMC官方回测器与开源回测器结果常不一致
- 不使用提示也能获得高VEF收益的策略很可能是过拟合
- Manual交易中各项独立服从均匀分布（每个有自己的分布参数）

---

## 2026-04-25 17:55

- #general: bot提示使用IV（隐含波动率）进行交易
- #algo-trading: 硬编码TTE（到期时间）应该不会被取消资格，硬编码规则主要针对基于tick的值
- #algo-trading: Hydrogel（氢凝胶）在3天回测器上有竞争力的目标PnL约100k
- #manual-trading: 关于Nash均衡的猜测值，有人提出819而非836

---

## 2026-04-25 18:20

- 不使用方向性押注的情况下，3天回测PnL大约在53k左右（对应网站约5k/天），超过此值通常意味着承担更高风险或过拟合
- 方向性押注（directional bets）可以人为放大PnL，但风险也相应上升
- 将trades数据文件喂给回测器会让无方向性策略的PnL进一步降低
- 网站使用的数据与回测器相同，但有选手认为网站上的市场regime与回测器不同
- 有选手报告无过拟合情况下回测达到72k、88k、108k

---

## 2026-04-25 18:35

- #algo-trading: Hydrogel单产品PnL范围观察——网站测试约16k，全日满量级有人达9k-150k，极端报告100M存疑；网站回测约为单日1/10数据
- #algo-trading: 数据集若使用受控随机种子加噪则无法完全复现，但可通过贝叶斯推断、置信区间等方式推测生成规律
- #algo-trading: 机器人行为难以100%精确模拟，只能做合理近似
- #manual-trading: 手动交易解空间稳定（参数小变化结果差异极小），b2_avg影响不大；排名差异主要取决于对手方/成交量分布而非解本身
- #manual-trading: 有人提交915-920区间重复20次作为参考报价

---

## 2026-04-25 18:35

- algo-trading: 历届IMC Prosperity 2和3的算法交易总PnL大约只有20-35万，过高的回测分数很可能是过拟合
- manual-trading: 第二个bid在最优值附近PnL曲线较平坦，但若出价低于最优值PnL会急剧下降（出价宁高勿低）

---

## 2026-04-25 18:50

- #algo-trading: 有选手提到通过研究比赛数据的生成方式（而非猜种子）可以获得有价值信息，暗示价格可能由正弦函数等周期性函数合成
- #algo-trading: 有人讨论circuit breaker（熔断机制）相关策略，得分在400-550区间
- #algo-trading: 有人询问IV scalping（隐含波动率剥头皮）策略是否有效，暂无明确回应
- #algo-trading: 有选手询问最终评测的价格是否会从历史数据第2天的结束价格继续延伸

---

## 2026-04-25 19:20

- #manual-trading: 本轮manual交易讨论中，忽略penalty的最优出价点为836（可通过简单优化计算得出）；考虑到他人会略微抬高，预估均值在841左右；若出价860则可能拿不到860对手方
- #manual-trading: 存在博弈论思路——若大家都猜841，实际可能会更高（需反向思考）

NO_INSIGHTS其余频道无实质策略信息

---

## 2026-04-25 19:30

- #algo-trading: 合法非过拟合策略在IMC官网OOS最高约50k；硬编码/DP方案因seed变化和regime shift在OOS上难以存活，IMC据称会封禁硬编码
- #algo-trading: VOLCANIC_ROCK_VOUCHER相关策略提到IV scalp、smile inversion、residual scalp等思路
- #algo-trading: 数据生成过程被认为是两个delta-1产品上的均值OU过程

---

## 2026-04-25 19:35

- algo-trading: IMC官网回测PnL超过30k可能存在过拟合嫌疑，超过50k几乎肯定是hardcoding/过拟合；3天回测合理目标约300k+
- algo-trading: 回测中drawdown极小也是过拟合的可疑信号
- algo-trading: 建议用1k tick数据验证均值回归策略，避免过拟合
- algo-trading: 可在IMC网站上部署简单MM策略来观察bot行为，估算成交上限
- manual-trading: 在manual交易出价中，许多人采用"+1"策略（如851等），导致整体underbid趋势

---

## 2026-04-25 19:55

- #manual-trading: 有人提到本轮manual最优出价为920双边，惩罚是乘法因子（非扣减）
- #algo-trading: 讨论中提示：回测无drawdown通常是过拟合信号；若策略backtest高利润但实盘负PnL，反向操作可能有效；可考虑将前一天末尾的信息编码传递到下一天
- #algo-trading: 有人尝试用IV和Black-Scholes（期权定价）但得到负PnL，提示该方法需谨慎使用

---

## 2026-04-25 20:05

- TTE（到期时间）讨论：day -2为7，day 0为8，day 1为5（即按day 0=8递减计算，day 1对应剩余5天）
- 有选手反映在Rust回测器上结果不错，但提交到官网PnL很差，怀疑过拟合回测器
- 有人认为榜单上35k的PnL可能过拟合，不会在回测中线性放大到1050k
- Hydrogel packs策略讨论：有人用短期均值回归，有人选择buy and hold

---

## 2026-04-25 20:20

- #algo-trading: 警告不要为网站PnL过度优化，10k timestamps数据无法反映全貌，容易过拟合
- #algo-trading: Day 2回测中均值回归策略与趋势冲突，表现较差
- #algo-trading: 部分选手使用Rust编写回测器
- #algo-trading: 策略思路：hydrogel用均值回归，velvet voucher用IV（隐含波动率），但PnL不稳定，后期会回吐
- #manual-trading: 关于B2出价猜测：主要集中在841或846附近，AI推荐价格可能在836左右，可作为博弈参考

---

## 2026-04-25 20:40

- #algo-trading: 官方网站回测（25k级别）比第三方完整3天回测（500k+）更具参考价值，因为TTE（到期时间）在第三方backtester中未正确建模
- #algo-trading: 每次执行有时间限制
- #algo-trading: 推荐的开源回测器：prosperity_rust_backtester (GeyzsoN/GitHub)
- #algo-trading: 使用backtester仍可帮助识别策略中的反趋势弱点
- #manual-trading: 每个卖家的reserve price是独立的均匀分布随机数，不是只投b1就行

---

## 2026-04-25 20:56

- 期权交易关键在于moneyness（价值状态），可基于此设计策略，理论可达53k PnL

---

## 2026-04-25 21:11

- 期权策略关键提示：使用 moneyness（行权价相对标的的位置）作为参数化变量
- 有选手观察到 implied vol 曲线相对平坦的现象（在询问是否普遍）

---

## 2026-04-25 21:15

- OTM期权可能由具有泊松流的机器人提供报价
- 期权moneyness影响定价方式与去年不同（暗示需要重新分析隐含波动率/曲面）
- Hydrogel（应为magnificent macarons相关或新产品）有30k左右盈利潜力
- 网站PnL超过30-40k可能是过拟合/硬编码的结果
- 有选手考虑买入并持有5400 call作为方向性策略

---

## 2026-04-25 21:21

- Round 5 manual/contract题：有选手提醒不要硬编码特定合约，应使用动态筛选逻辑，否则次日可能出现大问题（暗示数据/参数会变化）
- VEV 5300合约有选手获得约4.3k收益

---

## 2026-04-25 21:25

- IMC网站模拟仅测试第2天的前1/10时段（time 20000-21000），而本地回测器覆盖day 0/1/2完整时间（0-30000），两者结果差异需注意
- 有选手认为本轮总分超过60k基本属于过拟合/硬编码结果

---

## 2026-04-25 21:26

- 期权定价讨论：TTE（到期时间）在使用smile逻辑或BS公式计算IV时才有意义；需自行追踪日期变化
- 经验提示：若不过拟合，回测曲线出现回撤是正常现象

---

## 2026-04-25 21:35

- #algo-trading: 有人通过bug整轮做空hydrogel在day2前1000 ticks获利10k，但其他日子会大亏，说明hydrogel做空策略不稳定
- #manual-trading: 选手对manual题目最终答案的猜测分别为751/836和670/920，后者被认为可能接近中位数（具体题目背景未明）

NO_INSIGHTS之外的信息有限，主要是PnL闲聊。

---

## 2026-04-25 21:37

- #algo-trading: Hydrogel均值回归思路——基于N期移动均值计算z-score，过高则卖、过低则买
- #algo-trading: Volcanic voucher期权的DTE被讨论为5天
- #manual-trading: 关于Nash均衡讨论，有人认为最优值约794，另有人认为22k队伍下Nash均衡为836，可能加价+5到+10

---

## 2026-04-25 21:40

- #manual-trading: 关于manual trading的Nash均衡讨论，b1的Nash值约为836，对应b2为751；部分选手考虑出价高于Nash（如900或919）以略微偏离最优来获取更多收益
- #manual-trading: 约30%的队伍实际完成了round 1和2，参赛队伍数从22k降至约4100

---

## 2026-04-25 21:50

- #algo-trading: 有选手反映本轮回测器与官网实盘表现差异较大（回测好实盘差，反之亦然），原因可能是Day2前10%的数据regime与其余部分差异显著，回测器参考价值有限。
- #algo-trading: 顶级队伍本轮基准PnL约200k。
- #manual-trading: 本轮manual的Nash均衡点约在836，但若足够多人选择900+，低于该值的会吃亏；关键在于多少人会"故意拉高"以及拉到900需要多少人才能显著抬升平均值。

---

## 2026-04-25 22:05

- #algo-trading: 有人提到hydrogel不做mean reversion，而是等待信号触发再交易；3天回测约90万PnL但方差较大
- #manual-trading: 关于manual题（疑似第二价格竞猜）讨论Nash均衡，836及以上均为Nash均衡点，但有人认为836作为Nash过于乐观，收敛后的Nash不在836

NO_INSIGHTS（其余消息多为闲聊或无实质信息）

---

## 2026-04-25 22:12

- #manual-trading: 关于manual交易的bid选择，有玩家投票显示团队平均b2在830-840区间，有人选择847；b1可能下调至750；普遍认为带有赌博性质
- #algo-trading: IMC网站的AI顾问对所有角色给出的建议本质相同，仅措辞不同，被认为没有实质价值

---

## 2026-04-25 22:15

- #algo-trading: IMC官方backtester使用day 2数据的前10%，需注意避免对该段数据过拟合
- #manual-trading: 本轮manual出价920几乎必中但margin为零（去年类似情况，跟随高价者亏损）；847被认为是相对平衡的最低出价点，但仍高于多数队伍的bid2平均水平
- #manual-trading: 经验提示——不要盲从聊天频道里的高价建议（往往是trolling），保持理性出价更可能取得好成绩

---

## 2026-04-25 22:17

- #manual-trading: 关于manual trading的bid 2，讨论中提到的Nash均衡参考值在670-832附近；有人提示847是一个权衡点（高于此面临惩罚风险，低于此则收益较低）
- #manual-trading: 提醒注意论坛中他人公开建议的出价数字（如847、920）可能是误导，真实策略是博弈论意义上的"略高于公开建议价"，需谨慎参考公开喊价
- #algo-trading: 选手讨论显示portal上日均盈利约16.6k即可达到约500k的目标，可作为不过拟合的合理收益基准

---

## 2026-04-25 22:20

- #algo-trading: 有人提到针对hydrogel（应为round相关产品）通过硬编码（hardcode）能获得约3M收益，500k可能并非上限，存在显著的过拟合/硬编码空间
- #manual-trading: 本轮manual为出价博弈，越接近/略低于平均值表现越好，高于平均反而PnL更差，因此选手倾向保守报价（如900）
- #manual-trading: 提醒注意有顶尖选手运行Discord爬虫抓取报价信息，频道里故意诱导高报价（如920）属于反向操纵行为，需独立判断

---

## 2026-04-25 22:22

- #algo-trading: 真实回合的tick数是网站测试的10倍，因此本地backtester（开源Rust版本）的PnL通常比IMC网站显示的高约10倍，这解释了两者差异
- #algo-trading: Volcanic/Velvet fruit策略PnL参考：网站约300k，部分选手33k级别被认为可能过拟合
- #manual-trading: Discord抓取的猜数答案被大量meme值（如920）严重偏斜，需剔除异常值，但整体仍不可靠
- #manual-trading: 有人指出在reserve价上下出价才有效；故意误导他人不会真正"击败"对手，反而抬高均值增加自己亏损概率

---

## 2026-04-25 22:30

- 回测器：可fork jmerle的Prosperity 3版本，让Claude调整适配Prosperity 4
- 回测与实盘差异：回测700k实际可能只有580k左右，存在明显衰减
- 策略建议：先在底层资产上做好mean reversion，再去做期权策略
- Hydrogel(?)产品有人获得约8.5k收益，被认为偏低

---

## 2026-04-25 22:32

- #algo-trading: 有人在Hydro产品上跑IV smile策略，PnL在10k-90k区间不等，700k为高位回测结果（可能过拟合）
- #algo-trading: prosperity4bt回测器假设市场单先于他人成交，但PnL影响可忽略
- #algo-trading: 推荐的回测工具仓库 github.com/nabayansaha/imc-prosperity-4-backtester
- #manual-trading: 关于B1价格讨论，有人认为B1独立于B2为751，被反驳
- #manual-trading: manual题机制讨论：交易未成交不得分，成交则从XIRECS扣除支付金额

---

## 2026-04-25 22:37

- #general: VOLCANIC_ROCK_VOUCHER (vev) 可用VWAP并交易其偏离值，单此策略可获约6k收益
- #algo-trading: hydrogel(可能指某product)中存在趋势但较难识别
- #manual-trading: 本轮manual交易买入价800卖出价上限920即可盈利，无法亏损但有惩罚（scalar形式）

---

## 2026-04-25 22:40

- #general: 有选手认为均值回归策略效果好于做市(MM)；做市期权可能是当前可挖掘的方向
- #algo-trading: 可通过IMC网站测试数据日志中的trader IDs进行反向工程构建alpha信号（训练数据中无此信息）
- #algo-trading: 提到IV呈现平直线（可能用于波动率交易策略参考）

NO_INSIGHTS（部分）但已有上述要点。

---

## 2026-04-25 22:42

- #general: hydrogel 的订单簿中存在做市机器人，可通过观察订单簿提取"true price"指标
- #general: 期权定价方面有人推荐使用二项式模型（binomial）优于 BS
- #algo-trading: 关注完整回测的 PnL 而非 web preview 的 1k tick PnL，预览 PnL 意义不大
- #algo-trading: VolcanicRock（vev）底层资产用 VWAP z-score 策略（不交易期权）可获得约 6k PnL，但随机种子变化可能导致策略失效
- #algo-trading: 提交时需注意随机种子敏感性，避免过拟合

---

## 2026-04-25 22:48

- #general: VEV策略思路——用VWAP偏离（z-score）作为信号，期权部分在ITM时加杠杆放大信号，OTM在高IV时做空（该选手称获得18k）
- #algo-trading: Hydro packs（疑似某product）本地回测合理PnL约为30-40k/天
- #manual-trading: Round 2顶尖队伍在speed上的配置约在40%左右（待确认）；speed有较重的惩罚

---

## 2026-04-25 22:50

- 硬编码z-score策略在第3天数据上可能失效，需谨慎（来自#general）
- 回测930k为不错成绩，但需在3天数据上都表现良好才稳妥（来自#general）
- 本地回测分数与网站分数比例参考：本地40k约对应网站4k（来自#algo-trading）
- 各模块得分参考：hydro 2k、volcanic（vev）6k、options 10k；其中hydro偏低，options可达较高水平（来自#algo-trading）

---

## 2026-04-25 22:58

- #manual-trading: 本轮manual的设计意图是用cubic惩罚右偏bid2分布，吓退不愿做数学的玩家；惩罚不是世界末日，承担惩罚去抢更高价位可能更优
- #manual-trading: bid2的PnL依赖于从bid1吸收的对手分布，两个bid之间存在非平凡关联，不能独立优化
- #algo-trading: voucher (期权) 用Black-Scholes拟合效果差，smile难以建模；有人建议直接买入持有到期作为简单策略

---

## 2026-04-25 23:00

- prosperity4btest这个python wheel包会把你的trader源代码发送到第三方API，存在代码泄露风险，建议自己写backtester
- 自己写backtester并校准误差是最可靠的方式

NO_INSIGHTS的其他闲聊已忽略

---

## 2026-04-25 23:05

- algo-trading: 有选手提到做空 Volcanic Rock Voucher (velve) 作为策略，称获得15k收益
- algo-trading: 提及"4个trader"中有可利用的，暗示通过分析bot/trader行为可以套利

NO_INSIGHTS 之外仅以上要点。

---

## 2026-04-25 23:10

- #general: 可视化工具链接 prosperity.equirag.com
- #algo-trading: voucher仓位限制为每个strike各300（非总和300）
- #algo-trading: 有选手在VOLCANIC_ROCK_VOUCHER_6000赚252k、6500赚215k，提示高行权价option可能有较大利润空间
- #algo-trading: 资产和算法将延续到接下来的两轮

---

## 2026-04-25 23:13

- hydrogel(资源)上有人通过承担一定风险将PnL刷到20k-35k；从2k提升到5k比从20k到25k更难
- iacobus提到一个微观结构信号：当L1 mid - L2 mid >= 1时，L1会均值回归约4个tick
- portal数据是backtest的子集，所以选手更关注backtest PnL

---

## 2026-04-25 23:23

- 波动率微笑(vol smile)拟合策略讨论：部分人对每个timestamp拟合抛物线，也有人认为时间太短、流动性太小，变化很小，可以每几千个timestamp更新一次而非每个timestamp都拟合
- 有时vol smile呈现"frown"形态（倒置）
- 移动止损(trailing stop loss)在比赛中表现不错

---

## 2026-04-25 23:35

- #manual-trading: 手动交易题被视为权衡问题——数学最优解（最高PnL）会带来惩罚，需在PnL与避免惩罚之间权衡找到Nash均衡点
- #manual-trading: 可参考Round 2算法竞价分布作为参照，若trolls多则竞价会异常偏高；实际Round 2结果接近Nash均衡，说明多数参赛者理性

---

## 2026-04-25 23:40

- manual-trading: 关于第二轮bid的平均值预测讨论，认为不会低于835，估计在835-867之间

</result>

NO_INSIGHTS（实际有少量信息，输出上面那条）

---

## 2026-04-25 23:48

- #algo-trading: 对于本轮3天数据，由于数据量少容易过拟合，portal backtester可靠性受质疑
- #manual-trading: B1和B2两个出价并非完全独立，B2只能成交B1未成交的部分；最终结果主要取决于平均出价；讨论中有人认为710是B1最优，669是另一个候选

---

## 2026-04-25 23:58

- backtester在本轮（涉及期权TTE）可能不可靠，因为实盘中天数在递增，而回测TTE是固定的
- 关于vol smile拟合，需要注意timestamp选取问题

NO_INSIGHTS（manual频道无实质信息）

---

## 2026-04-26 00:09

=== #manual-trading ===
- 关于manual trading reserve价格机制讨论：算法可能为每个价格判断"我是否低于b1"等，逐步比较；reserve可能从分布中随机选取
- b2出价需要考虑其他人的分布，做优化决策
- 有人建议最高出价应在915+以避免惩罚

=== #algo-trading ===
- 100k timestep回测PnL参考：单日约300k-650k为不错水平，三天合计约100万属合理范围
- 早期轮次产品的数据在当前回测中为null
- 网页可视化的PnL参考价值有限

NO_INSIGHTS（其余多为闲聊和工具讨论，无实质策略内容）

---

## 2026-04-26 00:14

- rust回测器(GeyzsoN/prosperity_rust_backtester)与IMC官方回测一致性约0.5%，被多数人采用
- 实盘PnL大致区间：最高130-150K/天(rust bt)，现实约100K，IMC portal上30-60K
- 本轮PnL不再是线性的，不能简单将web PnL乘10来估算总PnL
- 建议自建backtester，用submission logs和round数据校准，否则回测无意义
- Manual trading第二出价的penalty是对bid2 PnL的乘数，并非影响整体PnL；915出价基本无效
- Manual较优解参考：bid1≈751，bid2≈841

---

## 2026-04-26 00:15

- #algo-trading: 警惕look-ahead bias——如果IMC回测PnL曲线与DP曲线高度吻合，可能存在前视偏差问题
- #algo-trading: 1M PnL在回测中不现实，建议用1k样本回测对比网页结果验证一致性
- #manual-trading: manual交易题目中bid 1、bid 2与平均bid 2存在公式联动关系，可通过对其他人bid 2的假设来估算平均bid 2

---

## 2026-04-26 00:19

来源: #algo-trading
- 有人声称在Rust回测器上3天合计达到1.4M PnL，但多数人认为不现实，估计上限在150–400k之间
- IMC自带回测器使用的衡量方式与Rust回测器不同，1k级别的PnL在IMC回测器上能对上
- Portal提供的100k ticks数据具体对应哪一天尚不明确
- 有人提到"DP oracle schedule"图，提示自己的资金曲线不应长成那样（可能暗示过拟合特征）
- 关于6000和6500行权价voucher是否值得交易仍有分歧

来源: #manual-trading
- Manual交易涉及两轮报价(b1, b2)：第一轮bid若高于某counterparty的reserve price则成交，否则进入第二轮；counterparty数量未公开
- 最优b1依赖于平均b2，因此估计平均b2更关键
- 报价值以5为增量，连续分布/多峰分布拟合效果有限，实践中用对若干合理出价(如836、850等)做加权平均近似
- 有人估计平均b2约为900
- 思路提示：可通过爬取Discord讨论获取b2的市场情绪，再反推b1
- 一位选手算法侧PnL约34k并自称无过拟合，可作为参考基准

---

## 2026-04-26 00:25

- algo-trading: PnL曲线过于线性可能意味着策略中泄露了未来数据（look-ahead bias）
- manual-trading: 当自己的第二次出价(b2)低于总体均值时会受到惩罚，惩罚为百分比形式

---

## 2026-04-26 00:34

#algo-trading
- Oracle最大可能PnL（拥有未来信息的算法）约为1.4M-1.8M，可作为回测上限参考
- Hydrogel公允价格存在争议，9992与10000两种观点，部分人采用10000
- 建议使用Rust回测器，避免硬编码以防过拟合
- 减少回撤的方法之一是hedge对冲
- 关于Volcanic Rock期权策略的讨论：smile mean-reversion vs smile market-making 哪个更优仍未定论

#manual-trading
- 第二轮bid很多人会出在840-850区间，可考虑差异化定价
- 策略权衡：在49%命中率拿更高单笔PnL vs 60%安全命中率，前者可能更优
- 920作为低价bid是更具竞争性的选择
- 出价机制理解：第一bid以下全部成交，第二bid若高于全局均值则两bid之间区间成交，否则触发penalty

---

## 2026-04-26 00:39

基于消息内容，提取以下有价值的见解：

=== #algo-trading ===
- 有选手在期权(vouchers)上获得显著收益：velvet 11.5k、Hydrogel 86.6k、Vouchers 44.5k，说明options是本轮主要利润来源
- 不同执行价(strike)的voucher需要采用不同策略，不能用统一方法
- 网站排名榜数据可能存在过拟合风险，约10k左右PnL的策略在day2很多变成负收益

=== #manual-trading ===
- 推荐使用本地backtester（如Rust backtester）而非依赖官方网站回测，网站结果不可靠
- Hydrogel在网站和Rust backtester之间数值差异很大，可能因为该产品随机性较强
- 在manual trading中，由于竞争对手众多，纯粹追求安全（60%置信度）不够，需要承担一定风险才能领先
- velvet extract在day2表现为正收益，hydrogel表现较差

---

## 2026-04-26 00:44

#algo-trading
- Round 3期权策略中，T的设置存在分歧（5 vs 5/252），需注意时间单位
- Hydrogel单日PnL超过30k基本属于过拟合；超过86k一日尤其可疑
- 期权做市效果不佳，有人改用take策略表现更好
- Hydrogel可能适合均值回归策略
- Velvet underlying可获得约6k的PnL

#manual-trading
- Manual题目网站显示的结果基于第2天前1/10数据，最终结算用完整第2天数据
- 估计avg bids时不应只假设热门b值，应该暴力枚举所有可能性以避免依赖他人行为假设
- 可通过模拟方式（brute force）来推算更稳健的最优出价

#general
- Round 3于EST时间早上6点结束

---

## 2026-04-26 00:50

=== #manual-trading ===
- Round的manual题bid2选手报价分布：840、880、890、890、900、919、920等，主流集中在880-920区间，少数极端值如672
- 该题分布可能是双峰：一峰追求PnL最大化，另一峰只为规避penalty而选safe zone
- 与上轮不同，这轮只有单一平均值参考（高于或低于），加权平均比多模态拟合更有效

=== #algo-trading ===
- 关于overfit边界的经验法则：特征只能从训练数据中提取，不能从隐藏数据反推
- Hydrogel等产品的非过拟合PnL基准争议：保守派认为三个产品合计约1200才算无overfit，hydrogel单品6k已属overfit；激进派称40k+确实可达
- 比赛中的options产品没有put，只有call

=== #general ===
- 各轮之间有约3小时的cooldown间隔

---

## 2026-04-26 00:55

- hydrogel和velvet之间存在小幅相关性（可能是OU生成过程中使用了相同函数，也可能只是噪声）
- delta 1产品的online OOS形状相同但波动率变窄，可能是样本量小导致
- 6000和6500 voucher可以做市（20 bid / 15 ask），通过加宽spread获利
- 4000 voucher的玩法：以低价（接近0）从bot买入，再以高价卖回（spread成本可能超过定价偏差，需谨慎）
- 跨天表现的连续性可作为衡量策略稳健性的指标（用第一天前10%预测后90%）

---

## 2026-04-26 01:00

- #algo-trading: 期权策略中，提示信息暗示需要使用IV(隐含波动率)和moneyness等因素
- #algo-trading: 部分选手在期权上亏损（约-40000至-1000），整体表现不佳，怀疑真实数据上会更好
- #general: 官方mods暗示在数据中藏了大量信号并用噪声混淆，可能难以反向工程
- #manual-trading: 共识为出价越低利润越高，但过低会被惩罚（penalize），属高风险高回报权衡
- #manual-trading: 关于是否需要出751来捕获750的bid存在讨论（边界值的处理细节）

---

## 2026-04-26 01:10

- 回测与官网PnL存在显著差异（如hydrogel回测正、官网负），bot行为可能是关键原因
- 本轮PnL不是评判重点，更应关注giveback/drawdown
- Hydrogel市场结构观察：约16的spread，存在两个做市bot加上一些随机订单簿bot和taker bot
- 算法挑战据推测以第2天的成绩评估，PnL上限约120k-150k级别

---

## 2026-04-26 01:15

- 订单簿通常包含两个做市商和偶尔的第三方限价单，可视化几个时间戳的订单簿即可发现规律
- 关于过拟合的争论：部分选手认为针对bot行为的过拟合是必要的，但bot行为是否在未来时间戳保持一致是未知数
- 当前竞争性PnL基准参考：单日250k+被认为有竞争力，顶尖可达650k+；options产品约48-50k水平
- Black-Scholes方程被讨论用于options定价（暗示对数收益率服从GBM假设）

---

## 2026-04-26 01:20

- 期权这轮订单簿非常稀疏，交易困难
- 5500和4000行权价值得交易，600和650行权价基本无法获利
- 有人反馈使用Black-Scholes模型效果差（仅5k），移除BS后反而提升至14k
- 波动率可作为交易信号
- 策略思路：用波动率微笑拟合，通过BS计算期权理论价后做市
- 期权回测算法部分PnL目标约30k

---

## 2026-04-26 01:35

- #algo-trading: 有选手指出最大硬编码（hardcode timestamps作弊）PnL约154-155k，超过此值基本不可能，疑似有人作弊
- #algo-trading: 单日PnL约30-45k被认为是较好水平，130k总分（不含options）属于较高
- #algo-trading: Options上400 PnL不算好
- #algo-trading: 警惕图表末端flatline和drawdown表现，overall PnL不是好指标
- #manual-trading: Manual round选手讨论gabagool报价，多人收敛在820-825附近，825被认为是较优选择
- #manual-trading: 提示题目中所谓"bids"实际是卖家的reserve prices（卖给你），需要正确理解题意

---

## 2026-04-26 01:40

- #manual-trading: 拍卖机制确认——出价能命中所有低于等于该价的reserve price；首次出价Nash均衡约836-840
- #manual-trading: Nash均衡推导思路：bid1取区间两端中点，bid2取bid1与上端中点
- #manual-trading: 部分玩家会抬高出价（如870）试图拉高平均价（第二轮规则相关）

NO_INSIGHTS（algo-trading频道大多为闲聊和PnL炫耀，无实质策略信息，仅提到Round 3涉及delta hedging和vouchers）

---

## 2026-04-26 01:45

- 满分预期：每轮峰值约 R1=200k、R2=220k、R3=250k；提交前网站测试通常只能拿到约10%
- TTE假设：网站测试时假设 Time-To-Expiry = 5

NO_INSIGHTS之外的有效信息已提取完毕。

---

## 2026-04-26 02:00

- #algo-trading: 有选手提到对美式期权可使用 Bjerksund-Stensland 近似进行定价
- #algo-trading: velvetfruit extracts 期权 spread 较大,边际利润被吃掉,难以套利
- #algo-trading: IMC 网站上的 PnL 普遍在 3k 左右,20k 可能是过拟合结果
- #manual-trading: 担心 manual 题中大量选手直接选 900 会显著影响最终结果分布

---

## 2026-04-26 02:05

- #manual-trading: 有人指出当平均值越接近920时，比平均值低1单位的惩罚比高1单位的惩罚更严重（说明manual题的支付函数对低于均值有偏向性惩罚）
- #manual-trading: 有传言称Claude给出的建议是795和905（可作为对手行为参考）
- #manual-trading: 上届约3%的队伍会刻意把答案偏向较低端
- #algo-trading: 期权策略主流做法是用Black-Scholes和波动率微笑估算fair value，但单纯依赖此方法效果有限，需要更好的vol估计方法
- #algo-trading: 讨论中提到DP oracle schedule（动态规划/预言机定价相关思路）

---

## 2026-04-26 02:30

#manual-trading
- 关于本轮manual交易（双价竞标），选手讨论的非博弈最优解大约是低价751、高价836；也有人认为是760和850，最终结果取决于其他人的出价分布

#algo-trading
- 有选手提到使用全部数据来拟合波动率smile的做法（针对期权类product）
- 本轮无overfit的合理PnL参考值约为30k；榜首水平接近900k多为hardcode/oracle调度（过拟合）
- 有人提到Prosperity 2和3中类似的"hardcode/oracle schedule"策略仍然奏效

---

## 2026-04-26 02:45

- #algo-trading: 有选手指出manual trading硬编码上限约为155k，对于报出600k+成绩的选手存疑（可能涉及过拟合或其他方式）
- #manual-trading: 有选手提出疑问——第一个bid的最大PnL是否独立于第二个bid的位置及他人的bid（涉及双bid manual题机制理解）

---

## 2026-04-26 02:50

- #algo-trading: 3天回测约600的PnL大致对应IMC官方portal上约20k（公式≈600/3/10）
- #algo-trading: 有人通过硬编码（hardcoding）在3日回测获得150k+，livesim能达90k
- #algo-trading: 普遍提到通过过拟合/硬编码来提升分数，未硬编码者难突破8k-20k
- #manual-trading: 第一标价(b1)应直接选择期望值最高的价格，不应偏离最大EV；因为b2价格高于b1，需在b1价位最大化中标量
- #manual-trading: 出价不必是5的倍数
- #manual-trading: 最优first bid与avgb2相关（有争议）

---

## 2026-04-26 03:10

#manual-trading
- 关于manual trading出价讨论：有人认为最优区间为761/851，有人提出867-766，820，800（用贝叶斯定理推算），750等不同方案，但普遍认为836及以上偏高

---

## 2026-04-26 03:25

- #algo-trading: 有选手提到spread成本约占200k，减少10%支付的spread可多赚20k，提示降低做市点差成本是优化方向
- #algo-trading: 选手反馈期权定价方式对PnL影响很大；hydrogel约7k为常见水平
- #algo-trading: 部分选手回测达90万-150万，0-4个参数即可，提示策略不需要过度复杂
- #manual-trading: 有选手猜测manual题目最优解在836附近（多人选择此值），讨论中有人担心其他人用小号刷极端值扭曲分布
- #general: "管道图片含隐藏alpha"等说法疑似玩笑，但提示有人尝试图像分类/时序预测方法（可能是trolling）

---

## 2026-04-26 03:26

- #algo-trading: 网站PnL受模型影响巨大，1k ticks样本不足以预测10k表现，10k-50k范围都算合理模型
- #manual-trading: 第二轮出价平均值约846；760/920被认为是关键价位；玩家普遍希望尽量接近835

---

## 2026-04-26 03:35

- #algo-trading: 有选手提到深度OTM call可作为对冲极端跳空的保险，价差成本约600。
- #algo-trading: 实盘模拟运行规模约为网站回测的10倍。
- #manual-trading: 手动交易讨论中，分析了出价相对于平均bid距离对每单位PnL的影响——越接近安全区PnL损失越小，且平均bid越高，远离安全区的代价越大；提示在bid选择时应权衡进入安全区的额外成本。

---

## 2026-04-26 03:36

- #general: volatility-smile定价、IV scalping和对冲是期权策略方向
- #general: velvetfruit extract最大PnL参考值约54k
- #algo-trading: 部分选手认为单一smile曲线效果不佳，讨论是否每个期权拟合各自的smile
- #algo-trading: time to expiry的计算方式存在疑问，可能影响定价
- #algo-trading: 有人反馈volatility smile策略实际不奏效
- #manual-trading: 手动交易中关于bid=reserve price是否成交的规则疑问，以及如何在不知道平均bid的情况下决定最优bid

---

## 2026-04-26 03:40

- #algo-trading: 期权回合基于IV和Black-Scholes难以找到alpha，最终提交才执行期权
- #algo-trading: 有选手采用"最优做市+趋势交易"组合策略，回测可达930k（day0/2约250k，day1约400k）
- #manual-trading: 关于安全区惩罚机制，800与796等价、795与791等价（即每5个单位为一档），需衡量超出安全区x单位的相对成本

---

## 2026-04-26 03:50

- algo-trading: 榜单上154k+的高PnL提交被指为硬编码（针对每个timestamp的oracle schedule优化），在final holdout阶段不会成立；不硬编码的真实水平约30k，略带过拟合可达120k左右
- algo-trading: 深度OTM voucher（如6k strike）有人作为应对底层价格大幅spike的尾部payoff来配置
- manual-trading: 出价末位选择（如xx5 vs xx1）在B1中无影响，但在其他题目中报价位数会影响中标量与利润，存在量与利的权衡，可考虑双边布局两头吃利润

---

## 2026-04-26 03:55

基于消息内容，提取有价值的见解：

=== #algo-trading ===
- 在线测试数据是Day 2的子集，过往轮次也是如此（不仅限于本轮）
- Hydrogel/期权策略表现：多数选手3天总PNL在75k-135k之间，能正确处理IV交易的选手可能领先一大截
- 有选手怀疑IV smile可能是误导信号（red herring）
- 一份较好的backtest参考结果：3天总PNL约851k，其中HYDROGEL_PACK ~146k，VELVETFRUIT_EXTRACT ~63k，VEV系列期权按行权价分布（5000档最高约137k，越偏离ATM贡献越小，6000/6500档无PNL）
- IMC官方backtester与本地backtester结果差异显著，需注意

=== #manual-trading ===
- 上轮manual提交多数集中在均衡点+10范围内，但并不存在唯一均衡
- 部分团队选择无风险speed=0策略，会使分布有偏

---

## 2026-04-26 04:31

- #algo-trading: 期权position limit疑问——300是单个期权还是总持仓的限制（未解答）
- #algo-trading: 有人提到IV在日内存在drift，可作为信号实现
- #manual-trading: 第二轮manual trading（疑似海龟竞拍/双出价题）选手出价集中在838-866区间，有人计划出价851左右

NO_INSIGHTS之外的有效信息已提取完毕。

---

## 2026-04-26 04:37

- Manual trading（猜价/出价题）讨论：去除离群值后平均出价约865，惩罚较重；第一次出价被认为是确定性的，可能取中位/中点最优。

---

## 2026-04-26 04:47

- #manual-trading: 本轮manual题目疑似为类似去年的双价竞拍（bid1/bid2），讨论中AI普遍给出836/841的报价；有人认为今年参赛者整体水平更高，会倾向于报更高价以减少损失，因此850以下被视为偏低
- #manual-trading: 有选手指出大量公开喊话试图压低别人报价，本身说明其对自己当前报价不够自信，可作为反向信号——若多数人想拉低均值，真实均值可能偏高
- #general: 比赛引擎对每次提交加入随机扰动，撮合时机的微小差异会影响成交，避免选手记住"完美交易"，提交结果存在波动

---

## 2026-04-26 04:55

#algo-trading
- reliveer1分享：VEV的bot flow信号未根据voucher订单簿状态进行gating；当4100和5200 voucher的spread扩大超过32 ticks时，t统计量显著上升；informed bot仅在能在tight surface中对冲时才触发——可作为信号判断informed flow的依据

#manual-trading
- 有人提出本轮manual trading（连续函数优化）的最优解为795（另有人猜791），可作为参考但未确认正确性

---

## 2026-04-26 05:05

===  #manual-trading ===
- Round 3 manual题为双价位投标博弈，热门组合包括766/866、771/871、761/856、761/851，普遍认为略高于5的倍数（如851而非850）更优
- 共识：在远离纳什均衡时，比常见整数报价高1的报价能击败聚集的对手
- 上限报价876被认为过高，多数人不愿超过836；最优区间被讨论在851-866之间

=== #algo-trading ===
- 一位选手分享Round 3算法成绩：3天回测Total PnL约337k，平均每时间步收益约1088，Sharpe约1.63，可作为基准参考
- 社区讨论中，回测PnL超过15k-50k通常被怀疑为过拟合
- 提及PEPPERS（Volcanic Rock相关）出现崩盘行情

NO_INSIGHTS（#general 无实质信息）

---
