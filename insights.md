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

## 2026-04-26 05:07

=== #manual-trading ===
- Round 3 manual题目为双bid竞价，多数选手围绕756/761/791等作为bid 1，bid 2集中在836-876区间，普遍认为低于840的bid 2会"被淘汰"
- 策略思路：bid 2取决于对群体平均bid 2的假设，optimal bid 1则是bid 2的函数；高bid虽跨5档但保证金损失更大
- 关键判断点在于预期成交价是846还是851，惩罚较高时倾向保守出价

=== #algo-trading ===
- Round 3顶尖PnL可达约440万（日均~140万、Sharpe~10），普通选手仅几千到几十万，差距巨大
- 多人提醒高PnL可能来自overfit，需警惕

---

## 2026-04-26 05:12

=== #manual-trading ===
- 历史经验：往年Prosperity manual trading的结果比预期更接近Nash均衡，建议参考Nash策略而非过度激进
- 弱Nash均衡点位于751和836（第三分位点），有人估计bid2最优值在895左右
- 部分玩家认为bid2 100%置信区间在838-848，押注接近Nash
- 惩罚策略思路：对冲选836者的最大有效惩罚约为876（再高成本过大）

=== #algo-trading ===
- Round 3提示：Pepper（新product）相关信息隐藏在wiki的"box"里，需仔细查看
- 讨论中提及position limit可能为50或200，需根据wiki确认
- 部分选手在Pepper上出现crash/巨额亏损，新product算法需谨慎处理边界情况

（其余多为玩梗和闲聊，无实质信息）

---

## 2026-04-26 05:18

- #manual-trading: 本轮manual题目为两轮出价博弈（b1/b2），纯Nash均衡解约为b1=751、b2=836；791为单变量profit EV最大化解；多人讨论b2均值预计约755-830区间，部分人选择略高于Nash以应对风险（如836+）
- #manual-trading: 去年数据显示平均出价仅比Nash高1点，可作参考
- #manual-trading: 双玩家场景下存在75+个纯Nash均衡；用mean而非median作评判会受极端alts账户影响
- #algo-trading: 教训——临deadline用Claude生成的算法直接提交风险极高（出现-28k等结果）；硬编码过拟合的方案在真实数据上往往亏损

---

## 2026-04-26 05:20

- #manual-trading: Round 3 manual出价区间约为670-920（5的倍数），有人用 (920-b)*(b-670)/250 的均匀分布EV模型求解，理论最优b≈中点795
- #manual-trading: 实战经验认为b2平均成交价>850，建议把b1压低以获取额外收益（前提是对b2能清算有信心）；常见提交组合如771/881、866/766
- #algo-trading: Round 3硬编码价格策略在真实数据上很可能反向亏损，回测>40k的硬编码方案在实盘会变负；30k左右才是较真实的可达水平

---

## 2026-04-26 05:30

- #manual-trading: Manual trading题目中，counterparty只在bid严格高于reserve时才卖出，因此bid恰好落在reserve价格上是被支配的——应该选择reserve+1（如796优于795，能多捕获一个pod，净收益+99，约+3.2%）。Reserves按5的步长分布（670, 675, ..., 800）。
- #manual-trading: Bid 2存在弱Nash均衡问题，836这类策略可被惩罚（4 EV成本换20 EV惩罚），但超过876后惩罚成本过高（cubic penalty）变得不划算。
- #manual-trading: 玩家选择示例：771/881、866 (b2)、835 (b2)，可作为bid分布的参考。
- #algo-trading: 官方backtester给出的timestamps数量是实盘数据的10倍以上，因此PnL不能简单按比例换算。
- #algo-trading: 普遍反馈Phase 2（含期权）比Phase 1难很多，但期权理论实际作用有限，hydrogel等其他product反而更值得投入时间。

---

## 2026-04-26 05:33

#manual-trading
- Round4手动交易：多位选手认为最优出价区间约为 b1≈796、b2≈861-866，部分人为安全起见上调5-10点（如796/876、750/850）

其余频道无实质有价值信息。

---

## 2026-04-26 05:38

来源: #manual-trading

- Manual round的最优出价讨论：bid1=766附近，bid2=866附近被多人认为最优
- 数学建模思路：bid1是 (x-670)(920-x) 的最大值 ≈ 796；bid2是 (b2-b1)(920-b2) 的最大值 ≈ 858（不含惩罚项的简化模型）
- 有人通过分布建模得出bid2均值在860-865区间
- 出价过高（如880+）被认为是错误策略，过低（835-836）则太保守
- 实际最优值取决于参赛者出价分布的先验假设

---

## 2026-04-26 05:48

#manual-trading
- 关于manual trading的两次出价博弈：最优b1并不是简单设为795（截断点），因为b2可以捕获b1和b2之间的成交，所以b1可以低于795，最优b1依赖于b2取值，约在765-780区间
- 部分选手报出价组合参考：766/866、771/872、676/696、720（b1）

---

## 2026-04-26 06:08

- manual-trading: 有人分析后认为本轮manual题目（疑似第二个数）的合理均值约861.5（剔除670/700/920等离群值），分布正偏使均值>中位数，预期实盘可能落在865附近
- manual-trading: 选手讨论的bid报价集中在850-880区间，普遍认为低于750或过于保守（如881）的报价不优
- algo-trading: 有选手指出R3产品（如pepper）信号比R2更难识别；OSM中存在informed trader/toxic flow，但该信号未延续到R3

---

## 2026-04-26 06:13

#manual-trading
- 关于manual round的双bid博弈题，社区讨论的最优解集中在762/857附近（基于投票分布），751/861、753.1等也是热门估算
- B1的最优值依赖于对手方分布而非独立数学题，与去年题目机制不同
- 约束条件：低bid必须小于高bid；B2过高会触发惩罚机制

NO_INSIGHTS对其他频道。

（实际输出）：

#manual-trading
- 本轮manual双bid题社区估算最优解约为762/857（基于投票分布反推），其他热门答案包括751/861、753.1等
- B1并非独立数学题，而是依赖对手方bid分布，与去年机制不同
- 题目存在B2过高触发惩罚的机制，且约束low bid < high bid

---

## 2026-04-26 06:19

- #manual-trading: 手动交易题需要同时优化 bid1 和 bid2（在同一函数中），而不是先定 bid1 再定 bid2；许多人误以为是顺序决策
- #manual-trading: 玩家出价分布观察：剔除低端离群值后均值约 865；常见出价范围 761-881，多数人在 836-860 之间
- #manual-trading: 储备价（reserve price）以 5 为间隔，因此出价取 5 的倍数即可，非 5 倍数无额外收益（如 790 与 791 等价）
- #manual-trading: 策略思路：bid 略高于均值可推高平均参与分；但需权衡惩罚与收益（penalty vs profit）

---

## 2026-04-26 06:24

#algo-trading
- Pepper和Osmium也作为隐藏alpha在被测试（wiki中有说明），很多人因此获得100k+收益
- 忽略pepper/osmium会导致大幅损失（提到-20k IV的代价）

#manual-trading
- Manual出价博弈：在无博弈假设下最优解为835和750
- 很多人误以为790/795是bid1的最优值，导致实际平均值偏高
- 策略关键：bid2需要高于平均值才有收益，否则会被立方折扣惩罚
- 奖励公式：若x>avg则reward=920-x；若x<avg则reward=(920-x)*cubic_disc(x,avg)
- bid1可视为独立于平均值；bid1 < bid2（命名为lower/higher bid）

---

## 2026-04-26 06:25

- 手动交易：根据数据(剔除异常值后)平均价约为865
- 手动交易Bid 1解题思路：在670-920区间内有k个5的倍数报价，最大化 (920-x)*(x-670) 形式的profit*quantity
- R2算法：有选手反馈直接提交R2算法即可work（暗示比赛中有未公开机制/隐藏alpha）

---

## 2026-04-26 06:29

=== #manual-trading ===
- 手动交易题（双bid拍卖）：当前观察到的中位数为861，分布前向偏斜，均值会高于中位数
- 双bid需联合优化b1和b2（而非分别优化），bid2分配在bid1之后进行
- 玩家实际投注示例：b1=766/856/851，b2=861/866，存在分歧
- 有传言部分玩家用小号操纵平均数

=== #algo-trading ===
- 有玩家通过隐藏alpha（疑似在源代码/wiki中）单日获得约15k-60k PnL提升，建议查看比赛源代码和wiki寻找隐藏信息
- 提示Pepper和Osmium在后续轮次可能不再值得交易（暗示产品退场或价值变化）

---

## 2026-04-26 06:34

#manual-trading
- 关于第三轮manual bid的讨论：多数玩家估计平均值在856附近，不会超过；常见出价组合如791/861、796/860、776/881
- 有人观察到讨论区样本（约126人）会使分布右偏，但4000剩余玩家中很多只用Nash均衡再加2

---

## 2026-04-26 06:49

- #manual-trading: 第三轮manual题目疑似为竞价/出价题，社区收集的bid 2中位数约861-862（150票样本），多数选手在870+；有人认为Nash均衡解约836，但分布右偏可假设mean>median
- #manual-trading: 注意社区分享的bids可能含虚假信息以操纵决策；官方可能按mean计算
- #algo-trading: 本轮IMC平台上传后好成绩门槛约90k；有选手反映回测150k但实盘上传仅25k，存在回测与实盘差异问题

</response>

---

## 2026-04-26 06:54

#general
- 有选手分享了Round 3 manual交易分布查看工具：https://prop-round-3.up.railway.app （PNL热图有bug，但分布数据可用）

#manual-trading
- 关于Round 3 manual（疑似两次出价博弈）讨论：当前均值受离群值影响，中位数可能是更好的估计器
- 蒙特卡洛模拟显示最优解低于850，多数人选择略高于Nash均衡但不超850
- 风险提示：出价正好低于均值会被严重惩罚
- 由于多数人知道Nash最优解，集体行为会推高平均值，需要相应调整出价

---

## 2026-04-26 07:10

- #manual-trading: 本轮manual交易使用cubic损失函数，惩罚公式形如((920 - b2avg)/(920 - b2))^3，分子对所有人相同；当b2低于均值时，b2越低越好，越高则惩罚越大（注意：低于均值时较高的b2受惩罚更多）

---

## 2026-04-26 07:10

- #manual-trading: 第二轮manual题目penalty函数为cubic，公式为((920 - b2avg)/(920 - b2))^3，分子对所有人相同；低于均值时，越接近均值（即b2越高但仍低于均值）惩罚越大，因此低b2优于高b2（在均值以下时）

---

## 2026-04-26 07:30

- #manual-trading: 本轮manual题目疑似猜平均数游戏，选手普遍认为平均值在860-862之间，主流答案集中在861-866，部分防守性选择885左右以避免低于均值

---

## 2026-04-26 07:35

- 手动交易本轮（双重出价博弈）：普遍认为最优bid为866，理论Nash约为751/836或791/871，但因多数人知道Nash会出现overbid，实际均值预期略高于861，故选866作为次优；低于均值的惩罚远大于高于均值
- 下轮预测：多数人押注会是带多腿成分的ETF/篮子产品，市场效率高且有随机rebalancing bot
- 期权IV smile策略实测多为负PnL，未能跑通
- Tomatoes（番茄）走势近随机，难找盈利策略；做市策略在round 3每天约10万PnL
- 多数玩家弃Tomatoes，专注Osium和Pepper

---

## 2026-04-26 07:50

- #manual-trading: 第二轮manual最优出价约为761/861（讨论中提到的最高分接近此组合），平均成交价约859
- #algo-trading: 网站显示的backtest使用的是Day 2前10000个时间戳，而Round 3实际在全新Day 3数据上运行，导致网站预览pnl与实际live pnl差异巨大（如backtest 510k vs live 50k）

---

## 2026-04-26 07:55

- #manual-trading: Round 3 manual题目疑似为博弈论(GTO)问题，可用蒙特卡洛求解；获得manual第一名的出价组合为761/861；有人提到857/858等其他出价。
- #algo-trading: 教程中的基础market making算法表现优于多数自制策略，提示基础MM策略在此轮有效。
- #algo-trading: 有选手提到该轮价格图表呈现mean reversion特征，可考虑均值回归策略。
- #algo-trading: 回测器结果与实盘差异较大，多人回测负PnL但实盘正PnL，回测可信度低。

---

## 2026-04-26 08:00

- #algo-trading: 有选手反馈纯mean reversion策略效果不错，未使用Black-Scholes/期权定价
- #algo-trading: 有团队认为portal数据波动太大不可靠，仅依赖backtest结果验证策略
- #manual-trading: Manual round最终最优出价的平均值为859，766/861组合可达第4名水平；最优解附近为860左右

---

## 2026-04-26 08:10

#algo-trading
- 高分选手在期权类产品使用Black-Scholes定价 + Gamma Scalping策略获取主要alpha

#manual-trading
- Round相关manual题最优出价为(761, 861)，bid1与bid2不独立：当bid2提高时，bid1也需相应提高，因为更高bid2会捕获更多订单但底部合约PnL降低，需通过提高bid1补偿
- 可通过对total_pnl函数求导推导最优解

---

## 2026-04-26 08:15

- 频道 #manual-trading: Manual round 最优投标对计算结果分歧——有人算出(761, 861)为最优，另一种分析认为(751, 836)是假设最优博弈的均衡解；由于参赛者倾向于高出价，部分队伍以5为步长上移均值并模拟代理分布来校准
- 频道 #manual-trading: (761, 861)是较宽范围的最优对之一；常见出价水平包括846、866等
- 频道 #algo-trading: Round 3 出现严重过拟合现象——很多队伍回测高收益（300k~672k）但提交后实际PnL接近0甚至负值（-2k至-12k）
- 频道 #algo-trading: Vouchers/期权策略普遍亏损，大量团队放弃期权部分；底层资产方向交易表现尚可但期权拖累整体PnL

---

## 2026-04-26 08:20

- #general: 官方澄清hardcoding规则——基于逆向工程bot行为或定义自身交易参数的硬编码可接受；但硬编码价格数据、引用外部数据、利用平台bug或使用非公开信息会被取消资格；对每个timestamp硬编码导致异常PnL会被怀疑，前排提交会被人工审核
- #algo-trading: Round 4 vouchers的关键变化是TTE（到期时间），有人在仅靠vouchers就赚到250k
- #algo-trading: 选手讨论使用Black-Scholes、IV、做市(MM)、delta hedging等期权策略；不做BS/IV/MM也能取得高分（暗示存在更简单的策略路径）
- #manual-trading: Round 4 manual题中众多选手在836附近出价，有选手参考去年数据假设不会偏离太远导致失误；存在一个外部网站(project-yqymi.vercel.app)显示出价分布，部分人故意overbid干扰

---

## 2026-04-26 08:30

- #algo-trading: 有选手提示重点应放在最大化网站PnL，并使用神经网络作为alpha来源
- #algo-trading: 观察到Hydrogel(Hydro)的spread为15仅出现在mid<10000时，spread为17仅出现在mid>10000时；某option上也有类似现象，可能是代码留下的人为痕迹，或可加以利用
- #algo-trading: 有人询问基础gamma scalping是否足以应对本轮（涉及options）
- #manual-trading: AC_50_P是Aether Crystal看跌期权，Strike=50 XIRECs，到期时间21个Solvenarian日（从Round 1在Intara开始计算）
- #manual-trading: 关于R4 manual（疑似类似goldfish博弈）：上一轮多数人b2出价约870；进化均衡收敛到b2=13；若清空了b1则b2 bid可能与PnL无关（讨论中存疑）

---

## 2026-04-26 08:30

- 观察：5400行权价的voucher（看涨期权）走势与其他行权价不同，可能存在差异化交易机会

---

## 2026-04-26 08:45

- #manual-trading: 报价时出价X只能成交低于X的订单，不包含等于X的（即bid 860不会吃到860的卖单），因此最优出价应为861/761，比平均价高一档
- #algo-trading: 回测与实盘结果差异巨大（有人回测-7k实盘+131k），原因是官方回测仅使用了Day2的子集数据，回测结果参考价值有限

---

## 2026-04-26 09:00

- #algo-trading: 排行榜portal显示的PnL约为完整一天的10%（即10x换算），最高算法约35万PnL
- #algo-trading: R4和R5使用相同产品，R3表现好的选手在R4/R5可继续优化
- #manual-trading: B2手动交易最优解为761/861
- #manual-trading: 注意B2底部总投资计算使用的是volume而非volume*3000的数量

---

## 2026-04-26 09:05

- #algo-trading: TTE计算讨论：T = max(0.03, 4.0 - timestamp/1_000_000) / 252，关于用252还是365交易日存在疑问
- #algo-trading: 提到trader.csv中包含trader id信息可以利用
- #algo-trading: 经验提示：用AI辅助编码时不要喂太多context，否则Claude会变蠢
- #algo-trading: IMC官方模拟器波动性大、Rust backtester在R3表现差，建议自建backtester
- #manual-trading: 本轮manual为竞价博弈类题目，纳什均衡约920附近（含escalation）；保守选择如756/856可得约78k；激进选751/836在群体均值偏低时可拿20-30k最高利润
- #manual-trading: 856-861区间属于运气/主观选择，超过群体均值才有高收益

---

## 2026-04-26 09:15

- #algo-trading: Round 4最终结算疑似在day 4运行（R3的backtest是day 2，最终是day 3；R4 backtest是day 3，最终在day 4），TTE随交易日推进而递减，影响期权定价
- #algo-trading: 大量选手在backtest上取得150k-300k PnL但最终结算暴跌甚至变为负数，典型的过拟合现象，提示策略应基于稳健的fair value（如硬编码均值）而非过度调参
- #algo-trading: 加止损（stop loss）在R4的options/volatility drift情况下救了不少人，建议在策略中加入风险控制
- #algo-trading: R4不过拟合的合理目标PnL约300k+
- #algo-trading: 可以用提供的测试数据自行验证backtest是否可靠

---

## 2026-04-26 09:20

- manual-trading: T+21期权按21个自然日（非交易日）到期；离散价格序列下，若任意时刻t有B≥S(t)则payout为0；不允许提前行权，PnL为成交价与到期实际产品价值的差值（100次模拟取平均）
- manual-trading: 总交易量为200×3000=600000 lots，AC本身也乘以3k

---

## 2026-04-26 09:30

- #manual-trading: Round 4手动题涉及期权定价，价格遵循随机过程，需估计可能路径来决定交易
- #manual-trading: 时间设定为年化252个交易日，每周5个交易日；T+21指距到期21个Solvenarian日（非全部为交易日），从Round 1的Intara开始计算
- #manual-trading: AC_40_BP为Aether Crystal二元看跌期权，行权价40 XIRECs，到期21日；若到期价低于40则支付固定10 XIRECs，否则归零
- #manual-trading: 由于模拟过程细节已知，给奇异期权定价并不比普通期权难多少

---

## 2026-04-26 09:50

- manual-trading: 有人通过Nash均衡得出报价(781, 886)，认为建模不确定性很重要因为更高bid更安全；另一人给出Nash公式 670+2(250)/3 ≈ 836
- manual-trading: Round 4的TTE为18个常规日（12个交易日），aether crystal从价格50开始模拟12个交易日
- algo-trading: 有人询问如何使用counterparty信息（self.buyer/self.seller返回值），暗示Round 4新增了交易对手方数据可用于策略

---

## 2026-04-26 09:51

- Manual trading（出价博弈题）：参赛者讨论双重出价的最优策略，认为bid1应匹配严格低于的reserve价，bid2再覆盖剩余对手；由于bid2收益依赖bid1，需联合优化。若只有单一bid，791或796可能最优，但联合优化下bid1应在761附近。
- 简化假设：可假设bids为1 mod 5以简化计算；不确定项是所有人的平均bid。

---

## 2026-04-26 09:55

- manual交易中的iterated nash均衡：通过对他人出价进行迭代演化可找到纳什均衡解
- 出价时需考虑他人会更高/更低出价，并将此预期纳入定价
- AC_45_KO敲出看跌期权：底层模拟为每交易日4步的离散网格，但根据"5天为一周、年化252天"的标注推断，敲出barrier按离散步监控

---

## 2026-04-26 09:56

- 网站门户显示的回测/测试PnL与最终隐藏数据PnL不同，门户显示+13k的算法最终可能-44k，提示存在过拟合风险
- manual题目AC_45_KO敲出看跌期权：底层模拟为每交易日4步离散价格网格，需确认敲出barrier是仅在离散步监测还是有连续监测调整（玩家提问，未获官方回复）

---

## 2026-04-26 10:05

- #manual-trading: 关于本轮期权题目的关键参数确认：二元支付为10、敲出条件是否含等于35、GBM是否包含-0.5σ²dt修正项
- #manual-trading: 时间推算讨论——已经过去Intara 6天+Solvenar 2天+约2非交易日，订单簿价格可能对应T-11
- #algo-trading: 多数选手回测有巨额利润但实盘表现差，警示过拟合风险；有人认为本轮实质上只有一个alpha来源

---

## 2026-04-26 10:10

- 风险管理教训：不要完全信任IMC网站或回测器，要关注实际drawdown；高drawdown的策略容易爆仓
- Round 3策略尝试：Black-Scholes + 隐含波动率拟合 + 均值回归 + scalping（效果不一）；Delta hedging是有效方法
- Round 3疑问：Black-Scholes中的无风险利率取值不明确
- Manual trading疑问：AC（底层资产）数量单位是否为lot制存在歧义，价格约50与衍生品3000 lot size对冲不匹配，倾向于离散时间设定
- 建议关注bot行为分析

---

## 2026-04-26 10:25

- Round 3中有选手靠hydrogel盈利80k，但其他品种亏损约40k，净盈利缩水
- 部分选手反映算法实盘表现与模拟回测结果一致，未超出预期
- Manual round对3周时间窗口（15或14个交易日）的定义不清晰，影响策略

---

## 2026-04-26 10:26

- AETHER_CRYSTAL底层模拟：几何布朗运动，零风险中性漂移，固定年化波动率251%，每交易日4步离散网格，每年252个交易日
- AC_45_KO是敲出看跌期权，underlying跌破barrier即作废；存在屏障监控是否仅在离散步发生的争议（连续监控会显著影响定价）
- T-21从intara第1天开始，期权剩余天数与100次模拟的时间安排不明确

---

## 2026-04-26 10:30

- #algo-trading: 有选手观察到5200和5300期权的已实现波动率始终高于隐含波动率，可考虑gamma scalping策略；5400期权的IV持续低于其他行权价，存在相对定价偏差
- #manual-trading: 关于barrier option的讨论——"ever trades below the barrier before expiry"应理解为离散时间步监控而非连续监控；T+21可能因节假日实际少于15个交易日
- #manual-trading: 推测当前期权剩余时间——已经过6天Intara+2天Solvenar+约2天非交易日≈10天，订单簿价格对应T-11

---

## 2026-04-26 10:35

- manual-trading: "21 Solvenarian Days"被解读为15个交易日（非日历日），据称有Mod确认；存在round 2开始时是否已过去一天的疑问
- manual-trading: 关于manual交易中price列与bid/ask的成交价格存在困惑

---

## 2026-04-26 10:45

- #manual-trading: 本轮manual交易b2平均值最终为859，有人预测在853-858区间并加了小buffer，bid 1被认为是最优策略

---

## 2026-04-26 10:56

- 算法交易频道：有选手用Black-Scholes + gamma scalping策略，认为可能存在过拟合问题，提示更简单/稳健的方案可能更有效
- 算法交易频道：尽管官方advisor提示使用隐含波动率和BS，但有选手建议不要完全依赖BS模型

---

## 2026-04-26 11:00

- manual-trading: 官方确认manual round机制——参赛者只在挑战开始时做一次决策，之后系统模拟所有交易日并按fair value结算
- manual-trading: 期权到期时间按日历日计算，但模拟基于252交易日/年假设，3周模拟期对应少于15个交易日
- algo-trading: Round 4使用相同数据，可以查看trader ID

NO_INSIGHTS对其他无实质内容部分

---

## 2026-04-26 11:05

- manual-trading: 期权波动率/时间参数确认 - "week"指5个交易日和7个日历日，标准每年252个交易日（用于年化计算）
- manual-trading: manual持仓将在Round 5开始时被估值并清算
- manual-trading: 选手观察期权定价正确，100次模拟下净利润为零，需要寻找不对称性来获利

---

## 2026-04-26 11:10

- Round 5期权手动交易讨论：期权按15个交易日定价，蒙特卡洛模拟100次得到净零利润，参赛者在寻找创造不对称性的方法
- 时间框架争议：21天总时长中部分为交易日，参考一年252个交易日构建网格点数量；有人提出21*4=84网格点用于模拟

---

## 2026-04-26 11:12

- #algo-trading: VOLCANIC_ROCK_VOUCHER期权持仓限制300，其他产品200（参考wiki）
- #algo-trading: 有选手认为Round 4与Round 3类似，counterparty信息难以带来改进
- #manual-trading: Round 4 manual题涉及标的资产，起始价格可能为50，剩余12个交易日（每周5天，共60步的讨论）

---

## 2026-04-26 11:20

#manual-trading
- 本轮manual为期权定价题，spot AETHER-CRYSTAL=50，给出了多个期权的市场中价与理论价对比（基于σ=251%, r=0%, paths=1M的模拟）
- 模型显示市场相对便宜（market cheap）的期权：_50_C (+0.0165)、_50_P_2 (+0.1417)、_50_C_2 (+0.1394)、KO_PUT (+0.0433)
- 模型显示市场偏贵的期权：CHOOSER (-0.3476)、BIN_PUT (-0.2869)、_40_P、_60_C 等
- 第一份合约疑似为标的本身，用于delta对冲；可用期权之间互相对冲净敞口
- 可用数量上限200，与期权notional（如15万）相比对冲能力有限，玩家普遍质疑数量设置
- 有人在bid1=796获得75.5k PnL（比761高约7k），bid2最优约861（有人出881略偏高）
- 该轮无统一最优解，依赖团队风险偏好

#algo-trading
- NO_INSIGHTS（仅排名闲聊）

---

## 2026-04-26 12:10

- 推荐的回测器：kevin-fu1 和 nabayansaha 的 IMC Prosperity 4 backtester（GitHub），nabayansaha 已更新支持 counterparty 信息
- Rust版本回测器被认为最准确，结果偏差通常源于过拟合
- IMC官方backtest也是可靠选择

---

## 2026-04-26 12:12

- IV曲线建模时应包含所有voucher，这样才能呈现volatility smile形态（#algo-trading）

---

## 2026-04-26 12:15

- 有玩家观察到：在hydrogel(volcano rock?)产品上，"Mark 38"始终是aggressor（主动吃单方），而"Mark 14"则被动接受其报价，可能可作为信号利用
- 深度实值期权(deep ITM)的定价表现异常，多名选手无法解释，常规模型在Prosperity 4中不适用
- Hydrogel交易思路讨论：可同时做market making和mean reversion

---

## 2026-04-26 12:22

- manual-trading: 本轮manual题目中b1和b2出价相互关联，b2会清理b1未成交的部分；b1的最优值取决于b2的出价水平
- manual-trading: 若b2设在纳什均衡，b1纳什值约为751；若b2过高，b1需调整到约796
- manual-trading: 策略上希望b1和b2买入数量大致相等
- manual-trading: 796只在打算放弃b2（即b2会失败）时才是最优选择

---

## 2026-04-26 12:25

- #manual-trading: 有选手认为期权手动交易应直接选最大EV策略，对冲会显著降低EV，只有在最差2-3%的模拟情景下对冲才有价值
- #algo-trading: 第3轮算法环节负PNL人数相比上轮翻了三倍，普遍反映难度大；有人在day 3数据上难以达到350k PNL

---

## 2026-04-26 12:27

- manual交易模拟参数：使用252个交易日/年，每天4步，n_steps=4*252，dt=1/n_steps用于蒙特卡洛模拟时间步进

NO_INSIGHTS其他频道

（保留有价值的一条）

---

## 2026-04-26 12:35

- #algo-trading: R3中通过bot trades可以发现额外信号（BT中每天可多赚约5k），但顶级选手可能利用了双重alpha来源，仅靠此信号难以达到300k
- #algo-trading: 顶级排名者可能识别出了数据生成的机制/对手方
- #algo-trading: 仅使用day 0/1/2数据进行策略开发，day 3达到300k目标被认为非常困难

---

## 2026-04-26 12:42

- #algo-trading: 有选手用第三方backtester跑全3天数据得到约870K PnL，提到的backtester仓库：nabayansaha/imc-prosperity-4-backtester 和 Xeeshan85/imc-prosperity-4-backtester
- #manual-trading: 期权到期时间设定为21 Solvenarian Days（从Round 1的Intara开始），被建模为15个交易日；manual trading界面中的"price"列疑似有bug，可忽略

---

## 2026-04-26 12:50

- 期权定价线索：T+14和T+21标签可能具有误导性，根据vanilla期权的实际计算，使用的是t=15/252和t=10/252（即按交易日而非日历日计算）
- Manual交易疑点：price列的含义、+14/21是交易日还是日历日、提交投资额显示的是仓位公允价值而非实际买卖价格
- Round 3反思：有选手在两个潜在策略中选错了

NO_INSIGHTS之外的有效信息已提取完毕。

---

## 2026-04-26 12:53

- manual-trading: 期权PnL计算的到期天数存在争议，Wiki写15个交易日，但moderator澄清按每年252交易日的标准估算（3周模拟），15天为高估，这一差异对策略盈亏影响巨大
- manual-trading: 关于报价机制——若fair value为中间价50，买入仍需支付ask价
- algo-trading: 有选手认为做市(MM)在当前轮次行不通，但spread过宽也难以market taking

---

## 2026-04-26 12:58

- manual-trading频道：网站显示的"price"有bug，应忽略；时间换算规则为每年252交易日，每天4步，每周5个交易日；T+14和T+21对应的年化时间为(14*5)/252和(21*5)/252（即周数转年数按5/252计算）

---

## 2026-04-26 13:13

- #manual-trading: 期权对冲存在规模错配——1张vanilla期权合约对应3000单位标的，但标的最大可交易量仅200，使得用标的对冲期权不现实

---

## 2026-04-26 13:18

- #manual-trading: manual options题目的合约规模实为3000（用于PnL缩放），所有数值（包括标的）都需按3000缩放，否则无法对冲。

---

## 2026-04-26 13:20

- #algo-trading: 有选手讨论 hydrogel(磁性树脂?)的 fair value 选择，是用动态FV还是固定10k锚定值仍有争议
- #manual-trading: manual交易合约规模确认为3000（作为PnL的线性scalar），需Wesley/Hrvoje确认后加入wiki

---

## 2026-04-26 13:40

- 比赛数据生成机制：连续生成数据后按每10k行（1m时间戳）切分，提供的3天历史数据中最后一天用于测试，最终运行在这3天之后的下一天

NO_INSIGHTS（其余消息无实质策略信息）

---

## 2026-04-26 14:03

- R3/R4存在空bid/ask侧的漏洞（无bot报价时可利用），R5被patch后噪音增加，不再能赚60M但仍是有效alpha
- 在空bid/ask的报价场景中，需在MM之后报价，hitting在之后发生
- Protein snackpacks属于R5产品，被误放入早期数据中
- Manual trading中关于Intara天数计算：7天=1周=5交易日，需考虑前2天是否为交易日

---

## 2026-04-26 14:30

- R5中bot行为存在多个alpha信号，部分较大，可作为交易机会挖掘方向

---

## 2026-04-26 14:35

- 期权观察：4000和4500行权价期权的隐含波动率（IV）行为与5000+行权价期权显著不同；做市velvet fruit的bot同时做市4000期权，而5200+期权由另一个不同的bot做市，可能存在可利用的差异。

---

## 2026-04-26 14:44

- Round 3回测PnL与实际holdout PnL差距很大，过拟合现象普遍（887k回测→206k实际，700k回测→-50k，42k→148k等）
- 高回测PnL未必好，参赛者建议谨慎对待回测结果
- Manual trading机制澄清：产品fair value按100次模拟的平均值标记；solvinarian days指日历日，trading days指交易日；本轮为独立挑战，不继承其他轮次的仓位

---

## 2026-04-26 14:45

#manual-trading
- 在手动交易的效用函数参数选择上：高风险厌恶（lambda 2.0、1.0）表现较差，最优区间在中等水平 lambda 0.3-0.5，其中 lambda 0.5 作为默认较稳健，lambda 0.3 也可接受

（#algo-trading 内容主要为闲聊和回测结果讨论，无实质策略信息）

---

## 2026-04-26 15:04

- Manual trading讨论：PnL计算方式为 买/卖价 - fair value，其中fair value是100次模拟的平均值
- Knockout产品的PnL可能需考虑未被敲出的比例：(买/卖价 - fair value) * 未敲出概率
- KO产品本质是赌100次模拟中最小值的均值是否超过35
- Chooser option的计算需注意是 mean(a-b) 而非 mean(a)-mean(b)（涉及非线性）

---

## 2026-04-26 15:14

- #manual-trading: 期权定价机制澄清——买入后持有至到期，到期时基于100次模拟计算期权"公允价值"（如knock-out看跌期权有时可能变成无价值），Wiki已更新说明

NO_INSIGHTS（其余消息为闲聊或提问无答案）

---

## 2026-04-26 15:19

- #manual-trading: manual round的价格模拟基于GBM（几何布朗运动），需关注drift、波动率sigma、风险r等参数，参考wiki可推导策略
- #algo-trading: 硬编码均值是常见做法，但加fallback机制如未充分回测可能反而出问题；建议构建MC backtester验证鲁棒性改动
- #algo-trading: 网站参考分数约40k为不错成绩，bench PnL约20k

---

## 2026-04-26 15:39

- Ljung-Box检验在本届比赛中对时间序列分析很有帮助（用于检测残差自相关性）

NO_INSIGHTS其余内容均为闲聊。

---

## 2026-04-26 16:04

- 观察：本届比赛中的bots似乎都不像去年的Olivia那样具有明显的信息优势，组织方可能刻意降低了内幕交易者的辨识度
- 思路：可考虑用voucher（期权）作为对底层资产的杠杆押注工具
- 技术细节：使用round 4数据计算voucher时，day 0/1/2对应的TTE分别为7/6/5

---

## 2026-04-26 16:15

#algo-trading
- 有选手仅用基础做市策略在所有资产上获得9k盈利
- 关于EMA计算fair value存在中间价vs最后成交价的选择讨论
- 关于vouchers策略：部分选手对所有类型都交易，但有人发现某些类型始终亏损，考虑筛选交易标的
- 观点：butterfly策略要到最后交易日才有效，因为每round相当于推进一天到期

#manual-trading
NO_INSIGHTS

---

## 2026-04-26 16:35

- manual-trading: 100次模拟样本量较小，结果方差极大（-500k到+500k范围），单纯选最高EV策略未必能赢
- manual-trading: 模拟数百万次后仍显示高度随机性，全押方向性策略风险大
- manual-trading: 选手估计manual部分难以盈利超过100-150k

---

## 2026-04-26 16:40

- manual交易讨论中提到模型隐含的预期收益上限约164k
- 有选手测试基本策略均值约160k，最大回撤数十万，最大上行超过120万，运气成分较大
- 团队众多导致风险管理良好的策略会被运气好的全压团队击败，最大EV满仓约为最大可能值的1/5
- 部分选手采用delta中性套利或IV smile scalping策略应对该manual

---

## 2026-04-26 16:55

- Olivia交易信息延迟一个timestamp可见：若Olivia在13100下单，需到13200才能从Trade类的buyer/seller字段识别（algo-trading）
- 有人提到hydro（可能指某product）存在明显套利/免费收益机会（algo-trading）

---

## 2026-04-26 17:00

- #algo-trading: 有人认为本轮的marks（参考价/标记价）信息量极低，不可靠
- #algo-trading: R3中高回撤的策略容易翻车，建议选择低回撤、稳健PnL的方案
- #manual-trading: Manual题最大EV约为166k左右，1.6M被认为不现实；要想拿高分需涉足高风险选项

---

## 2026-04-26 17:30

- 期权模拟机制：从day 0开始，2周期权用10个交易日（40 ticks到期），3周期权用15个交易日（60 ticks到期），运行100条几何布朗运动路径

---

## 2026-04-26 17:50

- #algo-trading: 将年化波动率转换为日波动率需除以sqrt(252)
- #algo-trading: 有人在尝试分别看bid和ask的隐含波动率（IV）来寻找IV moneyness的alpha
- #manual-trading: 推测manual交易的fair value由系统模拟100次计算得出，根据持仓方向决定是bid-fair还是fair-ask
- #manual-trading: manual界面最右侧的"price"列是bug，可忽略，原本应显示每个产品的投入金额

---

## 2026-04-26 20:21

- AETHER_CRYSTAL底层用几何布朗运动模拟，零风险中性漂移，年化波动率251%，每交易日4步离散网格（252交易日/年），knock-out只在离散点判定。

---

## 2026-04-26 20:36

- Mark 67 bot 策略推测：当 spread 较窄、订单簿买方占优、近5个tick价格下跌、且处于近50个tick的局部低点时，会主动吃掉内盘卖单
- 有观点认为产品间无相关性、无领先滞后关系、也无bot间依赖关系

---

## 2026-04-26 20:40

- #algo-trading: 本轮被普遍认为是hardcode题，网站分数超过20k可能已属过拟合，建议探索数据中未被发掘的思路而非调参
- #algo-trading: 官方暗示可以hardcode但需要聪明地处理

human_acknowledgment: NO_INSIGHTS不适用，已提取见解。

---

## 2026-04-26 20:56

- #algo-trading: 第4轮与第3轮相比，产品相同但增加了交易对手方(counter party)信息；实盘交易可能不提供买卖方名称，仅历史数据中有
- #algo-trading: 数据中可能存在多个"alpha"信号(基于交易对手方)，第一阶段只有Lauchy发现了
- #manual-trading: manual交易中的"price"列只是显示投资成本，与PnL无关，可忽略不影响决策
- #manual-trading: 仅100次模拟可能出现较多异常值，建议多跑几组100次模拟来评估稳定性

---

## 2026-04-27 00:20

- delta 1 产品策略：使用 grand mean 而非 rolling mean，可以重建 OU 过程
- informed flows 在 round 3 已存在但被掩盖，现在可能通过 trader IDs 显式展现
- 第 3 轮某选手单轮获利 220K+，但需警惕过拟合风险

---

## 2026-04-27 00:45

- #algo-trading: 有选手认为backtester在不同round间表现不一致，每轮都需要重新调参，怀疑backtester并非round独立。
- #manual-trading: 有选手反映R3算法过于保守导致排名下滑，本轮策略转向高风险博弈，而manual部分则倾向理性稳健操作以获取排位收益。

---

## 2026-04-27 01:07

- manual-trading: 本轮manual中vanilla期权接近公允定价，已price in所有信息，对100条路径平均的期望PnL不会很高
- algo-trading: 本轮manual不应使用Black-Scholes定价，需考虑其他方法

---

## 2026-04-27 01:10

- #algo-trading: 本轮manual trading中IMC在wiki明示价格路径遵循GBM，使用Black-Scholes定价vanilla期权时，主要被打破的假设仅为离散监测
- #manual-trading: 有选手观察到AC_50_P（执行价50的看跌期权）价格显著高于其他期权，原因待解（可能存在定价异常或套利机会）

---

## 2026-04-27 01:53

- Manual trading round讨论：选手认为该轮manual的最高期望值约为160-166k，超过这个数字（如233k）几乎不可能，更高收益伴随极高STD风险
- 数据变化提醒：本轮数据相比上轮有较大变化，上轮+40k PnL的策略在本轮变成-20k，需重新校准
- 有选手认为算法中可能缺失了trader flow（订单流）这一信号维度

---

## 2026-04-27 01:58

- #manual-trading: 关于本轮manual题目，认为运气成分不像看起来那么大——由于大家都会使用有限的几种正EV组合，关键是看与最优组合之间PnL的偏差；相关性回撤比非相关回撤更大，因此排名方差可能没那么高
- #algo-trading: 有人推测数据/bot行为中隐藏着多个alpha信号值得挖掘

---

## 2026-04-27 02:58

- Manual交易：price列可忽略（仅用于展示总投资额，部分显示不准），决策不应受其影响
- Manual交易：每个合约获得10 XIRECS，再乘以合约规模
- Manual交易：所有玩家的交易都在相同的100次模拟上评估（公平统一）

---

## 2026-04-27 05:24

- Manual交易: Chooser option机制澄清——若行权价高于标的资产价格则变为put，若低于则变为call

=== #algo-trading ===
NO_INSIGHTS

---

## 2026-04-27 05:35

- Round 4 manual交易涉及Chooser option：行权价高于标的价时变成put，低于时变成call
- 标的资产价格为100次GBM模拟的平均值，样本量适中，价格波动会被平滑，相对可预测
- Manual交易盈亏按3000倍缩放（仓位上限相关）
- Round 4 manual结果含运气成分，但100次模拟已显著降低随机性

---

## 2026-04-27 06:10

- #manual-trading: 期权合约T+14代表14天后到期但GBM模拟只走10个交易日(40步)，T+21对应15个交易日(60步)
- #manual-trading: 模拟共运行60步，T14到期合约在第40步后PnL保持恒定，T21合约PnL在40-60步之间继续波动

---

## 2026-04-27 06:30

- Round选手分享：官方PnL约66k，其中hydrogel贡献约1.6k，options约5k（每个产品有差异）
- Manual交易提示：KO（Knock-Out）是普通看跌期权，除非被敲出失效

---

## 2026-04-27 06:45

- #algo-trading: IMC官网回测样本量较小（约为完整一天数据的1/10 ticks），实际提交分数大约是回测的10倍；R4使用的是Day 3数据
- #manual-trading: Manual提交以100次模拟运行，结果存在较大不确定性，最终成绩按均值计算；某些选项组合方差极大，应优先选择结果较为稳定的组合

---

## 2026-04-27 07:15

- #algo-trading: 选手提到round 3和round 4的IV（隐含波动率）存在根本性差异，可能影响voucher定价策略
- #manual-trading: 讨论manual trading中分布预测的策略，提到使用CVaR优化而非纯EV最大化以应对潜在的不利seed风险

---

## 2026-04-27 07:40

- #manual-trading: 有选手指出本轮manual交易满仓做多是负EV策略，满仓做空EV更低，暗示中性或小仓位更优。

NO_INSIGHTS适用于algo频道（仅闲聊和排名讨论，无具体策略信息）。

---

## 2026-04-27 07:55

- #algo-trading: Round 3官方PnL与本地Backtest差距巨大（2.5k本地 vs 40k官方），提示backtest与实际bot行为差异显著，需研究bot行为
- #algo-trading: 今年波动率smile拟合不如去年，简单抛物线拟合效果差，部分选手尝试结合bot行为来改进smile拟合
- #manual-trading: Round 4 manual题估算：约1000总成交量 × 3000合约规模 × 0.05价差，可作为收益量级参考
- #manual-trading: 有选手用12 seeds × 30000 paths的蒙特卡洛模拟来评估manual策略
- #manual-trading: 讨论提到manual round存在通过极端"hail mary"策略博取小概率高排名的玩法

---

## 2026-04-27 08:10

- #algo-trading: 有选手用未修改的round 3算法在round跑出45k，提示无需过拟合即可取得不错成绩
- #algo-trading: VelvetFruit策略思路——观察到Velvet价格下跌足够多时假设进入下跌趋势并做空
- #manual-trading: Chooser option机制解读：买方选择PUT/CALL方向后，合约自动转为in-the-money的一侧；可通过选择out-of-the-money方向进行对冲
- #manual-trading: 关于manual trading的策略博弈观点——优化EV会让顶部选手聚集，反而最小化EV或押注极端情况（如赌70次运行都到45）可能在头部排名中胜出（增加方差）

---

## 2026-04-27 08:30

- Magnificent Macarons:spread过大且异常报价通常成交量低，难以从中获利

</result>

NO_INSIGHTS

Wait, let me reconsider. The message about spread being too large and outliers having low volume is actually a useful microstructure observation.

- spread过大且异常报价通常是低成交量quotes，难以从中获利

---

## 2026-04-27 08:35

- IV/期权策略实测收益有限：选手反映即使做IV scalping、smile fitting等期权策略，每天最多20k左右，远不及在主资产上做mean reversion能轻松赚到的200k
- R3排行榜高分主要来源于在主资产上做激进的mean reversion策略，而非期权套利
- 去年期权IV策略有效是因为spread普遍为1，今年spread变大后alpha显著缩水

---

## 2026-04-27 08:35

- algo-trading: 对每个voucher独立计算IV，跟踪IV的EMA作为"公允价格"进行交易的策略思路

---

## 2026-04-27 08:40

- #algo-trading: 多数选手在voucher（期权）产品上的盈利超过常规产品
- #algo-trading: IV的EMA策略效果不佳，因IV波动幅度太小无法覆盖买卖价差，快速期权交易容易被spread吃掉
- #algo-trading: 简单的阈值策略思路：价格高于阈值卖出，低于阈值买入（mean reversion）
- #algo-trading: 即使IV预测准确，也常因spread导致亏损

---

## 2026-04-27 08:46

- Volcanic rock凭证(options)的spread过大，单纯做mean reversion策略难以盈利

</result>

---

## 2026-04-27 08:50

- #algo-trading: ITM期权无外在价值，用BSM计算IV不合理；应关注bid/ask的IV，它们很少交叉
- #algo-trading: 今年期权策略难度显著提高，去年BSM做空+IV回归策略不再适用；存在合理的期权策略但收益很小
- #algo-trading: 4500行权价期权的IV变化对盈利贡献极小
- #algo-trading: 提到一个Rust回测器资源：https://github.com/GeyzsoN/prosperity_rust_backtester
- #manual-trading: Chooser option机制——2周后根据当时ATM情况自动转为call或put，之后还有1周到期

---

## 2026-04-27 09:05

- #general: 最终评测使用1天数据；网页PnL实际只是第3天前10万tick的数据，容易过拟合
- #algo-trading: 期权策略能做到300k算很强；mean reversion做到900k多半是过拟合；最终算法只在1天数据上测试

NO_INSIGHTS之外的内容已提取完毕。

---

## 2026-04-27 09:06

- Round 5测试数据说明：选手拥有Day 0,1,2,3的历史数据（来自R3和R4），最终在Day 4上被测试

---

## 2026-04-27 09:10

- Portal显示的是Day 3的10%结果，最终提交将在Day 4上运行（往届第5轮在Day 3测试，本轮在Day 4评估）

---

## 2026-04-27 09:25

- Hydrogel产品在提交日志中PnL大约1.8k左右（有选手反馈卡在该水平）
- 有选手观察到bot行为中存在隐藏alpha，只有在主动参与市场时才会显现，但需要反复提交测试才能验证

---

## 2026-04-27 09:35

- manual-trading: "price"列只是装饰性的投资成本展示，与PnL无关，可忽略；本轮manual交易无交易成本

NO_INSIGHTS之外仅此一条有价值信息。

---

## 2026-04-27 09:36

- vouchers策略：均值回归结合方向性押注，优先买入300（执行价）的voucher

---

## 2026-04-27 10:00

- 期权策略受bid-ask spread影响严重，即使预测准确也难以盈利

---

## 2026-04-27 10:26

- EMA策略在hydro品种上有效（约10k/天），需要较大的阈值和窗口，但回撤大，可能转负
- 由于hydro的regime不会像osmium那样变化，可考虑切换到SMA策略

---

## 2026-04-27 10:35

- manual-trading: Chooser Option机制讨论——3周后到期，2周后买家选择变为call或put（选择实值方向），最后一周作为标准期权。关于"周"是5天还是7天的天数定义存在歧义（wiki写3周，网站写21天）。
- manual-trading: 最终得分为100次底层资产模拟的PnL平均值（机制需进一步明确）。

---

## 2026-04-27 10:37

- OTM期权在round 5的第3天会出现衰减效应，前几天有效的策略在第3天可能失效，需要针对性调整

</result>

---

## 2026-04-27 11:52

- Round 4 中通过分析bot行为寻找交易模式的尝试普遍困难：发现的统计显著模式往往无法盈利，缺乏明显的时序套利机会
- 怀疑高分（90k）选手是通过事后过拟合数据集，根据观察到的标的价格走势直接做多/空voucher，这种策略在比赛中容易崩盘（上一轮已有先例）
- 有选手将mark信号作为算法约30%的组成部分，回测可达约800k

---

## 2026-04-27 12:00

- VOLCANIC_ROCK (VEF) 上有一个明显的informed bot信号，可作为交易参考
- HYDROGEL (volcanic_rock vouchers相关) 上有两个"dumb bots"，可被利用
- 这些bot行为清晰但难以有效变现

---

## 2026-04-27 12:08

- #algo-trading: 有选手认为Mark22挂单中存在买方edge机会（如VEV5300等档位edge约+0.51到+0.74/单位），但edge偏小，可利用性存疑
- #algo-trading: Mark49被认为是较有alpha潜力的信号，t-stat较高，但可能是噪声
- #algo-trading: 有选手提示Mark系列产品本身是可交易的，不仅是参考信号

---

## 2026-04-27 12:13

- 有选手认为4个bot中有3个是可利用的（exploitable），可针对性设计策略
- 提及对手价格67可作为持续买入的参考点

---

## 2026-04-27 12:33

- Round 3包含day 0,1,2数据；Round 4包含day 1,2,3数据；Round 4提交将在Day 4数据上运行
- 有选手认为IMC在bot数据中加入了过多噪声，bot行为不像去年那样明显

---

## 2026-04-27 12:43

- 建议利用已识别的bot trader IDs，回溯前几轮数据交叉分析，可能发现新的信号
- 提醒：Round 4的Day 1和Day 2数据与Round 3相同，重复回溯并无新增信息

---

## 2026-04-27 13:03

#general
- Round 4 manual题(VEV)分产品收益参考：HYDROGEL_PACK +82k、VELVETFRUIT_EXTRACT +74k、VEV_5000 +104k、VEV_5100 +99k、VEV_5200 +76k为主要盈利来源；VEV_4000/4500/5500收益很低，总计约+467k
- Round 4 manual按天收益：Day1 +232k、Day2 +84k、Day3 +150k

#algo-trading
- HYDROGEL单品采用均值回归策略可获约2.6k PnL，更高收益（如70k）疑似过拟合

#manual-trading
- 讨论manual题3000x乘数可能存在机制漏洞/过强影响

---

## 2026-04-27 13:09

- #algo-trading: 有选手观察到当价格偏离本地均值±6时，价格几乎必定回归（反向移动），但也有人提示这可能是过拟合信号
- #algo-trading: 跟随bot "Mark 67"（疑似优质交易员）的方向是一种alpha策略，反向操作（向他抛货）效果不佳
- #algo-trading: 通过在不同回测器或官方IMC表现间对比一致性可以判断策略是否过拟合
- #manual-trading: Round 4 manual题目理论最大收益>8.5M但需极大运气；建议不要全仓投入全部可用volume，否则单次亏损可能抹掉Round 3/4/5全部利润

---

## 2026-04-27 13:29

- manual-trading: 本轮manual交易理论最高收益约8.5M，虽然不太可能达到但技术上可行
- algo-trading: 有人提到对magnificent macarons (mark 67)估算fair value效果不错，但可能过拟合

---

## 2026-04-27 14:04

- hydrogel信号不可靠，不要盲目信任（即使informed traders也在交易hydrogel）
- 期权/时间相关计算中，BASE_DAYS_LEFT=4.0，DAYS_PER_YEAR取值（365 vs 252）存在讨论

---

## 2026-04-27 14:14

- #manual-trading: 有选手提到用Black-Scholes对vanilla options定价后，部分期权明显存在错误定价，可作为套利机会
- #algo-trading: 有选手指出本轮如果不利用bot（mark方）的交易行为，策略基本没竞争力，bot behavior是关键alpha来源

---

## 2026-04-27 14:24

- #algo-trading: 用midprice计算波动率曲线时得到的是frown（倒微笑）而非smile，需注意计算方式
- #algo-trading: 波动率smile相关计算可直接在代码中实现
- #manual-trading: 订单簿中显示数量为50

---

## 2026-04-27 14:40

- #algo-trading: 仅Velvet及其期权策略在历史数据回测PNL达385k，但IMC官方backtester显示负PNL；提示注意官方backtester只用了第3天10%数据，参考价值有限，需警惕过拟合和bot行为依赖。
- #manual-trading: Aether Crystal的描述文本（"存储和稳定能量波动"）可能是题目隐藏信号；AC_50_P为执行价50 XIREC、到期21天的看跌期权，从Round 1开始计时，目前在Round 4，需根据已过天数计算剩余到期时间。

---

## 2026-04-27 14:45

- algo-trading：Round 4策略需在Day 2这种极端行情下仍保持稳健，许多人因未对Day 2情景做压力测试（如3倍极端化）而爆亏-100k；可用蒙特卡洛模拟验证策略鲁棒性
- algo-trading：Round 4网站非troll最高PnL约67.5k，使用与上轮相同策略
- manual-trading：manual交易中price列只是装饰性显示"投资成本"，与PnL无关，不应影响决策

---

## 2026-04-27 14:55

=== #algo-trading ===
- 有选手在IMC回测中第3轮PnL约7.5k，与本地回测差异较大，需注意是否过拟合
- 讨论使用Rust回测器时的 --trade-match-mode worse 参数选择
- 不含informed bots时策略约790k，含bots后约60-90k范围

=== #manual-trading ===
- Round 4期权题(AC_50_CO等)：notion说明t=0买入持有至到期，但界面显示21/14天到期，需澄清实际剩余时间
- 期权"自动转为in the money方向"机制中，若到期价格恰好等于行权价(ATM)，默认行为不明
- 100次模拟取平均的设定下，押注极端outlier不是可行策略
- 模拟实验显示：纯max EV策略在100k次对比中超过50%的情况击败hedging策略，尽管IMC似乎暗示鼓励hedging

---

## 2026-04-27 15:00

#manual-trading
- 据Jack确认，本轮manual交易模拟使用真随机种子（同样的100条随机游走对所有队伍，但seed随机），不会针对max EV策略选择对抗性种子
- 部分选手怀疑提示中暗示会用对抗性seed，对max EV策略持保留态度

其余频道无有价值信息。

---

## 2026-04-27 15:05

- #algo-trading: 算法测试只跑当天数据的10%（约200k-300k之间的开始时间段），需要注意backtester样本与实测样本可能差异较大
- #algo-trading: 上轮有人website backtest仅7-7.5k但最终排到第3名，说明IMC官方backtester分数与最终排名相关性不强，不要过度优化backtester
- #algo-trading: 多数人在volcanic rock vouchers相关产品上把fair value锚定在5250（Claude推荐值），但锚定该价位的策略在实际round中容易翻车
- #manual-trading: IMC官方确认（tomas5880）测试时不会刻意挑选有利/不利seed，但若历史数据与最终测试数据因随机性差异过大（如历史均值回归vs最终单边趋势），会重新随机抽取seed以避免不公平
- #general: 建议用round 4的数据来回测验证策略在不同环境下的鲁棒性

---

## 2026-04-27 15:10

- Micro-price论文推荐：Sasha Stoikov的《The Micro-Price: A High Frequency Estimator of Future Prices》(SSRN 2970694)，micro-price是基于bid-ask spread和order book imbalance对mid-price的调整，可作为fair value估计
- 有选手询问OBI(order book imbalance)是否适用于VOLCANIC_ROCK_VOUCHER (velvet)，暗示OBI可能不是该品种的主要alpha来源

---

## 2026-04-27 15:15

- #manual-trading: 据称IMC官方确认（manual round相关）scaling是线性的
- #algo-trading: 有选手在hydro上用EMA做market making；OBI（订单簿不平衡）被提及作为常见策略；有人尝试用历史数据拟合隐含波动率smile/分布

NO_INSIGHTS之外其余多为闲聊和求助，无实质策略细节。

---

## 2026-04-27 15:35

- #algo-trading: 顶尖队伍（前2名）可能采用了与其他人不同的alpha策略（特别是在期权方面），其余前10与前20的差距更多来自mean reversion参数拟合的运气
- #algo-trading: 参数调优建议使用Optuna，选择"最稳健"的参数而不是回测最优参数（避免过拟合），最优参数实际表现往往更差
- #algo-trading: Hydrogel和VOLCANIC_ROCK_VOUCHER (VFE/VEV)可以通过informed flow（知情订单流）来改进策略
- #algo-trading: 改进voucher策略的方向：inventory skew、toxic flow filter（如Mark 67）、dynamic voucher scaling
- #manual-trading: 第4轮manual题目方差巨大，同样规则的100次模拟平均值在不同run间从-90k到+800k波动，建议对冲或选择性下注而非全押

---

## 2026-04-27 15:40

- #algo-trading: 选手建议使用稳定参数而非最高目标分数的参数（避免过拟合）
- #algo-trading: 主要PnL来源于VOLCANIC_ROCK_VOUCHER（VEV）的杠杆操作
- #algo-trading: Mean Reversion策略可用合理fair value估计并在其周围建band
- #algo-trading: 多alpha叠加（stacking alphas）的思路
- #algo-trading: 模型选择：分别调参后再比较目标分数选最佳
- #algo-trading: 对VFE，volatility smile不一定直接用于mean reversion信号
- #manual-trading: Manual题评估机制讨论：可能是100次试验的平均PnL，提示倾向风险调整后的PnL，暗示风险管理+高波动率组合较优

---

## 2026-04-27 15:45

- #algo-trading: 参数调优应避免"虚假精度"，建议以0.05为步长设置参数（如用5.60而非5.6161151515），避免过拟合
- #algo-trading: Hydrogel/VFE等产品可结合MR（均值回归）和MM（做市）策略
- #manual-trading: 一旦确定正确的fair value，最优均值期望是确定性的；高方差高均值策略风险大，意义不大

</result>

NO_INSIGHTS的反面——已提取见解如上。

---

## 2026-04-27 15:55

- #algo-trading: voucher产品基于Velvfruit而非hydrogel（wiki中明确说明）
- #algo-trading: fair value估计可用快慢EMA组合，每个tick更新（状态机方式）
- #algo-trading: 顶尖队伍可能掌握额外alpha，最终成绩仍含运气因素（最终holdout种子）

---

## 2026-04-27 16:00

- IMC portal提交测试用的是day 3的前1000个tick（有讨论说法不一，也有人认为是day 4的前100k tick）
- 提交代码时不要打印过多数据，会拖慢模拟速度
- 代码可以使用多种特征（mid price、microprice等）的线性组合作为信号
- 600行代码可能过于复杂，参考量级在40KB左右

---

## 2026-04-27 16:05

- Portal提交测试运行的是Round 3的Day 3前1000个tick的数据（可用旧代码跑出Round 3 PnL来验证）
- Manual trading各队伍使用相同的随机种子

---

## 2026-04-27 16:10

- #algo-trading: 寻找信号的方法建议——将CSV图表与纯多头算法对比，从counterparty入手，结合size等组合可能产生可预测的正向信号
- #algo-trading: 有人提到HG（可能指某product）每天可盈利100万+
- #manual-trading: 官方不会通过hack seed限制PnL，但会避免样本外数据与历史数据风格差异过大（如历史均值回归、样本外单边趋势）的误导性情况

---

## 2026-04-27 16:30

=== #manual-trading ===
- 多数人会跑max EV策略，PnL约150-160k；想要胜出可考虑略偏离max EV并加入方向性押注以博取运气
- 由于大部分参赛者实际无风险压力，单纯max EV可能不足以拔尖

=== #algo-trading ===
NO_INSIGHTS

---

## 2026-04-27 16:35

- #algo-trading: 有选手提到Magnificent Macarons (Mark 67?) 用 T-score 和 wall mid 策略赚钱较容易；Hydrogel (Volcanic Rock?) 策略PnL约200k
- #manual-trading: Round 5 manual题模拟40次不同策略，最大EV结果方差极大（-200k到600k），结果高度不确定

---

## 2026-04-27 17:00

- manual-trading: 关于AC_50_P看跌期权TTE的讨论，wiki中说明实际TTE应为15天而非题面所述的21天，且与Round 1无关联

---

## 2026-04-27 17:05

#algo-trading
- IMC题目通常设计了特定的数学函数关系，参数较少，神经网络不是最优解，应该尝试找出准确的函数关系而非拟合大量参数
- 历史数据量不足以正确训练LSTM或做regime classification
- 有人提到去掉delta hedge后web PnL达到9k（高风险但可能高收益的方向性押注）
- Magnificent Macarons (mark67?) 的规律简单，可用简单规则准确预测，无需神经网络

---

## 2026-04-27 17:10

#algo-trading
- 有选手认为可通过识别Mark67交易spike的规律来"前置交易"他，认为该模式存在于数据中但需要brute force寻找
- 有观点认为Mark67不是内部人，其行为基于K线（candlestick）规则
- 回测工具推荐：jmerle的backtester（已被fork并适配本年Round 4）和Rust版backtester，两者PnL一致
- 注意：依赖bot成交行为的策略在回测中可能无法真实反映，因回测订单是"预设"而非真实撮合

---

## 2026-04-27 17:46

- #algo-trading: 有选手提示观察每个 velvet_extract（疑似产品）价格图表的起始部分，可能藏有规律线索
- #manual-trading: T+14 解读为 2 周交易日 = 10 个交易日，对应 4×10 步（manual交易题目机制解读）

---

## 2026-04-27 18:15

- #general: run方法每次调用应在900ms内返回（平均≤100ms），否则超时，需保证算法轻量
- #algo-trading: HYDROGEL被建议采用均值回归策略
- #algo-trading: IV scalping策略上午盈利明显，但中午后vol定价趋于合理，担心在10倍tick数据上不可扩展
- #manual-trading: 本轮manual交易若达到+0.5 SD约可获利100万，1 SD下行风险约600万，方向性押注潜在收益巨大

---

## 2026-04-27 18:20

- 有选手发现：基于偏离滚动均值的概率进行随机交易，效果竟然比原本认真设计的策略更好（algo-trading频道）

NO_INSIGHTS对manual-trading部分（仅为关于下注策略的闲聊讨论）

---

## 2026-04-27 18:30

- #algo-trading: 有选手反馈mean reversion策略在velvet上有效，但在hydrogel上不奏效

---

其余消息均为闲聊或玩笑，无实质交易信息。

---

## 2026-04-27 18:35

- Round 3算法策略：使用EMA可达200k，加入线性回归可能提升至350k；用R3 day 0/1/2回测，day 3作为隐藏验证集（避免过拟合，且对R4更稳健），可做到800k
- Manual round被认为是本届最重要一轮，押注方向性可能带来巨额收益，但也可能扰乱算法表现

---

## 2026-04-27 20:20

- #algo-trading: 有人对hydrogel采用均值回归策略，需要确定计算窗口大小和偏离均值的阈值参数

---

## 2026-04-27 20:22

- 关于gamma scalping：可以做但实现不容易
- backtester中策略上限大约在800k左右

---

## 2026-04-27 20:30

- #general: 回测和实盘表现不一致是普遍现象；上一轮获胜策略在实盘有负PnL，因此实盘表现的可信度需谨慎评估
- #algo-trading: 尝试操纵产品22未成功；但存在可利用的bot行为模式，不过收益较薄

---

## 2026-04-27 20:50

- #manual-trading: 有选手认为manual题不需要对冲，直接最大化期望收益(max_E)即可

NO_INSIGHTS其余内容多为闲聊。

（最终输出仅一条有效见解）
- #manual-trading: 对于本轮manual题，有人主张无需hedge，直接最大化期望值

---

## 2026-04-27 20:57

- manual-trading: 一份期权合约对应3000份Ancient Coins (AC)，单合约最大3000

NO_INSIGHTS之外仅此一条有价值信息。

---

## 2026-04-27 21:10

- IMC官方网站回测器只有10万个时间戳，方差较大，不应过度依赖；策略基本面合理即可
- 实际比赛表现与回测分数不一定一致（有人回测4k实际超过回测15k的人）

</content>
</invoke>

---

## 2026-04-27 21:37

- Prosperity平台显示的PnL大约是最后一天数据前1/10部分的结果（参赛者推测）

---

## 2026-04-27 22:30

- Gamma scalping策略据称单日可达20万收益（但有人反对认为不值得花时间）

---

## 2026-04-27 22:43

- #algo-trading: 某策略在round 3数据平均89k，round 4三天分别为90262/89204/86810，首日91k，含少量硬编码
- #algo-trading: 有人尝试预测mark 38出现时点未果
- #algo-trading: 对hydrogel质疑，认为除了均值回归+做市没有其他alpha
- #manual-trading: 二元期权payoff为10，但有下行风险，模拟器显示的高平均值可能错误
- #manual-trading: 手动交易EV约+1M，标准差约1.5M

---

## 2026-04-27 23:54

- Round 4 (equirag/可能是magnificent macarons相关) 硬编码最优解的最大收益约为28万

---

## 2026-04-28 00:09

=== #manual-trading ===
- 期权到期时间换算：每7天对应5个交易日；14天和21天到期分别还剩10和15个交易日
- 有人推测volcanic rock vouchers的标的均值需要落在50附近，否则会让大量队伍获得巨额利润
- 满仓押注（full porting）单一方向并非最优策略，除非已经落后需要博一搏
- 有团队在manual交易上达到约45万PnL（95分位）

=== #algo-trading ===
- 讨论用jump-diffusion过程或Volterra过程对期权/标的建模，可能涉及regime change
- 提醒注意当前submission测试可能在样本内数据上跑，存在过拟合风险

(其他多为闲聊，无实质信息)

---

## 2026-04-28 00:14

- #general: 官方确认会监控数据合理性，若历史与最终测试数据差异过大（纯随机导致），会用新的random_seed重跑以保公平。
- #algo-trading: Hydrogel存在强均值回归特性，可考虑使用mean field market making策略。
- #algo-trading: 提到用stochastic HMM做regime change point检测的思路。
- #manual-trading: 期权类题目Time To Expiry为21个Solvenarian Days（从Round 1的Intara开始）。

---

## 2026-04-28 01:20

- 部分选手在第3天主要靠交易VOLCANIC_ROCK_VOUCHER_6000和6500获得大部分收益
- 有选手通过交易5000和5100行权价的voucher获得约25万收益
- 提示：6000 voucher可能因深度虚值而有特定交易机会（"Mark 55"分享过相关思路）

---

## 2026-04-28 02:15

- #manual-trading: 官方确认round 4 manual所有队伍共用同一组100次模拟运行（不是每队独立模拟）
- #algo-trading: 第3轮算法收益普遍存在过拟合现象，portal回测20k实测48k或portal13k实测-45k等偏差较大；无过拟合的合理上限约96-97k
- #algo-trading: 有人怀疑hydrogel（新product）适合均值回归策略

---

## 2026-04-28 02:25

- #algo-trading: 有选手提醒本轮要警惕regime shift，不要过度拟合短窗口测试，drawdown可能反而是面对未来regime的好信号；认为40-50k回测PnL是合理上限，过高可能是过拟合
- #algo-trading: 网页模拟器(web sim)结果不稳定，不适合作为基准，建议使用本地backtester并查看day-wise PnL以及跨天一致性来检验稳健性
- #algo-trading: IMC服务器测试只跑Day 3的约10%数据
- #general: 有选手提到bots只能用来微调fair value，没有很强的预测性
- #manual-trading: 关于Solvenarian交易日换算的疑问 - 21 Solv日对应15交易日，14 Solv日对应10交易日（未确认）

---

## 2026-04-28 02:30

基于这些消息，提取的有价值信息有限，主要集中在manual trading的期权定价上。

- #manual-trading: 用15个交易日校准的波动率，AC50C/AC50P在T+21的理论价约12.03，与市场报价12.00/12.05一致；若用21天则fair约14——表明应使用15天历史波动率进行校准
- #manual-trading: 模拟基于约100条预定义路径（可在Wiki确认）
- #algo-trading: 网页端PnL存在方向性运气成分，单次成绩参考价值有限
- #algo-trading: 多数选手判断本轮期权存在mispricing，关键在于风险缓释和库存优化，而非单纯加大size做市
- #algo-trading: 高PnL截图可能是对特定regime的过拟合，未必稳健

---

## 2026-04-28 02:35

- #algo-trading: 大多数选手在期权上采用均值回归策略，认为期权未定价错误且波动率不足以做市。
- #algo-trading: 常见做法是直接将期权头寸顶到最大库存做空，以最大化PnL博取排名。
- #algo-trading: 由于比赛排名机制，承担最大风险（pnlmaxx）比追求低回撤更有优势，但存在被组委会"惩罚高风险"轮次的担忧。
- #algo-trading: 均值回归策略可达到约50k PnL量级作为参考基准。

---

## 2026-04-28 02:35

- algo-trading: 选手普遍在做空OTM期权
- manual-trading: 有人选择在KO（看跌期权？）上顶满仓位作为赌博性下注，期望博取头名

---

## 2026-04-28 02:40

- #manual-trading: Round 3 manual收益基于100次模拟的平均值，但因底层波动大且有杠杆，运气好的种子组合可能让PNL进入百万级（有人测出25%分位达到2M的策略），不过若不在25%分位内则可能爆亏出局
- #manual-trading: 策略思路两极——要么全力优化博运气拿大奖，要么直接放弃manual部分
- #algo-trading: 部分选手出现回测(Rust 427k)与IMC官方回测(95k)差距巨大，疑似严重过拟合的信号

---

## 2026-04-28 02:55

- #general: 机器人生态可解码为星型拓扑结构，包含显性做市方、被动流动性提供者及预测型机器人（如47号在订单簿偏斜、买盘较重时逢低买入），但这些信号大多不可直接交易，主要是基于配对或部分信息/速度优势
- #general: 均值回归（MR）策略主导整体PnL；从MR腾出容量去跟随机器人或在hydro被动报价并不划算
- #algo-trading: 有传言mythos建立了操纵VOLCANIC_ROCK_VOUCHER_6000和6500市场的机器人，效果不错（可关注这两个期权产品的异常）
- #algo-trading: 大量选手PnL曲线形状相同只是幅度不同，提示主流策略高度同质化、可能过拟合R3数据；建议抛弃LLM建议，直接看价格图人工设计策略
- #manual-trading: Round5手动题约100次模拟下平均EV普遍在180k左右，更高数值（如867k）多为样本不足或计算错误
- #manual-trading: 纯max-EV策略受低概率长尾影响较大，考虑中位数或处理左偏分布可能更稳健；多数人不会对所有衍生品都下注以控制风险

---

## 2026-04-28 03:00

- 网站显示的PnL是基于day 3数据，而最终得分是基于选手看不到的day 4隐藏数据集运行的，应以网站为准
- 提交结果的PnL限制在前10k个timestamps内
- Manual round的100次模拟路径对所有人相同（非各自随机）
- 有选手提及Volcanic Rock voucher 6000和6500的盈利情况存疑，可能不赚钱

NO_INSIGHTS之外的零散信息有限，多数为闲聊。

---

## 2026-04-28 03:10

- 一个回测PnL分布：Day1 138.6k / Day2 124.3k / Day3 147.1k，三天总计约410k，可作为合理算法基准（非过拟合）。
- 观点：若Day3前10%ticks就能赚到当日45%的PnL，几乎可判定为过拟合。
- 观点：3天总PnL超过400k的算法多为过拟合，105k单日也偏过拟合。

#manual-trading
- 本轮manual类似期权结构，最大可能盈利量级约5m，但需极端运气；存在可设计成50/50 coin flip的策略（赢得百万或全损）。
- "Total investment"列会从PnL中扣除，"price"列只是装饰性显示，无实际作用。
- EV计算时可考虑把25,000美元奖金纳入。
- 有选手提到能构造出Sortino比率约20的策略，提示存在风险调整后较优的稳健解。

---

## 2026-04-28 03:55

- #general: 回测显示单层 mid±7 做市效果最佳（25,787 PnL），优于 mid±6（21,820）和 mid±5（16,748）；所谓多层挂单的收益其实只是继承了内层的edge，并未带来额外捕获，属于回测假象
- #manual-trading: 第4轮期权TTE相关讨论，有人提到是10天（需自行验证）

NO_INSIGHTS 之外的有效信息已提取。

---

## 2026-04-28 04:00

- 期权仓位限制为每个300（algo-trading频道，syna2671提到）

NO_INSIGHTS其余内容为闲聊。

---

## 2026-04-28 04:10

- R4最终模拟中所有bot都已"revealed"，不会有新bot出现，但并非每个bot都会在每个timestamp都交易
- manual trading策略讨论：若无保底压力，可在最大EV组合基础上叠加对底层资产的多/空头寸，使用约0.9的Sharpe筛选
- 由于manual奖金上限5000，落后者倾向于赌博式下注

---

## 2026-04-28 04:25

- #manual-trading: 有选手用Monte Carlo模拟（7600万组合）寻找manual round的最优期权组合，结果显示AC_45_KO（敲出期权）大量做多+AC裸空200+多个put/call组合可获566k均值，但标准差极高（3000万+），Q5为-5700万，本质是高方差赌博
- #manual-trading: 该策略的核心是大量持有AC_45_KO（500手）配合空AC标的和复杂的put/call spread结构
- #algo-trading: round 4 backtest表现：顶尖选手平均每日PnL在320k-400k+，网站显示68k但backtest达620k+的策略被认为相对不overfit
- #algo-trading: 部分选手担忧round 4末段VFE等volcano相关产品价格可能剧烈波动（如冲到7k），策略对结尾行情表现存在不确定性

---

## 2026-04-28 04:31

- algo-trading: 使用本地backtester比官方网站更准确，网站结果不可靠（部分选手反映网站排名差但实际进入T10）
- manual-trading: 多数人选择买入KO，预计大部分玩家收益会达到+3M左右，因此跟随大众买入难以拉开差距

NO_INSIGHTS之外的有效信息已提炼完毕。

---

## 2026-04-28 04:35

- #manual-trading: 本轮manual被认为是做空标的至200最优，EV为正（基于100k次模拟），但方差较低，胜负主要靠技巧
- #manual-trading: 有选手用模拟（100k次×100条路径）评估策略EV，确认做空方向EV为正
- #algo-trading: round 4提交时TTE=4，网页显示TTE=5（时间到期参数差异需注意）

---

## 2026-04-28 04:45

- 期权策略警告：不要在BT的day 1/2数据上过度拟合，因为有theta衰减——越接近到期某些期权会变得一文不值，过拟合容易产出垃圾策略
- 同样不要过度拟合web上的day 3数据，需在两者之间保持平衡
- 有选手BT达到329k PnL但提交后只有4k，提示BT高分往往是过拟合的信号
- Manual轮次：有人提到"做空KO是meta"，但也警告该轮存在巨大下行风险，可能是高方差赌博性质
- 怀疑Manual的样本可能被组织方挑选过以偏向低方差策略

---

## 2026-04-28 05:16

- #algo-trading: 本轮（最后一轮）排名靠前的队伍多采用均值回归(mean reversion)策略

NO_INSIGHTS（manual-trading部分无实质信息）

---

## 2026-04-28 05:20

- #manual-trading: 比赛排名只有前5名有实际价值，类似看涨期权——最优策略是增大方差（做高波动决策），即使是负EV也可能因提升入围概率而合理
- #manual-trading: 据传组织方可能会剔除异常seed或在某人赚到极端收益时重跑manual round，参赛者应有此心理预期
- #algo-trading: 第3轮可使用与前一轮相同的提交文件（说明策略具有跨轮稳定性）
- #algo-trading: 有团队尝试通过操纵fair price进行交易，并怀疑Mark（大额交易者）是按VWAP执行
- #algo-trading: 关于期权类产品定价，提到使用GBM + 蒙特卡洛模拟来确定交易阈值

---

## 2026-04-28 05:45

- #manual-trading: 有选手将本轮manual trading视为call option模拟，使用优化器最大化P(PnL>2M)的概率（约30%），而非最大化期望，PnL分布近似正态，0均值、3.5M标准差
- #manual-trading: 本轮的效用函数类似扑克ICM，倾向选择高风险高回报选项以最大化方差
- #algo-trading: 历史数据频谱分析在3000和8888处有尖峰但无实际用处；DSP方法效果不佳，统计方法更有效（去年冠军即统计派）

---

## 2026-04-28 05:57

- Rust回测器与官方网站行为基本一致，仅需少量改动；但回测器中没有bot逻辑，网站上才有
- 观察到价格抬高1单位时，bot 67会从买变卖，可用于推测bot行为

NO_INSIGHTS的部分：manual频道仅是闲聊，无实质信息

---

## 2026-04-28 06:05

- #algo-trading: R3 中做市（making）比吃单（taking）更难，是普遍共识。

---

## 2026-04-28 06:12

- Manual round对冲分析：裸chooser策略相比对冲策略平均PnL多约$41k，但std增加$200k+，Sharpe从0.364降至0.11，属于无补偿的方差，对冲锁定$60k无风险收益更优
- Manual round预期PnL区间根据风险承受度在40k-160k之间

---

## 2026-04-28 06:17

=== #general ===
- Hydrogel（疑似新product）表现为趋势+均值回归特性，有10个期权合约

=== #manual-trading ===
- 观点：比赛奖励结构类似深度OTM看涨期权，PNL波动率比PNL绝对值更重要，应该追求高方差策略而非稳健提升
- 有人尝试用MILP（混合整数线性规划）求解manual round

---

## 2026-04-28 06:20

- #manual-trading: 深度OTM期权的价值在于"错误定价的凸性"(mispriced convexity)，而不是因为累积波动率；评估基于所有模拟的平均值
- #manual-trading: 用例子说明凸性价值——给定2M行权价的call，N(0, 3M²)分布比N(100k, 300k²)更有价值，因为高方差带来更大尾部收益
- #manual-trading: 若对算法部分有信心，manual部分可以保守选择
- #manual-trading: 有人观察到LLM普遍建议"最大化凸性"，反向操作可能是个思路（半开玩笑）

---

## 2026-04-28 06:42

- Round 5 manual交易最高EV策略约为160-166k，存在另一个EV略低（163k）但方差降低40%的备选策略
- 高方差策略是赢得本轮的关键，降低方差会显著降低期望收益
- Round 5算法交易中存在小规模套利/alpha机会（约10k级别），可能涉及篮子交易（basket trading）

---

## 2026-04-28 06:52

- manual-trading: 最终PnL基于100次模拟平均，因此无需特意选种子
- manual-trading: 在winner-take-all的比赛机制下，最大化方差（而非EV）才是最优策略，因为负EV在百万级标准差面前不重要
- manual-trading: 存在"最优赌博"策略——根据排行榜位置动态选择风险水平（SR）以最大化夺冠概率

---

## 2026-04-28 07:07

- #manual-trading: 有人先优化P>2M的概率,再针对预测他人会用的10种策略微调胜率和胜幅作为manual题策略
- #manual-trading: 提示max EV策略可能会被组织方"惩罚",纯粹追求最大期望值不一定最优
- #manual-trading: 替代思路—先估计每个选项fair value的分布,再基于分布最大化PnL
- #manual-trading: 一个参考结果:期望PnL约28.6万,标准差22.3万,胜率90.27%,99% VaR约-20.4万

---

## 2026-04-28 07:13

- #manual-trading: 有选手分析认为买入KO（看涨）只在约4.7%的情况下盈利，否则趋向归零，暗示该manual题目期望值偏负，需谨慎下注。

NO_INSIGHTS其余消息基本为闲聊。

---

## 2026-04-28 07:23

#manual-trading
- manual题目可能并非随机模拟，措辞"100 simulations of the underlying"暗示结果间存在相关性，采用+EV策略时需考虑结果相关性
- 由于结果相关，做与高EV策略相反方向的对冲/反向选择也可能是一种博弈思路

---

## 2026-04-28 08:08

#manual-trading
- Round 4 manual题最优策略是选高EV低方差组合；实际结果方差被降低，只要EV为正基本都能赚到，纯粹追高EV更优
- 许多选手因追求高EV高方差导致负收益，排名大跌
- 75-80k被认为是不错的manual分数

#algo-trading
- Round 4 算法回测样本内PnL参考：好的选手3天约90万-100万（约30万/天），一般水平约20万/天
- Round 5 引入约50个新产品
- 公开测试集并非第4天前10%数据，与正式评测数据不同

---

## 2026-04-28 08:13

- #algo-trading: IMC官网回测图表只显示最后一个历史数据日的前10%，需自行下载完整10k ticks日志分析
- #algo-trading: 本轮某product仓位限制仅10，参赛者认为偏小
- #algo-trading: 选手认为本轮存在大量alpha机会
- #manual-trading: Manual轮中位数约17k，顶尖成绩~170k；最大EV策略约34k
- #manual-trading: 关于Manual期权题——组织者似乎选择了低方差种子（如波动率精确251%而非随机扰动），导致最优策略是直接选最高EV、忽略方差，而非最大化方差去博尾部

---

## 2026-04-28 08:19

- #algo-trading: 有选手分享R5策略——使用硬编码的均值回归参数，并用ITM期权作为杠杆仓位
- #algo-trading: 有人在R3通过做市+吃单获得平稳PnL曲线但仅15k；提醒纯MM策略在后期回合收益有限
- #manual-trading: manual题目中如果所有选项都按mid price定价，整体会亏损spread（-300k到0k之间）
- #manual-trading: 蒙特卡洛模拟显示manual题存在两个峰值的最优解

NO_INSIGHTS（其余为闲聊）

---

## 2026-04-28 08:29

基于消息内容，提取以下有价值见解：

=== #algo-trading ===
- R3/R4算法部分核心策略：每个产品做市 + 均值回归即可，简单的"低买高卖"足以进入全球前20
- Hydrogel（水凝胶）适用OU过程均值回归策略
- 期权underlying也呈现均值回归特性
- 该轮理论最大收益约2500万，Day 2-4顶尖成绩在2467万-2497万区间
- 硬编码（hardcode）策略在holdout数据上无效，往年硬编码者只是复制去年数据做平移

=== #manual-trading ===
- Manual环节疑似使用固定seed（低方差seed），导致结果偏离EV
- 通过蒙特卡洛优化6个组合可达到99.2%概率获得270k以上收益
- 保守参数策略可获得约25k收益且几乎无亏损风险

---

## 2026-04-28 08:54

来自 #algo-trading：
- R3 voucher策略思路：在标的价格约5250附近过滤voucher价格，拟合曲线预测fair value，将曲线向左平移以预测未来fair，低于阈值买入、高于阈值卖出。

来自 #manual-trading：
- R5 manual为基于新闻文章的方向性预测题，需判断每条新闻对应product涨跌；若能预测分布则可最大化PnL。
- 全仓单一新闻方向（full send）实测仅约12k收益，说明并非最优策略，需结合分布/仓位分配。
- 公式中存在fee项，被扣预算（used budget）会从PnL中扣除，fee是否额外扣除尚不明确。

---

## 2026-04-28 08:59

- R5 manual策略讨论：最优策略是高EV(+34k)但高方差(500k std)的方案，而非+170k EV/350k std；最高EV方案严重short Gamma
- R5 manual据称是基于option类策略，有人采用straddle（年化波动率251%下押注突破盈亏平衡点），事后认为short strangle会更优
- 一种较好的R5 manual策略：P(loss)=28%, mean profit=160k，位于Pareto frontier上
- R5算法交易简单策略：买入所有strike<5250的期权，卖出所有strike>5250的期权，可达算法榜top 200

---

## 2026-04-28 09:30

=== #general ===
- Round 5 的价格将基于 Round 4 第4天结束时的价格延续
- Round 5 包含全新产品

=== #manual-trading ===
- 一份高分Manual交易组合示例（利润177,980）：BUY 160 AC + 多个put（35/40/45/50行权价 T+21）+ 卖出60C/50C2/50CO/40BP/45KO等组合，可作为期权组合策略参考
- 讨论中质疑：若160 AC是好的，为何不买200（暗示该解未必是利润最大化，而是评分最高的提交）
- 有人质疑45_KO在被低估时却被max sell，提示组合中可能存在反直觉的最优解

---

## 2026-04-28 10:00

- R4 voucher市场结构观察：实际只有1个taker且交易量很小，其余都是做市商；taker会以不利价格成交。这种结构下IV smile做市+delta对冲难以盈利，简单的均值回归策略反而表现更好
- 构建voucher的IV smile时，可分别用voucher ask配underlying bid、voucher bid配underlying ask构建两条smile；ask smile较干净，bid smile呈阶梯函数形态（反映其他做市商的多空敞口）
- R4最终榜单顶级策略多为均值回归（而非复杂的期权做市/IV策略）
- 前两轮主要alpha为velvet均值回归 + 用ITM期权放杠杆
- R5最终轮可能PnL上限约1.5M左右
- R5 manual：文章只给方向性提示，但有手续费机制，缺乏量级估计信息，难以精确决策

---

## 2026-04-28 10:05

- manual-trading: 新闻主要影响公司，但交易的是产品（如黑曜石餐具生产停止会导致公司亏损），需要将公司新闻映射到对应产品的影响
- manual-trading: 即使确定某商品（如lava cake）会上涨，但如果历史上从未有过1%以上的波动，分配过多资金仍不明智；移动幅度难以估计

---

## 2026-04-28 10:15

- #manual-trading: 选手对manual题目存在困惑——不确定交易标的是商品本身还是公司股票，文本表述模糊（如lava cake的垄断描述）
- #manual-trading: 有人提出由于交易费用较高，单日内价格波动可能难以覆盖费用，考虑全部押注一个资产或全部填0的极端策略
- #manual-trading: UI显示卖出时会从预算中扣除，疑似bug或机制需澄清
- #manual-trading: 有选手提示"phoenix from the ashes"暗示可能与资产复活/重生相关（参考avatar Ozai等线索），属于题目叙事性提示

---

## 2026-04-28 10:30

- #general: IV错误定价本身具有均值回归特性，可以用IV估值做偏斜做市，同时利用错误定价的均值回归进行交易
- #general: 上届Prosperity前20对本届奖金资格无影响，仅前10在全球Top25队伍中受关注
- #manual-trading: 第4轮manual普遍亏损严重(多人报告-150k到-200k)，提示该轮难度高/陷阱多，需谨慎下注
- #manual-trading: 有选手认为第4轮manual最优策略是多目标优化中的帕累托最优采样

---

## 2026-04-28 10:35

- R4 manual题目核心：根据wiki提供的价格生成方式，逐bar计算各期权的公允价值并下注
- R4策略思路：需要假设组织方不会允许超过500k的疯狂PnL，据此做出条件采样决策
- 经验教训：1-sigma事件下押注2.5m的高回报诱惑通常不值得，多数随机价格路径较温和（68%概率不会出现极端情况），更安全的做法是接受100k左右的mean收益
- 提到R4手动交易中"商品"指可在平台建仓的标的；价格区间和锚点预配置，但参赛者提交会在区间内影响最终结果

---

## 2026-04-28 10:45

- 利用bots在低效报价时建仓，当错误定价显著时加仓并预期其回归到0，可整合到做市的库存管理系统中：目标库存与估值都基于mispricing（一个来自真实mispricing，一个来自其中的均值回归信号）
- 交易对手方信息可用于判断何时及如何吃单，避免被spread吃掉

---

## 2026-04-28 10:50

- Round 3 voucher套利在几何价差结构中已存在，并非市场参与者alpha；V5000有较强均值回归但其他voucher的IV错误定价仅±2，单独交易会被价差吃掉，估计alpha仅2-3k
- Round 3/4 vouchers的执行（skewed MM实现）比想法本身更重要，可能比单纯均值回归算法（~250k）更优
- Round 4 manual为100次模拟的随机游走，第一份样本即用作结算；若模拟诚实，最优策略是最大化方差（因下行有限、上行无限），中位数低于起点但均值持平

</thinking>

- Round 3 voucher套利在几何价差结构中已存在，并非市场参与者alpha；V5000有较强均值回归但其他voucher的IV错误定价仅±2，单独交易会被价差吃掉，估计alpha仅2-3k
- Round 3/4 vouchers的执行（skewed MM实现）比想法本身更重要，可能比单纯均值回归算法（~250k）更优
- Round 4 manual为100次模拟的随机游走，第一份样本即用作结算；若模拟诚实，最优策略是最大化方差（因下行有限、上行无限），中位数低于起点但均值持平

---

## 2026-04-28 10:55

- #general: 做市商通常追求库存中性，但本赛题反而需要根据估值主动建立库存，并结合反向工程的bot行为进行报价
- #general: R4的对手方信息显示只有1个taker和多个MM，taker每次只吃2-4手，导致难以靠捕捉spread盈利，纯MM策略无法获得好排名
- #general: 均值回归策略本届表现极佳，能直接进入前2名；这是Prosperity 2以来的主流有效策略
- #general: 深度ITM期权可以直接当作标的资产交易效果不错；中等ITM/ATM/OTM则用错误定价的均值回归更优
- #general: 当某个bot稳定提供流动性时，若fair value远离均值，可高概率被成交以建立库存，这是结合MM与均值回归的方式
- #manual-trading: 对数收益是可加的，但转换回实际收益需要相乘；-50%等价于+100%的非对称性导致大多数模拟结果低于起点（中位数低于均值），100次模拟样本不足时这点很关键
- #manual-trading: 组织方提供了模型和波动率并跑100次模拟，按理应支持精确EV计算，但实际却倾向于"安全"策略，存在矛盾

---

## 2026-04-28 11:05

- R5是"探索题"风格，相对R1-R4更exotic；MR（均值回归）策略在Prosperity中非常常见，因实际数据往往呈MR特征
- 有玩家用初版策略在backtester上达到约100万PnL（3天）
- Taker bot即使在spread达200的极宽情况下仍偶有成交，触发逻辑难以识别规律
- R5题目疑似涉及"基于交易成本的成交概率"机制
- Manual交易中volume_for_specific_product疑为百分比分配（如20代表20%），PnL计算机制及"used budget扣减"含义仍存疑

---

## 2026-04-28 11:10

- #manual-trading: 手动交易PnL计算说明——投入100k涨20%变120k时，PnL只增加20k（扣除本金），与平台费用公式逻辑一致
- #manual-trading: 每个产品的次日收益基于设定区间与玩家平均交易行为共同决定（确保不会出现单边错误定价）
- #manual-trading: 此轮新机制仅适用于manual trading，时间范围为1天

---

## 2026-04-28 11:25

- #general: Mark 67 (Volcanic Rock相关)可以被操纵——约22.3%的时间能让它以远好于穿越spread的价格与你交易；成交概率取决于报价距离mid的远近
- #manual-trading: R4 manual题目最大EV策略可能拿到约160k EV，但实际结果方差极大（最差39%约23.5k，高31%约354k），运气成分高

---

## 2026-04-28 11:35

- #general: 本轮CSV中buyer/seller字段为空，没有counterparty信息，但有许多新的bot参与
- #algo-trading: 可以交易全部50个产品，不限于单一类别
- #algo-trading: 有选手尝试自适应贝叶斯信任过滤器（根据insider近期胜率动态启用/禁用），但效果转负
- #manual-trading: 手动交易最大EV预期PnL约为159-161k

---

## 2026-04-28 11:45

- Round 4中bot信号的alpha不如R2明显，多数选手只发现轻微的skew信号
- Mark67被识别为VEV产品上的激进taker（非insider）：当挂单越过真实FV时会以小量(3-5)成交，可作为FV参考
- Mark38、Mark55、Mark01被认为是R4中alpha最强的几个bot
- 应对insider/激进bot的简单策略：在其交易后调整自己的fair value估计

---

## 2026-04-28 11:50

- R4期权可视为Delta 1策略的杠杆工具，不需要做gamma scalping，直接用于加杠杆方向性押注
- 全程均值回归（MR）策略在R4可获得约220-225k PnL
- 临近交易日结束需提高入场阈值，因为剩余时间不足以让MR策略回本，回本概率呈指数衰减（tick 8k约97%，9k约94%，9.5k约60%，9.8k约10%）
- 可通过OU过程拟合推导出解析式衰减函数，比硬编码截止时间更稳健（硬编码会在末尾被反向行情打爆）
- 当算法PnL波动较大时，可考虑设置盈利上限（如达到300k后停止D1交易）以锁定收益

---

## 2026-04-28 11:55

- #algo-trading: 在VEL期权上，单纯做市比gamma scalping收益更高；可通过IV计算gamma并测量已实现波动率来估算gamma scalping的理论最大收益（不含spread成本）。
- #algo-trading: 网站上Round的PnL基准约为25k，且无需对波动率smile建模即可达到该水平。

---

## 2026-04-28 12:00

- 期权策略：moneyness与spread存在明显关系，每个期权的最优阈值取决于VEL（波动率）偏离均值程度、期权平均spread和delta
- 通过解析公式推导的参数表现接近对各期权单独过拟合超参数的效果

---

## 2026-04-28 12:01

- 针对Volcanic Rock (VEL)的策略：使用硬编码的mean anchor，配合innovation和theta构成3参数模型来确定进出场阈值
- 建议加入fallback机制以防主策略失效

---

## 2026-04-28 12:26

- krokson的R5分析：最佳alpha是同组产品族内部的相对价值（cluster-local relative value），组内平均收益相关性约0.062，跨组仅0.006，强约一个数量级
- 未发现稳定的lead-lag关系，有用的关系基本是同步的，因此edge来自spread均值回归而非预测
- 最强的walk-forward spread对：MICROCHIP_RECTANGLE vs MICROCHIP_SQUARE（日2→3和3→4平均Sharpe约1.73，累计PnL约3888）
- 次优稳定spread对：SNACKPACK_RASPBERRY vs SNACKPACK_VANILLA、GALAXY_SOUNDS_SOLAR_FLAMES vs SOLAR_WINDS、SLEEP_POD_LAMB_WOOL vs NYLON
- 验证集与测试集表现差异极大（12k vs -10k PnL），提示存在过拟合风险
- 有选手声称解码了所有bot的触发交易过程，认为Mark 67的盈利源于偶然方向正确而非信息优势

---

## 2026-04-28 12:30

- #general: Mark67被部分选手认为只会买入，但实际上可以通过触发条件让其卖出，存在双向套利可能；其触发逻辑为"N个tick内最低价加验证条件"，相对于真实过程来说是次优的，可被利用
- #general: R4/R5中可通过解码识别哪些counterparty是informed的，部分Mark标签的对手并非真正informed
- #manual-trading: 有选手反映R4 manual题相同提交得到不同PnL，乘数计算方式不透明，存在疑问

---

## 2026-04-28 12:41

- #general: 思路是把IMC的题目当作"有优雅解"的问题来处理，并思考IMC会如何生成数据，从而识别数据中的规律。
- #manual-trading: 有选手观察到底层资产15天从50涨到232.21，年化波动率251%，认为这种走势在100次试验下作为平均PnL不太合理；另有人指出最+EV的策略是考虑mods会capping参数（否则会破坏比赛）。

---

## 2026-04-28 13:22

- #general: 关于bot "67"的挂单机制观察——大量挂单在mid附近不会被吃，只有小量挂单才会被吃；吃单概率与给出的edge成正比，且成交量也与edge成正比；67的锚点不是基于true mean

---

## 2026-04-28 14:05

- Pebbles篮子套利分析：篮子组合总和均值约49999.94，标准差仅2.7985，极低标准差暗示存在硬编码套利机会
- Microchips协整性检验：ADF统计量-2.7697，p值0.0627（接近但未通过5%显著性）
- 当前回合策略组合：统计套利 + 做市 + 均值回归，可达到2.3M+回测分数
- 手动交易回合支持做空：先以高价卖出再以低价买回获利
- 有选手认为本回合新闻文章与现实事件（如2025美国政府EV税收政策对应Pyroflex Cells）存在对应关系

---

## 2026-04-28 14:45

- snackpacks（picnic basket相关产品）家族存在协整关系，可用于配对/篮子套利策略

---

## 2026-04-28 14:50

- Manual round 和 algo round 的资产相互独立，没有关联（官方确认）

---

## 2026-04-28 15:28

#manual-trading
- 手动交易费用公式：fee = 100x²（x为分配百分比）
- 价格回报受预定义区间[x,y]和参与者提交方向共同影响：若多数人买入，最终回报靠近y端
- 讨论假设：新闻是否已被price in 是关键判断点
- 影响者(如自封财务大师)的买入推荐通常表现不佳，可作为反向信号参考

#algo-trading
- 有选手提到SnackPack/类似产品的生成器似乎使用耦合冲击(coupled shocks)同时抽取，价格变动存在相关性可推断
- 部分选手用神经网络(NN)交易，输入输出shuffle后比直接对调更稳定

---

## 2026-04-28 15:30

- #manual-trading: 新闻题材中部分故事的情绪已被过去价格部分消化（priced in），不同故事对应不同的时间框架，需注意区分
- #manual-trading: 其他参赛者的出价会影响自己的PnL，但影响较小，主要作为预测错误时的修正因子

NO_INSIGHTS对algo-trading频道（无实质策略信息，仅闲聊和调侃）

---

## 2026-04-28 15:33

- manual交易费用公式：fee = (volume/100)² × budget，其中volume为分配百分比（如20%即代入20），等价于100·x²（x为百分比小数形式）

---

## 2026-04-28 15:43

- Manual trading第5轮要求对冲：Wiki从一开始就说明需要对冲，未对冲会受到惩罚（最优EV解决方案收益接近0或负值）
- 题目中的range指的是股价变化范围（在可交易区间内）

---

## 2026-04-28 16:08

- #algo-trading: pebbles类商品的等权重组合表现出明显的均值回归特性，但单个产品看起来随机或趋势化，且5个产品间的相关性日间不稳定，难以单独交易底层产品（可考虑组合层面的均值回归策略）。

---

## 2026-04-28 16:38

- R4 manual round争议：选手认为IMC使用预定seed而非真正布朗运动导致结果方差极大，#1可能是17k EV/1M std的"垃圾组合"靠运气取胜；建议作废该轮或在规则中加入PnL上限（如±300K）
- 简单策略思路：用日终价值作为fair value，低于买高于卖，回测表现良好

---

## 2026-04-28 16:54

=== #manual-trading ===
- R5手动题分析：AC对冲在该轮并非免费风险降低。买卖价49.975/50.025下，每手AC期望亏损约75（穿越价差），200手对冲直接损失约15,000期望PnL
- AC仅能对冲一阶spot delta风险，无法对冲期权组合中的主要风险来源：chooser凸性、二元期权跳跃风险、敲出障碍/路径风险、短期gamma
- AC仓位上限仅200，相对期权账本太小太粗糙，协方差项不足以抵消新增方差，模拟显示对冲既降低均值PnL又增加标准差
- 对冲策略评估应基于：扣除交易成本和仓位限制后是否真正改善目标函数，而非默认"对冲总是更好"
- R5玩家定位对价格的影响机制存疑：是否线性、是否在[x,y]边界附近有阈值/封顶效应、[x,y]宽度是否跨产品一致、是否独立于新闻信号强度

---

## 2026-04-28 17:39

- Manual trading round分析：做多gamma只在RV>IV时有利；由于RV已知，关键在于最大化Sharpe比率而非variance；100次模拟样本量足够大，运气成分有限
- 不建议通过预测其他队伍行为来制定manual trading策略

---

## 2026-04-28 18:04

- #manual-trading: 关于最后一轮manual的关键分析：因题目说明价格是100次模拟的平均值，平均后波动率大幅降低（分布近似N(50, ~3)），全仓gamble组合的标准差约为3.7M，相当于押注±0.1 sigma事件；许多抱怨者错误地用单次GBM而非100次GBM平均来做蒙特卡洛模拟。
- #manual-trading: 由于波动被平均压缩，理性策略可能反而是被迫满仓gamble，否则会因±0.1 sigma的小波动而落败。

---

## 2026-04-28 18:50

- #manual-trading: 玩家投资量会影响标的价格走势（官方在频道中确认），他们设定了一个价格区间，会根据玩家成交量在区间内偏向某一侧
- #manual-trading: 有人尝试用不同QR扫描器重建二维码寻找隐藏线索，但像素不足，可能是死胡同

---

## 2026-04-28 18:55

- #algo-trading: 有选手提示basket（篮子）存在均值回归特性，可作为交易信号思路
- #manual-trading: 关于manual题目的策略讨论，多数选手认为不需过多考虑玩家心理博弈，部分选手尝试用NLP/情感分析将其量化处理

---

## 2026-04-28 19:00

- basket之间的spread也存在均值回归特性，可考虑跨basket套利
- Basket 50k策略表现尚可，但并非决定性的alpha来源

</result>

---

## 2026-04-28 20:00

- algo-trading: 比赛回测使用的是capsule中提供的最新一天的前10%作为测试数据
- manual-trading: 本轮manual交易机制——使用的预算会从PnL中扣除，未使用的预算不会加回利润而是直接作废，因此若投资回报低于预算支出PnL会变为负数，存在较大风险

---

## 2026-04-28 20:40

- #manual-trading: Volcanic（火山香）的最终价格高度依赖玩家下注分布，因此被认为是manual trading中PnL潜力最大的产品

---

## 2026-04-28 21:15

- 上一轮真实回合结构与回测非常相似，若策略表现差距大可能是过拟合而非信号不强
- IMC官网的回测数据基于Day 4
- 有选手在3天回测中Macarons（Galaxy）单品获得87k收益作为参考基准

---

## 2026-04-28 22:10

- Rust回测与网站结果差异大可能是过拟合；网站只显示Day 4的10%数据，需注意
- 关于"追踪元素指数的基金"是购买实物reactors还是Sulfur Ltd股权的疑问（现实市场通常是股权）

---

## 2026-04-28 23:10

- 跨3天回测时，每个产品组PnL最低应达到约100k，总PnL可达800k+
- Magnificent Macarons (microchips相关产品)难度较高，方差大，但可做到约120k PnL
- 部分玩家遇到traderState大小和文件大小限制问题
- 手动交易题目中"forever feathers"和"eternal feathers"可能是同一公司（疑似拼写问题）

---

## 2026-04-28 23:20

- Microchips/panels信号存在但不明显，有选手通过通用代码（无硬编码/参数调优）在回测中获得100万+ PnL
- 部分选手在chip交易上day 2出现负收益，策略稳定性是挑战
- Rust backtester和Kevin backtester都被认为好用
- 本轮可能存在隐藏的easter egg彩蛋，难度较高

---

## 2026-04-28 23:45

- #algo-trading: Pebbles被认为主要是套利策略
- #algo-trading: Panel和Chips被普遍认为是较难做的产品；有选手在Panel上3天获得40k-95k PnL，Chips约130k
- #algo-trading: 提醒——若策略依赖market taking，网站PnL与CSV回测结果可能差异较大，需警惕参数过拟合

---

## 2026-04-29 01:10

- 有选手分享了Rust回测器：https://github.com/GeyzsoN/prosperity_rust_backtester/releases/tag/v0.4.3
- 有选手猜测本轮可能是配对交易类型的alpha题目

---

## 2026-04-29 01:16

- Manual trading PnL公式：assets - budget - fees = PnL（fees需计入）

---

## 2026-04-29 01:35

- manual-trading: 本轮manual基于情绪数据的回报区间是预先定义的(x,y)，但其他参赛者的选择会影响最终落点，更看好的情绪会将结果推向上限y

---

## 2026-04-29 02:55

- 网站模拟器是确定性的，据称已移除随机化的交易量

</result>

</invoke>

---

## 2026-04-29 03:57

- #manual-trading: 手动交易的资金分配策略讨论——简单按比例缩放到100%可能不奏效；投入更多资金会降低初始投资部分的收益率，因此存在一个平衡点（即分配比例与边际收益之间需要权衡优化）

---

## 2026-04-29 04:18

- #algo-trading: 有人提出观察到时间序列具有强漂移性，建议用regime detector识别趋势状态后做"买入并持有直到regime切换"的策略（反之亦然）。

---

## 2026-04-29 04:33

- Manual puzzle 4提示：对于不想交易的商品，需要将value设为0并任选一边（side），可以尝试这种方式

---

## 2026-04-29 06:05

- #algo-trading: 有选手认为协整配对交易（cointegration pair trading）效果可能更好
- #algo-trading: 关于赛末未平仓头寸如何计入P&L的疑问（未得到解答）

---

## 2026-04-29 06:24

- #algo-trading: 关于回测器与网站的选择建议——若策略依赖于操纵bot成交则用网站，若只关注价格走势则用backtester
- #algo-trading: 网站显示的PnL只是1/10 tick，不可全信，应以backtester为准
- #algo-trading: Pebble产品3天100k timestamps的backtester PnL参考值约为40k-80k
- #manual-trading: 该轮manual挑战基于新闻事件，但缺乏上下文（当局反应、bot反应、参赛者反应、判断依据），不确定性本身可能就是挑战核心；举例说明市场反应未必与表面新闻直接相关（如C罗可乐事件，股票在事件前已下跌且当日已回升）

---

## 2026-04-29 06:29

- #manual-trading: 官方说明manual交易评分机制——每个产品有预设的回报区间和锚点，参赛者的提交会在该区间内移动实际回报（类似群体投票影响价格），需要批判性阅读新闻判断利好/利空/已price in，最终有"群体智慧"修正元素，但锚点与参赛者权重未公开。

---

## 2026-04-29 06:34

- #manual-trading: 本轮manual题本质是猜测产品收盘价相对开盘candle是涨是跌，下注金额需考虑手续费呈指数级增长，因此置信度决定仓位大小；评分机制更像是对锚定偏差的修正，博弈论成分不如R2/R4 manual那么核心。

---

## 2026-04-29 06:44

- #manual-trading: 官方说明manual round评分机制——每个产品有预设的回报区间和锚点，参赛者的集体提交会在区间内移动实际回报（多数人看多则价格涨得更多），需批判性阅读新闻判断利好/利空/已被定价

NO_INSIGHTS适用于algo-trading部分（仅闲聊PnL和过拟合担忧，无实质策略信息）

---

## 2026-04-29 06:54

- manual-trading: 有人提出manual题PnL公式为 PnL = budget × v × (r − v)，其中r为回报率，v为交易量（来源为Claude，未经证实，需自行验证）

NO_INSIGHTS对algo频道（关于pebbles的讨论无实质内容）

---

## 2026-04-29 07:04

- 有选手使用神经网络（NN）策略，将权重硬编码到脚本中，单日PnL可达约140万，整体回测3天达510万
- NN在本届比赛被认为是热门meta策略
- Round 4 manual普遍亏损，正PnL较少（有人2k）

---

## 2026-04-29 07:10

- #algo-trading: 最终评测基于完整的第4天数据，而网站显示的成绩仅基于该天初始一小段，因此可以考虑不在第4天数据上训练，避免污染
- #algo-trading: 有选手建议训练神经网络时仅使用Day 2、3数据
- #algo-trading: 有团队在Quantum/此轮中尝试用VAE（变分自编码器）建模
- #algo-trading: 部分选手认为NN不会过拟合，网站表现好的队伍在未见数据上通常也表现好（争议性观点，仅供参考）

---

## 2026-04-29 07:20

- 关于神经网络（NN）策略的讨论：有人认为NN性能与Ridge回归相近，不值得投入；但也有人称排名35及更高的选手在用NN
- 过拟合策略思路：极端过拟合训练数据获得250k PnL后，让代码在10万tick后停止交易并清仓——但被指出仅会过拟合训练日，实盘无效
- 有人尝试创建合成数据来扩充训练集
- Claude tokens建议用于stat arb和做市策略开发，而非NN

---

## 2026-04-29 07:35

- #algo-trading: 有团队用动态规划(DP)硬编码最优策略——通过提交一次拿到日志，计算每个时间戳的最佳交易后再次提交
- #algo-trading: NN训练思路：用GBM估计漂移和波动率，生成数千天的合成数据来训练神经网络
- #algo-trading: 有选手发现oxygen_shake_garlic在可视化网站的价格走势与下载的实际数据不一致
- #manual-trading: Ashflow Alpha可视化在4月29日12:00 CEST有小幅更新（具体变动未明）

---

## 2026-04-29 08:00

- #algo-trading: 训练神经网络数据不足时，可将各timestamp的数据打乱顺序，生成多条时间序列作为合成数据供NN学习
- #manual-trading: 新闻文章发布于交易日之间，可假设市场尚未对该新闻定价（除少数已提前部分定价的情况）；新闻文章可能会被更新，需要留意

---

## 2026-04-29 08:10

- 多数产品具有均值回归倾向（可能是为了制造噪声而编码的），可结合趋势处理做MR策略
- 市场微观结构观察：激进买家吃掉被动卖家
- Manual round费用机制说明：投资费用从PnL中扣除，未使用的预算不会返还（视为需归还的无息贷款），即使不投资也会因费用产生负PnL影响总分

---

## 2026-04-29 09:20

- 有选手反馈：在portfolio benchmark上，pebbles策略可获约2.5（0回撤），snackpack约3.3（一致性较差）
- robot_dishes可在day 4数据上过拟合到400k收益，但回合末数据不一定有效
- 有观点认为robots类资产不适合做alpha/均值回归博弈，信噪比差
- 部分选手只在pebbles和snackpack上找到了篮子级别可利用的pattern，其他资产收益平庸

---

## 2026-04-29 11:00

- #algo-trading: 有选手反馈本轮挑战较难，曾因比赛结束时间早于期权到期日，通过做空所有深度实值（deep ITM）期权使排名从700升到150
- #algo-trading: lead-lag策略普遍较难做，多人反映挣扎
- #manual-trading: 有选手怀疑manual题目中的图片是PDF417条形码，但用扫描器无法解读，可能图片过窄或不完整；尝试加quiet zone无效，建议根据PDF417标准结构反向工程

---

## 2026-04-29 11:55

- #algo-trading: 有选手用神经网络算法在portal上获得120k PnL，方法是训练NN后将权重硬编码到策略中
- #manual-trading: 手动交易中费用是二次方（quadratic）结构

---

## 2026-04-29 14:37

- pepper root（辣椒根）使用均值回归策略效果良好
- 提到emeralds和tomatoes之间可能存在叉积（cross product）信号（真实性存疑，可能是玩笑）

---

## 2026-04-29 14:47

- 有人提到关注trader "mark67"可能能找到alpha信号（来自traders数据）
- 有玩家声称仅在偶数timestamp交易回测可达200K（疑似玩笑/反讽，不可信）
- 普遍认为本轮（round 5）正PnL难度较高，奖金池估计在500K左右，而非1.5M

---

## 2026-04-29 15:08

- 配对交易（pair trading）在回测中最高约65万，效果有限，被认为是"加强版赌博"
- 上一轮期权挂钩的标的被脚本限制在一个区间内，有较大的硬编码套利空间
- 警示：过度硬编码风险大，IMC可能在后续轮次反向操纵图表
- 单纯的均值回归/配对交易不够，需要更深入的策略

---

## 2026-04-29 15:33

- algo-trading: 本轮策略核心是配对交易（pairs trading）优化，提示中已暗示寻找好的配对组合并设计交易策略

---

## 2026-04-29 15:38

- #manual-trading: 手动交易费用计算公式确认为 100 * x * x（x为百分比数值，如25%分配则手续费=100*25*25=62500）

---

## 2026-04-29 15:58

- #algo-trading: snack pack与van（vanilla）和choc（chocolate）被识别为配对交易组合，建议寻找更多配对
- #algo-trading: 网站提交与完整迭代回测的分数差异巨大（如46k vs 460k），提示回测器结果仅供参考，注意过拟合风险
- #algo-trading: position limit为10（针对某个product）

NO_INSIGHTS（manual-trading部分无实质信息）

---

## 2026-04-29 16:48

- 回测PnL与官方网站PnL差异大通常源于回测撮合更宽松（成交了实际不会成交的订单），需警惕过拟合
- IMC官方回测使用约10万个时间步（3天数据）

NO_INSIGHTS（manual-trading频道无实质信息）

---

## 2026-04-29 17:34

- Round 5被认为是更偏"alpha quant research"的一轮（寻找隐藏信号），而非执行层面的优化
- 有人提到spread是价格的函数，这一机制消耗了不少调试时间
- Manual trading中关于thermalite，有人选择卖出（dump）

---

## 2026-04-29 19:00

- #manual-trading: 选手讨论本轮manual trading的方向判断，普遍认为虽然故事描述听起来bearish（且Claude/Gemini等AI也判断为bearish），但实际是bullish的供应冲击；参考去年走势预计最大驱动因素可能移动50-60%，多数人计划下注20-25%偏多
- #manual-trading: 提示注意去年manual round中价格波动幅度很大，AI判断方向可能误导，需独立思考
- #algo-trading: 有选手提到Round 5的算法收益约在20mil级别（后承认夸大，建议查leaderboard核实）
- #algo-trading: 提到Round 3是容易翻车的关键回合

---

## 2026-04-29 19:05

- #manual-trading: 新闻轮中分析事件影响时需注意持有期 —— 我们只持有1天，而指数纳入/重组等利好往往需要数月才兑现，因此即便基本面强烈看涨，单日价格未必能跳涨30%，措辞细节很关键
- #manual-trading: 硫(sulfur)被部分玩家看作强烈看涨（基于指数纳入会迫使基金再平衡买入的逻辑），但多数玩家预期涨幅仅约+2%，存在预期分歧
- #manual-trading: 评估新闻类题目时，关键不是真实世界结果，而是参与者的共识预期如何驱动价格
- #manual-trading: 玩家普遍认为指数题和税收题表述较难理解，参与者信心较低

---

## 2026-04-29 19:20

- #algo-trading: 有传言称IMC网站当前展示的回测使用的是Day 4数据，意味着现在的测试可能存在样本内拟合（in-sample fit）情况，但最终提交时不会用此数据
- #algo-trading: 推荐论文 "Virtue of Complexity in Return Prediction" by B. Kelly，研究使用大量参数提升OOS预测效果，可参考用于NN方法
- #manual-trading: Magma Ink在手动交易中价格基本已被市场定价合理（priced in），套利空间有限

---

## 2026-04-29 19:30

- #algo-trading: 建议将均值回归类策略建模为OU过程而非简单使用Z-score，可降低交易成本损耗
- #manual-trading: 去年manual交易的费用结构为二次型(quadratic)
- #manual-trading: Thermalite的定位类似去年的VR(可能指无需特别配置/已被price in的品类)

---

## 2026-04-29 20:05

- PEBBLES家族策略思路：计算五个PEBBLES中间价之和减去50000；和过低时按ask买入XL，过高时按bid卖出XS（XS吸收失衡）；接近平衡时只交易XL（用XL对其他四个的均值作为一维因子，配合momentum规则在bid+1/ask−1下单并设上限）。利用家族篮子近似恒定+XS/XL分别清理失衡+XL为高beta腿。
- 管理员警告：若多支队伍合谋作弊将全部DQ，且会通报HR/招聘，影响录用。
- 有选手提到无作弊情况下live PnL大约75k–168k区间，1.5M被怀疑为overfitting/异常。

---

## 2026-04-29 20:20

- #manual-trading: 本轮manual交易150k被认为是可达成的好成绩（取决于资产回报）
- #algo-trading: 官方在R3/R4对疑似不公平合作的队伍发出警告，重复违规将被DQ；此前案例未带来实质PnL优势

</response>

---

## 2026-04-29 21:00

- 有用户提示：如果PnL为负，反转策略方向可能直接变正（针对pebble等品类）
- 有选手反映panels在回测器表现良好但实盘上线后大幅下滑，存在过拟合风险

---

## 2026-04-29 21:05

- 策略方向讨论：有人提到策略可能是均值回归（mean reversion），也有人调侃"买入并持有 PURIFICATION XL，卖出其他"
- 神经网络（NN）不是有效alpha
- 回测器与官网结果不一致：使用 nabayansaha/imc-prosperity-4-backtester 时，回测3天盈利12万，但官网提交后表现下降
- 一位选手分享回测分日数据：day-2约43.5万，day-3约41.8万，day-4约42.1万，三天合计约127万
- 警示：过高的Sharpe比率可能意味着过拟合

---

## 2026-04-29 21:10

- 第5轮(round 5)前150万左右利润主要来自pairs trading策略，可通过绘制各pair价差图寻找机会
- 均值回归价格比率(ratio mean reversion)是pairs trading的可行思路
- 一个backtest参考PnL基准：day2约339k、day3约393k、day4约427k，总计约119万
- Manual trading的PnL计算可能存在手续费/汇率损耗，买卖价差直接相减与实际PnL会有偏差

---

## 2026-04-29 21:20

- #manual-trading: R3 manual题低于平均值会被惩罚，本年平均值约859.06（mean=859），单点最优解为751和836，但群体均值偏高导致单点优化者被罚分
- #manual-trading: 罚分公式近似为 (371680-345420)*((920-859.06)/(920-855))^3，可用于估算结果

NO_INSIGHTS对algo-trading频道（仅闲聊无实质信息）

---

## 2026-04-29 21:30

- 在线网站PNL参考：选手讨论中提到合理范围约为150k-180k，最高单轮约500k（超过会大幅影响前几轮结果）

</result>

NO_INSIGHTS

（注：上述PNL数据点信息价值有限，仅作为大致参考。如严格按"有价值交易信息"判断，此段消息基本只是闲聊和PNL炫耀，无具体策略、产品规律或机制分析。）

NO_INSIGHTS

---

## 2026-04-29 21:55

- 多位选手暗示本轮（round 5）核心策略是均值回归（mean reversion），官方提示可能是误导
- 有选手提到非过拟合的网页回测PnL上限约3k，10k可能已过拟合
- 提及IMC回测数据仍可能存在20%随机化的问题（未确认）

---

## 2026-04-29 22:00

- #algo-trading: 选手观察到目前为止除pepper外，几乎所有product都呈现均值回归特性
- #manual-trading: 关于cutlery的多空讨论——供应减少看涨，但污染原因可能也带来看跌解读

NO_INSIGHTS之外仅这些有限信息。

---

## 2026-04-29 22:10

- #manual-trading: 关于manual trading新闻事件分析的讨论：好产品的供应损失通常推高价格；若是因污染等问题需要停产则偏空；去年类似题目（被需求的产品+下月开始生产）实际价格大幅下跌，提示新闻表面利好可能反向。

---

## 2026-04-29 22:22

- 提示关注过拟合问题，分享了相关学术论文（David H. Bailey关于回测概率的论文）
- 策略建议：应该交易能找到可泛化alpha的产品，通常按产品类别（而非单个产品）来研究
- Manual交易观点：cutlery 看涨，因为供应至少边际减少是确定的，而需求方向不明

---

## 2026-04-29 22:40

- Manual trading讨论：选手倾向卖出Lava Cake和Ashes，买入Sulfur和Thermalite，部分人也买Cutlery；Sulfur反应被认为可能超出1天时间框架但仍正向

---

## 2026-04-29 22:42

- #algo-trading: 50个产品被分为10组，每组5个，组内产品有相似性可整体分析；但同组内策略未必通用，应只交易能找到稳健alpha的产品，而非强行覆盖全部50个
- #manual-trading: 选手猜测manual交易选品本身无关紧要，关键在于背后的理由逻辑；个别选手押注cutlery上涨、lava cakes大幅下跌

---

## 2026-04-29 22:50

- manual-trading: 有人认为incense（香）的锚定价格设为0，对他人交易行为高度敏感，风险大，选择不交易以避免手续费
- manual-trading: 讨论paste的长期价值可能也接近0，但相比之下paste有实际用途，可能价值略高
- manual-trading: 共识是没有明确信号时不要交易冷门商品，以避免手续费损耗
- algo-trading: 对于看似奇怪的产品组合统计套利信号，建议视为噪声，不要浪费时间

---

## 2026-04-29 22:53

=== #manual-trading ===
- 关于manual trading产品情绪分析：incense被认为已经过度上涨，可能是pump and dump，存在崩盘风险但也有继续挤空可能
- 结果是部分确定性、部分依赖参赛者答案的混合机制
- 多数人认为前4个产品信号最强

=== #algo-trading ===
- 建议专注于wiki中已分组的50个产品，不要在冷门思路上浪费时间

---

## 2026-04-29 22:58

- #manual-trading: 计划中的指数买入理论上无法被完全price in，因为指数会以市价买入，无视价格高低
- #manual-trading: 参考过往比赛，产品价格变动幅度可能超过50%

NO_INSIGHTS其余为闲聊和回测器使用差异讨论。

---

## 2026-04-29 23:00

- Prosperity 4回测器fork可用：https://github.com/nabayansaha/imc-prosperity-4-backtester
- 手动交易讨论：历史上某资产曾涨45-60%，最后一轮可考虑加大风险
- 手动交易费用结构使得50/50产品的EV为负，需谨慎下注
- 参考资料：Frankfurt Hedgehogs的README含历史数据信息

---

## 2026-04-29 23:03

- #algo-trading: 有选手提示本轮50个资产应按10个分组来分析，而不是逐一或两两比较
- #manual-trading: 去年第5轮manual交易实际涨跌幅总和接近200%，可参考TimoDiehm的GitHub repo (imc-prosperity-3) 历史数据

---

## 2026-04-29 23:08

- manual-trading: 今年manual新闻信号强弱较为均匀分布，不像往年有明显的强弱信号区分，预期收益可能不会出现极端分化

</result>

---

## 2026-04-29 23:23

- 有选手指出：若策略基于 mid_price 减去 rolling_mean 做均值回归，在测试集中也能保持高PnL表现（mean reversion 围绕滚动均值的思路有效）

---

## 2026-04-29 23:45

- Round 5主要靠从数据中找出"泄露"信息（leaks），这是决定排名的关键回合，即使排名1000的选手也可能在R5冲到前列

---

## 2026-04-29 23:55

- manual-trading: 集体共识选项反而会被过度选择，导致相对收益下降，应避免跟随主流选择

---

## 2026-04-30 00:03

- Manual交易机制：官方设定一个"锚点"作为基础价格变动，玩家的提交行为会在一定范围内、以一定敏感度影响最终回报；多数人强烈看多会让价格在区间内上移更多

---

## 2026-04-30 00:54

- Round 3天最高约4.8-4.9m被认为是不过拟合的理论上限，DP（动态规划）最大值约为24m

---

（manual-trading频道无有价值信息）

---

## 2026-04-30 01:09

- #algo-trading: 有人提示 GALAXY_SOUNDS_BLACK_HOLES 与 SLEEP_POD_POLYESTER 之间存在 leader-lag 关系，建议 lookback 设为 100（信息真实性存疑，发帖者自称排名垫底）

---

## 2026-04-30 01:29

- 比赛榜单显示的是第4天的1/10数据（不是第5天），这是回测器与IMC日志结果不同的原因之一
- 正弦曲线价格预测在某些产品上有效

NO_INSIGHTS对于algo-trading频道

---

## 2026-04-30 02:40

- 回测器资源：kevin-fu的IMC Prosperity 4可视化器和回测器（github.com/kevin-fu1/imc-prosperity-4-backtester）
- 策略观察：lead-lag策略对部分资产有效；部分资产适合mean reversion
- 重要提醒：回测PnL与IMC官方平台模拟结果差异显著，回测表现好的策略在官方模拟可能表现差，反之亦然（多位选手报告：回测50万-70万，官方portal只有3万-8万）

NO_INSIGHTS之外的实质信息有限，大部分为闲聊。

---

## 2026-04-30 02:50

- Manual trading最后一题策略讨论：在缺乏信息的情况下提交>90%置信度很难，即使方向对但波动率判断错也会输；有人认为低风险策略可能更优，因为很多人会激进押注
- 观察：上轮组织者似乎刻意压制极端事件/赌博性押注的回报，因此预期不会有"剧烈方向性变化"奖励，但负向极端没人关注
- 最终排行榜不会当天公布，需要约2周时间核查作弊和alt账号

NO_INSIGHTS对其他频道（闲聊为主）

---

## 2026-04-30 03:25

- #algo-trading: 可通过在 Trader.run() 内用 time.time() 包裹打印耗时，监控代码执行速度；run block 每个 tick 调用一次，越快越好，可硬编码不依赖历史数据的部分以加速
- #algo-trading: 有选手提到通过修复仓位未正确建立的小bug，PnL 从 80k 提升到 600k+，提示检查 position 逻辑细节
- #algo-trading: 有人提出关注 bots 同时买卖资产的现象作为潜在信号，但尚未公开有效用法
- #manual-trading: 关于 obsidian 多空争论 —— 工厂停产利好供给端涨价，但污染议题与替代品可能压制需求；多数人偏看涨
- #manual-trading: 有观点认为大量买入 sulphur 缺乏依据，提示其上涨空间有限

---

## 2026-04-30 03:45

=================
#manual-trading
- Manual交易费用机制：fee = budget × scale，gross PnL = total profit - fee（fee已包含投资额），投资越多扣费越多，但投资过少利润有限，存在权衡
- 进入市场时点为"明天"（交易日之间）

#algo-trading 无有价值信息（NO_INSIGHTS）

---

## 2026-04-30 04:00

- 最终提交日（次日10k行数据）的模式与历史3天测试数据完全相同（来自Tomas的官方说明）
- numba库不被允许使用
- 关于硬编码历史数据：被建议改为用历史数据进行warmstart信号，而非硬编码
- Manual交易中Obsidian Cutlery被解读为买入信号，理由是问题不在产品本身而是供应受限

---

## 2026-04-30 04:55

#algo-trading
- 有选手发现3个basket产品之间存在lead-lag关系，实施后增加约14万利润
- 参与者交易记录(trades)中也存在lead-lag信号可利用
- 解决配对策略冲突可通过分配优先级来处理
- Round 5单日backtest约80万利润仍非顶级水平，顶队单日backtest可能在120万左右

#manual-trading
- 本轮manual题目可能存在类似Round 3 prop的"红旗"陷阱信号，需谨慎判断；有选手倾向小幅买入(4)

---

## 2026-04-30 05:00

- #algo-trading: 有选手观察到pebbles中间价之和基本固定在50k附近且呈均值回归特性，可作为交易信号
- #manual-trading: 关于新闻交易时点的判断——若为收盘后新闻，次日（减税生效时）很可能已被price in；若为盘前新闻，开盘买入仍可能受益
- #manual-trading: 有选手提到Thermalite仅作为电池用途的信息分类判断

---

## 2026-04-30 05:10

#algo-trading
- 有选手分享Round 5策略：long sleeppods和garlic，pair trade snackpacks，其余产品中性仓位做市（基于trader behavior）
- 有人提到：snackpack分spread A和spread B交易；raspberry-pistachio配对；vanilla用CHoCH（结构突破）信号
- Round 2 alpha：所有交易发生在每天的同一时间点（时间规律）

#manual-trading
- Round 5新闻题：obsidian cutlery（黑曜石刀具）观点分歧——有人认为类似"red flags+cacti needle"混合，因刀过于锋利造成事故和工厂损毁，可能不利好；保守策略0%
- volcanic incense被认为是rugpull陷阱，最佳出价0%（不知何时dump且只交易一天）
- Magma Ink、Ashes 也建议出价0%

---

## 2026-04-30 05:11

- 比赛机制：第1、2轮为Phase 1，需达到200k XIRECs才能晋级Phase 2（包含第3、4、5轮）；Phase 1结束后排行榜重置，3-5轮难度显著高于1-2轮，决定最终冠军；Phase 1和Phase 2可交易的产品不同
- 提交规则：未提交脚本者不计分；提交但未调用bid()则计为0
- 手动交易：去年Round 4曾将3个产品的真实价值设为0；本轮预期最大收益约125-150k

---

## 2026-04-30 05:15

### #algo-trading
- 官方确认Round 5无新regime变化，只是"调味"性质，不会有大变动
- Round 5 Manual中Sulfur Reactor的文本已更新：Elemental Index 118将在下一次rebalance加入Sulfur Reactor，跟踪该指数的基金会调整持仓
- Round 3和4的数据多由OU过程生成，可用阈值型均值回归（类似Bollinger Bands）交易；参考论文 arxiv.org/pdf/1504.04682
- Options在本届实际无价值，Implied Volatility无有效信号；直接对价格做MR是更优选择
- Round 2中针对roots和osmium的市场操纵思路：吃掉一边订单簿后挂高/低1%重新挂单，需逆向工程bot触发条件
- 本轮每笔交易（除2个family外）size和timestamp相同，可基于此设计策略
- 服务器迭代次数约为本地回测的1/10，回测PnL与实盘比例需相应估算
- Top选手策略示例：五引擎ensemble，把50个产品分配到子trader（T040/T100/T399/TMSG6），主要alpha为basket spread均值回归（组内OU mid - peer_avg）和OU自身mid回归；TMSG6含translator事件驱动配对；CF_PAIR协整（CHOC/VAN β=-0.974）；FORCED_BANDS对已知方向偏置产品做硬限位；v504加入probe挖出的bot exploit（被动BVFV/MID挂单在spread内）
- 具体配对/leader-lag参数示例：PEBBLES_M+PEBBLES_XS→PEBBLES_XL (lb50, pol-1, th418.25)、MICROCHIP三角/方形残差breakout (β=-0.742, th326.2)、UV_VISOR_ORANGE→GALAXY_SOUNDS_DARK_MATTER (lag75)、PANEL_1X4→1X2(lag1)、ROBOT_VACUUMING→ROBOT_DISHES (lag750) 等众多leader-lag与pair residual breakout信号

### #manual-trading
- Round 5 Manual策略：Lava Cake重押65%基准，分配57%预算覆盖范围，最多可用至61%
- Thermite cores被认为是最大陷阱，预计上涨约15%
- 关键差异：本轮文本说的是"forecast"而非"quarterly report"，需注意

---

## 2026-04-30 05:16

- #algo-trading: 有选手用OU过程+布林带做均值回归策略，并怀疑Round 4的残差也遵循OU过程
- #algo-trading: 选手反映难以从bot订单流中找到信号，最终改为通过被动挂单从taker获得更好成交
- #algo-trading: 提到一篇相关SSRN论文（abstract_id=5713082），可作均值回归参考

---

## 2026-04-30 05:20

#algo-trading
- Day 3 数据在几乎所有交易对上都是非平稳的，跨家族拟合配对大概率失败
- 存在一个交易"超大篮子"（覆盖全部50个资产）的篮子交易者，导致约8个家族中的交易高度同步
- 所有 pebble mid 的总和固定为 50,000，因此正常做市所有产品时持仓总价值会 < 50,000
- UV-visor 系列（除 amber 外）相对存在半天的右移，量级不同，可作小额套利
- 离散时间步下用 AR(p) 即可，无需 OU
- 67号交易者只在低成交量时吃单；给的 edge 越大成交概率越高，但直接吃 L1 收益接近
- Round 5 有效策略示例：按产品的 OU 均值回归集成（篮子内残差 mid - peer_avg 与各自 mid）、协整配对叠加、事件驱动的 translator 价差交易、库存带方向性先验，加上在 BVFV 锚定价位内被动报价以从特定对手方获取正 markout
- Round 5 没有期权
- 警告：跨多日不一致的复杂模型多半是过拟合，应专注分离信号而非在噪声中找模式

#manual-trading
- Obsidian Cutlery 类似玩家囤积票券再转售的现象，社区情绪偏看涨
- Round 4 的题目被普遍认为高度随机，Round 5 相对更可分析

---

## 2026-04-30 05:21

- Galaxy snacks中存在两对资产，其底层价格变动相互抵消（可做配对/价差交易）
- 资产间存在lead-lag关系，价格变动有完美时滞，但因量级不同难以通过统计方法发现
- 尝试通过t-2时刻的信息推断bot在t-1的下单逻辑（追踪因果），但未发现有意义的信号
- Round 5手动交易：Obsidian的故事偏负面，需关注区间偏向正面的难易程度

---

## 2026-04-30 05:25

- 配对交易思路：将协整和滞后视为同一概念（lag 0），如果A与B配对，A-B[-t]可能服从OU过程，参数拟合后用约170行代码实现，关键在于找到beta系数
- A-B并非随机游走相减，而是具有均值回归统计特性
- 跨日关系验证警示：分别检查3天数据和每日单独的关系，可能不一致，需谨慎避免过拟合
- 第5轮回测PnL参考：保守策略约400k，普通水平约1.5M，靠前选手可达更高水位
- 第5轮存在约450k的"无风险"alpha机会被部分人捕获
- 手动交易第5轮：Volcanic Incense被多人跳过；Obsidian故事alpha接近零不值得赌；Red Flags被认为是唯一可行的替代选项

---

## 2026-04-30 05:30

#algo-trading
- chrono55的options策略（R3第26→R4第10）：用bootstrapped AR(2)过程估计mean和level的置信区间，再转换到contract空间得到对应区间；若contract空间bands太小则不交易，自动考虑了theta decay对spread收缩的影响
- chrono55策略变体：原版~600k PnL用确定性趋势，换成EMA后更稳健，得到~400k PnL（3天回测）
- chrono55补充：主要寻找I(1)协整配对，估计漂移项，基于VECM bootstrap估计的alpha对篮子成分进行均值回归交易

---

## 2026-04-30 05:31

- #algo-trading: 针对ROBOTS产品，可在每个timestamp拟合多项式 a + b*area + c*width^2 + d*height^2，对偏差进行scalping
- #algo-trading: Day 4 ROBOTS出现大幅离散跳跃，存在约400k无风险PnL机会，可通过观察图表发现

---

## 2026-04-30 05:35

#algo-trading
- 部分资产的收益率在lag 1处呈现轻微负自相关，可考虑取上一次跳动方向的反向作为信号
- Round 5 资产分组结构：1组为传统配对交易；1组sum(prices)=固定值且各为随机游走；1组为lead-lag（相关性几个百分点，原本用户测试时高达80%）；其余多为纯噪声；跨组之间无关系
- 60M edge 来源于 lead-lag 组
- R3/R4 选项题更偏赌博，缺乏真实期权alpha

#manual-trading
- Thermalite 类资产存在追涨情绪，部分人主张满仓多头
- 关于Cutlery（餐具）事件：工厂事故导致营收损失、监管/责任/停工/资本支出风险，且可能波及其他工厂，基本面偏空；唯一反向理由是市场情绪推升

NO_INSIGHTS（其余为闲聊）

---

## 2026-04-30 05:36

- #algo-trading: Microchips存在lead-lag关系，每个分别以50/100/150/200的滞后跟随Circle
- #algo-trading: 50个products是有意设计的，目的是模拟真实场景——需要筛选出真正有edge的可交易标的，多数是噪声
- #algo-trading: R5信号统计显著性较难发现，Exotic options难度较高，整体存在流动性不足的问题
- #algo-trading: R3/R4执行细节对收益影响显著
- #algo-trading: 据Tan Le文档，cross-category products之间均存在leader-follower关系（但相关性可能太弱难以盈利）

---
