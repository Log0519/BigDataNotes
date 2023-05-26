为什么要加上ROW FORMAT DELIMITED NULL DEFINED AS ‘’;？
功能：将Hive的这张表中的null设置为空
注意：Hive中的NULL值是假NULL，Hive底层的数据是文件，Hive中的NULL值实际是\n
为什么特殊指定？
如果将Hive中的NUll导出到MySQL中，就不能成功
