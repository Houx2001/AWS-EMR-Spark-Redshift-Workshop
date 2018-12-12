# AWS-EMR-Spark-Redshift-Workshop
AWS EMR Spark Redshift Workshop

使用 Spark 创建集群
启动安装了 Spark 的集群

Open the Amazon EMR console at https://console.amazonaws.cn/elasticmapreduce/.

选择 Create cluster 以使用 Quick Create。

对于 Software Configuration，选择 Amazon 发行版 emr-5.14.0 或更高版本。

对于 Select Applications，选择 All Applications 或 Spark。

根据需要选择其他选项，然后选择 Create cluster。

访问 Spark 外壳
Spark 外壳基于 Scala REPL (读取-求值-输出-循环)。它让您能够以交互方式创建 Spark 程序并将工作提交到框架。您可以通过 SSH 连接主节点并调用 spark-shell,从而访问 Spark 外壳。

使用 SSH 连接主节点
您可以使用 SSH 连接到 Amazon EMR 主节点，以运行交互式查询，检查日志文件，提交 Linux 命令等。

Mac/Linux
打开终端窗口。在 Mac OS X 上，选择“应用程序”>“实用工具”>“终端”。在其他 Linux 发行版上，终端通常位于“应用程序”>“附件”>“终端”。
要建立与主节点的连接，请键入以下命令。请将 ~/你的密钥.pem 替换为用于启动集群的私有密钥文件 (.pem) 的位置和文件名。
ssh -i ~/你的密钥.pem hadoop@你的主节点DNS
键入 yes 以取消安全警告。

Windows
从以下位置将 PuTTY.exe 下载到您的计算机：
http：//www.chiark.greenend.org.uk/~sgtatham/putty/download.html
启动 PuTTY。
在 Session (会话) 列表中，单击 Category (类别)。
在“主机名”字段中，键入 hadoop@你的主节点DNS
在“类别”列表中，依次展开“连接”>“SSH”，然后单击“身份验证”。
对于用于身份验证的私有密钥文件，单击“浏览”，然后选择用于启动集群的私有密钥文件 (你的密钥.ppk)。
单击打开。
单击“是”以取消安全警告。

默认情况下，Spark 外壳创建其自己的 SparkContext 对象 (称作 sc)。如果 REPL 中需要，您可以使用此上下文。sqlContext 也可在此外壳中使用，它是一种 HiveContext。

将数据源文件上传到S3上

Open the Amazon S3 console at https://console.amazonaws.cn/s3/.

创建 sparksample bucket

点击上传，将people.json文件上传到S3

打开SSH主节点，执行下列命令进入spark-shell界面:
./bin/spark-shell --jars 

在scala>环境下依次执行下列代码：

import org.apache.spark.sql.Row

import org.apache.spark.sql.SparkSession

import org.apache.spark.sql.types._

val spark = SparkSession
      .builder()
      .appName("Spark SQL basic example")
      .config("spark.some.config.option", "some-value")
      .getOrCreate()

import spark.implicits._

val df = spark.read.json("s3a://sparksample/people.json")

// Displays the content of the DataFrame to stdout
df.show()
    // +----+-------+
    // | age|   name|
    // +----+-------+
    // |null|Michael|
    // |  30|   Andy|
    // |  19| Justin|
    // +----+-------+
    
import spark.implicits._
  
df.printSchema()
    // root
    // |-- age: long (nullable = true)
    // |-- name: string (nullable = true)

// Select only the "name" column
df.select("name").show()
    // +-------+
    // |   name|
    // +-------+
    // |Michael|
    // |   Andy|
    // | Justin|
    // +-------+

 // Select everybody, but increment the age by 1
 df.select($"name", $"age" + 1).show()
    // +-------+---------+
    // |   name|(age + 1)|
    // +-------+---------+
    // |Michael|     null|
    // |   Andy|       31|
    // | Justin|       20|
    // +-------+---------+

df.filter($"age" > 21).show()
    // +---+----+
    // |age|name|
    // +---+----+
    // | 30|Andy|
    // +---+----+

// Count people by age
df.groupBy("age").count().show()
    // +----+-----+
    // | age|count|
    // +----+-----+
    // |  19|    1|
    // |null|    1|
    // |  30|    1|
    // +----+-----+

// Register the DataFrame as a SQL temporary view
df.createOrReplaceTempView("people")

val sqlDF = spark.sql("SELECT * FROM people")

sqlDF.show()
    // +----+-------+
    // | age|   name|
    // +----+-------+
    // |null|Michael|
    // |  30|   Andy|
    // |  19| Justin|
    // +----+-------+

 // Register the DataFrame as a global temporary view
 df.createGlobalTempView("people")

 // Global temporary view is tied to a system preserved database `global_temp`
 spark.sql("SELECT * FROM global_temp.people").show()
    // +----+-------+
    // | age|   name|
    // +----+-------+
    // |null|Michael|
    // |  30|   Andy|
    // |  19| Justin|
    // +----+-------+

// Global temporary view is cross-session
spark.newSession().sql("SELECT * FROM global_temp.people").show()
    // +----+-------+
    // | age|   name|
    // +----+-------+
    // |null|Michael|
    // |  30|   Andy|
    // |  19| Justin|
    // +----+-------+
  
 


