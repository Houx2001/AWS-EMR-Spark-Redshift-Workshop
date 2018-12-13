# AWS-EMR-Spark-Redshift-Workshop 

## 1. 使用 Spark 创建集群
启动安装了 Spark 的集群

Open the Amazon EMR console at https://console.amazonaws.cn/elasticmapreduce/.

选择 Create cluster ,使用 Quick Create。

对于 Software Configuration，选择 Amazon 发行版 emr-5.14.0 或更高版本。

对于 Select Applications，选择 All Applications 或 Spark。

根据需要选择其他选项，然后选择 Create cluster。

## 2. 访问 Spark 外壳
Spark 外壳基于 Scala REPL (读取-求值-输出-循环)。它让您能够以交互方式创建 Spark 程序并将工作提交到框架。您可以通过 SSH 连接主节点并调用 spark-shell,从而访问 Spark 外壳。

## 使用 SSH 连接主节点
您可以使用 SSH 连接到 Amazon EMR 主节点，以运行交互式查询，检查日志文件，提交 Linux 命令等。

## Mac/Linux

打开终端窗口。在 Mac OS X 上，选择“应用程序”>“实用工具”>“终端”。在其他 Linux 发行版上，终端通常位于“应用程序”>“附件”>“终端”。
要建立与主节点的连接，请键入以下命令。请将 ~/你的密钥.pem 替换为用于启动集群的私有密钥文件 (.pem) 的位置和文件名。
ssh -i ~/你的密钥.pem hadoop@你的主节点DNS
键入 yes 以取消安全警告。

## Windows

从以下位置将 PuTTY.exe 下载到您的计算机：
http：//www.chiark.greenend.org.uk/~sgtatham/putty/download.html
启动 PuTTY。
在 Session (会话) 列表中，单击 Category (类别)。
在“主机名”字段中，键入 hadoop@你的主节点DNS
在“类别”列表中，依次展开“连接”>“SSH”，然后单击“身份验证”。
对于用于身份验证的私有密钥文件，单击“浏览”，然后选择用于启动集群的私有密钥文件 (你的密钥.ppk)。
单击打开。
单击“是”以取消安全警告。

## 3. 登录到EMR主节点后，下载redshift JDBC驱动程序：
wget --no-check-certificate   https://s3.amazonaws.com/redshift-downloads/drivers/jdbc/1.2.16.1027/RedshiftJDBC42-1.2.16.1027.jar

## 4. 将数据源文件上传到S3上
Open the Amazon S3 console at https://console.amazonaws.cn/s3/.

创建 sparksample bucket

点击上传，将people.json文件上传到S3

## 5. Amazon Redshift 使用控制台创建集群
要创建、修改、调整、删除、重启和备份集群，您可以使用 Amazon Redshift console 中的 Clusters 部分。

## 快速启动集群过程
登录 AWS 管理控制台 并通过以下网址打开 Amazon Redshift 控制台：https://console.amazonaws.cn/redshift/。

在主菜单中，选择您要在其中创建集群的区域。在本教程中，选择 宁夏。

在 Amazon Redshift 控制面板上，选择 Quick launch cluster (快速启动集群)

在“Cluster specifications (集群规格)”页面上，输入下列值，然后选择 Launch cluster (启动集群)：

Node type (节点类型)：选择 dc2.large。

Number of compute nodes (计算节点数)：保留默认值 2。

Master user name (主用户名)：保留默认值 awsuser。

Master user password (主用户密码) 和 Confirm password (确认密码)：输入主用户账户的密码。

Database port (数据库端口)：接受默认值 5439。

在“Clusters”页面上，选择您刚刚启动的集群，然后查看 Cluster Status 信息。确保 Cluster Status 为 available 且 Database Health 为 healthy，然后再根据本教程的后续步骤尝试连接到数据库。

## 6. 下载Redshift客户端应用
使用 SQL Workbench/J 连接到您的Refshift集群

## 7. 安装 SQL Workbench/J
http://www.sql-workbench.net/

转到安装并启动 SQL Workbench/J 页面。按照说明操作，在您的系统上安装 SQL Workbench/J。

## 8. 下载 Amazon Redshift JDBC 驱动程序，配置SQL Workbench/J
https://s3.amazonaws.com/redshift-downloads/drivers/RedshiftJDBC42-1.2.8.1005.jar

具体配置Redshift步骤，请访问 http://docs.amazonaws.cn/redshift/latest/mgmt/connecting-using-workbench.html

## 9. 使用Spark进行数据分析：
打开SSH主节点，执行下列命令进入spark-shell界面:
sudo spark-shell --jars RedshiftJDBC42-1.2.16.1027.jar 

默认情况下，Spark 外壳创建其自己的 SparkContext 对象 (称作 sc)。如果 REPL 中需要，您可以使用此上下文。

## 在scala>环境下依次执行下列代码：

val df = spark.read.json("s3://sparksample/people.json")

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
  
df.write.format("jdbc").option("driver","com.amazon.redshift.jdbc42.Driver").option("url", "你所创建的redshift集群JDBC URL").option("dbtable", "public.t_person").option("user", "redshift用户").option("password", "redshift密码").save() 

## 10. 使用SQL Workbench/J查看数据
使用 SQL Workbench/J 连接到您的Refshift集群查看Spark已经将数据存入redshift表public.t_person中
> 
> 
