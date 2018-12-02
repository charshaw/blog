---
layout: post
title:  "Spark local模式的java代码示例"
date:   2018-12-03 20:31:01 +0800
categories: 大数据
tag: spark编程
---



Spark有多种部署方式，比如standalone、mesos、yarn(yarn-client yarn-cluster)

spark还提供一种local模式，方便学习测试使用，最基本的代码如下，通过main函数能够启动一个SparkContext，用线程来模拟执行器，
并且提供了http://localhost:4040 SparkUI，能够查看任务信息。这样不需要安装任何其他包就能够运行spark任务，只需要有jdk环境即可启动

```java
public class TestSpark {

    private static SparkSession session;

    static {
        SparkConf conf = new SparkConf().setAppName("SparkTest").setMaster("local[4]");
        session = SparkSession.builder().config(conf).getOrCreate();
    }


    public static void main(String[] args){
         Dataset<Row> dataset = session.read().parquet("/home/test/*****");
         dataset.show();
         System.out.println(dataset.count());
    }
}
```