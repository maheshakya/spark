---
layout: global
title: Feature Extraction, Transformation, and Selection - SparkML
displayTitle: <a href="ml-guide.html">ML</a> - Features
---

This section covers algorithms for working with features, roughly divided into these groups:

* Extraction: Extracting features from "raw" data
* Transformation: Scaling, converting, or modifying features
* Selection: Selecting a subset from a larger set of features

**Table of Contents**

* This will become a table of contents (this text will be scraped).
{:toc}


# Feature Extractors

## Hashing Term-Frequency (HashingTF)

`HashingTF` is a `Transformer` which takes sets of terms (e.g., `String` terms can be sets of words) and converts those sets into fixed-length feature vectors.
The algorithm combines [Term Frequency (TF)](http://en.wikipedia.org/wiki/Tf%E2%80%93idf) counts with the [hashing trick](http://en.wikipedia.org/wiki/Feature_hashing) for dimensionality reduction.  Please refer to the [MLlib user guide on TF-IDF](mllib-feature-extraction.html#tf-idf) for more details on Term-Frequency.

HashingTF is implemented in
[HashingTF](api/scala/index.html#org.apache.spark.ml.feature.HashingTF).
In the following code segment, we start with a set of sentences.  We split each sentence into words using `Tokenizer`.  For each sentence (bag of words), we hash it into a feature vector.  This feature vector could then be passed to a learning algorithm.

<div class="codetabs">
<div data-lang="scala" markdown="1">
{% highlight scala %}
import org.apache.spark.ml.feature.{HashingTF, Tokenizer}

val sentenceDataFrame = sqlContext.createDataFrame(Seq(
  (0, "Hi I heard about Spark"),
  (0, "I wish Java could use case classes"),
  (1, "Logistic regression models are neat")
)).toDF("label", "sentence")
val tokenizer = new Tokenizer().setInputCol("sentence").setOutputCol("words")
val wordsDataFrame = tokenizer.transform(sentenceDataFrame)
val hashingTF = new HashingTF().setInputCol("words").setOutputCol("features").setNumFeatures(20)
val featurized = hashingTF.transform(wordsDataFrame)
featurized.select("features", "label").take(3).foreach(println)
{% endhighlight %}
</div>

<div data-lang="java" markdown="1">
{% highlight java %}
import com.google.common.collect.Lists;

import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.ml.feature.HashingTF;
import org.apache.spark.ml.feature.Tokenizer;
import org.apache.spark.mllib.linalg.Vector;
import org.apache.spark.sql.DataFrame;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.RowFactory;
import org.apache.spark.sql.types.DataTypes;
import org.apache.spark.sql.types.Metadata;
import org.apache.spark.sql.types.StructField;
import org.apache.spark.sql.types.StructType;

JavaRDD<Row> jrdd = jsc.parallelize(Lists.newArrayList(
  RowFactory.create(0, "Hi I heard about Spark"),
  RowFactory.create(0, "I wish Java could use case classes"),
  RowFactory.create(1, "Logistic regression models are neat")
));
StructType schema = new StructType(new StructField[]{
  new StructField("label", DataTypes.DoubleType, false, Metadata.empty()),
  new StructField("sentence", DataTypes.StringType, false, Metadata.empty())
});
DataFrame sentenceDataFrame = sqlContext.createDataFrame(jrdd, schema);
Tokenizer tokenizer = new Tokenizer().setInputCol("sentence").setOutputCol("words");
DataFrame wordsDataFrame = tokenizer.transform(sentenceDataFrame);
int numFeatures = 20;
HashingTF hashingTF = new HashingTF()
  .setInputCol("words")
  .setOutputCol("features")
  .setNumFeatures(numFeatures);
DataFrame featurized = hashingTF.transform(wordsDataFrame);
for (Row r : featurized.select("features", "label").take(3)) {
  Vector features = r.getAs(0);
  Double label = r.getDouble(1);
  System.out.println(features);
}
{% endhighlight %}
</div>

<div data-lang="python" markdown="1">
{% highlight python %}
from pyspark.ml.feature import HashingTF, Tokenizer

sentenceDataFrame = sqlContext.createDataFrame([
  (0, "Hi I heard about Spark"),
  (0, "I wish Java could use case classes"),
  (1, "Logistic regression models are neat")
], ["label", "sentence"])
tokenizer = Tokenizer(inputCol="sentence", outputCol="words")
wordsDataFrame = tokenizer.transform(sentenceDataFrame)
hashingTF = HashingTF(inputCol="words", outputCol="features", numFeatures=20)
featurized = hashingTF.transform(wordsDataFrame)
for features_label in featurized.select("features", "label").take(3):
  print features_label
{% endhighlight %}
</div>
</div>


# Feature Transformers

## Tokenizer

[Tokenization](http://en.wikipedia.org/wiki/Lexical_analysis#Tokenization) is the process of taking text (such as a sentence) and breaking it into individual terms (usually words).  A simple [Tokenizer](api/scala/index.html#org.apache.spark.ml.feature.Tokenizer) class provides this functionality.  The example below shows how to split sentences into sequences of words.

Note: A more advanced tokenizer is provided via [RegexTokenizer](api/scala/index.html#org.apache.spark.ml.feature.RegexTokenizer).

<div class="codetabs">
<div data-lang="scala" markdown="1">
{% highlight scala %}
import org.apache.spark.ml.feature.Tokenizer

val sentenceDataFrame = sqlContext.createDataFrame(Seq(
  (0, "Hi I heard about Spark"),
  (0, "I wish Java could use case classes"),
  (1, "Logistic regression models are neat")
)).toDF("label", "sentence")
val tokenizer = new Tokenizer().setInputCol("sentence").setOutputCol("words")
val wordsDataFrame = tokenizer.transform(sentenceDataFrame)
wordsDataFrame.select("words", "label").take(3).foreach(println)
{% endhighlight %}
</div>

<div data-lang="java" markdown="1">
{% highlight java %}
import com.google.common.collect.Lists;

import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.ml.feature.Tokenizer;
import org.apache.spark.mllib.linalg.Vector;
import org.apache.spark.sql.DataFrame;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.RowFactory;
import org.apache.spark.sql.types.DataTypes;
import org.apache.spark.sql.types.Metadata;
import org.apache.spark.sql.types.StructField;
import org.apache.spark.sql.types.StructType;

JavaRDD<Row> jrdd = jsc.parallelize(Lists.newArrayList(
  RowFactory.create(0, "Hi I heard about Spark"),
  RowFactory.create(0, "I wish Java could use case classes"),
  RowFactory.create(1, "Logistic regression models are neat")
));
StructType schema = new StructType(new StructField[]{
  new StructField("label", DataTypes.DoubleType, false, Metadata.empty()),
  new StructField("sentence", DataTypes.StringType, false, Metadata.empty())
});
DataFrame sentenceDataFrame = sqlContext.createDataFrame(jrdd, schema);
Tokenizer tokenizer = new Tokenizer().setInputCol("sentence").setOutputCol("words");
DataFrame wordsDataFrame = tokenizer.transform(sentenceDataFrame);
for (Row r : wordsDataFrame.select("words", "label").take(3)) {
  java.util.List<String> words = r.getList(0);
  for (String word : words) System.out.print(word + " ");
  System.out.println();
}
{% endhighlight %}
</div>

<div data-lang="python" markdown="1">
{% highlight python %}
from pyspark.ml.feature import Tokenizer

sentenceDataFrame = sqlContext.createDataFrame([
  (0, "Hi I heard about Spark"),
  (0, "I wish Java could use case classes"),
  (1, "Logistic regression models are neat")
], ["label", "sentence"])
tokenizer = Tokenizer(inputCol="sentence", outputCol="words")
wordsDataFrame = tokenizer.transform(sentenceDataFrame)
for words_label in wordsDataFrame.select("words", "label").take(3):
  print words_label
{% endhighlight %}
</div>
</div>

## Binarizer

Binarization is the process of thresholding numerical features to binary features. As some probabilistic estimators make assumption that the input data is distributed according to [Bernoulli distribution](http://en.wikipedia.org/wiki/Bernoulli_distribution), a binarizer is useful for pre-processing the input data with continuous numerical features.

A simple [Binarizer](api/scala/index.html#org.apache.spark.ml.feature.Binarizer) class provides this functionality. Besides the common parameters of `inputCol` and `outputCol`, `Binarizer` has the parameter `threshold` used for binarizing continuous numerical features. The features greater than the threshold, will be binarized to 1.0. The features equal to or less than the threshold, will be binarized to 0.0. The example below shows how to binarize numerical features.

<div class="codetabs">
<div data-lang="scala" markdown="1">
{% highlight scala %}
import org.apache.spark.ml.feature.Binarizer
import org.apache.spark.sql.DataFrame

val data = Array(
  (0, 0.1),
  (1, 0.8),
  (2, 0.2)
)
val dataFrame: DataFrame = sqlContext.createDataFrame(data).toDF("label", "feature")

val binarizer: Binarizer = new Binarizer()
  .setInputCol("feature")
  .setOutputCol("binarized_feature")
  .setThreshold(0.5)

val binarizedDataFrame = binarizer.transform(dataFrame)
val binarizedFeatures = binarizedDataFrame.select("binarized_feature")
binarizedFeatures.collect().foreach(println)
{% endhighlight %}
</div>

<div data-lang="java" markdown="1">
{% highlight java %}
import com.google.common.collect.Lists;

import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.ml.feature.Binarizer;
import org.apache.spark.sql.DataFrame;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.RowFactory;
import org.apache.spark.sql.types.DataTypes;
import org.apache.spark.sql.types.Metadata;
import org.apache.spark.sql.types.StructField;
import org.apache.spark.sql.types.StructType;

JavaRDD<Row> jrdd = jsc.parallelize(Lists.newArrayList(
  RowFactory.create(0, 0.1),
  RowFactory.create(1, 0.8),
  RowFactory.create(2, 0.2)
));
StructType schema = new StructType(new StructField[]{
  new StructField("label", DataTypes.DoubleType, false, Metadata.empty()),
  new StructField("feature", DataTypes.DoubleType, false, Metadata.empty())
});
DataFrame continuousDataFrame = jsql.createDataFrame(jrdd, schema);
Binarizer binarizer = new Binarizer()
  .setInputCol("feature")
  .setOutputCol("binarized_feature")
  .setThreshold(0.5);
DataFrame binarizedDataFrame = binarizer.transform(continuousDataFrame);
DataFrame binarizedFeatures = binarizedDataFrame.select("binarized_feature");
for (Row r : binarizedFeatures.collect()) {
  Double binarized_value = r.getDouble(0);
  System.out.println(binarized_value);
}
{% endhighlight %}
</div>

<div data-lang="python" markdown="1">
{% highlight python %}
from pyspark.ml.feature import Binarizer

continuousDataFrame = sqlContext.createDataFrame([
  (0, 0.1),
  (1, 0.8),
  (2, 0.2)
], ["label", "feature"])
binarizer = Binarizer(threshold=0.5, inputCol="feature", outputCol="binarized_feature")
binarizedDataFrame = binarizer.transform(continuousDataFrame)
binarizedFeatures = binarizedDataFrame.select("binarized_feature")
for binarized_feature, in binarizedFeatures.collect():
  print binarized_feature
{% endhighlight %}
</div>
</div>

## PolynomialExpansion

[Polynomial expansion](http://en.wikipedia.org/wiki/Polynomial_expansion) is the process of expanding your features into a polynomial space, which is formulated by an n-degree combination of original dimensions. A [PolynomialExpansion](api/scala/index.html#org.apache.spark.ml.feature.PolynomialExpansion) class provides this functionality.  The example below shows how to expand your features into a 3-degree polynomial space.

<div class="codetabs">
<div data-lang="scala" markdown="1">
{% highlight scala %}
import org.apache.spark.ml.feature.PolynomialExpansion
import org.apache.spark.mllib.linalg.Vectors

val data = Array(
  Vectors.dense(-2.0, 2.3),
  Vectors.dense(0.0, 0.0),
  Vectors.dense(0.6, -1.1)
)
val df = sqlContext.createDataFrame(data.map(Tuple1.apply)).toDF("features")
val polynomialExpansion = new PolynomialExpansion()
  .setInputCol("features")
  .setOutputCol("polyFeatures")
  .setDegree(3)
val polyDF = polynomialExpansion.transform(df)
polyDF.select("polyFeatures").take(3).foreach(println)
{% endhighlight %}
</div>

<div data-lang="java" markdown="1">
{% highlight java %}
import com.google.common.collect.Lists;

import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.mllib.linalg.Vector;
import org.apache.spark.mllib.linalg.VectorUDT;
import org.apache.spark.mllib.linalg.Vectors;
import org.apache.spark.sql.DataFrame;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.RowFactory;
import org.apache.spark.sql.SQLContext;
import org.apache.spark.sql.types.Metadata;
import org.apache.spark.sql.types.StructField;
import org.apache.spark.sql.types.StructType;

JavaSparkContext jsc = ...
SQLContext jsql = ...
PolynomialExpansion polyExpansion = new PolynomialExpansion()
  .setInputCol("features")
  .setOutputCol("polyFeatures")
  .setDegree(3);
JavaRDD<Row> data = jsc.parallelize(Lists.newArrayList(
  RowFactory.create(Vectors.dense(-2.0, 2.3)),
  RowFactory.create(Vectors.dense(0.0, 0.0)),
  RowFactory.create(Vectors.dense(0.6, -1.1))
));
StructType schema = new StructType(new StructField[] {
  new StructField("features", new VectorUDT(), false, Metadata.empty()),
});
DataFrame df = jsql.createDataFrame(data, schema);
DataFrame polyDF = polyExpansion.transform(df);
Row[] row = polyDF.select("polyFeatures").take(3);
for (Row r : row) {
  System.out.println(r.get(0));
}
{% endhighlight %}
</div>

<div data-lang="python" markdown="1">
{% highlight python %}
from pyspark.ml.feature import PolynomialExpansion
from pyspark.mllib.linalg import Vectors

df = sqlContext.createDataFrame(
  [(Vectors.dense([-2.0, 2.3]), ),
  (Vectors.dense([0.0, 0.0]), ),
  (Vectors.dense([0.6, -1.1]), )],
  ["features"])
px = PolynomialExpansion(degree=2, inputCol="features", outputCol="polyFeatures")
polyDF = px.transform(df)
for expanded in polyDF.select("polyFeatures").take(3):
  print(expanded)
{% endhighlight %}
</div>
</div>

# Feature Selectors

