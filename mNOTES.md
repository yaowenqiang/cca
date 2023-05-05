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

