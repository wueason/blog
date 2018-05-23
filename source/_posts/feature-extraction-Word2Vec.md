---
title: 特征提取之词向量
date: 2018-05-22 11:34:30
tags: [spark,Word2Vec,词向量,特征提取]
---

### 特征提取-词向量

Word2Vec（词向量），计算每个单词在其给定语料库环境下的分布式词向量（Distributed Representation）。

如果词的语义相近，那么它们的词向量在向量空间中也相互接近，这使得词语的向量化建模更加精确，可以改善现有方法并提高鲁棒性。词向量已经在许多自然语言处理场景中得到应用，如：命名实体识别，消歧，标注，解析，机器翻译等。

### 代码示例

相关API ：[`Word2Vec`](http://spark.apache.org/docs/latest/api/python/pyspark.ml.html#pyspark.ml.feature.Word2Vec)

```python
from pyspark.ml.feature import Word2Vec

spark = SparkSession.builder.master("local").appName("Word2Vec").getOrCreate()

documentDF = spark.createDataFrame([
    ("Hi I heard about Spark".split(" "), ),
    ("I wish Java could use case classes".split(" "), ),
    ("Logistic regression models are neat".split(" "), )
], ["text"])

word2Vec = Word2Vec(vectorSize=3, minCount=0, inputCol="text", outputCol="result")
model = word2Vec.fit(documentDF)

result = model.transform(documentDF)
for row in result.collect():
    text, vector = row
    print("Text: [%s] => \nVector: %s\n" % (", ".join(text), str(vector)))
```

参考[文章](https://blog.csdn.net/chunyun0716/article/details/64133028)

