---
layout: post
title:  "Spark functions表达式"
date:   2018-12-03 21:31:01 +0800
categories: 大数据
tag: spark编程
---



   Spark 提供了functions表达式的方式，能够执行响应的函数，能够执行用户输入的表达式，比直接调用函数更方便。

   spark functions.expr("")，函数表达式中 直接输入的名称是字段名称，如果需要输入常量，则需要用\"constant\"的方式传入
   
   如果需要调用UDF函数，可以直接调用，不需要用callUDF


```java
public class TestSpark {

    private static SparkSession session;

    static {
        SparkConf conf = new SparkConf().setAppName("SparkTest").setMaster("local[4]");
        session = SparkSession.builder().config(conf).getOrCreate();
        session.udf().register("obfuscation", UDFUtils.obfuscation, DataTypes.StringType);
    }


    public static void main(String[] args){
         Dataset<Row> dataset = session.read().parquet("/home/test/*****");
         Column col1 = functions.expr("length(name)");  //name为字段名
         Column col2 = functions.expr("obfuscation(name, \"replace_regex\")"); //obfuscation为UDF名称，name为字段名，replace_regex为常量
         dataset = dataset.withColumn("col1", col1);
         dataset = dataset.withColumn("col2", col2);
         dataset.show();
         System.out.println(dataset.count());
    }
}
```