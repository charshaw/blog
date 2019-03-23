---
layout: post
title:  "java远程调用kettle任务"
date:   2019-03-23 14:00:00 +0800
categories: 大数据
tag: ETL
---


   ketlle是一款开源的ELT工具，包括Spoon Pan CHEF Kitchen。

    * Spoon 允许通过图形界面来设计ETL转换过程
        ![spoon](../media/kettle_spoon.png)
    * Pan 允许批量运行有Spoon设计的ETL转换。后台执行程序，没有图形界面
    * CHEF 允许创建Job。
    * Kitchen 允许批量使用由Chef设计的任务，kitchen也是后台程序
    
另外kettle还提供了Carte，Carte是一个轻量级web服务器，允许远程进行监控、启动、停止在Carte服务上运行的job和trans，Carte也是本文的重点，
只有通过Carte才能通过http的方式实现远程执行ETL任务，其实际运行是在Carte服务器上。

```java
    public static void main(String[] args) throws Exception {
        KettleEnvironment.init();
        //创建DB资源库
        KettleDatabaseRepository repository = new KettleDatabaseRepository();
        DatabaseMeta databaseMeta = new DatabaseMeta("***", "mysql", "jdbc", "192.168.1.1", "kettle", "3306", "username", "pwd");
        //选择资源库
        KettleDatabaseRepositoryMeta kettleDatabaseRepositoryMeta = new KettleDatabaseRepositoryMeta("kettle", "kettle", "Transformation description", databaseMeta);
        repository.init(kettleDatabaseRepositoryMeta);
        //连接资源库
        repository.connect("admin", "admin");
        RepositoryDirectoryInterface directoryInterface = repository.loadRepositoryDirectoryTree();
        List<RepositoryElementMetaInterface> names = repository.getJobObjects(null, false);
        ObjectId  objectId = repository.getJobId("Job_first_dir", directoryInterface);
        JobMeta jobMeta = repository.loadJob("Job_first_dir", directoryInterface, null, null);
        SlaveServer remoteSlaveServer = getSlaveServer();
        JobExecutionConfiguration jobExecutionConfiguration = new JobExecutionConfiguration();
        jobExecutionConfiguration.setRemoteServer(remoteSlaveServer); **// 配置远程服务**
        jobExecutionConfiguration.setRepository(repository);

        String lastCarteObjectId = Job.sendToSlaveServer(jobMeta, jobExecutionConfiguration, repository, null);
        System.out.println("lastCarteObjectId=" + lastCarteObjectId);
        SlaveServerJobStatus jobStatus = null;
        do {
            jobStatus = remoteSlaveServer.getJobStatus(jobMeta.getName(), lastCarteObjectId, 0);
        } while (jobStatus != null && jobStatus.isRunning());
        Result oneResult = new Result();
        System.out.println(jobStatus);
        if (jobStatus.getResult() != null) {
            // 流程完成，得到结果
            oneResult = jobStatus.getResult();
            System.out.println("Result:" + oneResult);
        } else {
            System.out.println("取到空了");
        }

    }
    private static SlaveServer getSlaveServer(){
        SlaveServer remoteSlaveServer = new SlaveServer();
        remoteSlaveServer.setHostname("192.168.1.1");// 设置远程IP
        remoteSlaveServer.setPort("8081");// 端口
        remoteSlaveServer.setUsername("cluster");
        remoteSlaveServer.setPassword("cluster");
        return remoteSlaveServer;
    }
```

如果不配置远程服务器，就会在本地执行，该远程服务即为Carte的端口号，这样就会把任务提交到Carte服务器上运行。其中提交给Carte运行的接口需要把JobMeta传入，其中JobMeta
可以是从krb文件中读取，也可以从数据库类型资源库中获取，本文是数据库中获取，通过配置KettleDatabaseRepository，从数据库中获取JobMeta，然后提交给Carte执行。


注：Carte是一个提供web端的任务监控、启动、停止的服务，spoon支持本地执行该任务，也支持远程执行将任务提交到Carte上执行，需要先配置slaveserver。


