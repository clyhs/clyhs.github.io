---
title: 搭建ELK实时日志分析系统
date: 2017-05-08 15:53:19
category: elk
tags: [elasticseach,java]
---
#  什么是ELK
开源实时日志分析 ELK 由 ElasticSearch 、 Logstash 和 Kiabana 三个开源工具组成

* Elasticsearch 是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制， restful 风格接口，多数据源，自动搜索负载等。

* Logstash 是一个完全开源的工具，他可以对你的日志进行收集、分析，并将其存储供以后使用（如，搜索）。

* kibana 也是一个开源和免费的工具，他 Kibana 可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助您汇总、分析和搜索重要数据日志。

## 部署

### 服务器

> IP: 192.168.0.16
> jdk: 1.8
> os: centos7 64
> hostname: centoss1.pascloud.com


安装包：
1. elasticsearch-5.4.0.rpm
2. kibana-5.4.0-x86_64.rpm
3. logstash-5.4.0.rpm
4. filebeat-5.4.0-x86_64.rpm

### 安装elasticsearch
```shell
$ rpm -ivh elasticsearch-5.4.0.rpm
```
修改 */etc/elasticsearch/elasticsearch.yml*
```shell
$ network.host: 192.168.0.16
```
启动：
```shell
$ sudo systemctl daemon-reload
$ sudo systemctl enable elasticsearch.service
$ sudo systemctl start elasticsearch.service

$ curl -X GET http://localhost:9200/

```
```shell
{
  "name" : "XMfXZ3K",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "xYql63WZTs6J2vOVukCLlQ",
  "version" : {
    "number" : "5.4.0",
    "build_hash" : "780f8c4",
    "build_date" : "2017-04-28T17:43:27.229Z",
    "build_snapshot" : false,
    "lucene_version" : "6.5.0"
  },
  "tagline" : "You Know, for Search"
}
```

### 安装kibana
```shell
$ rpm -ivh kibana-5.4.0-x86_64.rpm
```
修改/etc/kibana/kibana.xml
```shell
$ vi /etc/kibana/kibana.xml

server.port: 5601
server.host: "192.168.0.16"
server.name: "centoss1.pascloud.com"
elasticsearch.url: "http://localhost:9200"
```
启动：
```shell
$ systemctl start kibana
$ netstat -tunpl | grep 5601
```
访问：http://192.168.0.16:5601

### 安装logstash
```shell
$ rpm -ivh logstash-5.4.0.rpm
```
生成SSL证书
```shell
$ cd /etc/pki/tls
$ openssl req -subj '/CN=centoss1.pascloud.com/' -x509 -days 3650 -batch -nodes -newkey rsa:2048 -keyout private/logstash-forwarder.key -out certs/logstash-forwarder.crt

```
创建一个01-logstash-initial.conf 文件
```shell
cat > /etc/logstash/conf.d/01-logstash-initial.conf << EOF
input {
  beats {
    port => 5044
    ssl => true
    type => "logs"
    ssl_certificate => "/etc/pki/tls/certs/logstash-forwarder.crt"
    ssl_key => "/etc/pki/tls/private/logstash-forwarder.key"
  }
}


filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
      add_field => [ "received_at", "%{@timestamp}" ]
      add_field => [ "received_from", "%{host}" ]
    }
    syslog_pri { }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }
}

output {
  elasticsearch { hosts => ["192.168.0.16:9200"] }
  stdout { codec => rubydebug }
}
EOF
```
验证：*01-logstash-initial.conf *
```shell
$ /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/01-logstash-initial.conf 
```
启动：
```shell
$ systemctl start logstash
$ netstat -tunpl | grep 5044
```

### 安装收集日志客户端：filebeat
```shell
$ rpm -ivh filebeat-5.4.0-x86_64.rpm
```


修改 /etc/filebeat/filebate.yml
```shell

- input_type: log
  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    ＃新增 指定到需要的日志文件
    - /usr/logs/stdout.log
    - /var/log/*.log

output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["192.168.0.16:9200"]
  
output.logstash:
  # The Logstash hosts
  hosts: ["centoss1.pascloud.com:5044"]
  ssl.certificate_authorities: ["/etc/pki/tls/certs/logstash-forwarder.crt"]
```
启动：
```shell
$ systemctl start filebeat

```
访问http://192.168.0.16:5601
![img](/images/elk/elk01.png)