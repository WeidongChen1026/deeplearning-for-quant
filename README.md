# deeplearning-for-quant
本项目是基于深度学习在多因子量化选股上的一种实践．
* 第一阶段主要完成了传统多因子选股在中国A股市场上的回测．实现的模型为Fama三因子和carhart四因子，（Fama五因子当时由于数据缺失没来得及做）
* 第二阶段整理影响股票超额收益率的因子（特征），完成了基于CNN的多因子量化选股模型（本阶段选用的因子仅为基于价格计算的因子）
* 第三阶段改进上一阶段的模型结构，将财务因子数据与价格因子数据融合起来，一起对股票超额收益率进行预测．

先放一张模型结构图：

![](/figures/模型结构%203.jpg)

## 实践过程总结

整个项目基于的假设和套利定价理论基于的假设是一样，项目实施的理论依据也是套利定价理论．

> 套利定价理论认为，套利行为是现代有效率市场（即市场均衡价格）形成的一个决定因素。如果市场未达到均衡状态的话，市场上就会存在无风险套利机会，并且用多个因素来解释风险资产收益，并根据无套利原则，得到风险资产均衡收益与多个因素之间存在（近似的）线性关系。

上述最关键的一句就是＂风险资产均衡收益与多个因素之间存在（近似的）线性关系＂，可以想到如果我们想要获得风险资产的超额收益，就得知道影响风险资产超额收益的因素有哪些，这样就能通过最简单的ML模型Linear Regression实现对风险资产超额收益的预测．但是现实情况是我们无法知道影响风险资产超额收益的所有因素，这也是股票预测为什么不准的主要原因．虽然我们无法获得所有的影响因素，但是如果能充分挖掘已知因素的影响能力，在很大概率上还是能战胜市场的．

目前整个项目用到的影响股票超额收益率的数据仅有价格数据和财务数据，当然现在已知的还有上市公司公告，新闻，分析师建议预测等文本信息，由于能力有限当前阶段没有将文本数据融合进模型，但自己已经有了一点想法，后续会简单说一下．

### 传统多因子选股流程

 传统多因子选股流程可以分为因子筛选，构建Alpha模型，构建风险模型，组合优化和业绩分析。

1. **因子筛选**

   因子分类：

   目前因子分为Beta，动量，规模，盈利性，波动性，成长性，价值，杠杆率，流动性这９大类，总计上百种不同的因子，这些因子都是与股票预期收益率具有一定相关程度的．当然这只是目前已知的，相信随着时间的推移还会有更多的因子被开发出来（目前能量化新闻等文本数据的因子还没有，能量化市场情绪的因子也没有）

   * 构建因子池

     从上述因子中挑选一定数量的因子作为当前候选因子。

   * 标准化因子（计算因子暴露）

     由于不同的因子度量不一样，各上市公司的市值也不同，所以需要对因子进行标准化

     $$(因子－因子市值加权平均)/因子标准差$$，当然这是最简单的方法．本项目中也会遇到因子标准化的问题，采用的方法是将因子暴露与上市公司对数市值和行业哑变量进行回归，去残差的方法，消除市值和行业对因子暴露的影响．

   * 单因子检验（筛选出与收益率具有强相关性的因子）

     可以采用相关性分析，IC辅助法分析，回归分析等方法。

   * 因子合成

     由于筛选出的因子有可能因子之间还存在相关性，所以还需要进一步因子和成。

     可以采用相关系数，主成分分析等其他方法。

   * 合成因子检验

     检验合成的因子的有效性。

2. **构建Alpha模型**

   经过上述因子筛选已经得到了与股票收益率具有强相关性的因子，现在需要利用这些因子对股票的预期收益率进行评价。

   评价方法有打分法和回归法

   打分法：

   * 计算各个因子权重

     可以采用最大化组合因子IR（信息率）的方法

   * 个股打分

     对个股的因子暴露（即标准化后的因子）进行加权平均，对个股进行打分。

   * 组合初步筛选

     对打分后的股票进行排序，选取一定量的股票构成初始组合。

    回归法：

   ​	对因子暴露（即标准化后的因子）和股票预期收益率在历史时间序列上进行回归，估计参数即因子收益率。利用估计的因子收益率和当前时期计算的因子暴露进行线性组合，对个股进行打分。组合筛选也是选取分数较高的前N个。


3. **构建风险模型**

   目前了解的风险模型有协方差矩阵和结构化风险模型（和APT有点像，就是将风险分解）。

   风险模型这块还没有深入了解。（有时候风险因素也会被考虑到alpha模型中）

4. **组合优化**

   对上述通过因子选取的组合进行优化。

   添加风险模型，交易成本模型等约束，求解筛选的股票的最优组合（即二次规划问题）

5. **业绩分析**

   对获取的超额收益进行分析，分析这些收益的来源和选取的股票的特点。

### 实验过程

本项目实施的是选股模型，并没有考虑风险，组合优化等问题（从最简化开始）

* 项目第一阶段验证了Fama三因子和Carhart四因子模型在中国A股市场上的表现，实验结果可以参照[第一阶段任务总结及成果分析展示](./第一阶段任务总结及成果分析展示.md)，Fama三因子和Carhart四因子的计算过程可以参考[第一阶段实施方案](./第一阶段实施方案.md)

  总的来说，表现随回测时间段波动较大，时好时坏．这一阶段用的数据主要是tushare上的数据，没有实现Fama五因子主要也是因为这个原因，因为Fama五因子要用到财务数据，而tushare的财务数据质量太差了，缺失有点多，用不来．后面的实验全是基于巨潮资讯的数据，毕竟还是收费的有质量保证．强烈推荐使用高质量数据，要不然都不知道是自己模型问题还是其他问题影响结果．

* 项目第二阶段搜集整理了目前常见的因子，在机器学习中可以称其为特征，并对这些特征进行了编码，详见[quant_factors](./quant_factors.md)，同时利用CNN对部分基于价格算出来的因子（特征）进行序列特征的提取，并利用提取的特征对股票进行了打分，具体实现细节可以参照[第二阶段任务总结及成果分析展示](./第二阶段任务总结及成果分析展示.md)．这个模型比较简单，但是取得的效果还是不错的，甚至有的人可能觉得和Fully Connected的网络没什么区别，但是强调一点，CNN的卷积核是权值共享的，因为要提取不同的因子的序列特征，而这些因子的序列特征可能存在某些共性的特点，所以这里用了CNN没用Fully Connected．

* 项目第三阶段处理了一批财务数据，并将这些财务因子和上述基于价格算出来的因子进行了整合，由于财务因子的粒度和上述因子的粒度还不一样，整合的过程中采用了一些技巧，主要就是通过position embedding和filter gate解决了粒度问题．具体的实现可以参照[第三阶段任务总结及成果分析展示](./第三阶段任务总结及成果分析展示)．目前这个模型还在改进．

### 展望与愿景

1. 后续希望对财务数据进行分箱，做一波基于FM的特征交叉，再看一下效果，毕竟基本面还是能提供一些信息，要想办法把这些信息挖出来，哪怕很少呢．
2. 当然模型还希望能加入上市公司公告，财经新闻，分析师预测等文本信息，这部分已经有人做了可以参考[Deep Learning for Event-Driven Stock Prediction](https://www.ijcai.org/Proceedings/15/Papers/329.pdf)

最后希望就是能利用最先进的ML和DL技术挖掘尽可能多的跟股票超额收益率有关的数据，并利用这些数据指导投资决策，虽然可能存在风险，但是至少能提供一些参考．

### 困难与不易

最让人头疼的地方就是数据预处理这块，这块几乎花费了80％的时间．

1. 首先要把巨潮咨询的数据转成自己能用的格式
2. 需要处理数据缺失，识别停牌，复牌，新上市和退市的股票
3. 需要计算各种指标
4. 需要处理成神经网络的输入，使梯度能够回传，模型能够收敛

一个人干这事，确实太痛苦了！！！可能处理后的数据还有点瑕疵．希望对这方面感兴趣的人能够一起做一个开源的量化因子库，因为现在巨头的量化因子库太贵了！！！动辄几十上百万．如果有了质量比较好的量化因子库，我相信更多的ML&DL爱好者会投入到模型研究中，就像ImageNet推动图像发展那样！！！
