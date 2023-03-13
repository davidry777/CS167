# Lab 8

## Update

If you see the following messages or errors when you compile and run your code, please use the new template in this page. There were a few bugs in the original template caused by unremoved codes from the same lab in the past year.

```text
Usage <input file>
<input file> path to a CSV file input
```

```text
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: 1
 at edu.ucr.cs.cs167.merlin.App$.main(App.scala:28)
 at edu.ucr.cs.cs167.merlin.App.main(App.scala)
```

---

## Objectives

* Run machine learning algorithms on big data using MLlib.
* Try out the different components of MLlib, i.e., transformers, estimators, evaluators, and validators.
* Build a machine learning Pipeline that describes the entire ML model creation process.

---

## Prerequisites

* Download the following file made available by [Kaggle](https://www.kaggle.com), [Sentiment data](sentiment.csv).

---

## Lab Work

### I. Project Setup (20 minutes) (In-home)

In this lab, we will use Scala as a programming language.

1. Follow the project setup part in [Lab 6](../Lab6/CS167-Lab6.md) with the same Spark version.
2. Add the following dependency to your `pom.xml` file.

    ```xml
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-mllib_${scala.compat.version}</artifactId>
        <version>${spark.version}</version>
    </dependency>
    ```

    Don't forget to add the following line into `<properties></properties>`.

    ```xml
    <spark.version>3.2.1</spark.version>
    ```

---

### II. Load Input (20 minutes) (In-home)

First, we will load the input file that we will use throughout the lab.
The input file name is passed as the first command line argument `args(0)`.

1. Add the following code stub and make sure it compiles.

    ```scala
    import org.apache.spark.SparkConf
    import org.apache.spark.ml.classification.{LogisticRegression, LogisticRegressionModel}
    import org.apache.spark.ml.evaluation.BinaryClassificationEvaluator
    import org.apache.spark.ml.feature.{HashingTF, StringIndexer, Tokenizer}
    import org.apache.spark.ml.param.ParamMap
    import org.apache.spark.ml.tuning.{ParamGridBuilder, TrainValidationSplit, TrainValidationSplitModel}
    import org.apache.spark.ml.{Pipeline, PipelineModel}
    import org.apache.spark.sql.{DataFrame, Dataset, Row, SparkSession}

    object App {

      def main(args : Array[String]) {
        if (args.length != 1) {
          println("Usage <input file>")
          println("  - <input file> path to a CSV file input")
          sys.exit(0)
        }
        val inputfile = args(0)
        val conf = new SparkConf
        if (!conf.contains("spark.master"))
          conf.setMaster("local[*]")
        println(s"Using Spark master '${conf.get("spark.master")}'")

        val spark = SparkSession
          .builder()
          .appName("CS167 Lab8")
          .config(conf)
          .getOrCreate()

        val t1 = System.nanoTime
        try {
          // TODO process the sentiment data
          val sentimentData: DataFrame = // ...

          val tokenzier = // ...

          val hashingTF = // ...

          val stringIndexer = // ...

          val logisticRegression = // ...

          val pipeline = // ...

          val paramGrid: Array[ParamMap] = new ParamGridBuilder()
            .addGrid(/* ... */)
            .addGrid(/* ... */)
            .build()

          val cv = new TrainValidationSplit()
            .setEstimator(pipeline)
            .setEvaluator(new BinaryClassificationEvaluator().setLabelCol("label"))
            .setEstimatorParamMaps(paramGrid)
            .setTrainRatio(0.8)
            .setParallelism(2)

          val Array(trainingData: Dataset[Row], testData: Dataset[Row]) = sentimentData.randomSplit(Array(0.8, 0.2))

          // Run cross-validation, and choose the best set of parameters.
          val logisticModel: TrainValidationSplitModel = cv.fit(trainingData)

          // val numFeatures: Int = logisticModel.bestModel.asInstanceOf[PipelineModel].stages(1).asInstanceOf[HashingTF].getNumFeatures
          // val regParam: Double = logisticModel.bestModel.asInstanceOf[PipelineModel].stages(3).asInstanceOf[LogisticRegressionModel].getRegParam
          // println(s"Number of features in the best model = $numFeatures")
          // println(s"RegParam the best model = $regParam")

          val predictions: DataFrame = logisticModel.transform(testData)
          predictions.select("text", "label", "prediction", "probability").show()

          val binaryClassificationEvaluator = new BinaryClassificationEvaluator()
            .setLabelCol("label")
            .setRawPredictionCol("prediction")

          val accuracy: Double = binaryClassificationEvaluator.evaluate(predictions)
          println(s"Accuracy of the test set is $accuracy")
            
          val t2 = System.nanoTime
          println(s"Applied sentiment analysis algorithm on input $inputfile in ${(t2 - t1) * 1E-9} seconds")
        } finally {
          spark.stop
        }
      }
    }
    ```

2. Complete all `// ...` and `/* ... */` based on the following sections.

---

### III. Logistic model on sentiment data (90 minutes)

In this part, we will build a model that estimates the sentiment (positive/negative) of textual data based on labeled data. To activate this part of the code, set the command line arguments to: `sentiment.csv`

1. Open the file `sentiment.csv` in a text editor and go through it to see some examples of the data. For your reference, the first few lines are included below.

    |                text|sentiment|
    |--------------------|---------|
    |My daughter liked it but I was aghast, that a character in this movie smokes. ...|      neg|
    |I... No words. No words can describe this. I will try for the sake of those  ... |      neg|
    |this film is basically a poor take on the old urban legend of the babysitter ... |      neg|
    |This is a terrible movie, and I'm not even sure why it's so terrible. It's ugly, ...|   neg|
    |First of all this movie is a piece of reality very well realized artistically. ...|     pos|

2. Load the file input file `sentiment.csv` as a CSV file using the following options.

    | option     | value    |
    | ---------- | -------- |
    | `"header"` | `"true"` |
    | `"quote"`  | `"\""`   |
    | `"escape"` | `"\""`   |

    Code

    ```scala
    ...
      .option("header", "true")
      .option("quote", "\"")
      .option("escape", "\"")
    ...
    ```

    Hint: See [Lab 6](../Lab6/CS167-Lab6.md#iv-query-the-dataframe-using-dataframe-operators-45-minutes)

    Print the schema and the first few lines to double check that the file was loaded correctly. It should look like the following.

    ```text
    root
    |-- text: string (nullable = true)
    |-- sentiment: string (nullable = true)

    +--------------------+---------+
    |                text|sentiment|
    +--------------------+---------+
    |My daughter liked...|      neg|
    |I... No words. No...|      neg|
    |this film is basi...|      neg|
    |This is a terribl...|      neg|
    |First of all this...|      pos|
    +--------------------+---------+
    ```

3. We will build a pipeline that consists of three transformations and one estimator described below.
4. First, we use a [`Tokenizer`](http://spark.apache.org/docs/latest/ml-features.html#tokenizer), which splits the text into words, and configure it as below.

    | method | parameter |
    | - | - |
    | [`setInputCol`](https://spark.apache.org/docs/latest/api/scala/org/apache/spark/ml/feature/Tokenizer.html#setInputCol(value:String):T) | `"text"` |
    | [`setOutputCol`](https://spark.apache.org/docs/latest/api/scala/org/apache/spark/ml/feature/Tokenizer.html#setOutputCol(value:String):T) | `"words"` |

5. Second, we use a [`HashingTF`](http://spark.apache.org/docs/latest/ml-features.html#tf-idf), which maps the words in each review to a vector of TF-IDF, and configure it as below.

    | method | parameter |
    | - | - |
    | [`setInputCol`](https://spark.apache.org/docs/latest/api/scala/org/apache/spark/ml/feature/HashingTF#setInputCol(value:String):HashingTF.this.type) | `"words"` |
    | [`setOutputCol`](https://spark.apache.org/docs/latest/api/scala/org/apache/spark/ml/feature/HashingTF#setOutputCol(value:String):HashingTF.this.type) | `"features"` |

6. Third, we use a [`StringIndexer`](http://spark.apache.org/docs/latest/ml-features.html#stringindexer), which maps the two sentiments `pos` and `neg` to integer values, and configure it as below.

    | method | parameter |
    | - | - |
    | [`setInputCol`](https://spark.apache.org/docs/latest/api/scala/org/apache/spark/ml/feature/StringIndexer.html#setInputCol(value:String):StringIndexer.this.type) | `"sentiment"` |
    | [`setOutputCol`](https://spark.apache.org/docs/latest/api/scala/org/apache/spark/ml/feature/StringIndexer.html#setOutputCol(value:String):StringIndexer.this.type) | `"label"` |
    | [`setHandleInvalid`](https://spark.apache.org/docs/latest/api/scala/org/apache/spark/ml/feature/StringIndexer.html#setHandleInvalid(value:String):StringIndexer.this.type) | `"skip"` |

7. Fourth, we run the data through a [`LogisticRegression`](http://spark.apache.org/docs/latest/ml-classification-regression.html#binomial-logistic-regression) model. The default configuration is fine as it uses the `features` and `label` columns as features and label columns, respectively.

8. Create a pipeline that combines the four stages in the above order.
  
    See [`Pipeline.setStages`](https://spark.apache.org/docs/latest/api/scala/org/apache/spark/ml/Pipeline.html#setStages(value:Array[_%3C:org.apache.spark.ml.PipelineStage]):Pipeline.this.type)

9. Create a [parameter grid](https://spark.apache.org/docs/latest/api/scala/org/apache/spark/ml/tuning/ParamGridBuilder.html) that tries the following parameters.

    | parameter                     | values                       |
    | ----------------------------- | ---------------------------- |
    | `hashingTF.numFeatures`       | `Array(10, 100, 1000)`       |
    | `logisticRegression.regParam` | `Array(0.01, 0.1, 0.3, 0.8)` |

10. Working with this dataset will take more time because it is bigger and more complex. To speed up the training process, we will use a [`TrainValidationSplit`](http://spark.apache.org/docs/latest/ml-tuning.html#train-validation-split) validator. We will create and configure it as below:

    ```scala
    val cv = new TrainValidationSplit()
      .setEstimator(pipeline)
      .setEvaluator(new BinaryClassificationEvaluator())
      .setEstimatorParamMaps(paramGrid)
      .setTrainRatio(0.8)
      .setParallelism(2)
    ```

11. Split the input data into training and test sets as below.

    ```scala
    val Array(trainingData: Dataset[Row], testData: Dataset[Row]) = sentimentData.randomSplit(Array(0.8, 0.2))
    ```

12. Run through the data and get the best model as follows.

    ```scala
    val logisticModel: TrainValidationSplitModel = cv.fit(trainingData)
    ```

    Note: If you want to get the parameters used by the best model, run the following code snippet.

    ```scala
    val numFeatures: Int = logisticModel.bestModel.asInstanceOf[PipelineModel].stages(1).asInstanceOf[HashingTF].getNumFeatures
    val regParam: Double = logisticModel.bestModel.asInstanceOf[PipelineModel].stages(3).asInstanceOf[LogisticRegressionModel].getRegParam
    ```

13. Now, apply the the model on the test set and print out a few cases for demonstration.

    ```scala
    val predictions: DataFrame = logisticModel.transform(testData)
    predictions.select("text", "label", "prediction", "probability").show()
    ```

    The output should look similar to the following:

    ```text
    +--------------------+-----+----------+--------------------+
    |                text|label|prediction|         probability|
    +--------------------+-----+----------+--------------------+
    |"Air Bud 2: Golde...|  1.0|       0.0|[0.68380935570389...|
    |"Cover Girl" is t...|  0.0|       0.0|[0.72511721634649...|
    |"Dominion" is a g...|  0.0|       0.0|[0.55437404786371...|
    |"Dungeon of Harro...|  1.0|       1.0|[0.43665553614488...|
    |"Everything is Il...|  0.0|       0.0|[0.92100407854036...|
    |"Hoods" doesn't d...|  1.0|       1.0|[0.07458240427025...|
    |"I haven't laughe...|  1.0|       1.0|[0.48317031993762...|
    |"Jaded" offers a ...|  1.0|       1.0|[0.30711688918009...|
    |"One True Thing" ...|  0.0|       0.0|[0.62604014511659...|
    |"Problem Child" w...|  1.0|       1.0|[0.07038002571342...|
    |"Summer of My Ger...|  0.0|       0.0|[0.85532203845878...|
    |"Tart" is a pathe...|  1.0|       1.0|[0.47321342214712...|
    |"The Ballad of th...|  1.0|       1.0|[0.45337525076680...|
    |"The Deer Hunter'...|  0.0|       0.0|[0.57247042379411...|
    |"The Phenix City ...|  0.0|       0.0|[0.85034976542746...|
    |"The Spirit of St...|  0.0|       0.0|[0.90987968937279...|
    |"The missing star...|  0.0|       0.0|[0.51433765691704...|
    |"Three" is a seri...|  1.0|       1.0|[0.37648809982384...|
    |"Tourist Trap" is...|  0.0|       0.0|[0.61016012355390...|
    |"War is in your b...|  1.0|       1.0|[0.26858342080044...|
    +--------------------+-----+----------+--------------------+
    ```

14. Let us also calculate the accuracy of the result.

    ```scala
    val binaryClassificationEvaluator = new BinaryClassificationEvaluator()
      .setLabelCol("label")
      .setRawPredictionCol("prediction")

    val accuracy: Double = binaryClassificationEvaluator.evaluate(predictions)
    println(s"Accuracy of the test set is $accuracy")
    ```

---

### IV. Submission (15 minutes)

1. Add a `README` file with all your information. Use this [template](CS167-Lab8-README.md).
2. No need to add a run script this time. However, make sure that your code compiles with `mvn clean package` prior to submission.
3. Similar to all labs, do not include any additional files such as the compiled code, input, or output files.

Submission file format:

```console
<UCRNetID>_lab8.{tar.gz | zip}
  - src/
  - pom.xml
  - README.md
```

Requirements:

* The archive file must be either `.tar.gz` or `.zip` format.
* The archive file name must be all lower case letters. It must be underscore '\_', not hyphen '-'.
* The folder `src` and two files, `pom.xml` and `README.md`, must be the exact names.
* The folder `src` and two filesm `pom.xml` and `README.md`, must be directly in the root of the archive, do not put them inside any folder.
* Do not include any other files/folders, otherwise points will be deducted.

See how to create the archive file for submission at [here](../MakeArchive.md).

---

## Hints

Spark tends to be very chatty on the console. If you would like to reduce the amount of logs written by Spark, create a file named `log4j.properties` under `src/main/resources` and include the following configuraiton in it.

```properties
# Set root logger level to INFO and its only appender to A1.
log4j.rootLogger=ERROR, A1

# A1 is set to be a ConsoleAppender.
log4j.appender.A1=org.apache.log4j.ConsoleAppender

# A1 uses PatternLayout.
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%-4r [%t] %-5p %c %x - %m%n
```
