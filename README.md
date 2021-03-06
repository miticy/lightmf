# lightmf
__A light-weight matrix factorization tool__

##Introduction

__lightmf__是一个轻量级的矩阵分解工具, 实现了推荐系统中其中一类重要的模型--__隐因子模型__的训练和预测.

功能上, __lightmf__使用了SGD实现了对带偏置的隐因子模型(__BiasMF__)的训练.
尽管没有像__SVDFeature__一样提供包括SVD++, Learning to Rank等衍生模型, 
也没有__libfm__提供除了SGD外的ALS, MCMC等多种模型训练方式, __lightmf__的易用性也是前两者无法比拟的, 体现在以下几个方面:

1. __训练和预测工具简单易用, 只需要提供2-3个必要的参数即可运行__
2. __输入数据格式简洁, 无需做任何预处理. 支持文本id数据__
3. __支持模型验证__, 可以指定训练过程, 训练数据中用于验证的比例
4. __训练模型明文保存__, 支持对模型进行分析和二次开发
5. 采用多线程加速, 模型训练速度快

##Change Log

__2015/05/15__: 增加__Sigmoid Matrix Factorization__功能

##Useage

运行`make`, 在output/bin目录下产出__lightmf-train__和__lightmf-test__这两个训练和测试工具.

若要针对__lightmf__进行二次开发, 可以使用output/include下面的头文件和output/lib/liblightmf.so

##lightmf-train

隐因子模型的训练数据包括有uid, iid和rating组成的三元组, 三元组之间以一个空格隔开：

__uid iid rating 附加信息__

uid和iid可以为文本id, __lightmf__内部实现了对文本id对整型id的映射. 附加信息不会影响到模型的训练.

除了训练数据路径, 第二个必须提供的参数是模型保存的路径, 该路径必须是一个目录, 用于保存每一轮迭代后的模型.

第三个必须提供的参数是__type__, 指定训练任务的类型: 1-回归问题, 2-分类问题.

除了__train__, __model__和__type__这三个参数外, 其他参数都是可选的, 包括：

1. __num\_factor__: 隐因子的数量. 隐因子数量越多, 训练速度越慢, 同时过拟合的可能性更高. 默认为__25__, 一般隐因子数量的选择范围是__20~50__
2. __sigma__: 随机初始化隐因子模型的正态分布的方差. 使用以0为均值的正态分布对隐因子模型进行初始化化是一种常见做法. 正态分布方差的选择对结果会有一定的影响. 默认值是__0.01__. 使用太小的方差会影响模型的收敛速度
3. __lambda__: L2正则化数据, 控制模型复杂度和泛化能力之间的权衡. 默认值是__0.005__
4. __max\_epoch__: 训练迭代轮数.默认值是__10__. 每一轮训练完成后, 中间结果都会保存到__model__指定的路径下
5. __alpha__: SGD的学习率. 默认值为__0.01__. 对于矩阵分解这类非凸模型来说, 学习率直接影响了模型最终是否能够收敛
6. __validate__: 验证数据的比例. 设置为n意味着1/n的训练数据将用不会用于训练, 而是单独用于验证模型的效果. validate默认值为0, 即不进行验证. 推荐的做法是, 使用一定比例的数据用于验证, 挑选出最优的参数后, 再使用最优参数和全量训练数据重新训练模型.

隐因子模型以明文的形式保存, 每一轮迭代产出的模型保存在以迭代次数编号的文件里. 

第一轮迭代后产出的模型包括以下3个文件: pass-0000, pass-0000.row, pass-0000.col, 以此类推. 
其中, pass-0000保存模型的元参数, 格式如下:

Line 1: __num\_factor__ __sigma__ __用户隐因子向量数量__ __物品隐因子向量数量__

Line 2: __用户隐因子模型保存路径__, 即pass-0000.row

Line 3: __物品隐因子模型保存路径__, 即pass-0000.col

Line 4: __全局打分平均值mu__

pass-0000\.row和pass-0000\.col保存了完整的用户/物品隐因子模型, 以行为单位, 每一行的数据使用空格分割, 字段的意义依次为: 

__用户/物品的文本id 内部整型id 偏置项 隐因子向量__

使用脚本语言(如python)可以很容易地实现对模型的二次开发

###lightmf-train命令行参数示例

    ./lightmf-train [OPTIONS]
    Options are:
     -train        (required)          Filename for training data 
     -model        (required)          Output path for model 
     -type         (required)          Output type, 1 for regression, 2 for classification
     -num_factor   (default = 25)      Number of latent factors 
     -sigma        (default = 0.01)    Initial std of normal distribution for latent factors 
     -lambda       (default = 0.005)   L2 regularizaton parameter 
     -max_epoch    (default = 10)      Max training iterations 
     -alpha        (default = 0.01)    Learning rate of SGD 
     -validate     (default = 0)       Proportion of training data for validation 
     -help                             Show this help 

##lightmf-test

预测的接口比较简单, 仅包含3个必须的参数. 测试数据的格式必须与训练数据一致.
__model__参数与训练工具的model参数略有不同, 这里的model指向的是一个元参数文件的路径.
__output__是预测的输出, 将在每一行测试数据之间加上一个预测的打分值.

###lightmf-test命令行参数示例

    ./lightmf-test [OPTIONS]
    Options are:
     -model    (required)  Latent factor model path 
     -test     (required)  Filename for test data 
     -output   (required)  Filename for output data 
     -help                 Show this help 

##Evaluation

### Movielens-1M

使用__movielens-1m__数据集评估__lightmf__的性能.
该数据集包含由6000+用户对4000+部电影的100万左右的打分数据.

随机划分得到的训练集包含__939, 809__样本, 测试集包括__60, 400__样本.
使用默认参数进行训练, 每一轮迭代的时间仅为__1秒__, stderr输出如下:

    $ ./lightmf-train -train ra.train -model model/
    [Train]: ra.train
    [Model]: model/
    [NumFactor]: 25
    [Sigma]: 0.01
    [Lambda]: 0.005
    [MaxEpoch]: 10
    [Alpha]: 0.01
    [Validate]: 0
    [INFO]: kv_dict size = 939809
    [INFO]: rows size = 6040
    [INFO]: cols size = 3682
    [INFO]: mean = 3.56966
    [INFO]: sum = 3.3548e+06
    [Iter] = 0000 [RMSE] = 0.951473 [Duration] = 0.93 Sec.
    [Iter] = 0001 [RMSE] = 0.912981 [Duration] = 1.11 Sec.
    [Iter] = 0002 [RMSE] = 0.908044 [Duration] = 1.10 Sec.
    [Iter] = 0003 [RMSE] = 0.905934 [Duration] = 1.10 Sec.
    [Iter] = 0004 [RMSE] = 0.903140 [Duration] = 1.11 Sec.
    [Iter] = 0005 [RMSE] = 0.895122 [Duration] = 1.09 Sec.
    [Iter] = 0006 [RMSE] = 0.882142 [Duration] = 1.10 Sec.
    [Iter] = 0007 [RMSE] = 0.868088 [Duration] = 1.08 Sec.
    [Iter] = 0008 [RMSE] = 0.853275 [Duration] = 1.09 Sec.
    [Iter] = 0009 [RMSE] = 0.838199 [Duration] = 1.08 Sec.

使用最后一轮的结果对测试集进行评估, RMSE为__0.889795__. stderr输出如下:

    ./lightmf-test -test ra.test -model model/0009 -output ra.test.out
    [Model]: model/0009
    [Test]: ra.test
    [Output]: ra.test.out
    RMSE on [ra.test]: 0.889795

### Movielens-20M

__movielens-20m__是由__GroupLens__最新发布的电影推荐数据集. 
该数据集包含由__138, 493__位用户对__26, 744__部电影的打分数据, 共计__20000263__份训练样本.
stderr输出如下:

    [Train]: ./ml-20m/ra.train
    [Model]: model/
    [NumFactor]: 25
    [Sigma]: 0.01
    [Lambda]: 0.005
    [MaxEpoch]: 20
    [Alpha]: 0.01
    [Validate]: 0
    [INFO]: kv_dict size = 20000263
    [INFO]: rows size = 138493
    [INFO]: cols size = 26744
    [INFO]: mean = 3.52553
    [INFO]: sum = 7.05115e+07
    [Iter] = 0000 [RMSE] = 0.884548 [Duration] = 22.62 Sec.
    [Iter] = 0001 [RMSE] = 0.856210 [Duration] = 23.51 Sec.
    [Iter] = 0002 [RMSE] = 0.827038 [Duration] = 23.38 Sec.
    [Iter] = 0003 [RMSE] = 0.798865 [Duration] = 23.39 Sec.
    [Iter] = 0004 [RMSE] = 0.775023 [Duration] = 23.35 Sec.
    [Iter] = 0005 [RMSE] = 0.755519 [Duration] = 23.33 Sec.
    [Iter] = 0006 [RMSE] = 0.739148 [Duration] = 22.88 Sec.
    [Iter] = 0007 [RMSE] = 0.726245 [Duration] = 23.30 Sec.
    [Iter] = 0008 [RMSE] = 0.716560 [Duration] = 22.94 Sec.
    [Iter] = 0009 [RMSE] = 0.709158 [Duration] = 23.07 Sec.
    [Iter] = 0010 [RMSE] = 0.703336 [Duration] = 23.20 Sec.
    [Iter] = 0011 [RMSE] = 0.698626 [Duration] = 22.90 Sec.
    [Iter] = 0012 [RMSE] = 0.694734 [Duration] = 22.80 Sec.
    [Iter] = 0013 [RMSE] = 0.691435 [Duration] = 22.96 Sec.
    [Iter] = 0014 [RMSE] = 0.688630 [Duration] = 22.99 Sec.
    [Iter] = 0015 [RMSE] = 0.686178 [Duration] = 22.97 Sec.
    [Iter] = 0016 [RMSE] = 0.684050 [Duration] = 23.20 Sec.
    [Iter] = 0017 [RMSE] = 0.682150 [Duration] = 23.19 Sec.
    [Iter] = 0018 [RMSE] = 0.680487 [Duration] = 22.72 Sec.
    [Iter] = 0019 [RMSE] = 0.678962 [Duration] = 23.14 Sec.

每一轮迭代时间仅为__23秒__左右. 
当把全量数据加载进内存后, __lightmf__的内存占用仅为__1637Mb__, 由此可见, 内存并不会成为瓶颈. 

##Todo

1. ~~实现__Sigmoid Matrix Factorization__, 增加对0-1打分数据的支持~~
2. 实现__读缓存__, 缓解大数据集带来的内存压力
