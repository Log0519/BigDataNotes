# 一、创建 DataFrame
@[toc]
在 Spark SQL 中 SparkSession 是创建 DataFrame 和执行 SQL 的入口，创建 DataFrame

## 1. 三种方式创建：

### ① 通过 Spark 的数据源进行创建；

    val df = spark.read.json("data/user.json")

### ② 从一个存在的 RDD 进行转换；需要写清RDD里面的结构，即每个数据代表什么意思

    val df = sc.makeRDD(List(("zhangsan",30), ("lisi",40))).map(t=>User(t._1, t._2)).toDF

### ③ 还可以从 Hive Table 进行查询返回

## 2. DataFrame操作

### ① 读取 JSON 文件创建 DataFrame

    scala> val df = spark.read.json("data/user.json")
    df: org.apache.spark.sql.DataFrame = [age: bigint， username: string]

### ② 对 DataFrame 创建一个临时表

`scala> df.createOrReplaceTempView("people")`

### ③ 通过 SQL 语句实现查询全表

    scala> val sqlDF = spark.sql("SELECT * FROM people")
    sqlDF: org.apache.spark.sql.DataFrame = [age: bigint， name: string]

# 二、DataSet

### 1. 利用样例类创建

    scala> case class Person(name: String, age: Long)
    defined class Person
    scala> val caseClassDS = Seq(Person("zhangsan",2)).toDS()
    caseClassDS: org.apache.spark.sql.Dataset[Person] = [name: string, age: Long]
    scala> caseClassDS.show

### 2. 使用基本类型的序列创建 DataSet

    scala> val ds = Seq(1,2,3,4,5).toDS
    ds: org.apache.spark.sql.Dataset[Int] = [value: int]
    scala> ds.show

<font color=#FF0000>注意：在实际使用的时候，很少用到把序列转换成DataSet，更多的是通过RDD来得到DataSet</font>

 <font color=#FF0000>DataSet可以用 表名.属性 使用列</font>

# 三、RDD 转换为 DataSet

### 1. SparkSQL 能够自动将包含有 case 类的 RDD 转换成 DataSet，case 类定义了 table 的结构，case 类属性通过反射变成了表的列名。Case 类可以包含诸如 Seq 或者 Array 等复杂的结构。

    scala> case class User(name:String, age:Int)
    defined class User
    scala> sc.makeRDD(List(("zhangsan",30), ("lisi",49))).map(t=>User(t._1, t._2)).toDS
    res11: org.apache.spark.sql.Dataset[User] = [name: string, age: int]

### 2. DataSet 转换为 RDD

DataSet 其实也是对 RDD 的封装，所以可以直接获取内部的 RDD

    scala> case class User(name:String, age:Int)
    defined class User
    scala> sc.makeRDD(List(("zhangsan",30), ("lisi",49))).map(t=>User(t._1, 
    t._2)).toDS
    res11: org.apache.spark.sql.Dataset[User] = [name: string, age: int]
    scala> val rdd = res11.rdd
    rdd: org.apache.spark.rdd.RDD[User] = MapPartitionsRDD[51] at rdd at 
    <console>:25
    scala> rdd.collect
    res12: Array[User] = Array(User(zhangsan,30), User(lisi,49))

# 四、DataFrame 和 DataSet 转换

### DataFrame 其实是 DataSet 的特例，所以它们之间是可以互相转换的。

### <font color=#FF0000>数据包含关系：DataSet > DataFrame > RDD ,从高转换到低很方便，低转换到高要给出高所需要的条件，DataFrame需要数据代表的意思，DataSet需要数据类型,DataFrame,DataSet都可以创建临时表</font>

RDD-----**toDF**----->DataFrame

DataFrame-----**rdd**----->RDD

DataFrame-----**as[样例类]**----->DataSet

DataSet-----**toDF**----->DataFrame

样例类RDD-----**toDS**----->DataSet，前提是rdd要把数据封装为样例类，直接使用也可以，但是不方便

DataSet-----**rdd**----->RDD

    scala> case class User(name:String, age:Int)
    defined class User
    scala> val df = sc.makeRDD(List(("zhangsan",30), 
    ("lisi",49))).toDF("name","age")
    df: org.apache.spark.sql.DataFrame = [name: string, age: int]
    scala> val ds = df.as[User]
    ds: org.apache.spark.sql.Dataset[User] = [name: string, age: int]

###DataSet 转换为 DataFrame

    scala> val ds = df.as[User]
    ds: org.apache.spark.sql.Dataset[User] = [name: string, age: int]
    scala> val df = ds.toDF
    df: org.apache.spark.sql.DataFrame = [name: string, age: int]

# 五. RDD、DataFrame、DataSet 三者的关系

在 SparkSQL 中 Spark 为我们提供了两个新的抽象，分别是 **DataFrame** 和 **DataSet**。

他们和 RDD 有什么区别呢？首先从版本的产生上来看：

    Spark1.0 => RDD 
    Spark1.3 => DataFrame
    Spark1.6 => Dataset

如果同样的数据都给到这三个数据结构，他们分别计算之后，都会给出相同的结果。不
同是的他们的执行效率和执行方式。在后期的 Spark 版本中，DataSet 有可能会逐步取代 RDD
和 DataFrame 成为唯一的 API 接口。

## 1. 三者的共性

➢ RDD、DataFrame、DataSet 全都是 spark 平台下的分布式弹性数据集，为处理超大型数
据提供便利;

➢ 三者都有惰性机制，在进行创建、转换，如 map 方法时，不会立即执行，只有在遇到
Action 如 foreach 时，三者才会开始遍历运算; 

➢ 三者有许多共同的函数，如 filter，排序等; 

➢ 在对 DataFrame 和 Dataset 进行操作许多操作都需要这个包:import spark.implicits._（在
创建好 SparkSession 对象后尽量直接导入）

➢ 三者都会根据 Spark 的内存情况自动缓存运算，这样即使数据量很大，也不用担心会
内存溢出

➢ 三者都有 partition 的概念

➢ DataFrame 和 DataSet 均可使用模式匹配获取各个字段的值和类型

## 2. 三者的区别

### ① RDD

➢ RDD 一般和 spark mllib 同时使用

➢ RDD 不支持 sparksql 操作

### ② DataFrame

➢ 与 RDD 和 Dataset 不同，DataFrame 每一行的类型固定为 Row，每一列的值没法直
接访问，只有通过解析才能获取各个字段的值

➢ DataFrame 与 DataSet 一般不与 spark mllib 同时使用

➢ DataFrame 与 DataSet 均支持 SparkSQL 的操作，比如 select，groupby 之类，还能
注册临时表/视窗，进行 sql 语句操作

➢ DataFrame 与 DataSet 支持一些特别方便的保存方式，比如保存成 csv，可以带上表
头，这样每一列的字段名一目了然(后面专门讲解)

### ③ DataSet

➢ Dataset 和 DataFrame 拥有完全相同的成员函数，区别只是每一行的数据类型不同。

DataFrame 其实就是 DataSet 的一个特例 type DataFrame = Dataset[Row]

➢ DataFrame 也可以叫 Dataset[Row],每一行的类型是 Row，不解析，每一行究竟有哪
些字段，各个字段又是什么类型都无从得知，只能用上面提到的 getAS 方法或者共
性中的第七条提到的模式匹配拿出特定字段。而 Dataset 中，每一行是什么类型是
不一定的，在自定义了 case class 之后可以很自由的获得每一行的信息

# 六. 数据访问方式

## 1. DataSet.show , DataFrame.show

## 2. 创建临时表后，spark . sql ( " select * (Dataset可以用 ‘.属性’访问)  from  表名 " )
