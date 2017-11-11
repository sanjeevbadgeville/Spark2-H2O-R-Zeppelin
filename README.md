# Spark2-H2O-R-Zeppelin
A stack for data mining using Spark2, H2O, R and Zeppelin running on Cloudera Hadoop Distribution

# Spark2 Setup

## Hadoop Version (tested with CDH5.8)
$hadoop version
  Hadoop 2.6.0-cdh5.11.0

## Download Spark 
http://spark.apache.org/downloads.html
Download Spark: spark-2.2.0-bin-hadoop2.6.tgz
wget http://apache.mirrors.ionfish.org/spark/spark-2.2.0/spark-2.2.0-bin-hadoop2.6.tgz

## Extract Spark2 downloaded file
- sudo mkdir /opt/spark
- sudo chown -R cloudera:cloudera /opt/spark
- cp /mnt/working/spark-2.2.0-bin-hadoop2.6.tgz /opt/spark
- tar xvzf /opt/spark/spark-2.2.0-bin-hadoop2.6.tgz
-  ln -s /opt/spark/spark-2.2.0-bin-hadoop2.6.tgz /opt/spark/current

## Update conf/spark-env.sh
- SPARK_HOME=/opt/spark/current
- HADOOP_CONF_DIR=/etc/hive/conf

## Update conf/spark-defaults.conf
- spark.master                       yarn
- spark.yarn.jars                    hdfs://localhost:8020/user/cloudera/spark-2.2.0-bin-hadoop2.6/*

## Create HDFS folder /user/cloudera (if not present)
- sudo -u hdfs hdfs dfs -mkdir /user/cloudera
- sudo -u hdfs hdfs dfs -chown -R cloudera /user/cloudera
- hdfs dfs -mkdir spark-2.2.0-bin-hadoop2.6
- hdfs dfs -copyFromLocal jars/*  spark-2.2.0-bin-hadoop2.6

## Test Spark2 Installation
- $ ./bin/run-example SparkPi 10 --master yarn
- $ ./bin/spark-shell --master yarn
- $ ./bin/pyspark
Yarn cluster mode
- $ ./bin/spark-submit --class org.apache.spark.examples.SparkPi     --master yarn     --deploy-mode cluster     --driver-memory 4g     --executor-memory 2g     --executor-cores 1     --queue thequeue     examples/jars/spark-examples*.jar     10
Yarn client mode
- ./bin/spark-submit --class org.apache.spark.examples.SparkPi     --master yarn     --deploy-mode client     --driver-memory 4g     --executor-memory 2g     --executor-cores 1     --queue thequeue     examples/jars/spark-examples*.jar     10

# H2O Sparkling-Water Setup

## Download Sparkling-Water
http://h2o-release.s3.amazonaws.com/sparkling-water/rel-2.0/0/sparkling-water-2.0.0.zip

## Update sparkling-env.sh
- SPARK_HOME=/opt/spark/current
- MASTER=yarn

## Copy Sparkling-Water Fat jar
- cp /opt/sparkling-water/current/assembly/build/libs/sparkling-water-assembly_2.11-2.0.0-all.jar  /opt/spark/current/jars

## Test Sparkling-Water Installation
- /opt/spark/current/bin/spark-submit --master=yarn --class water.SparklingWaterDriver --conf "spark.yarn.am.extraJavaOptions=-XX:MaxPermSize=384m -Dhdp.version=current"  --driver-memory=8G --num-executors=3 --executor-memory=3G --conf "spark.executor.extraClassPath=-XX:MaxPermSize=384m -Dhdp.version=current"  /opt/spark/current/jars/sparkling-water-assembly_2.11-2.2.2-all.jar

# Install R

$ sudo yum install R
$ sudo yum install libxml2-devel
$ sudo yum install libcurl-devel

- install.packages('knitr', repos="http://cran.rstudio.com/",dependencies = TRUE)
- install.packages('data.table', repos="http://cran.rstudio.com/",dependencies = TRUE)
- install.packages('curl', repos="http://cran.rstudio.com/",dependencies=TRUE)
- install.packages("httr", repos="http://cran.rstudio.com/", dependencies=TRUE)
- install.packages("plotly", repos="http://cran.rstudio.com/", dependencies=TRUE)
- install.packages("devtools", repos="http://cran.rstudio.com/", dependencies=TRUE)
- devtools::install_github("ropensci/plotly")
- devtools::install_github('ramnathv/rCharts')

# Apache Zeppelin

## Download Zeppelin 
https://zeppelin.apache.org/download.html
zeppelin-0.7.3-bin-all.tgz
wget http://mirror.reverse.net/pub/apache/zeppelin/zeppelin-0.7.3/zeppelin-0.7.3-bin-all.tgz

## Update /opt/zeppelin/current/conf/zeppelin-env.sh
- export MASTER=yarn
- export SPARK_HOME=/opt/spark/current
- export SPARK_APP_NAME=zeppelin-cdh
- export HADOOP_CONF_DIR=/etc/hive/conf
- export SPARK_SUBMIT_OPTIONS="--jars /opt/spark/current/jars/sparkling-water-assembly_2.11-2.2.2-all.jar"

## Run Zeppelin with Spark2, Sparkling-water, and R
- /opt/zeppelin/current/bin/zeppelin.sh -Pspark-2.0

## Test Zeppelin Installation
http://localhost:8080/#/

- %spark
- import org.apache.spark.sql._
- val sqlContext = new SQLContext(sc)
- import sqlContext.implicits._
- import org.apache.spark.h2o._
- val h2oContext = H2OContext.getOrCreate(sc) 
- import h2oContext._ 

- val df: DataFrame = sc.parallelize(1 to 1000, 100).map(v => IntHolder(Some(v))).toDF
- val hf = h2oContext.asH2OFrame(df)
- val newRdd = h2oContext.asDataFrame(hf)(sqlContext)

# Oracle Access

## Use the ojdbc7.jar in the lib folder as it has the file defaultConnectionProperties.properties file updated with oracle.jdbc.timezoneAsRegion=false

## Use the Postgres Interpreter
- postgresql.driver.name	oracle.jdbc.driver.OracleDriver
- postgresql.max.result	1000
- postgresql.password	    [PASSWORD]
- postgresql.url	        jdbc:oracle:thin:@[HOST_IP]:[HOST_PORT]:[SID]
- postgresql.user	        [USERNAME]

Dependencies
/opt/zeppelin/current/lib/ojdbc7.jar

## Add Oracle jdbc driver to Spark Interpreter

Dependencies
/opt/zeppelin/current/lib/ojdbc7.jar

## Test Zeppelin to access Oracle using %psql
- %psql
- SELECT * FROM DUAL

## Test Zeppelin to access Oracle using %spark
- %spark
- sc.version
-- val pdf = sqlContext.load("jdbc", Map("url" -> "jdbc:oracle:thin:[USERNAME]/[PASSWORD]@[HOST_IP]:[HOST_PORT]:[SID]", "driver" -> "oracle.jdbc.driver.OracleDriver", "dbtable" -> "dual") )
- pdf.printSchema()
- pdf.registerTempTable("pdf")

- %sql SELECT count(*) FROM pdf

# Vertica

# Use the vertica-jdbc-8.0.0-1.jar in the lib folder

## Use JDBC Interpreter
- default.driver	com.vertica.jdbc.Driver
- default.password	[PASSWD]
- default.url	jdbc:vertica://[HOST]:[HOST_PORT]/[DB]?user=[USERNAME]&password=[PASSWD]
- default.user	[USERNAME]

Dependencies
/opt/zeppelin/current/lib/vertica-jdbc-8.0.0-1.jar

## Test Zeppelin to access Vertica using %jdbc
- %jdbc
- SELECT count(*) FROM [SCHEMA].[TABLE]

## Test Zeppelin to access Vertica using %spark
- %spark
- sc.version 
- val pdfv = sqlContext.load("jdbc", Map("url" -> "jdbc:vertica://[HOST]:[PORT]/[DB]?user=[USERNAME]&password=[PASSWD]", "driver" -> "com.vertica.jdbc.Driver", "dbtable" -> "[SCHEMA].[TABLE]", "fetchsize" -> "100") )
- pdfv.printSchema()
- pdfv.registerTempTable("pdfv")

- %sql SELECT * FROM pdfv

## Impala
/opt/zeppelin/current/lib/ojdbc7.jar	
/tmp/toSratch/2.5.36.1056/ImpalaJDBC41.jar	
/opt/zeppelin/current/lib/vertica-jdbc-8.0.0-1.jar	
/tmp/toSratch/2.5.36.1056/commons-logging-1.1.1.jar	
/tmp/toSratch/2.5.36.1056/hive_metastore.jar	
/tmp/toSratch/2.5.36.1056/hive_service.jar	
/tmp/toSratch/2.5.36.1056/httpclient-4.1.3.jar	
/tmp/toSratch/2.5.36.1056/httpcore-4.1.3.jar	
/tmp/toSratch/2.5.36.1056/libfb303-0.9.0.jar	
/tmp/toSratch/2.5.36.1056/libthrift-0.9.0.jar	
/tmp/toSratch/2.5.36.1056/log4j-1.2.14.jar	
/tmp/toSratch/2.5.36.1056/ql.jar	
/tmp/toSratch/2.5.36.1056/slf4j-api-1.5.11.jar	
/tmp/toSratch/2.5.36.1056/slf4j-log4j12-1.5.11.jar	
/tmp/toSratch/2.5.36.1056/TCLIServiceClient.jar	
/tmp/toSratch/2.5.36.1056/zookeeper-3.4.6.jar

# References
https://www.linkedin.com/pulse/running-spark-2xx-cloudera-hadoop-distro-cdh-deenar-toraskar-cfa
https://github.com/h2oai/sparkling-water/blob/master/DEVEL.md#SparklingWaterZeppelin
http://www.cloudera.com/documentation/enterprise/5-8-x/topics/cdh_ig_running_spark_on_yarn.html
- find IPAddress: docker inspect [container_id] | grep IPAddress
- sudo iptables -t nat -A DOCKER -p tcp --dport 8080 -j DNAT --to-destination [container_ip]:8080
- sudo iptables -t nat -A POSTROUTING -s [container_ip] -j MASQUERADE -p tcp --dport 8080 -d [container_ip]
Redhat 6.5 if yum install R fails
"wget http://mirror.centos.org/centos/6/os/x86_64/Packages/lapack-devel-3.2.1-4.el6.x86_64.rpm
wget http://mirror.centos.org/centos/6/os/x86_64/Packages/blas-devel-3.2.1-4.el6.x86_64.rpm
wget http://mirror.centos.org/centos/6/os/x86_64/Packages/texinfo-tex-4.13a-8.el6.x86_64.rpm
wget http://vault.centos.org/6.2/updates/x86_64/Packages/libicu-devel-4.2.1-9.1.el6_2.x86_64.rpm
sudo yum localinstall *.rpm


