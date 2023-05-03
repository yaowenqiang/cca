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

### Running queries

> sqoop  eval  --connect jdbc:mysql://localhost:3306/retail_db --username root --password cloudera  --query "select * from orders limit 10"

> sqoop  eval  --connect jdbc:mysql://localhost:3306/retail_db --username root --password cloudera  --query "insert into  orders values(100000000, '2023-07-25 00:00:00.0', 1000, 'CLOSED')"
