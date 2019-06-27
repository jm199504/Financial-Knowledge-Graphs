Machine Learning With A Heart

**竞赛链接**

<https://www.drivendata.org/competitions/54/machine-learning-with-a-heart/>

**竞赛排名**（截止于2019.06.06）

![rank](images\rank.png)

**数据介绍**

![introduction](images\introduction.png)

**数据观察**

由于数据特征中包含类别数据，因此使用哑变量转换

![observe](images\observe.png)

**添加衍生数据特征**

通常建模过程中需要尝试添加衍生特征来增强模型性能，如各样本与其各类别对应特征中位数/均值差/绝对值差等等

![add](images\add.png)

**数据归一化**

由于各数据特征的尺度不同，因此需要考虑对数据归一化，如标准归一化、最大最小归一化等

![standard](images\standard.png)

**数据特征排名**

通常将全部数据特征放入预测模型中效果反而不佳，可以考虑特征选择，如随机森林、XGBOOST等树模型进行特征排名

![frank](images\frank.png)

**构建模型和寻找参数**

![model](images\model.png)

**预测结果处理**

由于本次预测竞赛是分类问题，因此对于模型输出0-1的预测值进行分类，其发现单一阈值法效果不佳的主要原因在于处于0.2-0.7间的部分样本模型预测有误，即存在0.3x的样本属于1类或者存在0.6x的样本属于0类的情况（个人认为测试数据集中存在异常点）

![result](images\result.png)

**模型融合**

尝试对模型融合发现效果不佳，主要原因是BayesianRidge模型已经处于很高得分