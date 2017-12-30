---
layout: blog
title: Elasticsearch01-入门教程
tags: [java, Elasticsearch,]
desc: centos安装Elasticsearch
---

### 1.centos安装Elasticsearch
官方安装教程：https://www.elastic.co/guide/en/elasticsearch/reference/6.0/zip-targz.html
#### 1.1 安装jdk
#### 1.2 安装Elasticsearch

##### 下载
```
[root@bobyang apps]# curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.0.1.tar.gz 
```
##### 解压
```
[root@bobyang apps]# tar -zxvf elasticsearch-6.0.1.tar.gz 
```


##### 官方教程

```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.0.1.tar.gz
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.0.1.tar.gz.sha512
shasum -a 512 -c elasticsearch-6.0.1.tar.gz.sha512 
tar -xzf elasticsearch-6.0.1.tar.gz
cd elasticsearch-6.0.1/ 
```

#### 1.3 启动

```
[root@bobyang apps]# cd elasticsearch-6.0.1/bin/
[root@bobyang bin]# ./elasticsearch
```






