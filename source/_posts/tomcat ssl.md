---
title: tomcat ssl
date: 2017-08-11 22:45:25
category: ssl
tags: tomcat
---
## tomcat ssl

https分为单项认证和双向认证。

一般https页面上的访问都是单项认证，服务端发送数字证书给客户端，客户单方面验证。而服务端不做验证。

而双向认证，需要双方都有证书，然后发送给对方进行验证。一般用于企业应用对接。



1、**单项认证**

服务端创建密钥对及密钥库。

```
keytool -genkey -alias tomcat -keypass 123456 -keyalg RSA -keysize 2048 -validity 365 -storetype JKS -keystore D:/ssl/keystore.jks -storepass 123456
```

![img](https://clyhs.github.io/images/java/ssl1.png)

看一下密钥库里的信息

```
keytool -list -v -keystore D:/ssl/keystore.jks
```

![img](https://clyhs.github.io/images/java/ssl2.png)

* tomcat7

conf/server.xml

```
<Connector port="8443" protocol="org.apache.coyote.http11.Http11Protocol"
               maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
               keystoreFile="D:/ssl/keystore.jks"
               keystorePass="123456"
               truststoreFile="D:/ssl/keystore.jks"
               truststorePass="123456"
               clientAuth="false" sslProtocol="TLS" />
```

注意：clientAuth="false"，这个参数值如果为true为双向认证，false为单项认证，我们用false。

* tomcat8.5

conf/server.xml

```
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
           maxThreads="150" SSLEnabled="true">
		<SSLHostConfig>
			<Certificate certificateKeystoreFile="d:/ssl/keystore.jks"
		   certificateKeystorePassword="123456"
						 type="RSA" />
		</SSLHostConfig>
	</Connector>
```

2、**双向认证：**

我们上边生成了服务端证书，并发送给客户端进行了验证。

(1)为方便导入浏览器，生成p12格式的密钥库。

```
keytool -genkey -alias client -keypass 123456 -keyalg RSA -keysize 2048 -validity 365 -storetype PKCS12 -keystore D:/ssl/client.p12 -storepass 123456
```

![img](https://clyhs.github.io/images/java/ssl3.png)

(3)然后可以看一下证书库里的密钥对

```
keytool -list -v -storetype PKCS12 -keystore D:/ssl/client.p12
```

(4)导出客户端证书。

```
keytool -export -alias client -keystore D:/ssl/client.p12 -storetype PKCS12 -keypass 123456 -file D:/ssl/client.cer
```

(5)把客户端证书添加到服务端的密钥库中并添加信任。

```
keytool -import -alias client -v -file D:/ssl/client.cer -keystore D:/ssl/keystore.jks -storepass 123456
```

(6)然后把客户端的密钥库client.p12导入到浏览器的证书管理中，放到个人下。【重要】如果你不导入这个文件是访问不到的。

![img](https://clyhs.github.io/images/java/ssl4.png)

![img](https://clyhs.github.io/images/java/ssl5.png)

![img](https://clyhs.github.io/images/java/ssl6.png)

* tomcat7

  将tomcat的server.xml中https配置参数clientAuth="false"改成true，双向认证。不然还是单向认证的，没有意义。

  ```
  <Connector port="8443" protocol="org.apache.coyote.http11.Http11Protocol"
                 maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
                 keystoreFile="D:/ssl/keystore.jks"
                 keystorePass="123456"
                 truststoreFile="D:/ssl/keystore.jks"
                 truststorePass="123456"
                 clientAuth="true" sslProtocol="TLS" />
  ```

  

3、**模拟CA实现对服务器证书的认证**

CA认证，是这个机构的根证书已经存在于浏览器中的【受信任的根证书颁发机构】，浏览器信任他，所以不会出现该证书不安全的提示。

（1）首先服务端需要生成一个带证书及主体信息的签名申请文件csr格式，CA需要用来制作签名证书。

```
keytool -certreq -keyalg RSA -alias tomcat -sigalg SHA256withRSA -keystore D:/ssl/keystore.jks -file D:/ssl/serverreq.csr
```

（2）CA也是有自己的密钥对和密钥库的，创建好。

```
keytool -genkey -alias rootca -keypass 123456 -keyalg RSA -keysize 2048 -validity 365 -storetype JKS -keystore D:/ssl/castore.jks -storepass 123456
```

![img](https://clyhs.github.io/images/java/ssl7.png)

（3）CA库有了，然后CA用自己的私钥对服务端申请签名文件中的证书进行签名操作。

```
keytool -gencert -alias rootca -keystore D:/ssl/castore.jks -infile D:/ssl/serverreq.csr -outfile D:/ssl/signedserver.cer
```

然后我们看一下，发布者是CA

```
keytool -printcert -file D:/ssl/signedserver.cer
```

![img](https://clyhs.github.io/images/java/ssl8.png)

（4）我们将CA库的证书导出来，这就是上边这个signedserver.cer的根证书。

```
keytool -export -alias rootca -keystore D:/ssl/castore.jks -storetype JKS -keypass 123456 -file D:/ssl/rootca.cer
```

（5）服务端导入根证书

```
keytool -import -v -alias rootca -file D:/ssl/rootca.cer  -keystore D:/ssl/keystore.jks
```

（6）服务端导入签名证书覆盖原证书

```
keytool -import -v -alias tomcat -file D:/ssl/signedserver.cer  -keystore D:/ssl/keystore.jks
```

（7）来看一下服务端的库

```
keytool -list -v -keystore D:/ssl/keystore.jks
```

这里边一共有三条信息：

　　（1）rootca证书信息

　　（2）刚刚导入的覆盖的别名叫tomcat的签名证书

　　（3）上一篇做的双向认证导入的客户端证书。

（7）把根证书rootca.cer发送给客户端，安装到浏览器中的【受信任的根证书颁发机构】。

4、**模拟的CA密钥库对客户端证书也进行签名颁发**

一个客户端和服务端对接就需要把这个客户端的证书拿来导入到服务端的密钥库中。那么很多客户端要对接，就要多次导入。

可以这样，让客户端发送证书的csr文件给我们，我们用模拟的CA密钥库对客户端证书也进行签名颁发。

然后把签名后的证书发送给他，让他自己导入到自己的客户端密钥库里【也需要先导入根证书，再导入签名证书】。

这样的话https交互时，

服务端发送签名证书给客户端：客户端能用安装在浏览器中的【受信任的颁发机构】中的rootca.cer根证书进行验证。

客户端发送签名证书给服务端：服务端也能用信任的库里的rootca根证书对客户端发来签名证书进行校验。

为了清晰点，从头开始做，就不展示图片了。

**服务端：**

1.创建CA库，用于对证书签名

```
keytool -genkey -alias rootca -keypass 123456 -keyalg RSA -keysize 2048 -validity 365 -storetype JKS -keystore D:/ssl/castore.jks -storepass 123456
```

2.创建服务端密钥库

```
keytool -genkey -alias tomcat -keypass 123456 -keyalg RSA -keysize 2048 -validity 365 -storetype JKS -keystore D:/ssl/keystore.jks -storepass 123456
```

3.创建服务端证书签名请求文件

```
keytool -certreq -keyalg RSA -alias tomcat -sigalg SHA256withRSA -keystore D:/ssl/keystore.jks -file D:/ssl/serverreq.csr
```

4.CA库对服务端证书进行签名，生成一个证书文件

```
keytool -gencert -alias rootca -keystore D:/ssl/castore.jks -infile D:/ssl/serverreq.csr -outfile D:/ssl/signedserver.cer
```

5.从CA库导出rootca根证书

```
keytool -export -alias rootca -keystore D:/ssl/castore.jks -storetype JKS -keypass 123456 -file D:/ssl/rootca.cer
```

6.将rootca导入到服务端密钥库

```
keytool -import -v -alias rootca -file D:/ssl/rootca.cer  -keystore D:/ssl/keystore.jks
```

7.将服务端签名证书导入到服务端的密钥库，覆盖原证书。

```
keytool -import -v -alias tomcat -file D:/ssl/signedserver.cer  -keystore D:/ssl/keystore.jks
```

8.tomcat的server.xml添加配置【clientAutl=true，双向认证】

```
<Connector port="8443" protocol="org.apache.coyote.http11.Http11Protocol"
               maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
               keystoreFile="D:/ssl/keystore.jks"
               keystorePass="123456"
               truststoreFile="D:/ssl/keystore.jks"
               truststorePass="123456"
               clientAuth="true" sslProtocol="TLS" sslEnabledProtocols="TLSv1,TLSv1.1,TLSv1.2" />
```

**客户端：**

1.创建客户端密钥库

```
keytool -genkey -alias client -keypass 123456 -keyalg RSA -keysize 2048 -validity 365 -storetype JKS -keystore D:/ssl/client.jks -storepass 123456
```

2.创建客户端证书签名请求文件

```
keytool -certreq -keyalg RSA -alias client -sigalg SHA256withRSA -keystore D:/ssl/client.jks -file D:/ssl/clientreq.csr
```

3.把客户端证书签名请求文件clientreq.csr发送给服务端，服务端的模拟CA密钥库对证书进行签名，生成一个证书文件**【以下这条是服务端执行】**

```
keytool -gencert -alias rootca -keystore D:/ssl/castore.jks -infile D:/ssl/clientreq.csr -outfile D:/ssl/signedclient.cer
```

4.服务端把生成的客户端签名文件signedclient.cer和根证书rootca.cer发送给客户，客户端导入密钥库。还是先导入根证书，再导入签名证书。

```
keytool -import -v -alias rootca -file D:/ssl/rootca.cer  -keystore D:/ssl/client.jks
keytool -import -v -alias client -file D:/ssl/signedclient.cer  -keystore D:/ssl/client.jks
```

5.浏览器不支持jks，所以把客户端的jks库转为p12格式库。

```
keytool -importkeystore -srckeystore D:/ssl/client.jks -srcstoretype JKS -deststoretype PKCS12 -destkeystore D:/ssl/client.p12
```

6.将客户端密钥库client.p12导入到浏览器-证书-个人

7.将rootca.cer导入到浏览器-证书-受信任的证书颁发机构

启动tomcat，访问成功