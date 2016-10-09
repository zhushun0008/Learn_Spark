## Problem Solved ##
### Spark Configuration ###
1. SparkConf' object has no attribute '_get_object_id' when using sc = pyspark.SparkContext(conf)

	```
	conf = SparkConf().setAppName('test_spark_configuration')
	sc = pyspark.SparkContext(conf) 
	```
	
	* ref: [How to change SparkContext properties in Interactive PySpark session](http://stackoverflow.com/questions/32362783/how-to-change-sparkcontext-properties-in-interactive-pyspark-session)
	* fixed
	
		```
		sc = pyspark.SparkContext(conf=conf) 
		```

### Optimization
#### Resource
* [ Spark1.0.x入门指南 ](http://www.cnblogs.com/Scott007/p/3849677.html)
* [ 阅读Spark排错与优化](http://blog.csdn.net/lsshlsw/article/details/49155087)
* [ Spark On YARN内存分配](http://blog.javachen.com/2015/06/09/memory-in-spark-on-yarn.html)
* [ 理解RDD](http://blog.csdn.net/bluejoe2000/article/details/41415087)
* [ Spark源码系列讲解](http://www.uml.org.cn/wenzhang/artsearch.asp?curpage=1)
* [Problems solved of running spark](https://github.com/AllenFang/spark-overflow/blob/master/README.md)

#### Problems Solved
1. Uncaught fatal error from thread [sparkDriver-akka.remote.default-remote-dispatcher-8] shutting down ActorSystem [sparkDriver] java.lang.OutOfMemoryError: Java heap space
	* [ parkDriver throwing java.lang.OutOfMemoryError: Java heap space](https://mail-archives.apache.org/mod_mbox/spark-user/201604.mbox/%3CCA+e75uvb+E93U53RxOoxpnPOik914G8g2ed0q=esuzcqyzmu2A@mail.gmail.com%3E)
	* [ Java heap space Error while running SVMWithSGD algorithm in MLlib](http://stackoverflow.com/questions/31916017/java-heap-space-error-while-running-svmwithsgd-algorithm-in-mllib)
	* [ SparkSql OutOfMemoryError](http://apache-spark-user-list.1001560.n3.nabble.com/SparkSql-OutOfMemoryError-td17468.html)
	* [java.lang.OutOfMemoryError: Java heap space with RandomForest](https://issues.apache.org/jira/browse/SPARK-5743)
	* [ scala spark编程常见问题总结](http://blog.csdn.net/sivolin/article/details/47105655)

2. Lost task Error communicating with MapOutputTracker
	* [ Re: Error communicating with MapOutputTracker from mail-archives.apache.org](https://mail-archives.apache.org/mod_mbox/spark-user/201505.mbox/%3CCAGHU-i0L9VBxM+auAi4XDECchaLurvUPaJa_MZXc+mAq_2JjAg@mail.gmail.com%3E)
		* increase spark.akka.askTimeout, I used --conf spark.network.timeout=300 to fix the this issue.

3. org.apache.spark.shuffle.MetadataFetchFailedException: Missing an output location
	* [ Spark Shuffle FetchFailedException解决方案](http://blog.csdn.net/lsshlsw/article/details/51213610)
	* [ using MEMORY_AND_DISK from stackoverflow](http://stackoverflow.com/questions/28901123/org-apache-spark-shuffle-metadatafetchfailedexception-missing-an-output-locatio)
	* [increase spark.yarn.executor.memoryOverhead from mail-archives.apache.org](https://mail-archives.apache.org/mod_mbox/spark-user/201502.mbox/%3CCAHentsTnKrdbKaFF2oRJTM26TViGacgVr9mFbovSdLM1ikWHYQ@mail.gmail.com%3E)
	* [ job keeps failing with org.apache.spark.shuffle.MetadataFetchFailedException: Missing an output location for shuffle 1](http://mail-archives.us.apache.org/mod_mbox/spark-user/201502.mbox/%3CCAHentsTnKrdbKaFF2oRJTM26TViGacgVr9mFbovSdLM1ikWHYQ@mail.gmail.com%3E)
	* [ Memory leak](https://issues.apache.org/jira/browse/SPARK-4996)
	* [ org.apache.spark.shuffle.MetadataFetchFailedException: Missing an output location for shuffle 0 in stackoverflow](http://stackoverflow.com/questions/28901123/org-apache-spark-shuffle-metadatafetchfailedexception-missing-an-output-locatio)
	* [ Spark Shuffle FetchFailedException解决方案](http://blog.csdn.net/lsshlsw/article/details/51213610)


4. Map output statuses can still exceed spark.akka.frameSize
Use spark-submit --conf spark.akka.frameSize=200 (set 200M for frameSize)
	* [Map output statuses can still exceed spark.akka.frameSize](https://issues.apache.org/jira/browse/SPARK-5077)
	* [ Spark broadcast error: exceeds spark.akka.frameSize Consider using broadcast](http://stackoverflow.com/questions/27218472/spark-broadcast-error-exceeds-spark-akka-framesize-consider-using-broadcast)
	* [ Apache spark message understanding](http://stackoverflow.com/questions/26904619/apache-spark-message-understanding)
	* [ 设置spark.akka.frameSize不生效](http://wenda.chinahadoop.cn/question/3120#!answer_form)
	* [ Fixing Spark](http://tech.grammarly.com/blog/posts/Petabyte-Scale-Text-Processing-with-Spark.html)


5. java.io.IOException: Unable to acquire 67108864 bytes of memory
	* [Disable the tungsten execution engine](http://alvincjin.blogspot.com/2016/01/unable-to-acquire-bytes-of-memory.html)
	* [Seems to be only a issue for spark 1.5](https://issues.apache.org/jira/browse/SPARK-10309#userconsent)

6. ERROR cluster.YarnScheduler: Lost executor xxxxxx remote Rpc client disassociated
***Problem***: when join the big table with a small table(about 40,000 records) and error occurs
	* try this ```--conf spark.yarn.executor.memoryOverhead=600``` in [How to prevent Spark Executors from getting Lost when using YARN client mode?](http://stackoverflow.com/questions/31728688/how-to-prevent-spark-executors-from-getting-lost-when-using-yarn-client-mode) but not work 
	* use similar to map_side_join to fix this problem
		* use a dictionary to store the small table and broadcast it
		* use map or filter operations to process the big table with key in dictionary
		* *** In conclusion: Use map side to bypass the shuffle stage and use dictionary instead for loop to speed up the process***
	```
	def filter_map_side_join(temp_row, m):
		return m.get(temp_row[0], False)

	rdd_selected_mother_user_log_within_duation = repartition_valid_rdd_user_pin_with_log_within_duation.filter(lambda temp_row: filter_map_side_join(temp_row, select_user_dict_broadcast.value)).map(lambda one_data: one_data[1])
	```
