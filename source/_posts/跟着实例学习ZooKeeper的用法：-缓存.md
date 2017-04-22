---
title: 跟着实例学习ZooKeeper的用法： 缓存
date: 2017-04-22 22:43:27
category: zookeeper
tags: [java,zookeeper]
---
可以利用ZooKeeper在集群的各个节点之间缓存数据。 每个节点都可以得到最新的缓存的数据。 Curator提供了三种类型的缓存方式：Path Cache,Node Cache 和Tree Cache。

# Path Cache
Path Cache用来监控一个ZNode的子节点. 当一个子节点增加， 更新，删除时， Path Cache会改变它的状态， 会包含最新的子节点， 子节点的数据和状态。
这也正如它的名字表示的那样， 那监控path。

实际使用时会涉及到四个类：

    PathChildrenCache
    PathChildrenCacheEvent
    PathChildrenCacheListener
    ChildData

通过下面的构造函数创建Path Cache:
```java
public PathChildrenCache(CuratorFramework client, String path, boolean cacheData)
```
想使用cache，必须调用它的start方法，不用之后调用close方法。
start有两个， 其中一个可以传入StartMode，用来为初始的cache设置暖场方式(warm)：

    NORMAL: 初始时为空。
    BUILD_INITIAL_CACHE: 在这个方法返回之前调用rebuild()。
    POST_INITIALIZED_EVENT: 当Cache初始化数据后发送一个PathChildrenCacheEvent.Type#INITIALIZED事件

public void addListener(PathChildrenCacheListener listener)可以增加listener监听缓存的改变。

getCurrentData()方法返回一个List对象，可以遍历所有的子节点。

这个例子摘自官方的例子， 实现了一个控制台的方式操作缓存。 它提供了三个命令， 你可以在控制台中输入。

    set 用来新增或者更新一个子节点的值， 也就是更新一个缓存对象
    remove 是删除一个缓存对象
    list 列出所有的缓存对象

另外还提供了一个help命令提供帮助。

设置/更新、移除其实是使用client (CuratorFramework)来操作, 不通过PathChildrenCache操作：
```java
client.setData().forPath(path, bytes);
client.create().creatingParentsIfNeeded().forPath(path, bytes);
client.delete().forPath(path);
```
而查询缓存使用下面的方法：
```java
for (ChildData data : cache.getCurrentData()) {
	System.out.println(data.getPath() + " = " + new String(data.getData()));
}
```
# Node Cache
Path Cache用来监控一个ZNode. 当节点的数据修改或者删除时，Node Cache能更新它的状态包含最新的改变。

涉及到下面的三个类：

    NodeCache
    NodeCacheListener
    ChildData

想使用cache，依然要调用它的start方法，不用之后调用close方法。

getCurrentData()将得到节点当前的状态，通过它的状态可以得到当前的值。
可以使用public void addListener(NodeCacheListener listener)监控节点状态的改变。

我们依然使用上面的例子框架来演示Node Cache。
```java
package com.colobu.zkrecipe.cache;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.Arrays;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.cache.NodeCache;
import org.apache.curator.framework.recipes.cache.NodeCacheListener;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.curator.test.TestingServer;
import org.apache.curator.utils.CloseableUtils;
import org.apache.zookeeper.KeeperException;
public class NodeCacheExample {
	private static final String PATH = "/example/nodeCache";
	public static void main(String[] args) throws Exception {
		TestingServer server = new TestingServer();
		CuratorFramework client = null;
		NodeCache cache = null;
		try {
			client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
			client.start();
			cache = new NodeCache(client, PATH);
			cache.start();
			processCommands(client, cache);
		} finally {
			CloseableUtils.closeQuietly(cache);
			CloseableUtils.closeQuietly(client);
			CloseableUtils.closeQuietly(server);
		}
	}
	private static void addListener(final NodeCache cache) {
		// a PathChildrenCacheListener is optional. Here, it's used just to log
		// changes
		NodeCacheListener listener = new NodeCacheListener() {
			@Override
			public void nodeChanged() throws Exception {
				if (cache.getCurrentData() != null)
					System.out.println("Node changed: " + cache.getCurrentData().getPath() + ", value: " + new String(cache.getCurrentData().getData()));
			}
		};
		cache.getListenable().addListener(listener);
	}
	private static void processCommands(CuratorFramework client, NodeCache cache) throws Exception {
		printHelp();
		try {
			addListener(cache);
			BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
			boolean done = false;
			while (!done) {
				System.out.print("> ");
				String line = in.readLine();
				if (line == null) {
					break;
				}
				String command = line.trim();
				String[] parts = command.split("\\s");
				if (parts.length == 0) {
					continue;
				}
				String operation = parts[0];
				String args[] = Arrays.copyOfRange(parts, 1, parts.length);
				if (operation.equalsIgnoreCase("help") || operation.equalsIgnoreCase("?")) {
					printHelp();
				} else if (operation.equalsIgnoreCase("q") || operation.equalsIgnoreCase("quit")) {
					done = true;
				} else if (operation.equals("set")) {
					setValue(client, command, args);
				} else if (operation.equals("remove")) {
					remove(client);
				} else if (operation.equals("show")) {
					show(cache);
				}
				Thread.sleep(1000); // just to allow the console output to catch
									// up
			}
		} catch (Exception ex) {
			ex.printStackTrace();
		} finally {
		}
	}
	private static void show(NodeCache cache) {
		if (cache.getCurrentData() != null)
			System.out.println(cache.getCurrentData().getPath() + " = " + new String(cache.getCurrentData().getData()));
		else
			System.out.println("cache don't set a value");
	}
	private static void remove(CuratorFramework client) throws Exception {
		try {
			client.delete().forPath(PATH);
		} catch (KeeperException.NoNodeException e) {
			// ignore
		}
	}
	private static void setValue(CuratorFramework client, String command, String[] args) throws Exception {
		if (args.length != 1) {
			System.err.println("syntax error (expected set <value>): " + command);
			return;
		}
		byte[] bytes = args[0].getBytes();
		try {
			client.setData().forPath(PATH, bytes);
		} catch (KeeperException.NoNodeException e) {
			client.create().creatingParentsIfNeeded().forPath(PATH, bytes);
		}
	}
	private static void printHelp() {
		System.out.println("An example of using PathChildrenCache. This example is driven by entering commands at the prompt:\n");
		System.out.println("set <value>: Adds or updates a node with the given name");
		System.out.println("remove: Deletes the node with the given name");
		System.out.println("show: Display the node's value in the cache");
		System.out.println("quit: Quit the example");
		System.out.println();
	}
}
```

# Tree Node
这种类型的即可以监控节点的状态，还监控节点的子节点的状态， 类似上面两种cache的组合。 这也就是Tree的概念。 它监控整个树中节点的状态。
涉及到下面四个类。

    TreeCache
    TreeCacheListener
    TreeCacheEvent
    ChildData

而关键的TreeCache的构造函数为

```java
public TreeCache(CuratorFramework client, String path, boolean cacheData)
```
想使用cache，依然要调用它的start方法，不用之后调用close方法。

getCurrentChildren()返回cache的状态，类型为Map。 而getCurrentData()返回监控的path的数据。

public void addListener(TreeCacheListener listener)可以增加listener来监控状态的改变。

例子依然采用和上面的例子类似， 尤其和Path Cache类似。
```java
package com.colobu.zkrecipe.cache;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.Arrays;
import java.util.Map;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.cache.ChildData;
import org.apache.curator.framework.recipes.cache.TreeCache;
import org.apache.curator.framework.recipes.cache.TreeCacheEvent;
import org.apache.curator.framework.recipes.cache.TreeCacheListener;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.curator.test.TestingServer;
import org.apache.curator.utils.CloseableUtils;
import org.apache.curator.utils.ZKPaths;
import org.apache.zookeeper.KeeperException;
public class TreeCacheExample {
	private static final String PATH = "/example/treeCache";
	public static void main(String[] args) throws Exception {
		TestingServer server = new TestingServer();
		CuratorFramework client = null;
		TreeCache cache = null;
		try {
			client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
			client.start();
			cache = new TreeCache(client, PATH);
			cache.start();
			processCommands(client, cache);
		} finally {
			CloseableUtils.closeQuietly(cache);
			CloseableUtils.closeQuietly(client);
			CloseableUtils.closeQuietly(server);
		}
	}
	private static void addListener(final TreeCache cache) {
		TreeCacheListener listener = new TreeCacheListener() {
			@Override
			public void childEvent(CuratorFramework client, TreeCacheEvent event) throws Exception {
				switch (event.getType()) {
				case NODE_ADDED: {
					System.out.println("TreeNode added: " + ZKPaths.getNodeFromPath(event.getData().getPath()) + ", value: "
							+ new String(event.getData().getData()));
					break;
				}
				case NODE_UPDATED: {
					System.out.println("TreeNode changed: " + ZKPaths.getNodeFromPath(event.getData().getPath()) + ", value: "
							+ new String(event.getData().getData()));
					break;
				}
				case NODE_REMOVED: {
					System.out.println("TreeNode removed: " + ZKPaths.getNodeFromPath(event.getData().getPath()));
					break;
				}
				default:
					System.out.println("Other event: " + event.getType().name());
				}
			}
		};
		cache.getListenable().addListener(listener);
	}
	private static void processCommands(CuratorFramework client, TreeCache cache) throws Exception {
		// More scaffolding that does a simple command line processor
		printHelp();
		try {
			addListener(cache);
			BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
			boolean done = false;
			while (!done) {
				System.out.print("> ");
				String line = in.readLine();
				if (line == null) {
					break;
				}
				String command = line.trim();
				String[] parts = command.split("\\s");
				if (parts.length == 0) {
					continue;
				}
				String operation = parts[0];
				String args[] = Arrays.copyOfRange(parts, 1, parts.length);
				if (operation.equalsIgnoreCase("help") || operation.equalsIgnoreCase("?")) {
					printHelp();
				} else if (operation.equalsIgnoreCase("q") || operation.equalsIgnoreCase("quit")) {
					done = true;
				} else if (operation.equals("set")) {
					setValue(client, command, args);
				} else if (operation.equals("remove")) {
					remove(client, command, args);
				} else if (operation.equals("list")) {
					list(cache);
				}
				Thread.sleep(1000); // just to allow the console output to catch
									// up
			}
		} finally {
		}
	}
	private static void list(TreeCache cache) {
		if (cache.getCurrentChildren(PATH).size() == 0) {
			System.out.println("* empty *");
		} else {			
			for (Map.Entry<String, ChildData> entry : cache.getCurrentChildren(PATH).entrySet()) {
				System.out.println(entry.getKey() + " = " + new String(entry.getValue().getData()));
			}
		}
	}
	private static void remove(CuratorFramework client, String command, String[] args) throws Exception {
		if (args.length != 1) {
			System.err.println("syntax error (expected remove <path>): " + command);
			return;
		}
		String name = args[0];
		if (name.contains("/")) {
			System.err.println("Invalid node name" + name);
			return;
		}
		String path = ZKPaths.makePath(PATH, name);
		try {
			client.delete().forPath(path);
		} catch (KeeperException.NoNodeException e) {
			// ignore
		}
	}
	private static void setValue(CuratorFramework client, String command, String[] args) throws Exception {
		if (args.length != 2) {
			System.err.println("syntax error (expected set <path> <value>): " + command);
			return;
		}
		String name = args[0];
		if (name.contains("/")) {
			System.err.println("Invalid node name" + name);
			return;
		}
		String path = ZKPaths.makePath(PATH, name);
		byte[] bytes = args[1].getBytes();
		try {
			client.setData().forPath(path, bytes);
		} catch (KeeperException.NoNodeException e) {
			client.create().creatingParentsIfNeeded().forPath(path, bytes);
		}
	}
	private static void printHelp() {
		System.out.println("An example of using PathChildrenCache. This example is driven by entering commands at the prompt:\n");
		System.out.println("set <name> <value>: Adds or updates a node with the given name");
		System.out.println("remove <name>: Deletes the node with the given name");
		System.out.println("list: List the nodes/values in the cache");
		System.out.println("quit: Quit the example");
		System.out.println();
	}
}
```