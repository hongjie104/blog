---
layout:     post
title:      "ElasticSearch 简介"
subtitle:   ""
date:       2017-11-20 12:00:00
catalog:    true
tags:
    - ElasticSearch
---

# Elasticsearch 简介
---
### 是什么
Elasticsearch是一个基于Apache Lucene(TM)的开源搜索引擎，被认为是迄今为止最先进、性能最好的、功能最全的搜索引擎库。

### 安装
1. Elasticsearch使用Java开发，所以要先安装[Java环境](http://www.java.com)。
2. 从[官方下载页面](https://www.elastic.co/downloads/elasticsearch)查到最新版版本号，替换掉下面链接中的版本号。今天是17年11月20号，最新版是6.0.0，那就以6.0.0版本举例：
    ```
    
    curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.0.0.zip
	unzip elasticsearch-6.0.0.zip
	cd  elasticsearch-6.0.0
    ```
    
    下载完成后验证安装：
    ```
    bin/elasticsearch
    ```
    稍等片刻，完成启动后，在浏览器地址栏中输入地址进行访问：

    ```
    http://localhost:9200/?pretty
    ```
    
    如无意外，浏览器上会显示类似以下的数据：
    ```
    {
      "name" : "ipypN4Q",
      "cluster_name" : "elasticsearch",
      "cluster_uuid" : "cfOYLuogQkazOi6cEFj03w",
      "version" : {
        "number" : "6.0.0",
        "build_hash" : "8f0685b",
        "build_date" : "2017-11-10T18:41:22.859Z",
        "build_snapshot" : false,
        "lucene_version" : "7.0.1",
        "minimum_wire_compatibility_version" : "5.6.0",
        "minimum_index_compatibility_version" : "5.0.0"
      },
      "tagline" : "You Know, for Search"
    }
    ```
    使用ctrl + C先关闭Elasticsearch，安装x-pack。
3. 安装x-pack扩展
    [x-pack](https://www.elastic.co/downloads/x-pack)是Elasticsearch的功能扩展包，对原本的 marvel、watch、alert 做了一个封装。
	安装方法：进入到elasticsearch目录下，执行如下命令：
	
    ```
    
    bin/elasticsearch-plugin install x-pack
    ```
    
    安装完成之后，再次启动Elasticsearch，在浏览器中再次访问
    ```
    http://localhost:9200/?pretty
    ```
    
    会要求输入账号和密码，这是因为安装了x-pack之后访问收到了权限控制。
    在命令行中输入
    ```
    bin/x-pack/setup-passwords auto
    ```
    
    会生成默认的账号和密码，使用elastic的账号和密码即可登录成功。

3. 安装kibana
    [kibana](https://www.elastic.co/downloads/kibana)是一个为 ElasticSearch提供数据分析的 Web 接口，可用它对日志进行高效的搜索、可视化、分析等各种操作。
    访问[下载网页]([kibana](https://www.elastic.co/downloads/kibana))选中符合自己操作系统的下载链接进行下载。
4. 是的撒多所
5. sdsd
6. 的双方的说法





Changed password for user kibana
PASSWORD kibana = 8ttTx7+~%7RdzL#Nri?%

Changed password for user logstash_system
PASSWORD logstash_system = 0C0tm$!sU!-MfD9n$%S8

Changed password for user elastic
PASSWORD elastic = !JF-tTE*pklKDjaALtEy
