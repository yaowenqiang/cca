# CCA 175


> https://labs.itversity.com
> brew install coursier/formulas/coursier && cs setup
> https://www.cloudera.com/about/training/certification/cdhhdp-certification/cca-spark.html


## HDFS quick overview

/etc/hadoop/conf/core-site.xml

> fs.defaultFS

/etc/hadoop/conf/hdfs-site.xml

+ dfs.blocksize
+ dfs.replication

> hadoop fs copyFromLocal /data/crime /usr/username/.

> hadoop fs -du -s -h /user/cloudera/
> hdfs fsck /usr/cloudera/ -files -blocks -locations

## YARN quick overview

> /etc/hadoop/conf/yarn-site.xml

+ yarn.resourceManager.address
+ yarn.resourceManager.webapp.address

### Spark default settings

> /etc/spark/conf/spark-env.sh

+ Number of executors - 2
+ Meomry - 1 GB

Quite offten we under utilize resources, Understanding memory settings throughly and then then mapping them with data size we are tring to process we can accelerate the execution of our jobs


### data sets

> https://github.com/dgadiraju/data
> git clone https://github.com/dgadiraju/data.git

## Sqoop

> https://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html
> sqoop help eval

> quickstart vm default username ad password is root and cloudera

### list databases

> sqoop list-databases  --connect jdbc:mysql://localhost:3306 --username root --password cloudera
> sqoop-list-databases  --connect jdbc:mysql://localhost:3306 --username root --password cloudera

> sqoop-list-databases  \
    --connect jdbc:mysql://localhost:3306 \
    --username root \
    --password cloudera

> sqoop-list-databases  \
    --connect jdbc:mysql://localhost:3306 \
    --username root \
    -P


> sqoop-list-databases  \
    --connect jdbc:mysql://localhost:3306 \
    --username root \
    --password-file filepath

### List tables


> sqoop list-tables  \
    --connect jdbc:mysql://localhost:3306/retail_db \
	--username root \
	--password cloudera 

### Running queries

> sqoop  eval  --connect jdbc:mysql://localhost:3306/retail_db --username root --password cloudera  --query "select * from orders limit 10"

> sqoop  eval  --connect jdbc:mysql://localhost:3306/retail_db --username root --password cloudera  --query "insert into  orders values(100000000, '2023-07-25 00:00:00.0', 1000, 'CLOSED')"


### Importing data

> sqoop import  \
    --connect jdbc:mysql://localhost:3306/retail_db \
	--username root \
	--password cloudera \
	--table order_items \
	--target-dir '/user/cloudera/sqoop_import/retail_db/order_item'
	--num-mappers 1


>  hadoop fs -tail /user/cloudera/filepath

> -m Use n map tasks to import in parallel

### execution life cycle

### dest parget dir if exists
> sqoop import  \
    --connect jdbc:mysql://localhost:3306/retail_db \
	--username root \
	--password cloudera \
	--table order_items \
	--target-dir '/user/cloudera/sqoop_import/retail_db/order_item'
	--num-mappers 1 \
	--delete-target-dir
	
### append data to target dir

> sqoop import  \
    --connect jdbc:mysql://localhost:3306/retail_db \
	--username root \
	--password cloudera \
	--table order_items \
	--target-dir '/user/cloudera/sqoop_import/retail_db/order_item'
	--num-mappers 1 \
	--append

### Using split by
> mysql> create table order_items_nopk  as select * from order_items;

#### things to remember for split by

+ column should be indexed
+ columns in the field should be sparse
+ also often it should be sequence generated or evenly incremented
+ it should not have null values


> sqoop import  \
    --connect jdbc:mysql://localhost:3306/retail_db \
	--username root \
	--password cloudera \
	--table order_items_nopk \
	--target-dir '/user/cloudera/sqoop_import/retail_db/order_item'
	--split-by order_item_order_id

#### split by text fieldso

> sqoop import  \
    -Dorg.apache.sqoop.splitter.allow_text_splitter=true \
    --connect jdbc:mysql://localhost:3306/retail_db \
    --username root \
    --password cloudera \
    --table orders \
    --warehouse-dir '/user/cloudera/sqoop_import/retail_db/' \
    --split-by order_status \

### auto reset to one mapper	

>  sqoop import  \
    -Dorg.apache.sqoop.splitter.allow_text_splitter=true \
    --connect jdbc:mysql://localhost:3306/retail_db \
    --username root \
    --password cloudera \
    --table order_items_nopk \
    --warehouse-dir '/user/cloudera/sqoop_import/retail_db/' \
    --autoreset-to-one-mapper	\
    --delete-target-dir	

### Different file formats (TODO)

> --as-textfile
> --as-avrodatafile	
> --as-sequencefile	
> --as-parquetfile	

### Using compressions

#### Compression algorithms

+ Gzip
+ Deflate
+ Snappy
+ Others

> sqoop import  \
    -Dorg.apache.sqoop.splitter.allow_text_splitter=true \
    --connect jdbc:mysql://localhost:3306/retail_db \
    --username root \
    --password cloudera \
    --table order_items_nopk \
    --warehouse-dir '/user/cloudera/sqoop_import/retail_db/' \
    --autoreset-to-one-mapper	\
    --delete-target-dir	 \
    --as-textfile \
    --compress
> default is gzip, and file format is .gz

> hadoop fs -get /user/cloudera/sqoop_import/retail_db/order_items_nopk  order_items_nopk
> gunzip part*.gz



> sqoop import  \
    -Dorg.apache.sqoop.splitter.allow_text_splitter=true \
    --connect jdbc:mysql://localhost:3306/retail_db \
    --username root \
    --password cloudera \
    --table order_items_nopk \
    --warehouse-dir '/user/cloudera/sqoop_import/retail_db/' \
    --autoreset-to-one-mapper	\
    --delete-target-dir	 \
    --as-textfile \
    --compress
    --compression-codec org.apache.hadoop.io.compress.SnappyCodec

> file format is .snappy

#### hadoop codec config 
> /etc/hadoop/conf/core-set.xml
> io.compression.codec

### Boundary Query

>    sqoop import  \
    --connect jdbc:mysql://localhost:3306/retail_db \
    --username root \
    --password cloudera \
    --table order_items \
    --target-dir '/user/cloudera/sqoop_import/retail_db/order_item' \
    --delete-target-dir	 \
    --boundary-query "select min(order_item_id), max(order_item_id) from order_items where order_item_id >=999999"

>   sqoop import  \
    --connect jdbc:mysql://localhost:3306/retail_db \
    --username root \
    --password cloudera \
    --table order_items \
    --target-dir '/user/cloudera/sqoop_import/retail_db/order_item' \
    --delete-target-dir	 \
    --boundary-query "select 1000000,1100000"

### Transformations and filtering

> Table and/or columns is mutually exclusive with query
> for query split-by is mandatory if num-mappers is greater then 1
> query should have a placehoulder \$CONDITIONS

>   sqoop import  \
    --connect jdbc:mysql://localhost:3306/retail_db \
    --username root \
    --password cloudera \
    --table order_items \
    --target-dir '/user/cloudera/sqoop_import/retail_db/order_item' \
    --delete-target-dir	 \
    --num-mappers 2 \
    --columns order_item_order_id,order_item_id,order_item_subtotal


> select o.*, sum(oi.order_item_subtotal) order_revenu 
from orders o join order_items oi
on o.order_id = oi.order_item_order_id
group by o.order_id, o.order_date, o.order_customer_id, o.order_status

#### Using filters

>     sqoop import  \
    --connect jdbc:mysql://localhost:3306/retail_db \
    --username root \
    --password cloudera \
    --target-dir '/user/cloudera/sqoop_import/retail_db/order_item' \
    --delete-target-dir	 \
    --split-by order_id \
    --query 'select o.*, sum(oi.order_item_subtotal) order_revenu from orders o join order_items oi on o.order_id = oi.order_item_order_id and $CONDITIONS group by o.order_id, o.order_date, o.order_customer_id, o.order_status '


### Delimiters 
### Handling nulls

>     sqoop import  \
    --connect jdbc:mysql://localhost:3306/retail_db \
    --username root \
    --password cloudera \
    --table order_items \
    --target-dir '/user/cloudera/sqoop_import/retail_db/order_item' \
    --delete-target-dir	 \
    --num-mappers 2 \
    --null-non-string -1 \
    --fields-terminated-by "\t"
    --line-terminated-by ":"

> search for sqoop ascii null delimiter

>     sqoop import  \
    --connect jdbc:mysql://localhost:3306/retail_db \
    --username root \
    --password cloudera \
    --table order_items \
    --target-dir '/user/cloudera/sqoop_import/retail_db/order_item' \
    --delete-target-dir	 \
    --num-mappers 2 \
    --null-non-string -1 \
    --fields-terminated-by "\000"
    --line-terminated-by ":"
### Incremental Loads

### Simple Hive Import

>   sqoop import  \
    --connect jdbc:mysql://localhost:3306/retail_db \
    --username root \
    --password cloudera \
    --table order_items \
    --delete-target-dir	 \
    --hive-import \
    --hive-database cloudera_sqoop_import \
    --hive-table order_items \
    --num-mappers 2


### Managing tables while performing Hive import

#### overwrite table 

>   sqoop import  \
    --connect jdbc:mysql://localhost:3306/retail_db \
    --username root \
    --password cloudera \
    --table order_items \
    --delete-target-dir	 \
    --hive-import \
    --hive-database cloudera_sqoop_import \
    --hive-table order_items \
    --hive-overwrite \
    --num-mappers 2

#### faile if table exists

>    sqoop import  \
    --connect jdbc:mysql://localhost:3306/retail_db \
    --username root \
    --password cloudera \
    --table order_items \
    --delete-target-dir	 \
    --hive-import \
    --hive-database cloudera_sqoop_import \
    --hive-table order_items \
    --hive-overwrite \
    --create-hive-table \
    --num-mappers 2

> files will be put into hdfs  /user/user/username/table_name/ dir , if import into the table successfully, the dir above will be deleted


>     sqoop import  \
    --connect jdbc:mysql://localhost:3306/retail_db \
    --username root \
    --password cloudera \
    --table order_items \
    --hive-import \
    --hive-database cloudera_sqoop_import \
    --hive-table order_items \
    --hive-overwrite \
    --num-mappers 2

This will be fail because the target dir hdfs://quickstart.cloudera:8020/user/cloudera/order_items already exists 

### Import all tables

+ import-all-tables
+ Limitations
  + --warehouse-dir is mandatory
  + Better to use auto-reset-to-one-mappe
  + Cannot specify many arguments such as --query --cols, --where
  + Incremental import is not possible


>     sqoop import-all-tables  \
    --connect jdbc:mysql://localhost:3306/retail_db \
    --username root \
    --password cloudera \
    --autoreset-to-one-mapper

The command above only import all tables into /user/cloudera/tablename dir


### Sqoop export

Typeical life Cycle

+ Data ingesting (using Sqoop - one of the approaches)
+ Process data
+ Visualize the processed data
  + Connect BI/Visualization tools to HDFS directly
  + Port the processed data into a database
+ sqoop export will help you in porting the process data to a database


run in hive

> create table daily_reverue as
    select order_date, sum(order_item_subtotal) daily_reverue
    from orders join order_items on
    order_id = order_item_order_id
    where order_date like '2013-07%'
    group by order_date

run in mysql

create database retail_export;
use retail_export;

create table daily_revenue (
	order_date varchar(30),
	revenue float
);



>     sqoop export  \
    --connect jdbc:mysql://localhost:3306/retail_export \
    --username root \
    --password cloudera \
	--table daily_revenue \
	--input-fields-terminated-by '\001' \
	--num-maps 1
	--export-dir hdfs://quickstart.cloudera:8020/user/hive/warehouse/cloudera_sqoop_import.db/daily_reverue


### Export Behavior
### Column Mapping



create table daily_revenue_demo (
	revenue float,
	order_date varchar(30),
	description varchar(255)
);


    sqoop export  \
    --connect jdbc:mysql://localhost:3306/retail_export \
    --username root \
    --password cloudera \
    --columns order_date,revenue \
	--table daily_revenue_demo \
	--export-dir hdfs://quickstart.cloudera:8020/user/hive/warehouse/cloudera_sqoop_import.db/daily_reverue \
	--input-fields-terminated-by '\001' 

if a column in the mysql table is not null and not specified in the hive table ,the export will fail


### Update and Upsert(TODO)

### Stage Tables

create table daily_revenue_stage (
	order_date varchar(30) primary key,
	revenue float
);

    sqoop export  \
    --connect jdbc:mysql://localhost:3306/retail_export \
    --username root \
    --password cloudera \
	--table daily_revenue \
	--staging-table daily_revenue_stage \
	--clean-staging-table \
	--export-dir hdfs://quickstart.cloudera:8020/user/hive/warehouse/cloudera_sqoop_import.db/daily_reverue \
	--input-fields-terminated-by '\001' 


> if the export is successed, the stage table will be truncated.

## Transform, Stage and Store
### Install spark on windows

### Documentations

> search for spark programming guide 1.6.3

> https://spark.apache.org/docs/1.6.3/programming-guide.html

### Initializing the job

> spark-shell --master yarn --conf spark.ui.port=12654

scala > sc
scala > sqlContext

spark-shell = scala + spark dependencies + implicit variables sc and sqlContext


> spark-shell --master yarn --conf spark.ui.port=12654 

default  value config

> /etc/spark/conf/spark-defaults.conf
> /etc/spark/conf/spark-env.sh



spark-shell --master yarn  \
	--conf spark.ui.port=12654 \
   --num-executors 1 \
   --executor-memory 512M


sc.stop
import  org.apache.spark.{SparkConf, SpartContext}
val config = new SparkConf().setAppName("daily Revernue").setMaster("yarn-client
val sc = new SparkContext(conf)
sc.getConf.getAll.foreach(println)
> sc.getConf.getAll.foreach(println)

### Create RDD using data from HDFS

+ RRD (Relilient Distribute Dataset)
  + in-memory
  + Distributed
  + Resilient
+ Reading files from HDFS
+ Reading files from local file system and create RDD
+ Quick overview of Transformations and Actions
+ DAG and lazy evaluation
+ Previewing the data using Actions

scala > val l = (1 to 1000).toList

scala > val orders = sc.textFile('/public/retail-db/orders')
scala > others.first()
scala > others.take(10)
scala > val productRaw = scala.io.Source.fromFile("/data/retail_db/productes/part-0000").getLines.toList
scala > val productRDD = sc.parallelize(productRaw)
scala > productRDD.take(10)
scala > val l_rdd = sc.parallelize(l)
scala > l = (1 to 20000).toList

> DAG (Directed Acyclic Graph)

> orders.takeSample(true, 100)
> orders.takeSample(true, 100).foreach(println)
> orders.collect
> orders.takeOrders

### Standard Transformations

+ Filtering(horizontal and vertical)
+ String Manipulation(Scala)
+ Row level transformation
+ Joins
+ Aggregation
+ Sorting
+ Ranking
+ Set Operations

#### String Manipulation(Scala)
> val s = orders.first
> val a = str.split(",")
> a(0)
> val orderId = a()).toInt
> a(1).contains("2023")
> valorderDate = a(1)
> orerDate.substring(0, 10)
> orerDate.replace('-', '/')
> orerDate.replace("07", "July')
> orerDate.indexOf("07")
> orerDate.indexOf("07", 2)
> orerDate.length
> orerDate.isEmpty

### Row level transformations

> str.split(',')(1).substring(0, 10).replace("-", "").toInt

val orderDates = orders.map((str: String) =>  {
    str.split(',')(1).substring(0, 10).replace("-", "").toInt
})
orderDates.take(10).foraech(println)

    val ordersPairedRDD = orders.map(order => {
        val o = order.split(",")
            (o(0).toInt, o(1).substring(0, 10).replace("-","").toInt)

            })



val orderItems = sc.textFile("/user/cloudera/retail_db/order_items/")
val orderItemsPairedRDD = orderItems.map(orderItem => {
    (orderItem.split(",")(1).toInt, orderItem)
})

// flatmap

val l = List("Hello", "How are you doing", "Let us perform word count","As part of the word count program", "We will see how many items each word repeat")
val l_rdd = sc.parallelize(l)
val l_map = l_rdd.map(ele => ele.split(" "))
val l_flatmap = l_rdd.flatMap(ele => ele.split(" "))

val wordCount = l_flatmap.map(word => (word, 1)).countByKey

### filtering (horizontal and vertical) (TODO)

### Joins


val orderItems = sc.textFile("/user/cloudera/retail_db/order_items/")


val ordersMap = orders.map(order => {
    (order.split(",")(0).toInt, order.split(",")(1).substring(0, 10))

        })

val orderItemsMap = orderItems.map(orderItem => {
    val oi = orderItem.split(",")
        (oi(1).toInt, oi(4).toFloat)

        })

val ordersJoin = ordersMap.join(orderItemsMap)


//Get all the orders which do not have corresponding entries in order items

val ordersMap = orders.map(order => {
    (order.split(",")(0).toInt, order)
})

val orderItemsMap = orderItems.map(orderItem => {
val oi = orderItem.split(",")
    (oi(1).toInt, oi)
})

val ordersLeftOuterJoin = ordersMap.leftOuterJoin(orderItemsMap)
