---
title: "Housing Prices (Kaggle Project)"
excerpt: "为了体验完整数据处理、分析、模型构建而做的练手项目。包括数据清洗、缺失值处理、类别编码、特征工程、特征选择、交叉验证与最终预测提交。<br>![](/files/Housing%20Prices/houseprice_cover.jpg){: width='70%' }"
collection: portfolio
order: 2
---


*There's no English version for the moment, check this project on Github*

*本页暂时没有英文版本，可以在Github上查看*

```
https://github.com/Lihao-academic/Kaggle_House_Prices
```


## 项目介绍（进度90%）

由于本人之前并没有完整的处理过有一定复杂度的数据集，所以选择这个Kaggle项目训练一下，Housing Prices是一个基础的项目，但是足够施展整个流程了。所以本项目的目的是把数据处理与分析的基本功做扎实，而不求达到很好的效果，只想落地工程。我在这个项目中，首先详细完整的探索了数据，为数据清洗做准备，接着就是数据清洗，然后搭建loader，接着是建模并优化，最后还有分析。这个项目整体虽然比较简单，但是跑得很完整。

项目代码主要基于jupyter notebook开发，最终也把代码固定到`.py`文件里，尽可能规范的实现了完整流程。这个项目中，我体验了一遍完整的问题处理流程，更加学习了Pandas和sklearn等工具。由于是刻意的练习项目，全程对AI使用保持克制，只咨询代码用法并讨论宏观流程。



## 数据介绍与探索


我直接从Kaggle官网手动下载数据，然后解压缩，放到`data/raw/`里面。里面有4个文件，`data_description.txt`是各行的介绍，或者说是这些列的介绍，`test.csv`和`train.csv`是用到的数据，在`train.csv`里面建模并做测试，然后最后用`test.csv`再测试，给出预测结果，提交上去。`sample_submission.csv`是一个提交样例，最后提交的结果就应该如此。我们的重点就是`train.csv`。

由于jupyter notebook可能会改动工作区域，我们使用`pathlib`模块来自动获取工作地址，保证只要目录如介绍的一样，就可以直接跑通。

---
数据预览的时候，我先用pandas把`.csv`文件读取成dataframe，探索了一下数据的基本情况。

数据大小为(1460, 81)，相当于总共有80个features，1个待预测的变量。数据类型很简单，只有3类，float64(3), int64(35), str(43)，只有3个是float64，大部分数值是int64。

不过，其中有的类别变量更适合当做数值来看，而有的数值变量可能更适合当类别来看。

比如，MSSubClass其实表示的是房屋建筑的类型，数字就是个纯编号，所以这个类看起来全都是数字，却决不能直接当做纯数字放到模型里面去。

还有就是房屋的id，这个自然也只能当类别变量，不能认为他的数值有意义。


有些数值变量看起来取值很少，这个要分情况，有的如HalfBath这种，即客用卫生间，配有可用卫生间的房子非常稀少，所以取值少。这是正常现象。


---

有关数据质量，经过统计以后，我发现大部分列都是全满的，但是少部分列不全，而还有的列几乎都是空的，全部都是NA。

NA的含义是一个很重要的问题，有的时候NA表示该房子没有对应设施，有的时候表示单纯的数据缺失，有的时候表示不适用或无意义

经过整理，我发现主要的，有缺失的列有：


* MiscFeature表示其他地方没有涉及的杂项，这个大部分数据都没有，也很合理。这个类的NA表示就是None，就是没有，因为该有的信息其他部分已经讲完了。

* MasVnrType是指墙外面贴的装饰性石材的类型，如果没有就是None，很多房子都没有，这是所以是0.59

* Alley是指房子挨着的小巷，绝大部分房子都没有这东西。

* PoolQC指游泳池评级，绝大多数房子没有配套的。

* FireplaceQu缺的也比较多，这个是NA指的是没有对应设施，0.45的房子没有这个设施。

* GarageType表示车库的比例，这个NA也是表示没有车库，不是真正的缺失值。它连带着一些其他的变量也是NA，不过绝大多数房子都有车库，比例非常低。包括其他Garage系列，GarageQual, GarageYrBlt, GarageFinish, GarageType, GarageCond，这些数据的缺失值情况和比例都一样。

* LotFrontage指的是房子到街道的距离，这个完全没说是什么类型数据，也没解释NA的含义，看来这个NA是真的彻底的数据缺失。

* Electrical是指供电类型，这个没有说明NA是什么意思，空缺的值应该是真的缺失值了。

总结一下，有至少4种类别：

|          | 真正的缺失值       | 没有相应设施或属性 |
|----------|--------------|-----------|
| 类别变量  | 如Electrical  | 如Alley    |
| 数值变量  | 如LotFrontage | ---       |


---

接下来我看了一下目标变量的行为。首先这个变量没有空缺值，取值数量也算合理，但是我发现这个变量并不是正态分布的，明显是right-skewed的，存在少量溢价很高的房子，这提醒我们，为了防止scale的影响，也许可以采用对数处理一下。

![](/files/Housing%20Prices/right_skewed.png)

---
接着进一步探索一下其他feature的情况。我选择了几个代表性的图片，先看一下情况：

* 面积LotArea
* 房间数量（卧室数量等）Bedroom，或者说BedroomAbvGrd
* 房子的评级之类，比如OverallCond
* 建造年份，YearBuilt

离散变量就用boxplot，连续变量就用scatter plot，然后发现他们基本上和价格是呈正比的。

![](/files/Housing%20Prices/boxplot.png)

我特别排查了一下BedroomAbvGr，即房间数量的行为，发现没有异常。在图上看会给人感觉他的取值有问题。

---
我们进一步直接看各个变量和目标变量SalePrice的correlation，取绝对值并排序，得到：

|特征  |相关程度|
|---|---|
|BsmtFinSF2       |0.011378|
|BsmtHalfBath     |0.016844|
|MiscVal          |0.021190|
|Id               |0.021917|
|...|...|
|TotalBsmtSF      |0.613581|
|GarageArea       |0.623431|
|GarageCars       |0.640409|
|GrLivArea       | 0.708624|
|OverallQual      |0.790982|
|SalePrice        |1.000000|

这里有趣的是，美国房价和车库的面积相关性很强，如果不是在当地生活的人可能不会清楚一这点。

相关性有个问题，就是只能计算数值变量，但是类别变量也需要勘察一下相关程度，我们这里选择使用互信息来处理。互信息的计算不允许空缺值，我首先填补了一下空缺，经过计算后，发现Neighborhood对SalePrice的互信息有0.5，这个数值不清楚具体是什么意义，但是至少非常合理，而BsmtQual, KitchenQual, ExterQual等也都会影响生活质量，因此对房价也有一定的影响。这些features都是我们日后建模可能用到的。

---
我们总结一下目前的探索，我认为至少已经把存在的问题看到了，虽然可能还不够详尽。总的来说，要点如下：

1. 数据81列1460行，也就是80个变量，1460个数据点，所以必定要做特征工程，大大减少特征数量才行
2. 数值变量有38行，大部分是整数，不过其中也有包括id之类的变量，显然需要后续排除
3. 有的数值变量虽然是数值变量，但实际上是类别变量，不过经过排查就id和MSSubClass需要区别对待
4. 不同变量的NA含义不同，有的表示没有该设备或属性，我们可以通过一些手段来处理，真缺失很少
5. 数据集的缺失值不多，除了LotFrontage这个变量有17.7%的真缺失以外，其他都是接近或低于5%的，可以手动处理
6. 目标变量SalePrice的分布是右偏的，可能需要使用对数处理
7. 不同的变量大小相差很大，可能需要scaling
8. 通过计算correlation找到了几个重点变量，不过只计算了数值变量的，其中GarageArea和GarageCars明显存在共线性
9. 通过计算Mutual Information确定Neighborhood等变量的重要性，这些一定也是要考虑进去的


## 清洗与处理

首先是有些列名和数据描述里面的不一样，我把它改成了和数据表述中的拼写；

然后开始处理缺失值。我首先统计了一下具有缺失值的列，然后发现，缺失值的形态各不相同，比例也不同，缺失值的含义也不同，因此不能使用完全一致的处理方法。这一点我们全面已经探索出来了。


对于真正的类别变量缺失值，我们把缺失值改成更明确的字符串，比如`PoolQC`的缺失值其实意思是没有游泳池，我们直接改成"NoPool"。这一轮设计到的变量有MiscFeature， Alley，PoolQC，FireplaceQu，Garage系列，Bsmt系列；

处理完以后又统计一下，剩下的是一些真正的缺失值，对于连续数值变量，我们采用中位数填补，如`LotFrontage`就是这样的真正缺失；

还有少数缺失值我直接删除掉了，总共删了9行数据（其实后来来看，不应该删除比较好）


最后留下MasVnrArea和MasVnrType，这两个比较特殊，因为MasVnrType里面既有None，也有NA，经过查阅元数据，发现NA非常少，大部分都是None，而似乎pandas把两种都合并在一起了，但只有NA才是真缺失，None只是NoMasVnr。既然pandas已经混淆了这两列，我们干脆先把所有的MasVnrType空缺值都充填为NoMasVnr，然后再用MasVnrArea删除行。因为探究发现，基本MasVnrArea的NA都来自于MasVnrType的空缺。

Garage系列也比较特殊，有一个“GarageYrBlt”，我发现我全部fillna()以后，数据类型变成了object，因为我之前把没有车库的空缺值都填补成"NoGarage"了，其他的都是类别变量，更具体来说都是"str"类型，我这么做就把本来是纯数值的年份也填补上了。但是GarageYrBlt又真的存在很多空缺值，这怎么办呢，我决定添加一个新的列"HasGarage"，有车库就是1，没有就是0，而GarageYrBlt的空缺值改成0，由于信息已经由HasGarage提供了，所以总信息没有变。这里有一些特征工程的意味，不过为了干净清理，这里就只能先这么做了。

处理完缺失值以后，就开始处理类别变量，因为有的类别变量实际上完全可以当做离散数值变量来看待，而还有一些是明确的类别变量，那些变量之间没有数值关系。我这里处理的原则是，直接把那些离散的变量编码成整数，并且我接受等距假设，认为一个类别变量的直接的举例是等距的，如`ExterQual`有5个取值，Ex, Gd, TA, Fa, Po，也就是Excellent, Good, Typical/Average, Fair, Poor，我直接编码成5-1；其实严格讲，这一步也是有点特征工程的意味，但是这里直接这么处理了，因为会简单很多。

还有一些数字变量，其实本质上是类别变量，典型的就是"MSSubClass"，他虽然是纯数字，但是这些数字表示的是建筑的类型，而不是某些具体的，与房价有关的含义，还有就是"Id"，这个是数据表的编号，显然对预测也没有用处。我们先保存Id，再删去，以免后面用到。而"MSSubClass"，我们直接用`.astype(int)`处理了。

---

其实到这一步，数据就已经清理完了，由于后面还需要做特征工程，然后才能做预处理，而我们这一步就是想要把预处理一起做完，所以我们中途保存了一下，保存为`train_cleaned.csv`。

顺便一提，这里也有本次学到的一个细节，csv不保存数据类型，而是根据数据情况推断的，这导致了后面每次读取，都需要额外再转换一下"MSSubClass"的类型，十分麻烦。以后记得可以保存成.parquet比较好。

---

数据清理干净以后，继续做预处理。这一部分其实是特征工程之后的事情，但是这里为了连贯性就先做了。预处理主要包括两个部分，一个是scaling，一个是1-hot encdoing。

还有个小细节就是目标变量"SalePrice"需要取log，这个一行代码就完事了。

Scaler我选择的是RobustScaler，是基于IQR做的，比较稳妥，不会受到异常值的影响。sklearn有这个函数，我们提取出"number"类型的数据，然后`fit_transform()`就可以了。

One hot encoding同理，先取出"str"类型的列，然后转换，然后再把这些新列拼回去。

最后处理完的形状是(1451, 241)，也就是241行，这些列展开以后扩充到这么多。新的列名命名方式为"原列名_取值"，如Alley本来有3个值，Grvl， Pave， NA, 而NA被我换成了"NoAlley"，所以就被展开成Alley_Grvl，Alley_NoAlley，Alley_Pave。

这样我们就搞清楚了数据预处理过程。

## 特征工程

我们这里的特征工程，非常简单直接，就是尝试手动构造一些可能对模型有帮助的变量，基本都是简单的算数组合，然后添加进去。基本上，我能想到的有这些构造方法：

* 年份可以用差值来代替建造时间

    GarageAge = YrSold - GarageYrBlt, 售卖年份减去车库建造年份，即售卖时，车库的年份

    HouseAge = YrSold - YearBuilt, 售卖年份减去建造年份，即售卖时房子的年份

    RemodAge = YrSold - YearRemodAdd, 售卖年份减去翻新年份，即售卖时距离上一次翻新的时间

* 可以把所有的面积加起来计算总使用面积

    TotalUtilSF = 1stFlrSF + 2ndFlrSF + TotalBsmtSF, 第一层加第二层加地下室总面积，即总建筑内可使用面积

* 卫生间等效加总

    EqvlBath = BsmtFullBath + 0.5 * BsmtHalfBath + FullBath + 0.5 * HalfBath

* 比例系列

    LivingRatio = GrLivArea/LotArea 可居住利用率

    BedroomRatio = Bedroom/TotalUtilSF单位使用面积的卧室数量

* 面积×质量

    OvrallMul = OverallQual * LotArea, 总体质量乘以面积

    BsmtMul = BsmtQual* TotalBsmtSF, 地下室质量乘以面积

    KitMul = Kitchen * KitchenQual, 厨房质量乘以面积

* Porch 总面积

    TotalPorchSF = OpenPorchSF + EnclosedPorch + 3SsnPorch + ScreenPorch, 门廊总面积

* 设施存在性

    HasPool, HasGarage, HasBsmt, HasFireplace, Has2ndfloor, HasPorch, HasFence各个设施的存在性

* 总设施数量

    Totalutil = HasPool + HasGarage + HasBsmt + HasFireplace + Has2ndfloor + HasPorch + HasFence, 总设施数量

* 根号项

    RootUtil = TotalUtilSF^0.5
    RootLotArea = LotArea^0.5


所以特征工程这一部分没什么代码难度。由此，我们也提出了我们的实验方案。我们不止跑一次，我们设计了总共三组特征处理，分别是：

Feature Set A (Baseline) : Data -> Preprocessing -> training

Feature Set B : Data -> BIC -> Preprocessing -> training

Feature Set C : Data -> Feature Eningeering -> BIC -> Preprocessing -> training

我们这一小节实现的就是Feature Eningeering

## 基线模型

为什么直接到基线模型了？不应该先写特征选择吗？因为我决定先跑通基线模型，而特征选择应该当做基线模型的某种改进或额外处理方法。

基线模型一开始也是数据处理。我直接读取了之前保存的`train_cleaned.csv`，这个数据集其实就已经可以喂给模型了。

不过和数据处理阶段不同的是，我们需要划分验证集，官方有测试集，但是测试集数据显然不能用来做优化，我们需要在训练数据里面额外画出来验证集。我们使用sklearn的函数划分，比例是20%

```
X_train, X_val, y_train, y_val = train_test_split(
    X,y,
    test_size=0.2, # 切分比例为20%
    random_state=42,# 种子设置为42
)
```

下一步是Scaling and One hot Encoding.同样是复制前面的代码。有个小问题是，encoder实际上是可以理解为一个可学习的功能器件，在做编码的时候，需要根据一定的数据来构造转换规则。而这样，如果在trainset上学习一下，可以在training set 和validation set两个地方使用，而不能在validation上使用。因为validation模拟的是不能得到的数据，如果让validation自己根据自己来做转化，肯定效果好，但这样不符合真实假设。说白了，就是轻微的数据泄露。


由于有可能某个类别在训练阶段没有见到全部的取值，而测试阶段遇到了新的取值，为了防止bug，需要让OneHotEncoder的handle_unknown设置为True.
```
encoder = OneHotEncoder(sparse_output=False, handle_unknown='ignore')
```

总之，代码和前面的一样，是重复处理。

训练模型就更简单了，也是调用sklearn的现成函数，如：

```
from sklearn.linear_model import LinearRegression
LRmodel = LinearRegression()
LRmodel.fit(X_train, y_train)
print("Fit succeed")
```

有个问题是，这个fit()结束了以后会自动产生一些输出，而这会引发Pycharm里的Jupyter Notebook的一个bug，就必须切断这个输出，下面再加一句什么，或者用分号阻断输出等。


模型训练完了以后，结果如：

R²： 0.893

RMSE： 0.13

MAE： 0.083


模型解释了89%的波动，还是比较有效果的。

我们直接下一步，做交叉验证。这样整体baseline就算完全跑通了。交叉验证也可以直接用sklearn里面的现成函数，不过出于学习的目的，我们采用自动化程度比较低的KFold.split()来处理，这样可以理解的更清晰。当然后面我们采用了自动化程度更高的方法。

首先我把预处理函数封装了一下，然后

```
def preprocessing(X_train, X_val, y_train, y_val):
    ...
    return X_train, X_val, y_train, y_val
```

数据处理代码写好后，利用KFold.split()辅助划分，这个函数返回一个列表结构，我们需要手动在这个列表里遍历，每次都需要重新处理，重新训练模型，然后记录分数。大致逻辑如下：

```
# 划分交叉验证
kf = KFold(n_splits=5, shuffle=True, random_state=42)

for fold, (train_index, val_index) in enumerate(kf.split(X_train), start=1):
    # 利用给定的indexes, 完成本轮的测试
    # 得到的是每一个fold的训练集，因此列名一样，但是长度很短（行数少）
    X_train_fold, X_val_fold = X_train.iloc[train_index], X_train.iloc[val_index]
    y_train_fold, y_val_fold = y_train.iloc[train_index], y_train.iloc[val_index]

    # 完成剩下的预处理
    ...
    # 训练模型
    ...
    # 添加分数
    ...
```
跑完以后，就得到分数了，基线模型跑完结果如下：

```
R^2:  [0.9169 0.8324 0.6873 0.8718 0.8083]
RMSE:  [0.0094 0.0149 0.0186 0.0119 0.0137]
MAE:  [0.0069 0.0085 0.0093 0.0081 0.008 ]
```

接着我又做了一次Pipeline的方法，重复了一遍。大致思路很简单，

1. 定义preprocessor

```
preprocessor = ColumnTransformer(
    transformers=[
        ("num", RobustScaler(), num_cols),
        ("cat", OneHotEncoder(handle_unknown="ignore"), cat_cols),
    ]
)
```

2. 用Pipeline把preprocessor和训练绑定起来

```
pipeline = Pipeline(
    steps=[
        ("preprocessor", preprocessor),
        ("LRmodel", LinearRegression()),
    ]
)
```

3. 使用cross_validate()自动执行交叉验证
同样需要利用KFold定义划分，但是不用自己写循环了，cross_validate()可以自动执行：

```
scores = cross_validate(
    pipeline,
    X,
    np.log(y),
    cv=cv,
    scoring=scorings,
    return_train_score=True
)
```

这里有个小细节，就是我们传入的y是取log的，因为preprocessor没有包括y的处理。

这一轮交叉验证跑通了Pipeline的方法，我们后面用的都是这种方法。最后得到的效果是这样的：

```
train_RMSE 0.09356 ± 0.00267
test_RMSE 0.15402 ± 0.0394
train_MAE 0.06565 ± 0.0013
test_MAE 0.08997 ± 0.0065
train_R2 -0.94506 ± 0.00306
test_R2 -0.84298 ± 0.08305
```

## 特征选择

我在统计建模课程里面学过选择的方法，BIC、AIC等选择准则，所以一开始我就想用这些准则来做特征选择。不过，其实我犯了一个致命的错误，就是忘了这玩意是干什么的了，这个BIC，是用来精简模型的，而绝非用来让模型预测更好的。不过考虑到，本来这一部分就叫做特征选择，所以也不算偏题。

首先，我们把原来的手动特征工程写成函数，然后写计算BIC的函数。因为sklearn没有提供这个函数，我们就需要自己写。公式倒是比较简单：
```
residual = y - y_pred
RSS = sum(residual²)
BIC = n * log(RSS / n) + k * log(n)
```
这里有个问题，就是每次计算BIC，都需要计算一次预测值，这就意味着对于240个列，每评估一个列，都需要拟合一次，这样，根据后面的结果，最后只剩下50多列，也就是需要做将近200次的fit和pred，这个运算量是比较大的。

计算BIC还是比较简单的，函数定义为`calculate_BIC(X_train, y_train)`，返回BIC值

根据BIC选择列就比较复杂了。首先需要获得当前的所有列名，然后计算一下删去每个列的BIC值，确定效果最好的BIC，和对应的列，删去新的列以后，再次做循环，概念代码如下：

```
def BIC_selector(X_train, y_train):
    # 先获得初始BIC, 列名
    ...
    # 进行循环，直到最后选出所有的列名
    while True:
        # 在当前的所有列里面循环
        for cols in all_cols_name:
            # 计算删除一个列之后的BIC
            new_BIC = calculate_BIC(X_train_copy.drop(cols, axis=1), y_train)
            # 如果刚刚删除一个列得到的BIC更小，就记下来，并更新当前最小的BIC
            ...
        # 如果一个循环结束，没有发现任何新的可删除列，就说明已经穷尽了，直接跳出while
        ...
        # 如果只剩下1列，也直接跳出
        ...
        # 更新阶段，把该删的删掉，更新“当前所有列”，记录历史
        ...
    return X_train_copy
```

代码写好以后，我们先看一下这两个函数能不能正常跑通，直接看效果

```
selected_X = BIC_selector(X,y)
selected_X.shape
```

结果是可以正常删列，但是我的电脑跑一次大概是13-14分钟，这个测试成本稍高。因为要删200列，每删一个列都需要遍历200个其他的列，所以是n(n-1)/2的复杂度，大概要拟合2万多次，计算量是比较大的。

在确认两个函数都可以正常运行以后，我们后面要做交叉验证，还需要使用Pipeline来自动化完成，所以需要想办法把BIC写进transformer兼容的形式。

这就是说，需要写两个类。这里的transformer兼容，具体来说，就是指需要类里面有两个函数，一个是fit()，一个是transform()；执行这个BIC删列的类的结构是这样的：

```

class BICSelector(BaseEstimator, TransformerMixin):
    def __init__(self):
        pass
    def fit(self, X_train, y_train):
        # 原本BIC_selector(X_train, y_train)的内容
        ...
        return self
    def transform(self, X_train):
        return X_train[self.selected_columns_]
```

我们经过测试，确定这个类可以正常使用，就开始组装pipeline，同样，先定义preprocessor，然后用pipeline组装起来，这里就需要多一步了

```
pipeline = Pipeline(
    steps=[
        ("preprocessor", preprocessor),
        ("selector", BICSelector()),
        ("LRmodel", LinearRegression()),
    ]
)
```

正因为我们前面把函数写成了兼容transformer的形式，才可以在这里放进Pipeline自动处理。

然后做了这个组合，也就是Set B的交叉验证，思路和前面相同，不再重复，交叉验证跑了55分钟，计算量相当大。现在我们得到的结果是：

|scorings| Set A(baseline)| Set B (only selection)|
|--------|----------------|------------------------|
|train_RMSE| 0.09356 ± 0.00267|0.1038 ± 0.0026|
|test_RMSE| 0.15402 ± 0.0394|0.1808 ± 0.0467|
|train_MAE| 0.06565 ± 0.0013|0.0741 ± 0.0013|
|test_MAE| 0.08997 ± 0.0065|0.0894 ± 0.008|
|train_R2| 0.94506 ± 0.00306|0.9324 ± 0.0035|
|test_R2| 0.84298 ± 0.08305|0.7792 ± 0.101|

哈哈，仔细看，发现其实SetB的表现明显要差不少，首先是RMSE，均方根误差，这次Set B的均方根误差，在测试集上的表现似乎不但没有明显提升，反而还稍微下降了一点。测试集上的则表现下降更明显，本来均方根误差只有0.15左右，现在变成了0.18左右；

MAE平均绝对值误差的表现类似，没有明显下降，基本持平甚至稍微变差一点点

R2最明显，训练集上面的差别不太明显，而测试集上的表现下降明显。

这其实提醒我们一件事，那就是BIC不是用来让预测变得更好的，他只是精简模型的，尤其是BIC的惩罚相对来说并不小，删列删的太多，影响总的预测结果也很正常。这里面有趣的是，MAE没有太大变化，即使是测试集也没有太大变化，而RMSE则降低稍微明显一点。我们知道RMSE是平方，对于比较大的误差，可以剧烈的放大，这也说明，可能是大部分普通数据的预测效果依然差不多保持了，但是对于少数难以预测的数据，误差被放大了。

并且，至少我们在这里证明，经过BIC选择的列，可以依旧保持和原来差距不太大的表现，原来有80列，经过编码以后，有210列，而BIC删去了160列，只剩50列，还能基本保持预测效果，这就凸显了其对模型的精简效果十分明显。


Set B完成以后，就得开始Set C的，根据我们之前的设计，Set C 就是多了一个特征工程的部分，同样，为了自动化测试，需要把特征工程写成transformer兼容的类，这个很简单，fit什么都不做，因为没有任何需要学习的参数，直接在transform()阶段返回结果就行了，非常简单：

```
class FeatureEngineering(BaseEstimator, TransformerMixin):
    def __init__(self):
        pass
    def fit(self, X, y=None):
        return self
    def transform(self, X):
        #  原本特征工程的函数
        ...

        return X_input
```

最后，得到新的结果

|scorings| Set A(baseline)| Set B (only selection)| Set C (FN+selection)|
|--------|----------------|------------------------|------|
|train_RMSE| 0.09356 ± 0.00267|0.1038 ± 0.0026|0.1025 ± 0.0022|
|test_RMSE| 0.15402 ± 0.0394|0.1808 ± 0.0467|0.1794 ± 0.0426|
|train_MAE| 0.06565 ± 0.0013|0.0741 ± 0.0013|0.0724 ± 0.0015|
|test_MAE| 0.08997 ± 0.0065|0.0894 ± 0.008|0.0895 ± 0.0078|
|train_R2| 0.94506 ± 0.00306|0.9324 ± 0.0035|0.9341 ± 0.0028|
|test_R2| 0.84298 ± 0.08305|0.7792 ± 0.101|0.7857 ± 0.0949|

如果仔细观察，就发现set C其实对比set B还是有轻微的提升的，RMSE稍微降低了，MAE也稍微降低了，R2也都稍微增加了，不过显然，手工特征工程似乎并不能抵挡BIC的损害。

不过进一步详细的分析我打算放到final Demo里面再做，因为总体的效果，毫无疑问其实是Baseline最好，我们干脆就用Baseline先生成一个提交数据，先把Kaggle提交了。

## 生成提交

到了这一步，有了前面的代码铺垫，就非常简单清楚了，直接训练baseline，不用交叉验证，训练完了以后预测生成提交样本就行了。

值得说一说的是，我发现测试集和训练集竟然数据性质不一样，测试集里面出现了训练集里没有出现的新的问题，而且还有个问题，那就是我们训练集处理缺失的时候，删去了几行，而测试集一定不能删，所以也必须想新的办法。

由于我在做这一步的时候，已经固定了许多代码，所以我的处理方法是，在数据清洗的步骤中固定的函数`clean_data()`中额外加一个参数，然后据此专门加一步处理：

```
def clean_data(df_train, test_data = False):
    .....
    if not test_data:
        ....
    else:
        ....
```

处理方法也稍有不同，原来有删列的方法，这里我改用众数、中位数等填充。

测试集和训练集性质不同，这是一个很有用的教训。最后预测生成，提交就好了。

## Final Demo （待完成）







