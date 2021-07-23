# 基于电影评分的推荐系统

•     基于个性化平均评分的推荐

•     基于电影相似度的协同过滤推荐

•     基于矩阵分解的协同过滤推荐

绘制大致**过程流程图**如下：

<img src="https://gitee.com/XiaoJing-C/images/raw/master/img/20210723130748.png" style="zoom:100%;" />

 

 

绘制大致**原理流程图**如下：

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723130847.png)

## 一、环境准备

### 实验环境：

`Anaconda 4.8.5 + Python 3.6.10` 

**所需程序包：**`lenskit 0.7.0 + pandas 1.1.1 + numpy 1.19.2 + seaborn 0.11.0`

除 `lenskit` 之外，其他程序包都可以通过 `pip` 命令直接安装，由于我的虚拟环境已经安装好这些基本的程序包，这里不再赘述。

**关于lenskit:**

`Lenskit` 是一组用于试验和研究推荐系统的Python工具集，它以灵活的方式为训练、运行和评估推荐程序算法提供支持，适合于研究和教学。

官方文档：https://lkpy.readthedocs.io/en/stable/

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723130905.png)

官方文档提供了三种安装方式：

1、使用anaconda 安装当前版本 

`conda install -c lenskit lenskit`

2、使用pip安装

`pip install lenskit`

3、直接从GitHub安装

`pip install git+https://github.com/lenskit/lkpy`

使用前两种方法安装，将安装最新版本 0.9.0，使用第三种方法安装，将安装稳定版0.11.1。两个版本都进行了尝试，发现在导包环节均会出现如下所示错误：

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723130924.png) 

错误提示信息显示无法导入方法 UnratedCandidates，查看 topn.py 源码，发现确实不存在该方法。 

查阅官方文档后发现这两个版本均已经摒弃了这个方法，于是查看实验平台所使用的lenskit版本号。

<div align=left><img src="https://gitee.com/XiaoJing-C/images/raw/master/img/20210723130950.png" style="zoom:100%;" /></div>

 

如上图所示，平台使用的 lenskit 为0.7.0版本。于是卸载已安装好的lenskit，重新进行安装。

<div align=left><img src="https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131040.png" style="zoom:100%;" /></div>

 

<div align=left><img src="https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131107.png" style="zoom:100%;" /></div>

到此，环境搭建已完成。

### 数据集：

下载地址：https://grouplens.org/datasets/movielens/

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131136.png)

下载 small 小型数据压缩包，其中包含四个数据集 ，我们使用 ratings.csv 和 movies.csv 完成本次实验。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131205.png)

ratings.csv 是电影评分历史数据集，包含 100836 个样本，每个样本包含 4 个属性，如下图所示：

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131217.png)

movies.csv 是电影信息数据集，包含 9742 个样本，每个样本包含 3 个属性，如下图所示：

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131225.png)

由于下载的数据集与平台数据集除样本数有差别外，其他完全一致，所以不用进行**数据清洗**。

## 二、探索性数据分析

#### 1、载入实验需要的软件包

•     lenskit

•     pandas：用于数据分析和处理

•     matplotlib：用于画图，实现数据可视化

•     numpy：用于存储处理数据

•     seaborn：用于数据可视化

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131239.png)

#### 2、数据读取

•     读取电影评分历史数据集 ratings，并修改变量名，使用 rename 函数将原数据表中 userId 字段改为 user ，movieId 改为 item，实现见名知意，且方便后续操作。打印数据维度并使用 head( ) 函数查看前几行。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131251.png)      

•     同样的，读取电影信息数据集 movies，修改变量名称，并查看数据集的维度和前几行。 

#### ![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131303.png)

#### 3、数据聚合与连接

•     将基于电影评分历史数据集 ratings，按照变量 item 做聚合，计算各电影的平均评分和评分数量，查看前几行。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131321.png)•     使用merge函数与电影信息数据集 movies 做内连接，添加电影的标题（变量 title ）和题材（变量 genres )，将连接后的数据存在新变量 movie_ratings 中， 查看前几行。 

–     可以看到新的数据集由 rating_mean、rating_count、items、title、genres 这五个属性构成。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131341.png) 

•     使用 dropna 函数删除缺失值

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131355.png)

#### 4、电影评分分析

•     使用 seaborn 库中的 countplot 函数用条形显示每个分箱器中的观察计数，直观显示出评分的分布图

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131409.png) 

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131418.png)

​      

​      

​      

•     displot 是直方图和密度图的集合体，绘制各电影平均评分的分布图

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131428.png)

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131436.png)

•    绘制各电影评分数量的分布图。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131524.png)    

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131537.png)

 

 

•     得到平均评分最高且有100条以上评分的前20部电影

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131549.png)

•     绘制这20部电影的平均评分

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131602.png) 

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131612.png)

 

•     将 ascending 参数设为 False ，实现降序排列，以此得到评分数量最多的前20部电影

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131624.png)

•     绘制这20部电影的评分数量

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131635.png)

​      

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131646.png)

 

## 三、数据预处理

### 1、交叉验证数据分割

将数据集分为5组训练集和测试集做5折交叉验证。具体方法为：

1. 将用户分割成5组测试集用户；

2. 对于每组测试集用户，再进一步抽取这些用户20%的评分样本作为测试集；

3. 创建对应的训练集，包含该组测试集用户的其余80%的评分样本以及所有非测试集用户的评分样本。

调用程序包 lenskit.crossfold 中的函数 partition_users() 将数据集分割为5折交叉验证中各折对应的训练集和测试集，其中 

•     第1个参数 data 表示用户评分数据框；

•     第2个参数 partitions 表示分割用户的分组数，即交叉验证的折数；

•     第3个参数 method 表示每个用户抽取评分样本的方法，这里为抽取20%；

•     返回结果为成对训练集和测试集的迭代器。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131707.png)

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131722.png)

查看每组训练集和测试集的形状。 

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131737.png)

可以看到每组训练集与测试集的维度如上图结果所示。

### 2、候选推荐数据准备

使用 concat 函数，利用 for 循环合并所有的测试集，为后续模型性能评估做准备。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131750.png)

在给用户做电影推荐时，我们需要给用户提供一个候选推荐列表。

但是数据集中有部分电影只有极少数的评分，并不稳定，不适合做推荐，所以这里我们剔除所有评分少于5条的电影记录。 

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131804.png)

## 四、基于个性化平均评分的推荐

用户电影偏移量评分预测算法定义为：

![](https://gitee.com/XiaoJing-C/images/raw/master/img/clip_image002.png)

其中，![](https://gitee.com/XiaoJing-C/images/raw/master/img/clip_image001.png)是全局平均评分，![](https://gitee.com/XiaoJing-C/images/raw/master/img/2.png)是电影偏移量![](https://gitee.com/XiaoJing-C/images/raw/master/img/1.png)是用户偏移量。进一步引入阻尼参数![](https://gitee.com/XiaoJing-C/images/raw/master/img/3.png)和![](https://gitee.com/XiaoJing-C/images/raw/master/img/4.png)，计算公式为

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723131856.png)

调用程序包 lenskit.algorithms.basic 中的构造函数 Bias() 创建偏移算法，其中

•     第1个参数 items 表示是否计算电影偏移量，默认为是；

•     第2个参数 users 表示是否计算用户偏移量，默认为是；

•     第3个参数 damping 表示阻尼参数。

我们可以查看lenskit的官方文档获得更加详细的介绍。

lenskit.algorithms.basic 模块包含个性化平均评分预测，具体用法如下：

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723132015.png)

 

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723132027.png)

我们主要采用两种方式评估推荐算法。

•     **第一种**是由推荐算法预测每个用户评分最高的前𝑘个电影列表，与实际该用户已评分的电影做对比，计算根据排名加权的重合度。

•     **第二种**是由推荐算法预测每个用户已评分的电影的评分，与真实值做对比计算误差。

​      

### 1、推荐列表

•     遍历5组训练集和测试集，每次迭代用训练集做训练，并给测试集中的所有用户推荐每个用户评分最高的前100个电影列表。
 

程序包 lenskit.batch 中的函数 recommend() 用于给用户批量推荐影片，其中

•     第1个参数 algo表示推荐算法；

•     第2个参数 users 表示需要推荐的用户列表，这里为所有测试集中的独立用户；

•     第3个参数 n 表示推荐的电影数量；

•     第4个参数 candidates 表示候选推荐列表，这里调用程序包 lenskit.topn 中的构造函数UnratedCandidates() 返回每个用户没有评分过的电影列表。

 

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723132727.png)

合并所有测试集的推荐列表，为模型性能评估做准备。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723132739.png)

使用偏移算法的属性 mean_、item_offsets_ 和 user_offsets_ 分别得到全局平均评分𝜇、标识符为1的电影（《玩具总动员1》)偏移量 𝑏𝑖 和标识符为145的用户偏移量 𝑏𝑢。 

![image-20210723132800609](C:\Users\1979\AppData\Roaming\Typora\typora-user-images\image-20210723132800609.png)

可以看到，𝜇 为 3.5002643829509283、𝑏𝑖 为 0.4599346220241948、𝑏𝑢 为 -0.23977443873998147

调用程序包 lenskit.topn 中的构造函数 RecListAnalysis() 创建推荐列表分析对象。进一步调用该对象的函数add_metric 添加评价指标，这里添加的指标为归一化衰减累积增益（normalized discounted cumulative gain，简称NDCG），这里调用程序包 lenskit.metrics.topn 中的函数 ndcg() 得到。

归一化衰减累积增益定义为：

 



![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723132820.png)

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723134424.png)

调用推荐列表分析对象的函数 compute() 计算评价指标，其中

•     第1个参数 recs 表示推荐评分的电影列表；

•     第2个参数 truth 表示用户真实评分的电影列表；

•     返回结果为每个用户的评价指标值，这里为 NDCG。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723132851.png) 

通过 mean 函数得到所有测试集用户的 NDCG 均值为 0.017595354789097562。

### ![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723132902.png)

### 2、评分预测

遍历5组训练集和测试集，每次迭代用训练集做训练，并给测试集中的所有用户和影片组合预测评分。

程序包 lenskit.batch 中的函数 predict() 用于给用户和影片组合批量预测评分，其中

•     第1个参数 algo 表示推荐算法；

•     第2个参数 pairs 表示需要预测评分的用户和影片组合，这里为所有测试集中的用户和影片组合。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723132919.png) 

合并所有测试集的评分预测，为模型性能评估做准备。

![image-20210723132935167](C:\Users\1979\AppData\Roaming\Typora\typora-user-images\image-20210723132935167.png) 

调用程序包 `lenskit.metrics.predict` 中的函数 `rmse()` 计算预测评分与真实评分的均方根误差，其中

•     第1个参数 predictions 表示预测评分；

•     第2个参数 truth 表示真实评分。

## ![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723132950.png)

## 五、基于电影相似度的协同过滤推荐

调用程序包 `lenskit.algorithms.item_knn` 中的构造函数 `ItemItem()` 创建基于电影相似度的协同过滤算法，其中

•     第1个参数 nnbrs 表示 k 近邻的数量。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133001.png)

这里我们的参数设置为20。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133028.png)

### 1、推荐列表

遍历5组训练集和测试集，每次迭代用训练集做训练，并给测试集中的所有用户推荐每个用户评分最高的前100个电影列表。 

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133039.png)

合并所有测试集的推荐列表，为模型性能评估做准备。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133053.png)使用基于电影相似度的协同过滤算法的属性 sim_matrix_ 得到电影的相似度矩阵，并经过一系列变换得到与标识符为1的电影（《玩具总动员1》)相似度最高的前10个电影。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133105.png) 

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133117.png)

 

同样的，计算测试集用户的NDCG均值。 

### ![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133130.png)

### 2、评分预测

遍历5组训练集和测试集，每次迭代用训练集做训练，并给测试集中的所有用户和影片组合预测评分。 

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133449.png)

合并所有测试集的评分预测，为模型性能评估做准备。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133506.png)

调用程序包 lenskit.metrics.predict 中的函数 rmse() 计算预测评分与真实评分的均方根误差，其中

•     第1个参数 predictions 表示预测评分；

•     第2个参数 truth 表示真实评分。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133518.png)



## 六、基于矩阵分解的协同过滤推荐

该算法使用交替最小二乘法（alternating least squares）训练偏差矩阵分解。

调用程序包 lenskit.algorithms.als 中的构造函数 BiasedMF() 创建基于矩阵分解的协同过滤算法，其中

•     第1个参数 features 表示隐因子的数量；

•     第2个参数 iterations 表示迭代次数；

•     第3个参数 reg 表示正则化参数；

•     第4个参数 damping 表示阻尼参数。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133532.png)

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133551.png)

### 1、推荐列表

遍历5组训练集和测试集，每次迭代用训练集做训练，并给测试集中的所有用户推荐每个用户评分最高的前100个电影列表。 

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133606.png)

合并所有测试集的推荐列表，为模型性能评估做准备。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133620.png)

计算测试集用户的NDCG均值，得到0.0775176690540789

### ![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133630.png)

### 2、评分预测

遍历5组训练集和测试集，每次迭代用训练集做训练，并给测试集中的所有用户和影片组合预测评分。 

![image-20210723133851824](C:\Users\1979\AppData\Roaming\Typora\typora-user-images\image-20210723133851824.png)

合并所有测试集的评分预测，为模型性能评估做准备。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133858.png) 

调用程序包 lenskit.metrics.predict 中的函数 rmse() 计算预测评分与真实评分的均方根误差，其中

•     第1个参数 predictions表示预测评分；

•     第2个参数 truth 表示真实评分。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133913.png) 

## 七、推荐算法的测试和评估

### 1、归一化衰减累积增益（NDCG）

绘制3种推荐算法用于推荐列表的NDCG。可以看出，基于个性化平均评分的推荐最差，基于矩阵分解的协同过滤推荐最好。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723133958.png)

### 2、均方根误差

绘制3种推荐算法用于预测评分的均方根误差。可以看出，基于个性化平均评分的推荐最差，基于矩阵分解的协同过滤推荐最好。 

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723134024.png)

### 3、推荐列表

得到标识符为145的用户在测试集中的电影列表。 

查看该用户在训练集中的电影列表，这些记录输入了推荐算法用于训练。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723134055.png)

查看基于个性化平均评分推荐的前20条推荐列表 

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723134111.png)

查看电影相似度的协同过滤推荐的前20条推荐列表 

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723134124.png)

查看基于矩阵分解的协同过滤推荐的前20条推荐列表 

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723134140.png)

### 4、评分预测

查看基于个性化平均评分推荐的评分预测。

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723134159.png)

查看电影相似度的协同过滤推荐的评分预测

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723134216.png)

查看基于矩阵分解的协同过滤推荐的评分预测

![](https://gitee.com/XiaoJing-C/images/raw/master/img/20210723134240.png)

 

通过预测结果与实际得分之间的差距可以看出，基于矩阵分解的协同过滤推荐预测效果更胜一筹。

## 八、总结

•     我们采用了三种不同的推荐算法进行测试，并且通过测试与评估可以发现，基于个性化平均评分推荐算法性能最差，而基于矩阵分解的协同过滤推荐算法性能最好。

•     协同过滤算法是一种较为著名和常用的**推荐算法**，它基于对用户历史行为数据的挖掘发现用户的喜好偏向，并预测用户可能喜好的产品进行推荐。也就是常见的“猜你喜欢”，和“购买了该商品的人也喜欢”等功能。它的主要实现由：

–      根据和你有共同喜好的人给你推荐

–      根据你喜欢的物品给你推荐相似物品

–      根据以上条件综合推荐

•     通过查找资料可以知道，协同过滤算法在大数据情况下，由于计算量较大，不能做到实时地对用户进行推荐。基于模型的协同过滤算法有效解决了这一问题，矩阵分解是基于模型的协同过滤算法中的一种，在基于模型的协同过滤算法中，利用历史数据训练得到模型，并利用该模型实现实时推荐。

•     我们应当根据实际情况以及实际需求，来选择合适的推荐算法。