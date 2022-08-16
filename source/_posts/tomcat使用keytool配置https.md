---
title: tomcat使用keytool配置https
date: 2017-08-11 22:45:25
category: https
tags: tomcat
---

## 一、生成证书

1、**生成服务器证书**

```
keytool -genkey -v -alias tomcat -keyalg RSA -keystore d:\tomcat.keystore -validity 36500
```

![img](https://clyhs.github.io/images/java/1.png)

2、**生成客户端证书**

```
keytool -genkey -v -alias mclient -keyalg RSA -storetype PKCS12 -keystore d:\mclient.p12
```

![img](https://clyhs.github.io/images/java/2.png)

3、**让服务器信任客户端证书**

(1)由于不能直接将PKCS12格式的证书库导入，必须先把客户端证书导出为一个单独的CER文件，使用如下命令：

```
keytool -export -alias mclient -keystore d:\mclient.p12 -storetype PKCS12 -storepass 123456 -rfc -file d:\mclient.cer
```

![img](https://clyhs.github.io/images/java/3.png)

(2)将该文件导入到服务器的证书库，添加为一个信任证书使用命令如下：

```
keytool -import -v -file d:\mclient.cer –keystore d:\tomcat.keystore
```

![img](https://clyhs.github.io/images/java/4.png)

(3)通过 list 命令查看服务器的证书库，可以看到两个证书，一个是服务器证书，一个是受信任的客户端证书：

```
keytool -list -keystore tomcat.keystore
```

![img](https://clyhs.github.io/images/java/5.png)

4、**让客户端信任服务器证书**

把服务器证书导出为一个单独的CER文件提供给客户端，使用如下命令：

```
keytool -keystore d:\tomcat.keystore -export -alias tomcat -file d:\tomcat.cer
```

![img](https://clyhs.github.io/images/java/6.png)

5、**经过上面操作，生成如下证书：**

![img](https://clyhs.github.io/images/java/7.png)

其中 tomcat.cer 提供给客户端，tomcat.keystore供服务器使用

## 二、证书使用

1、**服务器tomcat的配置**

```
<Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true" 
      maxThreads="150" scheme="https" secure="true" 
      clientAuth="false" sslProtocol="TLS" 
      keystoreFile="conf/cer/tomcat.keystore" keystorePass="123456" 
      truststoreFile="conf/cer/tomcat.keystore" truststorePass="123456" /> 


https://localhost:8443
```

2、 **导入服务器公钥证书（tomcat.cer）**

由于是自签名的证书，为避免每次都提示不安全。这里双击tomcat.cer安装服务器证书。

注意：将证书填入到“受信任的根证书颁发机构”

![img](https://clyhs.github.io/images/java/8.png)

![img](https://clyhs.github.io/images/java/9.png)