---
title: 了解java的RMI
date: 2017-04-10 22:57:25
category: java
tags: [RMI,java]
---
# java RMI
Java RMI 指的是远程方法调用 (Remote Method Invocation)。它是一种机制，能够让在某个 Java 虚拟机上的对象调用另一个 Java 虚拟机中的对象上的方法。

## Java RMI概念
在Java中，只要一个类继承了`java.rmi.Remote`接口，即可成为存在于服务器端的远程对象，供客户端访问并提供一定的服务。JavaDoc描述：Remote 接口用于标识其方法可以从非本地虚拟机上调用的接口。任何远程对象都必须直接或间接实现此接口。只有在“远程接口”（扩展 `java.rmi.Remote` 的接口）中指定的这些方法才可远程使用。

### 编写一个RMI的步骤

1. 定义一个远程接口，此接口需要继承Remote
2. 开发远程接口的实现类
3. 创建一个server并把远程对象注册到端口
4. 创建一个client查找远程对象，调用远程方法

####一个Hello Word的远程调用实例
定义一个远程接口

编写RMI应用的第一步就是先定义远程接口。远程接口必须继承`java.rmi.Remote`接口，并且声明自己的远程方法。为了处理远程方法发生的各种异常，每一个远程方法必须抛出一个`java.rmi.RemoteException`异常。

```java
public interface RemoteHelloWord extends Remote {
  String sayHello() throws RemoteException;
}
```
这个远程接口只定义了一个远程方法 sayHello()，远程方法在调用的时候有可能失败比如发生网络问题或者server挂掉，此时远程方法会抛出RemoteException异常。
开发接口的实现类

开发接口的实现类，即具体的远程对象，在远程对象中实现远程接口中定义的方法。

```java
public class RemoteHelloWordImpl  implements RemoteHelloWord{
    @Override
    public String sayHello() throws RemoteException {
        return "Hello Word!";
    }
}
```
创建一个Server并把对象注册到端口

在server端只需要做两件事：

   1. 创建并导出远程对象
   2. 用Java RMI registry 注册远程对象

下面是一个server端的程序：
```java
public class RMIServer {
public static void main(String[] args) {
    try {
        RemoteHelloWord hello=new RemoteHelloWordImpl();
        RemoteHelloWord stub=(RemoteHelloWord)UnicastRemoteObject.exportObject(hello, 9999);
        LocateRegistry.createRegistry(1099);
        Registry registry=LocateRegistry.getRegistry();
        registry.bind("helloword", stub);
        System.out.println("绑定成功!");
    } catch (RemoteException e) {
        e.printStackTrace();
    } catch (AlreadyBoundException e) {
        e.printStackTrace();
    }
}
}
```
创建一个client查找远程对象，调用远程方法
```java
public class RMIClient {
  public static void main(String[] args) {
         try {
              Registry registry = LocateRegistry.getRegistry("localhost");
              RemoteHelloWord hello = (RemoteHelloWord) registry.lookup( "helloword");
              String ret = hello.sayHello();
              System. out.println( ret);
        } catch (RemoteException e) {
               e.printStackTrace();
        } catch (NotBoundException e) {
               e.printStackTrace();
        }
  }

}
```

启动server，然后在启动client，控制台打印：
`Hello Word!`