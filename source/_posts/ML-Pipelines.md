---
title: ML工作流（Pipelines）
date: 2018-04-08 16:28:47
tags: [工作流,Pipelines]
---

## ML工作流（Pipelines）

### ML工作流（Pipelines）中的一些概念

+ DataFrame：使用Spark SQL中的DataFrame作为数据集，它可以容纳各种数据类型。 DataFrame中的列可以是存储的文本，特征向量，真实标签和预测标签等。

+ Transformer：转换器，是一种可以将一个DataFrame转换为另一个DataFrame的算法。比如一个模型就是一个Transformer。

+ Estimator：评估器，基于算法实现了一个`fit()`方法进行`拟合`，输入一个DataFrame，产生一个Transformer。

+ PipeLine：管道将多个工作流阶段（转换器和估计器）连接在一起，形成机器学习的工作流。

+ Parameter：用来设置所有转换器和估计器的参数。

### Pipelines如何运转

一个工作流被指定为一系列的阶段，每个阶段都是Transformer或Estimator。这些阶段按顺序运行，输入的DataFrame在通过每个阶段时会进行转换。对于Transformer阶段，会在DataFrame上调用`transform()`方法。对于Estimator阶段，调用`fit()`方法来拟合生成Transformer（它将成为PipelineModel或拟合管道的一部分），并在DataFrame上调用Transformer的transform()方法。

![流水线](images/ML-Pipeline.png "流水线")

上图中，顶行表示具有三个阶段的管道。前两个（Tokenizer和HashingTF）是Transformers（蓝色），第三个（LogisticRegression）是Estimator（红色）。底行表示流经管道的数据，其中圆柱表示DataFrames。在原始DataFrame上调用`Pipeline.fit()`方法拟合，它具有原始的文本和标签。`Tokenizer.transform()`方法将原始文本拆分为单词，并向DataFrame添加一个带有单词的新列。 `HashingTF.transform()`方法将字列转换为特征向量，向这些向量添加一个新列到DataFrame。然后，由于LogisticRegression一个Estimator，Pipeline首先调用`LogisticRegression.fit()`拟合产生一个LogisticRegressionModel。如果管道有更多的Estimator，则在将DataFrame传递到下一个阶段之前，会先在DataFrame上调用LogisticRegressionModel的`transform()`方法。

PipeLine本身也是一个Estimator。因而，在工作流的`fit()`方法运行之后，它产生了一个PipelineModel，它也是一个Transformer。这个管道模型将在测试数据的时候使用。下图展示了这种用法。

![流水线模型](images/ML-PipelineModel.png "流水线模型")

在上图中，PipelineModel具有与原始Pipeline相同的阶段数，但是原始Pipeline中的所有估计器Estimators都变为变换器Transformers。当在测试数据集上调用PipelineModel的`transform()`方法时，数据按顺序通过拟合的管道。每个阶段的transform()方法更新数据集并将其传递到下一个阶段。Pipelines和PipelineModels有助于确保训练数据集和测试数据集通过相同的特征处理步骤。

### 代码示例

#### 理解Estimator，Transformer和Param

相关API ：[`Estimator`](http://spark.apache.org/docs/latest/api/python/pyspark.ml.html#pyspark.ml.Estimator)，[`Transformer`](http://spark.apache.org/docs/latest/api/python/pyspark.ml.html#pyspark.ml.Transformer)，[`Params`](http://spark.apache.org/docs/latest/api/python/pyspark.ml.html#pyspark.ml.param.Params)

```python
from pyspark.ml.linalg import Vectors
from pyspark.ml.classification import LogisticRegression

# 准备训练数据集(label, features)元组
training = spark.createDataFrame([
    (1.0, Vectors.dense([0.0, 1.1, 0.1])),
    (0.0, Vectors.dense([2.0, 1.0, -1.0])),
    (0.0, Vectors.dense([2.0, 1.3, 1.0])),
    (1.0, Vectors.dense([0.0, 1.2, -0.5]))], ["label", "features"])

# 创建一个LogisticRegression示例，也就是Estimator
lr = LogisticRegression(maxIter=10, regParam=0.01)

# 打印出所以的参数和默认值信息
print("LogisticRegression parameters:\n" + lr.explainParams() + "\n")

# 用训练数据集训练模型，这一步骤会使用到lr中的parameters
model1 = lr.fit(training)

# 现在，model1成为了一个Model（Estimator产生的transformer）
# 我们查看一下拟合过程所用到的parameters
print("Model 1 was fit using parameters: ")
print(model1.extractParamMap())

# 修改参数
paramMap = {lr.maxIter: 20}
paramMap[lr.maxIter] = 30
paramMap.update({lr.regParam: 0.1, lr.threshold: 0.55})

# 合并参数
paramMap2 = {lr.probabilityCol: "myProbability"}  # 修改输出列名
paramMapCombined = paramMap.copy()
paramMapCombined.update(paramMap2)

# 现在基于新的参数进行拟合
model2 = lr.fit(training, paramMapCombined)
print("Model 2 was fit using parameters: ")
print(model2.extractParamMap())

# 测试数据集
test = spark.createDataFrame([
    (1.0, Vectors.dense([-1.0, 1.5, 1.3])),
    (0.0, Vectors.dense([3.0, 2.0, -0.1])),
    (1.0, Vectors.dense([0.0, 2.2, -1.5]))], ["label", "features"])

# 使用Transformer.transform()对测试数据进行预测
# LogisticRegression.transform只会使用'features'列，myProbability列既是probability列，之前我们做过更改
prediction = model2.transform(test)
result = prediction.select("features", "label", "myProbability", "prediction").collect()

for row in result:
    print("features=%s, label=%s -> prob=%s, prediction=%s" % 
    (row.features, row.label, row.myProbability, row.prediction))
```

可参考[`examples/src/main/python/ml/estimator_transformer_param_example.py`](https://github.com/apache/spark/tree/v2.3.0/examples/src/main/python/ml/estimator_transformer_param_example.py)

#### Pipeline

相关API ：[`Pipeline`](http://spark.apache.org/docs/latest/api/python/pyspark.ml.html#pyspark.ml.Pipeline)

```python
from pyspark.ml import Pipeline
from pyspark.ml.classification import LogisticRegression
from pyspark.ml.feature import HashingTF, Tokenizer

# 训练数据集(id, text, label)元组.
training = spark.createDataFrame([
    (0, "a b c d e spark", 1.0),
    (1, "b d", 0.0),
    (2, "spark f g h", 1.0),
    (3, "hadoop mapreduce", 0.0)
], ["id", "text", "label"])

# 配置pipeline，连接tokenizer，hashingTF和lr
tokenizer = Tokenizer(inputCol="text", outputCol="words")
hashingTF = HashingTF(inputCol=tokenizer.getOutputCol(), outputCol="features")
lr = LogisticRegression(maxIter=10, regParam=0.001)
pipeline = Pipeline(stages=[tokenizer, hashingTF, lr])

# 拟合
model = pipeline.fit(training)

# 测试数据集(id, text)元组
test = spark.createDataFrame([
    (4, "spark i j k"),
    (5, "l m n"),
    (6, "spark hadoop spark"),
    (7, "apache hadoop")
], ["id", "text"])

# 预测结果
prediction = model.transform(test)
selected = prediction.select("id", "text", "probability", "prediction")
for row in selected.collect():
    rid, text, prob, prediction = row
    print("(%d, %s) --> prob=%s, prediction=%f" % (rid, text, str(prob), prediction))
```

可参考[`examples/src/main/python/ml/pipeline_example.py`](https://github.com/apache/spark/tree/v2.3.0/examples/src/main/python/ml/pipeline_example.py)

