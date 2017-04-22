---
title: 跟着实例学习ZooKeeper的用法：分布式锁
date: 2017-04-22 19:40:45
category: zookeeper
tags: [java,zookeeper]
---

[TOC]

# 锁
分布式的锁全局同步， 这意味着任何一个时间点不会有两个客户端都拥有相同的锁。

## 可重入锁Shared Reentrant Lock
首先我们先看一个全局可重入的锁。 Shared意味着锁是全局可见的， 客户端都可以请求锁。 Reentrant和JDK的ReentrantLock类似， 意味着同一个客户端在拥有锁的同时，可以多次获取，不会被阻塞。
它是由类InterProcessMutex来实现。
它的构造函数为：
```java
public InterProcessMutex(CuratorFramework client, String path)
```
通过acquire获得锁，并提供超时机制：
```java
public void acquire()
Acquire the mutex - blocking until it's available. Note: the same thread can call acquire
re-entrantly. Each call to acquire must be balanced by a call to release()
public boolean acquire(long time,
                       TimeUnit unit)
Acquire the mutex - blocks until it's available or the given time expires. Note: the same thread can
call acquire re-entrantly. Each call to acquire that returns true must be balanced by a call to release()
Parameters:
time - time to wait
unit - time unit
Returns:
true if the mutex was acquired, false if not
```

通过release()方法释放锁。
InterProcessMutex 实例可以重用。

**Revoking**
ZooKeeper recipes wiki定义了可协商的撤销机制。
为了撤销mutex, 调用下面的方法：
```java
public void makeRevocable(RevocationListener<T> listener)
将锁设为可撤销的. 当别的进程或线程想让你释放锁是Listener会被调用。
Parameters:
listener - the listener
```
如果你请求撤销当前的锁， 调用Revoker方法。
```java
public static void attemptRevoke(CuratorFramework client,
                                 String path)
                         throws Exception
Utility to mark a lock for revocation. Assuming that the lock has been registered
with a RevocationListener, it will get called and the lock should be released. Note,
however, that revocation is cooperative.
Parameters:
client - the client
path - the path of the lock - usually from something like InterProcessMutex.getParticipantNodes()
```
**错误处理**
还是强烈推荐你使用ConnectionStateListener处理连接状态的改变。 当连接LOST时你不再拥有锁。

首先让我们创建一个模拟的共享资源， 这个资源期望只能单线程的访问，否则会有并发问题。
```java
package com.colobu.zkrecipe.lock;
import java.util.concurrent.atomic.AtomicBoolean;
public class FakeLimitedResource {
	private final AtomicBoolean inUse = new AtomicBoolean(false);
	public void use() throws InterruptedException {
		// 真实环境中我们会在这里访问/维护一个共享的资源
		//这个例子在使用锁的情况下不会非法并发异常IllegalStateException
		//但是在无锁的情况由于sleep了一段时间，很容易抛出异常
		if (!inUse.compareAndSet(false, true)) { 
			throw new IllegalStateException("Needs to be used by one client at a time");
		}
		try {
			Thread.sleep((long) (3 * Math.random()));
		} finally {
			inUse.set(false);
		}
	}
}
```
然后创建一个ExampleClientThatLocks类， 它负责请求锁， 使用资源，释放锁这样一个完整的访问过程。
```java
package com.colobu.zkrecipe.lock;
import java.util.concurrent.TimeUnit;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.recipes.locks.InterProcessMutex;
public class ExampleClientThatLocks {
	private final InterProcessMutex lock;
	private final FakeLimitedResource resource;
	private final String clientName;
	public ExampleClientThatLocks(CuratorFramework client, String lockPath, FakeLimitedResource resource, String clientName) {
		this.resource = resource;
		this.clientName = clientName;
		lock = new InterProcessMutex(client, lockPath);
	}
	public void doWork(long time, TimeUnit unit) throws Exception {
		if (!lock.acquire(time, unit)) {
			throw new IllegalStateException(clientName + " could not acquire the lock");
		}
		try {
			System.out.println(clientName + " has the lock");
			resource.use(); //access resource exclusively
		} finally {
			System.out.println(clientName + " releasing the lock");
			lock.release(); // always release the lock in a finally block
		}
	}
}
```
最后创建主程序来测试。
```java
package com.colobu.zkrecipe.lock;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.curator.test.TestingServer;
import org.apache.curator.utils.CloseableUtils;
public class InterProcessMutexExample {
	private static final int QTY = 5;
	private static final int REPETITIONS = QTY * 10;
	private static final String PATH = "/examples/locks";
	public static void main(String[] args) throws Exception {
		final FakeLimitedResource resource = new FakeLimitedResource();
		ExecutorService service = Executors.newFixedThreadPool(QTY);
		final TestingServer server = new TestingServer();
		try {
			for (int i = 0; i < QTY; ++i) {
				final int index = i;
				Callable<Void> task = new Callable<Void>() {
					@Override
					public Void call() throws Exception {
						CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
						try {
							client.start();
							final ExampleClientThatLocks example = new ExampleClientThatLocks(client, PATH, resource, "Client " + index);
							for (int j = 0; j < REPETITIONS; ++j) {
								example.doWork(10, TimeUnit.SECONDS);
							}
						} catch (Throwable e) {
							e.printStackTrace();
						} finally {
							CloseableUtils.closeQuietly(client);
						}
						return null;
					}
				};
				service.submit(task);
			}
			service.shutdown();
			service.awaitTermination(10, TimeUnit.MINUTES);
		} finally {
			CloseableUtils.closeQuietly(server);
		}
	}
}
```

代码也很简单，生成10个client， 每个client重复执行10次 请求锁--访问资源--释放锁的过程。每个client都在独立的线程中。
结果可以看到，锁是随机的被每个实例排他性的使用。

既然是可重用的，你可以在一个线程中多次调用acquire,在线程拥有锁时它总是返回true。

你不应该在多个线程中用同一个InterProcessMutex， 你可以在每个线程中都生成一个InterProcessMutex实例，它们的path都一样，这样它们可以共享同一个锁。
## 不可重入锁Shared Lock
这个锁和上面的相比，就是少了Reentrant的功能，也就意味着它不能在同一个线程中重入。
这个类是InterProcessSemaphoreMutex。
使用方法和上面的类类似。

首先我们将上面的例子修改一下，测试一下它的重入。
修改ExampleClientThatLocks.doWork,连续两次acquire:
```java
public void doWork(long time, TimeUnit unit) throws Exception {
	if (!lock.acquire(time, unit)) {
		throw new IllegalStateException(clientName + " could not acquire the lock");
	}
	System.out.println(clientName + " has the lock");
	if (!lock.acquire(time, unit)) {
		throw new IllegalStateException(clientName + " could not acquire the lock");
	}
	System.out.println(clientName + " has the lock again");
	
	try {			
		resource.use(); //access resource exclusively
	} finally {
		System.out.println(clientName + " releasing the lock");
		lock.release(); // always release the lock in a finally block
		lock.release(); // always release the lock in a finally block
	}
}
```

注意我们也需要调用release两次。这和JDK的ReentrantLock用法一致。如果少调用一次release，则此线程依然拥有锁。
上面的代码没有问题，我们可以多次调用acquire，后续的acquire也不会阻塞。
将上面的InterProcessMutex换成不可重入锁InterProcessSemaphoreMutex,如果再运行上面的代码，结果就会发现线程被阻塞再第二个acquire上。 也就是此锁不是可重入的。

## 可重入读写锁Shared Reentrant Read Write Lock
类似JDK的ReentrantReadWriteLock.
一个读写锁管理一对相关的锁。 一个负责读操作，另外一个负责写操作。 读操作在写锁没被使用时可同时由多个进程使用，而写锁使用时不允许读 (阻塞)。
此锁是可重入的。一个拥有写锁的线程可重入读锁，但是读锁却不能进入写锁。
这也意味着写锁可以降级成读锁， 比如请求写锁 --->读锁 ---->释放写锁。 从读锁升级成写锁是不成的。

主要由两个类实现：

    InterProcessReadWriteLock
    InterProcessLock

使用时首先创建一个InterProcessReadWriteLock实例，然后再根据你的需求得到读锁或者写锁， 读写锁的类型是InterProcessLock。
```java
public InterProcessLock readLock()
public InterProcessLock writeLock()
```
例子和上面的类似。
```java
package com.colobu.zkrecipe.lock;
import java.util.concurrent.TimeUnit;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.recipes.locks.InterProcessMutex;
import org.apache.curator.framework.recipes.locks.InterProcessReadWriteLock;
import org.apache.curator.framework.recipes.locks.InterProcessSemaphoreMutex;
public class ExampleClientReadWriteLocks {
	private final InterProcessReadWriteLock lock;
	private final InterProcessMutex readLock;
	private final InterProcessMutex writeLock;
	private final FakeLimitedResource resource;
	private final String clientName;
	public ExampleClientReadWriteLocks(CuratorFramework client, String lockPath, FakeLimitedResource resource, String clientName) {
		this.resource = resource;
		this.clientName = clientName;
		lock = new InterProcessReadWriteLock(client, lockPath);
		readLock = lock.readLock();
		writeLock = lock.writeLock();
	}
	public void doWork(long time, TimeUnit unit) throws Exception {
		if (!writeLock.acquire(time, unit)) {
			throw new IllegalStateException(clientName + " could not acquire the writeLock");
		}
		System.out.println(clientName + " has the writeLock");
		
		if (!readLock.acquire(time, unit)) {
			throw new IllegalStateException(clientName + " could not acquire the readLock");
		}
		System.out.println(clientName + " has the readLock too");
		
		try {			
			resource.use(); //access resource exclusively
		} finally {
			System.out.println(clientName + " releasing the lock");
			readLock.release(); // always release the lock in a finally block
			writeLock.release(); // always release the lock in a finally block
		}
	}
}
```

在这个类中我们首先请求了一个写锁， 然后降级成读锁。 执行业务处理，然后释放读写锁。

## 信号量Shared Semaphore
一个计数的信号量类似JDK的Semaphore。 JDK中Semaphore维护的一组许可(permits)，而Cubator中称之为租约(Lease)。
有两种方式可以决定semaphore的最大租约数。第一种方式是有用户给定的path决定。第二种方式使用SharedCountReader类。
如果不使用SharedCountReader, 没有内部代码检查进程是否假定有10个租约而进程B假定有20个租约。 所以所有的实例必须使用相同的numberOfLeases值.

这次调用acquire会返回一个租约对象。 客户端必须在finally中close这些租约对象，否则这些租约会丢失掉。 但是， 但是，如果客户端session由于某种原因比如crash丢掉， 那么这些客户端持有的租约会自动close， 这样其它客户端可以继续使用这些租约。
租约还可以通过下面的方式返还：
```java
public void returnAll(Collection<Lease> leases)
public void returnLease(Lease lease)
```
注意一次你可以请求多个租约，如果Semaphore当前的租约不够，则请求线程会被阻塞。 同时还提供了超时的重载方法。
```java
public Lease acquire()
public Collection<Lease> acquire(int qty)
public Lease acquire(long time, TimeUnit unit)
public Collection<Lease> acquire(int qty, long time, TimeUnit unit)
```
主要类有:

    InterProcessSemaphoreV2
    Lease
    SharedCountReader

下面是使用的例子：
```java
package com.colobu.zkrecipe.lock;
import java.util.Collection;
import java.util.concurrent.TimeUnit;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.locks.InterProcessSemaphoreV2;
import org.apache.curator.framework.recipes.locks.Lease;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.curator.test.TestingServer;
import org.apache.curator.utils.CloseableUtils;
public class InterProcessSemaphoreExample {
	private static final int MAX_LEASE = 10;
	private static final String PATH = "/examples/locks";
	public static void main(String[] args) throws Exception {
		FakeLimitedResource resource = new FakeLimitedResource();
		try (TestingServer server = new TestingServer()) {
			CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
			client.start();
			InterProcessSemaphoreV2 semaphore = new InterProcessSemaphoreV2(client, PATH, MAX_LEASE);
			Collection<Lease> leases = semaphore.acquire(5);
			System.out.println("get " + leases.size() + " leases");
			Lease lease = semaphore.acquire();
			System.out.println("get another lease");
			resource.use();
			Collection<Lease> leases2 = semaphore.acquire(5, 10, TimeUnit.SECONDS);
			System.out.println("Should timeout and acquire return " + leases2);
			System.out.println("return one lease");
			semaphore.returnLease(lease);
			System.out.println("return another 5 leases");
			semaphore.returnAll(leases);
		}
	}
}
```
首先我们先获得了5个租约， 最后我们把它还给了semaphore。
接着请求了一个租约，因为semaphore还有5个租约，所以请求可以满足，返回一个租约，还剩4个租约。
然后再请求一个租约，因为租约不够，阻塞到超时，还是没能满足，返回结果为null。

上面说讲的锁都是公平锁(fair)。 总ZooKeeper的角度看， 每个客户端都按照请求的顺序获得锁。 相当公平。
## 多锁对象 Multi Shared Lock
Multi Shared Lock是一个锁的容器。 当调用acquire， 所有的锁都会被acquire，如果请求失败，所有的锁都会被release。 同样调用release时所有的锁都被release(失败被忽略)。
基本上，它就是组锁的代表，在它上面的请求释放操作都会传递给它包含的所有的锁。

主要涉及两个类：

    InterProcessMultiLock
    InterProcessLock

它的构造函数需要包含的锁的集合，或者一组ZooKeeper的path。
```java
public InterProcessMultiLock(List<InterProcessLock> locks)
public InterProcessMultiLock(CuratorFramework client, List<String> paths)
```
用法和Shared Lock相同。

例子如下：
```java
package com.colobu.zkrecipe.lock;
import java.util.Arrays;
import java.util.concurrent.TimeUnit;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.locks.InterProcessLock;
import org.apache.curator.framework.recipes.locks.InterProcessMultiLock;
import org.apache.curator.framework.recipes.locks.InterProcessMutex;
import org.apache.curator.framework.recipes.locks.InterProcessSemaphoreMutex;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.curator.test.TestingServer;
public class InterProcessMultiLockExample {
	private static final String PATH1 = "/examples/locks1";
	private static final String PATH2 = "/examples/locks2";
	
	public static void main(String[] args) throws Exception {
		FakeLimitedResource resource = new FakeLimitedResource();
		try (TestingServer server = new TestingServer()) {
			CuratorFramework client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
			client.start();
			
			InterProcessLock lock1 = new InterProcessMutex(client, PATH1);
			InterProcessLock lock2 = new InterProcessSemaphoreMutex(client, PATH2);
			
			InterProcessMultiLock lock = new InterProcessMultiLock(Arrays.asList(lock1, lock2));
			if (!lock.acquire(10, TimeUnit.SECONDS)) {
				throw new IllegalStateException("could not acquire the lock");
			}
			System.out.println("has the lock");
			
			System.out.println("has the lock1: " + lock1.isAcquiredInThisProcess());
			System.out.println("has the lock2: " + lock2.isAcquiredInThisProcess());
			
			try {			
				resource.use(); //access resource exclusively
			} finally {
				System.out.println("releasing the lock");
				lock.release(); // always release the lock in a finally block
			}
			System.out.println("has the lock1: " + lock1.isAcquiredInThisProcess());
			System.out.println("has the lock2: " + lock2.isAcquiredInThisProcess());
		}
	}
}
```
新建一个InterProcessMultiLock， 包含一个重入锁和一个非重入锁。
调用acquire后可以看到线程同时拥有了这两个锁。
调用release看到这两个锁都被释放了。

再重申以便， 强烈推荐使用ConnectionStateListener监控连接的状态。 