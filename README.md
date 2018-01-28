# Data Science Primer

## Apache Spark

| **Spark Term**                           | **Description**                          |
| ---------------------------------------- | ---------------------------------------- |
| **Spark Architecture**                   | ![spark-architecture-diagram](https://spark.apache.org/docs/latest/img/cluster-overview.png) |
| Spark Driver                             | sends work we have 1 of those.           |
| Spark Worker                             | like multiple of them, we want enough RAM memory connecting to hdfs |
| RDD                                      | Think distributed collection with good api, immutable, fault tolerant |
| `#Partitions > #Executors`               | * Optimize: At least many partitions (shard) to data like num of executors<br />* So that each executor can work in parallel on some partition of the data |
| Immutability                             | FirstRDD points to SecondRDD points to ThirdRDD (transformations) |
| Fault Tolerance                          | If spark-worker fails work restarts on another spark-worker by spark-driver |
| Partitioning                             | Data is broken into partitions 3         |
| Worker node 1* Executor                  |                                          |
| Transformations VS Actions               | * Transformations [Lazy]: map, flatMap, filter, groupBy, mapValues, ...<br />* Actions: take, count, collect, reduce, top |
| **Skeleton Project**                     |                                          |
| dependency                               | * "org.apache.spark" %% "spark-core"<br />* create fatJar in assembly |
| **Use Spark**                            |                                          |
| create SparkContext                      | * `val sparkConf = new SparkConf().setAppName("somename").set("spark.io.compress.codec", "lzf")` <br />* `val sc = new SparkContext(conf)`<br />* `main` standard java main method to run your code. |
| submit it                                | `spark-submit com.mypackage.MySparkApp —master local[*] my-fat-jar-without-spark.jar` <br />// spark-submit is part of spark we downloaded<br />// [*] use cpu cores as many as you have<br />// To deploy to cluster you are going to have [many more params](https://spark.apache.org/docs/latest/submitting-applications.html) |
| **Spark API**                            |                                          |
| `spark-shell`                            | Explore the api with spark shell, already has spark context `sc` |
| `sc.makeRDD(List(1,2))`                  |                                          |
| `rdd.map`                                | `val myRdd = rdd.map(_ * 2)` // returns RDD |
| `myRdd.collect()`                        | `Array(2,4)`                             |
| `rdd.cache()`                            | So that if you have multiple `.action()` like `.collect()` data won't be referched for the transformations rdd again. |
| **Local dev**                            |                                          |
| Spark Dependency                         | <script src="https://gist.github.com/tomer-ben-david/9068a65e798e226a979765c359ae8b31.js"></script> |
| Main                                     | `object SparkExample extends App<br />override def main(args: Array[String]): Unit = {` |
| Spark conf and context                   | `val conf = new SparkConf().setAppName("parse my book").setMaster("local[*]")<br />val sc = new SparkContext(conf)` |
| Load text from http                      | <script src="https://gist.github.com/tomer-ben-david/d94bcd0060b8a9acb04857903d71cd81.js"></script> |
| Reduce by top words                      | <script src="https://gist.github.com/tomer-ben-david/5662e1e709a74e7a69cb7d942c822fbc.js"></script> |
| **Performance**                          |                                          |
| *ByKey                                   | reduceByKey much more efficient than reduce, no shuffle. *byKey. |
| Driver Node RAM                          | Result < RAM on driver machine, result returned through driver Otherwise out of memory. |
| minimize shuffles                        | The less you have the better spark will utilize data locallity and memory |
| **Beginning NLP**                        |                                          |
| High dimentionality                      | text analytics is a high dimentionality problem, it's not infrequent to have 100K features. |
|                                          | Watch this great series here: https://www.youtube.com/playlist?list=PL8eNk_zTBST8olxIRFoo0YeXxEOkYdoxi |
| Preprocessing                            |                                          |
| Step 1: Observe                          | Observe the data, see what it is do some plotting |
| Step 2: PreprocFilter                    | Casing, Puncutation, Numbers, Stop words, Symbols, Stemming |
| Step 3: Tokenization                     | Tokenization                             |
| Step 4: DFM                              | Document Frequency Matrix, high dimention. with DFM we had high dimention we have the below line for each text message, so we need to do diemntion reduction.![doc freq matrix](https://tinyurl.com/docfreqmatrix) |
| Step 5: Create Model                     | Use `cross validation`                   |
| Step 5.1: Decision Tree Model based on Bag of words | Simplest model, based on bag of words, word count |
| Step 5.2: Tune: Normalize doc length     | Normalize based on doc length it's obvious that the longer the document is the higher the count of words it would have for each of the above, we need to normalize.  `TF frequnecy(word) / sum(frequency(all words)) so we normalize to the proportioin of the count of word in doc relative to other words` |
| Step 5.3: Tune: TF-IDF                   | Penalize words that appear cross corpus. `IDF(t) = log (N number of docs / count(t))` so if term appears on all docs log( 44 / 44) == 0 so we don't take into account that word. |
| Step 6: ngram                            | We counted for single words 1-gram but can we count cobination of words? ngram is not combination of words it's just consequetive words 2gram each 2 consequetive words.  bigram more than 2X matrix size.  The curse of dimentionality problem.  This creates a very sparse matrix. |
| Step 7: Random Forest                    | Bigram can reduce the accuracy! so we would need to combine them with Random Forests. |
| Step 7.1: LSA - Latent Semantic Analysis | Money, Loan, … => collapse to => Depth! Matrix Factoriation : Feature reduction, based on dot product of similar docs, reduce them, based on SVD - singular value decomposition - decompose a matrix - break it down into smaller chunks, reduce the huge size of our ngram sparse matrix.  Which will allow us to use Step 7 random forsest otherwise would take too much time.  Geometrically how close are two vectors (two rows each row is a vector), dot product gives an estimation of how close two vectors are, it's less precise than cosine correlation but it's part of it.<br />LSA - collapses together the term-document and document-term (rows and columsn) and treats them together collapse both to higher order construct. |
| Step 8: Random Forest                    | Now that we collapsed the matrix we run a much better algorithm random forest and improe results by 2% . |
| Step 9: Specifity/Sensitivity            | Decide if you prefer sensitivity or specifity and do feature engineering to prefer accuracy in one of them.  Example add feature $textLength we saw from visual plot that long emais are spam.  Repeat the model creation.  Feel free to also look if your accuracy and specifity and sensitivity are all gong up. |
| Step 10: Feature engineering VarImpPlot  | check which features are important.  In many cases the feature you engineer as a human like the textLength are far far more important and predict much much well than the discovered features, this is where you as a data scientist add value - feature engineering.<br />Among the engineered features: #1 spam similarity.  #2 Text langth  howeverif some feature is like too much good i predicting it can be an indicatin of overfitting. |
| Step 10.1: Adding cosine similarity engineered feature |                                          |
| Step 11: Test - Test Data                | You need to make sure your columns in test data are same in size and meaning as in train data.  R "dfm_select" does exactly that. |
|                                          |                                          |
| Dot product                              | dotproduct(doc 1 closer - doc 2) > dotproduct(doc 3 farther doc4) |
| Transended features                      | features such as text length are transendent probably, meaning it's a good feature because over time people use :) ad other smilies with trends but feature of text length is pretty much correct over time. |
| Confusion matrix                         | confusionMatrix(realLabels, predictedLabels) => table columns: actual: ham/spam rows: what was predicted ham/spam, in this case we want to reduce false positive. |
| Cosine Similarity                        | the angle between the two vectors. values [0,1] 0.9 does not mean 90% similar, however 0.9 vs 0.7 means 20% more similar. orks well also in high dimentions.  we then create a new feature of cosine similarity with the mean of all spam similarities.  <br />. https://www.youtube.com/watch?v=7cwBhWYHgsA&t=75s![cosine similarity](https://tinyurl.com/cosinesimilarity1) |
| False Positive/Negative                  | Do me a favour first dfine what negative class is and what positive class is and only then talk about false positive and false negative. |
| Accuracy metric                          | `(TP + TN) / (TP + TN + FP + FN)`        |
| Semsitivity metric                       | (TP) / (TP + FN) (correct ham)           |
| Speficity metric                         | TN / (TP + FN) (correct spam)            |
| SVD no free lunch                        | 1. compute intensitve.  2. reduced factorized matrixes are approximations of origina.  3. project new data into the computed matrix. |
| Truncated SVD                            | truncate(svd) => svd.output.take(top 300) // top n |
| bag of words —> TFIDF —> SVD             | each matrix transormation improves quality of prediction.  without SVD we cannot do random forests as it would take long time. |
| Vectors Dot product on rows              | an estimation of how close two vectors are, it's less precise than cosine correlation but it's part of it.  DOT product of all docs (rows) is X * X transpose. |
| Dot product on columns terms             | how close is each term one to another loan, money, … (if appears in similar docs) |
| Term collapse                            | So terms that are close in meaning, we can collapse to reduce matrix size |
| Cleanup                                  | Remove whitespaces, split by new lines, ... |
| Remove header                            | `val noHeader = rdd.filter(!_.contains("something from first line"))` |
| Clean Text                               | tokenize, remove whitespaces, filter empty strings, wors to lower case |
| Load text                                | `scala.io.Source.fromURL("http://www.gutenberg.org/files/2701/2701-0.txt").mkString` |
| Spark reads with list                    | `mobyDickText.split("\n")`               |
| to RDD                                   | `sc.parallelize(mobyDickLines)`          |
| Tokenize                                 | `mobyDickRDD.flatMap(_.split("[^0-9a-zA-Z]"))` |
| Remove empty                             | `.filter(!_.isEmpty)`                    |
| Lower case                               | `.map(_.toLowerCase())`                  |
| Compute                                  | Word count                               |
| map 1 for each word                      | `.map((_, 1))`                           |
| Count for each word                      | `.reduceByKey(_ + _)`                    |
| Take top 10                              | `.takeOrdered(10)(Ordering[Int].reverse.on(_._2))` |
| Print                                    | `.foreach(println)`                      |
| Document Frequency Matrix == Bag of words (order not preserved) | Rows: datums (sms..), Columns: token (after tokenization), cell: count(datum, token), de fator standard for classification. ![doc freq matrix](https://tinyurl.com/docfreqmatrix) (ref: https://youtu.be/Y7385dGRNLM)<br />Order not preserved: Bag of Words<br />ngram: you add back the word ordering<br />Especially useful for classification: spam, .. |
|                                          |                                          |
| **Data Frame**                           |                                          |
|                                          | DataFrame == Table == Matrix             |
| R Dataframe                              | ` pt_data <- data.frame(subject_name, temperature, flu_status,  gender, blood, symptoms, stringsAsFactors = FALSE)` |
| Word count example                       | <script src="https://gist.github.com/barkhorn/c419cfd9ba450bbaa868ed4bfea067b0.js"></script> |
| Spark Session                            | `val spark = org.apache.spark.sql.SparkSession.builder().appName("someapp").getOrCreate()` |
| Data Frame                               | <script src="https://gist.github.com/tomer-ben-david/7c3495ae903bb083b290ec9a69bdaffe.js"></script> |
| Show DF                                  | `df.show(); df.printSchema()`            |
| case classes                             | instead of referring to columns with "some column_name" refer to them with case classes |
| nested json explode                      | The *explode()* function creates a new row for each element in the given map column: `explode(col("Data.close")).as("word")` |
| parse json example                       | <script src="https://gist.github.com/tomer-ben-david/6c2cbae6af7ba1846db804e873e9dcef.js"></script> |
| **Test Spark**                           |                                          |
| SparkSuite                               | <script src="https://gist.github.com/tomer-ben-david/7531e451c62f10addb3c997f5b2d125e.js"></script> |
| **Data Science Terms**                   |                                          |
| Nearest Neightbouts KNN                  | `KNN(unlabeledData): labelOf(nearest neightbours)` .  Classification algorithm, used for OCR, movie recommendation. |
| Binary Classification                    | The task of predicting a binary label. E.g., is an email spam or not spam? Should I show this ad to this user or not? Will it rain tomorrowor not? This section demonstrates algorithms for making these types of predictions. |
| HyperLogLog                              | on machine 1 for each user => hll1.add(user), on machine 2: for each user hll2.add(user);  hll1.unify(hll2) .  hll.size() will return how many users estimation with low memory. |
| Features in practice                     | look at your tabular data.  For example table of stocks, we want to find which are similar, let's say we are already told which are similar, we want to find a feature that cause them to be similar or different, in that field fieldx, we expect the similar stocks to have diff close to zero and for different stocks to see this field as close to 1 [AAS 38] |
| Colaborative Filtering                   | Recommendation system without access to specific features of users/films just to whether user a liked movie a, without the features of the movies or users. |
| Latent factor                            | Explain observed inteactions between large number of users and items through relativly small number of unobserved underlying reasons, like people that bought math books also buy mozart discs, so it's a kind of taste. |
| Matrix Factorization                     | ex. rows users, y purchased something or listened to song. we have one large matrix.  mostly going to be 0 - sparse.  Can be described by smaller matrix multiplication, as sparse matrix can be smaller multiplied matrix, we don't need all the waste.  [AAS Figure 3.1] AKA *matrix completion algorithms* However no perfect solution, makes sense. |
| Alternating Least Squares                | Estimate the matrix factorization solution.  in Spark MLIB ALS. |
| **RStudio**                              |                                          |
| read text file                           | `mylog <- readLines("./reputation") # => R read load text file` |
| **pipe**                                 | `%>%`                                    |
| **dplyr**                                | source datacamp                          |
| Introduction                             | select columns `select("column1", "column2", ...)`, `filter` rows, `arrange` rows, change columns, add columns, summary statistics |
|                                          |                                          |
| **Sparklyr SparkR**                      | Source datacamp                          |
| connect to spark cluster                 | `spark_conn <- spark_connect(master = "local"); spark_disconnect(sc = spark_conn)` |
| copy data to spark cluster               | `track_metadata_tbl <- copy_to(spark_conn, track_metadata, overwrite = TRUE)` |
| show tables                              | `src_tbls(spark_conn)`                   |
| link to table in spark                   | `track_metadata_tbl <- tbl(spark_conn, "track_metadata")` |
| print 5 linked table                     | `print(x = track_metadata_tbl, n = 5, width = Inf)` |
| Examine linked table structure summary   | `slimpse(track_metadata_tbl); str(track_metadata_tbl)` |
|                                          | glimpse()                                |
| **BigData**                              |                                          |
| Columnar storage                         | A file is a column or section of file, instead of a row, imagine a column with country name, this means your compression of this column is much more effective therefore columnar storages tend to be much more effective.  Also some queries require a single column so faster. |
| Parquest                                 | An implementation columnar storage, a file is a column (or section in file) |


| Topic                     | HOWTO                                    |
| ------------------------- | ---------------------------------------- |
| **Study Resources**       |                                          |
| General Big Data          | <a target="_blank"  href="https://www.amazon.com/gp/product/1946383481/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1946383481&linkCode=as2&tag=planetizer0c-20&linkId=58766618ae5432f218a7c8db17c0d4e5"><img border="0" src="//ws-na.amazon-adsystem.com/widgets/q?_encoding=UTF8&MarketPlace=US&ASIN=1946383481&ServiceVersion=20070822&ID=AsinImage&WS=1&Format=_SL250_&tag=planetizer0c-20" ></a><img src="//ir-na.amazon-adsystem.com/e/ir?t=planetizer0c-20&l=am2&o=1&a=1946383481" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" /> |
| Apache Spark              | <a target="_blank"  href="https://www.amazon.com/gp/product/0672338513/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0672338513&linkCode=as2&tag=planetizer0c-20&linkId=9e3d739aad73dc61faee301221c4a8b9"><img border="0" src="//ws-na.amazon-adsystem.com/widgets/q?_encoding=UTF8&MarketPlace=US&ASIN=0672338513&ServiceVersion=20070822&ID=AsinImage&WS=1&Format=_SL250_&tag=planetizer0c-20" ></a><img src="//ir-na.amazon-adsystem.com/e/ir?t=planetizer0c-20&l=am2&o=1&a=0672338513" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" /> |
| Amazon EMR                | https://aws.amazon.com/emr/getting-started/ |
| Data Science RND Workflow | https://alexioannides.com/2016/08/16/building-a-data-science-platform-for-rd-part-1-setting-up-aws/<br />Zeppelin, AWS, Spark |

* AAS - Advanced Analytics with Spark





|      |      |
| ---- | ---- |
|      |      |
|      |      |
|      |      |

