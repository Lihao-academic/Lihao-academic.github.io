---
title: "Talents, Industry, and Richness: How are they related? "
excerpt: "数据可视化课程项目，自行搜索数据并提出研究问题，讨论了OECD国家的工业发展、购买力水平和STEM领域毕业生三者直接的关系<br>![](/files/DataVisualization/dataviz_cover.png)"
collection: portfolio
order: 4
---
## 项目说明

本项目为Data Visulization课程的课程项目。课程目的是使学生掌握多种数据可视化工具，然而可视化永远只是辅助，如何讲好故事才是关键。本项目自主搜索数据，并提出研究目标，经数据整合后，探讨了日常生活中人们常常关心的问题，工业发展，工资，购买力水平，这三者的联系与内在关联
## 研究问题

购买力，STEM学科学生的比例，工资，税，以及国家工业发展，人们常常讨论这些问题，但是他们是如何相互联系的呢？这个项目就是想直观展示一下这些概念的联系。研究问题有：

* STEM学生的比例及其工资，与国家工业发展有关吗？比如国家工业发展会不会因为STEM学生多一点而得到提升，国家工业水平会影响STEM的薪资溢价吗？

* 购买力水平和STEM学生的比例有关系吗？税后购买力和薪资呢？是工业导致国家富有，但是会因为国家太富有，导致人们不愿意去学STEM学科？是平均购买力，还是税后购买力更能影响人们选择STEM的意愿，还是单纯是工资影响？

* 税率本身会影响工业发展吗？提升还是抑制？

## 数据

这个项目的数据是自己搜索的，因此一开始story-driven的目标没办法顺利推进，在不断循环修改以后，最终才确定了使用的数据库。再加上数据量比较小，因此也没有使用任何API或者爬虫等下载数据，全部都是手动下载。本项目使用的数据来自WorldBank和OECD，全部都提供Data Explorer。包括：

PPP数据
```
https://data.worldbank.org/indicator/NY.GDP.MKTP.PP.CD
```
Taxing Wages
```
https://data-explorer.oecd.org/vis?df[ds]=DisseminateFinalDMZ&df[id]=DSD_TAX_WAGES_COMP%40DF_TW_COMP&df[ag]=OECD.CTP.TPS&dq=.AV_TW..S_C0.AW100._Z.A&lom=LASTNPERIODS&lo=10&to[TIME_PERIOD]=false&vw=tb
```
Share of STEM students
```
https://data-explorer.oecd.org/vis?fs[0]=Topic,1%7CEducation%20and%20skills%23EDU%23%7CStudents%23EDU_STU%23&pg=0&fc=Topic&bp=true&snb=31&df[ds]=dsDisseminateFinalDMZ&df[id]=DSD_EAG_UOE_NON_FIN_STUD%40DF_UOE_NF_DIST_FIELD&df[ag]=OECD.EDU.IMEP&df[vs]=1.0&dq=.......A.......&pd=,&to[TIME_PERIOD]=false&lb=bt&vw=tb
```
Percentage of Income of STEM workers
```
https://data-explorer.oecd.org/vis?lc=en&df[ds]=dsDisseminateFinalDMZ&df[id]=DSD_EAG_LSO_EA%40DF_LSO_EARN_ALL&df[ag]=OECD.EDU.IMEP&df[vs]=1.0&dq=AUS%2BAUT%2BBEL%2BCAN%2BCHL%2BCOL%2BCRI%2BCZE%2BDNK%2BEST%2BFIN%2BFRA%2BDEU%2BGRC%2BHUN%2BIRL%2BISR%2BITA%2BJPN%2BKOR%2BLVA%2BLTU%2BLUX%2BMEX%2BNLD%2BNZL%2BNOR%2BPOL%2BPRT%2BSVK%2BSVN%2BESP%2BSWE%2BCHE%2BTUR%2BGBR%2BUSA%2BARG%2BBRA%2BBGR%2BPER%2BROU%2BZAF._T.Y25T34%2BY25T64%2BY35T44%2BY45T54%2BY55T64.ISCED11A_5T8.F05T07............A3&pd=2000,2025&to[TIME_PERIOD]=true
```
Industry value added (% of GDP)
```
https://data.worldbank.org/indicator/NV.IND.TOTL.ZS
```

这些数据的License全都是(CC-BY 4.0)，我们用来做课程项目并公开展示没有任何问题。

尽管数据质量已经比较高了，但依然存在缺失值和异常值，需要进行删除、填补和核验。这就是数据处理的部分了。


## 概念引入

在正式开始之前，作为报告，我们需要先理清楚概念，这些概念是本项目讨论中涉及到的，这里给出简要说明，不是严格定义。

- 购买力平价 (PPP)：衡量一个国家公民真实财富的指标，通过调整不同的价格水平和汇率，实现各国购买力的公平比较。

- Tax Wedge：员工缴纳的税款和社保缴款总额与其劳动总成本的比率。本质上，它是政府从工资中抽取的份额。

- 工业增加值占GDP的百分比(Industry value added, percentage of GDP)：一项宏观经济指标，显示一个国家经济总产出中工业部门（如制造业、采矿业和建筑业）的份额，而非服务业或金融业的份额。**本文项目中提到的工业水平或工业发展，都是指这个指标，尽管这个指标实际上难以完全衡量国家的工业水平**

- STEM：科学、技术、工程、数学的缩写，这些学科推动着一个国家的高科技研究、创新和产业劳动力发展。

- STEM从业人员收入 (%)：STEM专业人员的收入溢价或相对工资，相对于全国平均工资而言。例如，118% 这个数值意味着 STEM（科学、技术、工程和数学）领域的从业者比普通从业者收入高出 18%。

## 方法论

首先，所有的操作都由python完成，因此可以保证复现，我在Pycharm中使用Jupter Notebook，配合一些数据处理库，包括numpy, pandas和matplotlib等。

我们的目标是把多个数据表直接合并成一个wide table, 由于数据集非常多，大部分的操作都是删除、合并。

具体来说，我们需要先确定一个主键，比如Country Code等可以串联起所有数据表的水平。不过值得一提的是，我还使用了一个可以自动获取Country Code的软件库，因为不是所有数据库都有精准干净的Country Code。

我使用PPP数据作为主要的表格，在确定好以后，就开始处理剩下数据表，并不断的left merge，按照顺序，先后合并了Taxing Wedges，STEM毕业生收入，工业指数

在处理数据的过程中，我发现STEM学生比例这个数据有少量缺失，我没有删除他们，而是采用forward fill/ backword fill等方法填补；


而工业指标的数据的缺失我采用了线性拟合插值的方法处理了。

最后合并大表格的时候，Country Code和Year都不能乱，因此采用的是双主键合并。


最后，我们不想研究时间趋势，但是时间信息也不愿意完全浪费，因此我把2016年到2024年的数据合并到了一起，我们采用了时间加权平均的办法，年份越近，权重越大。最终效果如图：

![final wide table](/files/DataVisualization/widetable.png)



## 结果分析

结果其实比较出乎意料，那就是我们认为会有一定关联的概念，实际上相关性非常弱。由于结果图片比较重复，这里也就不逐个放了，如果感兴趣可以去Github上看。

我们直接给出结论。首先“STEM成绩占比”与“工业指标”的关系就比较弱，
```
r = 0.151，R² = 0.023，p = 0.364
```

这是非常弱的正相关，关键是统计上不显著。

STEM salary premium → Industry 更弱，大概：
```
r = 0.065，R² = 0.004，p = 0.784
```

应该说，没有观察到明显证据表明STEM毕业生比例或者STEM工资溢价与工业增加值占GDP比例具有较强线性关系

第二组是PPP ↔ STEM 比例，更接近于完全没有关系：
```
PPP与STEM比例：约 r = 0.052
税后PPP与STEM比例：约 r = -0.011
```
所以至少从这个样本看，国家更富裕并不明显对应更多学生选择 STEM。

第三组是 Tax Wedge ↔ Industry，大约：
```
r = -0.186，R² ≈ 0.035，p ≈ 0.263
```
有一点负方向，但仍然很弱。

最后，我们还检验了STEM从业人员的工资溢价是否与STEM学生比例有关。

结果依然非常接近于零：
```
r = -0.035，R² = 0.001，p = 0.883
```
也就是说，在当前样本中，并没有观察到STEM从业人员拥有更高工资溢价的国家，会有更多学生选择STEM领域。

这个结果其实很有意思。直觉上，工资可以被理解为劳动力市场提供的一种经济激励，如果某一领域收入明显更高，我们可能预期更多学生会选择这一领域。然而至少在这些OECD国家中，这种关系并没有明显体现出来。

当然，这并不意味着工资对个人专业选择没有影响，而只能说明国家层面的STEM工资溢价与STEM学生比例之间没有表现出明显的线性关系。

## 进一步探索

由于前面的线性相关分析几乎都没有得到明显的关系，我们又进行了一个探索性的分组比较。

首先按照Tax Wedge的中位数将样本国家分为高税负和低税负两组。样本中的中位数约为37.52%。

分组以后得到：

![high vs low](/files/DataVisualization/low_vs_high.png)

一个比较有意思的现象是，高税负国家的STEM比例反而略高，但是工业增加值占GDP的比例更低。

随后我们进一步选择Tax Wedge最高和最低的各5个国家进行极端组比较。这种差异变得更加明显：低税负国家的平均工业占比更高，而高税负国家的STEM比例反而更高。

![highest vs lowest](/files/DataVisualization/highest_vs_lowest.png)

不过，这里的分析只能被理解为探索性结果。极端组比较会主动放大样本之间的差异，而且没有控制其他变量，因此不能由此推断税负导致了工业结构或者STEM教育选择的变化。

## 讨论与局限

本项目最终得到的最明显结果，反而是大部分我们原本认为可能相关的变量之间并没有表现出很强的线性关系。

这并不是说这些因素之间不存在联系，而更可能说明国家层面的教育选择和产业结构无法通过少数几个宏观经济指标简单解释。

首先，本项目存在大量未纳入的潜在变量。例如教育制度、产业政策、移民结构、性别差异、文化环境以及不同职业在社会中的吸引力，都可能影响STEM领域的人才供给。

其次，不同数据集的完整程度并不一致。尤其是STEM收入数据最终只有约21个OECD国家能够用于分析，因此相关结果的样本量较小。

此外，为了整合不同年份和不同来源的数据，本项目对少量缺失值进行了前向/后向填充或趋势拟合，并进一步通过时间加权平均把多年数据压缩成国家级横截面数据。这提高了不同数据源之间的可比性，但也牺牲了一部分时间变化信息。

最后，本项目本质上是一个探索性的数据可视化项目。相关系数、回归趋势和分组比较只能用于发现结构性关联，而不能证明因果关系。

## 项目总结

从结果上看，这个项目并没有找到一个令人惊讶的强相关关系，但这本身也是一次有价值的探索。

项目最初是story-driven的：先提出自己感兴趣的问题，再寻找能够回答这些问题的数据。然而实际过程中，很快就遇到了数据定义不同、年份不一致、国家覆盖范围不同以及缺失值等问题。因此，项目最后很大一部分工作实际上转变为了如何将World Bank与OECD多个不同来源的数据整理成一个可以共同分析的数据集。

从技术上，本项目完成了多源数据清洗、长宽表转换、双主键合并、缺失值处理、时间加权聚合以及相关性和可视化分析。

更重要的是，结果提醒了我们：一个看似非常合理的故事，并不一定能够在数据中得到支持。可视化的作用并不是证明最初的猜想，而是帮助我们更直观地理解数据真正呈现出的结构。




