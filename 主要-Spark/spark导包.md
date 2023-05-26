package com.log.spark.core.acc

import org.apache.spark.rdd.RDD
import org.apache.spark.util.{AccumulatorV2, LongAccumulator}，自定义累加器用
import org.apache.spark.{SparkConf, SparkContext}

import scala.collection.mutable，使用mutable.Map的时候用