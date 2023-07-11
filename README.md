# Linux-BigDataCommand
大数据等相关启动命令
flink启动命令
/opt/module/flink/bin/flink run -s hdfs://mycluster/checkpoint/db2XppoTpodd02/1f481ba76beaa22db9f2d86d1b7bdcca/chk-1544/_metadata -t yarn-per-job 
-Djobmanager.memory.process.size=8096m -Dtaskmanager.memory.process.size=4096m -Dyarn.appmaster.vcores=5 -Dyarn.containers.vcores=1 
-Dyarn.application.queue=flink -Dyarn.application.name=db2XppoTpodd02 -c db2XppoTpodd02 /data/myjars/original-XppoTpodd02Table-1.0-SNAPSHOT.jar


--local ./sql-client.sh embedded
--yarn session提交方式  
./yarn-session.sh -Dyarn.application.queue=flink -Djobmanager.memory.process.size=10240m -Dtaskmanager.memory.process.size=4096m -d

./sql-client.sh embedded -s yarn-session
set sql-client.execution.result-mode=tableau;
set execution.checkpointing.interval=3sec;

--查看yarn session 提交后的端口，直接在刚启动的日志查看或者可从jobmanager中搜索关键词 Rest endpoint listening at TVVMON0006:33938
set execution.checkpointing.mode=EXACTLY_ONCE;
set execution.checkpointing.timeout=30 min;--  30min
set execution.checkpointing.interval=1 min ; -- 1min
set execution.checkpointing.externalized-checkpoint-retention=RETAINONCANCELLATION;
-- ExecutionConfigOptions
set table.exec.state.ttl=1 day;  -- 1 day
set table.exec.mini-batch.enabled=true; -- enable mini-batch optimization
set table.exec.mini-batch.allow-latency=1 s; -- 1s
set table.exec.mini-batch.size=1000;
set table.exec.sink.not-null-enforcer=drop;


--spark 提交方式
spark-sql \
  --master yarn \
  --queue flink \
  --conf 'spark.serializer=org.apache.spark.serializer.KryoSerializer' \
  --conf 'spark.sql.catalog.spark_catalog=org.apache.spark.sql.hudi.catalog.HoodieCatalog' \
  --conf 'spark.sql.extensions=org.apache.spark.sql.hudi.HoodieSparkSessionExtension'

hudi-cli.sh --queue flink

spark-sql --master yarn --driver-cores 1 --driver-memory 5g --num-executors 3 --executor-cores 2 --executor-memory 10G --queue spark


--刷新分区
MSCK REPAIR TABLE table_name;
展示分区
show partitions table_name
--刷新表
REFRESH TABLE default.SINK_USER_EVENT_ATTR_Latest;

create table user_behavior.user_property_5 using hudi location '/hudi_Zgio.db/user_property_5_i';

---kafka相关命令
--查看Kafka版本
find ./libs/ -name \*kafka_\* | head -1 | grep -o '\kafka[^\n]*'
 ./kf.sh start  启动Kafka
bin/kafka-server-start.sh config/server.properties >>/dev/null 2>&1 &
--查看Kafka中的所有topic
bin/kafka-topics.sh --list --bootstrap-server 10.80.29.38:9092
bin/kafka-topics.sh --list --zookeeper 10.80.59.69:2181,10.80.59.70:2181,10.80.59.71:2181
--创建一个topic
bin/kafka-topics.sh --create --topic db2_xppi_tpicw26_kafkaTest --partitions 5 --replication-factor 5 --bootstrap-server 10.80.29.38:9092
bin/kafka-topics.sh --create --topic zgiobehavior --partitions 3 --replication-factor 3 --zookeeper 10.80.59.69:2181,10.80.59.70:2181,10.80.59.71:2181
---删除topic
bin/kafka-topics.sh --bootstrap-server 10.80.29.38:9092 --delete --topic db2_xppi_tpicw26_kafkaTest
bin/kafka-topics.sh --zookeeper 10.80.59.10:2182,10.80.59.11:2182,10.80.59.12:2182 --topic sc_caputer_change_topic --delete
--查看某个主题的信息
bin/kafka-topics.sh ---bootstrap-server 10.80.29.38:9092  --describe  --topic db2_xppi_tpirk03_kafka 
bin/kafka-topics.sh --zookeeper 10.80.29.38:2181,10.80.29.39:2181,10.80.29.40:2181  --describe  --topic db2_xppi_tpirk03_kafka 
--查看所有主题的详细信息
bin/kafka-topics.sh --zookeeper 10.80.59.69:2181,10.80.59.70:2181,10.80.59.71:2181 --describe

--消费Kafka某个topic数据 后缀from-beginning 表示从最早位置，不加默认从最新位置

如果是想看数据中是否包含某个字段可以通过 grep   | grep G21773832
bin/kafka-console-consumer.sh --topic sc_caputer_change_topic_pro --bootstrap-server 10.80.9.11:9092 | grep t_gd_commodity_price
查看消费的数据且携带事件时间
bin/kafka-console-consumer.sh --topic sc_caputer_change_topic_pro --bootstrap-server 10.80.9.11:9092 --from-beginning --property print.timestamp=true --property print.timestamp.format="YYYY-MM-dd HH:mm:ss:SSS"

bin/kafka-console-consumer.sh --topic pay_zg_total --bootstrap-server 10.80.251.7:9092 --from-beginning > /data/Zgiodata/pay_zg_total

--生产数据
bin/kafka-console-producer.sh --topic AT_LEAST_ONCE_topic --broker-list  10.80.59.69:9092,10.80.59.70:9092,10.80.59.71:9092
--查看topic的每个分区的偏移量
kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list 10.80.251.7:9092,10.80.251.8:9092,10.80.251.9:9092 --topic pay_zg_total

bin/kafka-topics.sh --bootstrap-server 10.80.29.97:9092 --describe --topic db2_xppi_tpirk03_topic


docker消费Kafka
sudo docker run -it --network=host edenhill/kcat:1.7.1  -b 10.80.251.7:9092,10.80.251.8:9092,10.80.251.9:9092 -t pay_zg_total -o beginning > /data/Zgiodata/pay_statisv2



夏竟翔
impala-shell -q "SELECT * FROM zanalytics.b_user_event_attr_5 WHERE event_id =134 AND yw=202243" -B --output_delimiter="," --print_header -o /home/baoadmin/behavior_data/b_user_event_all_5_20221003.csv

db2测试环境
服务器账号密码 dbtest11/1qazXSW@
查看db2数据库的进程
db2top -d doedvlp

查看db2的cdc 进程是否正常
1. db2  2.connect to doedvlp  3.VALUES ASNCDC.ASNCDCSERVICES('status','asncdc')
cd ~
/datatest11/dbtest11/sqllib/bin/asncap capture_schema=asncdc capture_server=DOEDVLP &
db2 get dbm cfg


当编译程序时需要用到某个jar包，但是又重远程仓库下载不下来可以先百度找到这个jar包然后手动编译到本地仓库
mvn install:install-file -Dfile=./pentaho-aggdesigner-algorithm-5.1.5-jhyde.jar -DgroupId=org.pentaho -DartifactId=pentaho-aggdesigner-algorithm -Dversion=5.1.5-jhyde -Dpackaging=jar


Kafka Schema-Registry
https://chbxw.blog.csdn.net/article/details/119594987

kafka Confluent Schema Registry 简单实践
https://blog.csdn.net/qq_40788398/article/details/124816170?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-1-124816170-blog-119594987.pc_relevant_default&spm=1001.2101.3001.4242.2&utm_relevant_index=4   
Flink with Avro Confluent Kafka-Registry
https://blog.csdn.net/wuxintdrh/article/details/119796500

Confluent g w
https://docs.confluent.io/platform/current/schema-registry/index.html#sr-overview

idm cdc 
https://www.ibm.com/docs/zh/idr/11.4.0?topic=eikcopk-write-avro-records-before-after-images-in-one-row

flink 官网中文 个人制作翻译
https://www.bookstack.cn/read/flink-1.15-zh/840accb10defbcc6.md
