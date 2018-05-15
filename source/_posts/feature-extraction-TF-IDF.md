---
title: 特征提取之TF-IDF算法
date: 2018-05-14 14:41:12
tags: [spark,TF-IDF,特征提取]
---

## 特征提取之TF-IDF

TF-IDF（词频-逆文档频率）算法是一种统计方法，用以评估一字词对于一个文件集或者一个语料库中的其中一份文件的重要程度。字词的重要性随着它在文件中出现的次数成正比增加，但同时会随着它在语料库中出现的频率成反比下降。

TFIDF的主要思想是：如果某个词或短语在一篇文章中出现的频率TF高，并且在其他文章中很少出现，则认为此词或者短语具有很好的类别区分能力，适合用来分类。TF-IDF实际上就是 TF\*IDF，其中 TF（Term Frequency），表示词条在文章Document 中出现的频率；IDF（Inverse Document Frequency），其主要思想就是，如果包含某个词 Word的文档越少，则这个词的区分度就越大，也就是 IDF 越大。对于如何获取一篇文章的关键词，我们可以计算这边文章出现的所有名词的 TF-IDF，TF-IDF越大，则说明这个名词对这篇文章的区分度就越高，取 TF-IDF 值较大的几个词，就可以当做这篇文章的关键词。

在此推荐[这篇文章](http://www.ruanyifeng.com/blog/2013/03/tf-idf.html)。

计算步骤

1. 计算词频（TF）

$$
词频={某个词在文章中的出现次数 \over 文章总次数}
$$

2. 计算逆文档频率（IDF）

$$
逆文档频率 = log {语料库的文档总数 \over 文章总次数}
$$

3. 计算词频-逆文档频率（TF-IDF）

$$
词频-逆文档频率 = 词频 \* 逆文档频率
$$

#### 代码示例

相关API ：[`HashingTF`](http://spark.apache.org/docs/latest/api/python/pyspark.ml.html#pyspark.ml.feature.HashingTF)，[`IDF`](http://spark.apache.org/docs/latest/api/python/pyspark.ml.html#pyspark.ml.feature.IDF)

```python
from pyspark.ml.feature import HashingTF, IDF, Tokenizer

spark = SparkSession.builder.master("local").appName("TF-IDF").getOrCreate()

sentenceData = spark.createDataFrame([
    (0.0, "Hi I heard about Spark"),
    (0.0, "I wish Java could use case classes"),
    (1.0, "Logistic regression models are neat")
], ["label", "sentence"])

tokenizer = Tokenizer(inputCol="sentence", outputCol="words")
wordsData = tokenizer.transform(sentenceData)

hashingTF = HashingTF(inputCol="words", outputCol="rawFeatures", numFeatures=20)
featurizedData = hashingTF.transform(wordsData)

idf = IDF(inputCol="rawFeatures", outputCol="features")
idfModel = idf.fit(featurizedData)
rescaledData = idfModel.transform(featurizedData)

rescaledData.select("label", "features").show()
```

可参考[`examples/src/main/python/ml/word2vec_example.py`](https://github.com/apache/spark/tree/v2.3.0/examples/src/main/python/ml/word2vec_example.py)

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>