
  def sampleTopKey( sparkSession: SparkSession, tableName: String, keyColumn: String ): Array[(Int, Row)] = {
    val df: DataFrame = sparkSession.sql("select " + keyColumn + " from " + tableName)
    val top10Key = df
      .select(keyColumn).sample(false, 0.1).rdd // 对key不放回采样
      .map(k => (k, 1)).reduceByKey(_ + _) // 统计不同key出现的次数
      .map(k => (k._2, k._1)).sortByKey(false) // 统计的key进行排序
      .take(10)
    top10Key