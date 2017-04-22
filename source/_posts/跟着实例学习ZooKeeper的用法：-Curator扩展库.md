---
title: 跟着实例学习ZooKeeper的用法： Curator扩展库
date: 2017-04-22 23:23:03
category: zookeeper
tags: [java,zookeeper]
---
还记得Curator提供哪几个组件吗？ 我们不妨回顾一下：

    Recipes
    Framework
    Utilities
    Client
    Errors
    Extensions
Recipes组件包含了丰富的Curator应用的组件。 但是这些并不是ZooKeeper Recipe的全部。 大量的分布式应用已经抽象出了许许多多的的Recipe，其中有些还是可以通过Curator来实现。
如果不断都将这些Recipe都增加到Recipes中， Recipes会变得越来越大。 为了避免这种状况， Curator把一些其它的Recipe放在单独的包中， 命名方式就是curator-x-,比如curator-x-discovery， curator-x-rpc。
本文就是介绍curator-x-discovery。
这是一个服务发现的Recipe。
我们在介绍临时节点Ephemeral Node的时候就讲到， 可以通过临时节点创建一个服务注册机制。 服务启动后创建临时节点， 服务断掉后临时节点就不存在了。 这个扩展抽象了这种功能，听过了一套API,可以实现服务发现机制。

# 服务类
我们先介绍一下例子中的服务类。
InstanceDetails定义了服务实例的基本信息,实际中可能会定义更详细的信息。
```java
package com.colobu.zkrecipe.discovery;
import org.codehaus.jackson.map.annotate.JsonRootName;
/**
 * In a real application, the Service payload will most likely be more detailed
 * than this. But, this gives a good example.
 */
@JsonRootName("details")
public class InstanceDetails {
	private String description;
	public InstanceDetails() {
		this("");
	}
	public InstanceDetails(String description) {
		this.description = description;
	}
	public void setDescription(String description) {
		this.description = description;
	}
	public String getDescription() {
		return description;
	}
}
```
ExampleServer相当与你在分布式环境中的服务应用。 每个服务应用实例都类似这个类， 应用启动时调用start， 关闭时调用close。
```java
package com.colobu.zkrecipe.discovery;
import java.io.Closeable;
import java.io.IOException;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.utils.CloseableUtils;
import org.apache.curator.x.discovery.ServiceDiscovery;
import org.apache.curator.x.discovery.ServiceDiscoveryBuilder;
import org.apache.curator.x.discovery.ServiceInstance;
import org.apache.curator.x.discovery.UriSpec;
import org.apache.curator.x.discovery.details.JsonInstanceSerializer;
/**
 * This shows a very simplified method of registering an instance with the
 * service discovery. Each individual instance in your distributed set of
 * applications would create an instance of something similar to ExampleServer,
 * start it when the application comes up and close it when the application
 * shuts down.
 */
public class ExampleServer implements Closeable {
	private final ServiceDiscovery<InstanceDetails> serviceDiscovery;
	private final ServiceInstance<InstanceDetails> thisInstance;
	public ExampleServer(CuratorFramework client, String path, String serviceName, String description) throws Exception {
		// in a real application, you'd have a convention of some kind for the
		// URI layout
		UriSpec uriSpec = new UriSpec("{scheme}://foo.com:{port}");
		thisInstance = ServiceInstance.<InstanceDetails> builder().name(serviceName).payload(new InstanceDetails(description))
				.port((int) (65535 * Math.random())) // in a real application,
														// you'd use a common
														// port
				.uriSpec(uriSpec).build();
		// if you mark your payload class with @JsonRootName the provided
		// JsonInstanceSerializer will work
		JsonInstanceSerializer<InstanceDetails> serializer = new JsonInstanceSerializer<InstanceDetails>(InstanceDetails.class);
		serviceDiscovery = ServiceDiscoveryBuilder.builder(InstanceDetails.class).client(client).basePath(path).serializer(serializer)
				.thisInstance(thisInstance).build();
	}
	public ServiceInstance<InstanceDetails> getThisInstance() {
		return thisInstance;
	}
	public void start() throws Exception {
		serviceDiscovery.start();
	}
	@Override
	public void close() throws IOException {
		CloseableUtils.closeQuietly(serviceDiscovery);
	}
}
```

# 发现中心
DiscoveryExample提供了增加，删除，显示，注册已有的服务的功能。
注意此处服务注册是由ExampleServer自己完成的， 这比较符合实际的情况。 实际情况是服务自己起来后主动注册服务。 但是此处启动又是由DiscoveryExample来调用， 纯粹为了演示使用。 你可以根据你自己的情况合理安排服务的注册和启动。

random命令提供了一个完全由DiscoveryExample控制的服务。 它负责注册一个服务并启动。

调用close就关闭了服务。
```java
package com.colobu.zkrecipe.discovery;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.Arrays;
import java.util.Collection;
import java.util.List;
import java.util.Map;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.apache.curator.test.TestingServer;
import org.apache.curator.utils.CloseableUtils;
import org.apache.curator.x.discovery.ServiceDiscovery;
import org.apache.curator.x.discovery.ServiceDiscoveryBuilder;
import org.apache.curator.x.discovery.ServiceInstance;
import org.apache.curator.x.discovery.ServiceProvider;
import org.apache.curator.x.discovery.details.JsonInstanceSerializer;
import org.apache.curator.x.discovery.strategies.RandomStrategy;
import com.google.common.base.Predicate;
import com.google.common.collect.Iterables;
import com.google.common.collect.Lists;
import com.google.common.collect.Maps;
public class DiscoveryExample {
	private static final String PATH = "/discovery/example";
	public static void main(String[] args) throws Exception {
		// This method is scaffolding to get the example up and running
		TestingServer server = new TestingServer();
		CuratorFramework client = null;
		ServiceDiscovery<InstanceDetails> serviceDiscovery = null;
		Map<String, ServiceProvider<InstanceDetails>> providers = Maps.newHashMap();
		try {
			client = CuratorFrameworkFactory.newClient(server.getConnectString(), new ExponentialBackoffRetry(1000, 3));
			client.start();
			JsonInstanceSerializer<InstanceDetails> serializer = new JsonInstanceSerializer<InstanceDetails>(InstanceDetails.class);
			serviceDiscovery = ServiceDiscoveryBuilder.builder(InstanceDetails.class).client(client).basePath(PATH).serializer(serializer).build();
			serviceDiscovery.start();
			processCommands(serviceDiscovery, providers, client);
		} finally {
			for (ServiceProvider<InstanceDetails> cache : providers.values()) {
				CloseableUtils.closeQuietly(cache);
			}
			CloseableUtils.closeQuietly(serviceDiscovery);
			CloseableUtils.closeQuietly(client);
			CloseableUtils.closeQuietly(server);
		}
	}
	private static void processCommands(ServiceDiscovery<InstanceDetails> serviceDiscovery, Map<String, ServiceProvider<InstanceDetails>> providers,
			CuratorFramework client) throws Exception {
		// More scaffolding that does a simple command line processor
		printHelp();
		List<ExampleServer> servers = Lists.newArrayList();
		try {
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
				} else if (operation.equals("add")) {
					addInstance(args, client, command, servers);
				} else if (operation.equals("delete")) {
					deleteInstance(args, command, servers);
				} else if (operation.equals("random")) {
					listRandomInstance(args, serviceDiscovery, providers, command);
				} else if (operation.equals("list")) {
					listInstances(serviceDiscovery);
				}
			}
		} finally {
			for (ExampleServer server : servers) {
				CloseableUtils.closeQuietly(server);
			}
		}
	}
	private static void listRandomInstance(String[] args, ServiceDiscovery<InstanceDetails> serviceDiscovery,
			Map<String, ServiceProvider<InstanceDetails>> providers, String command) throws Exception {
		// this shows how to use a ServiceProvider
		// in a real application you'd create the ServiceProvider early for the
		// service(s) you're interested in
		if (args.length != 1) {
			System.err.println("syntax error (expected random <name>): " + command);
			return;
		}
		String serviceName = args[0];
		ServiceProvider<InstanceDetails> provider = providers.get(serviceName);
		if (provider == null) {
			provider = serviceDiscovery.serviceProviderBuilder().serviceName(serviceName).providerStrategy(new RandomStrategy<InstanceDetails>()).build();
			providers.put(serviceName, provider);
			provider.start();
			Thread.sleep(2500); // give the provider time to warm up - in a real
								// application you wouldn't need to do this
		}
		ServiceInstance<InstanceDetails> instance = provider.getInstance();
		if (instance == null) {
			System.err.println("No instances named: " + serviceName);
		} else {
			outputInstance(instance);
		}
	}
	private static void listInstances(ServiceDiscovery<InstanceDetails> serviceDiscovery) throws Exception {
		// This shows how to query all the instances in service discovery
		try {
			Collection<String> serviceNames = serviceDiscovery.queryForNames();
			System.out.println(serviceNames.size() + " type(s)");
			for (String serviceName : serviceNames) {
				Collection<ServiceInstance<InstanceDetails>> instances = serviceDiscovery.queryForInstances(serviceName);
				System.out.println(serviceName);
				for (ServiceInstance<InstanceDetails> instance : instances) {
					outputInstance(instance);
				}
			}
		} finally {
			CloseableUtils.closeQuietly(serviceDiscovery);
		}
	}
	private static void outputInstance(ServiceInstance<InstanceDetails> instance) {
		System.out.println("\t" + instance.getPayload().getDescription() + ": " + instance.buildUriSpec());
	}
	private static void deleteInstance(String[] args, String command, List<ExampleServer> servers) {
		// simulate a random instance going down
		// in a real application, this would occur due to normal operation, a
		// crash, maintenance, etc.
		if (args.length != 1) {
			System.err.println("syntax error (expected delete <name>): " + command);
			return;
		}
		final String serviceName = args[0];
		ExampleServer server = Iterables.find(servers, new Predicate<ExampleServer>() {
			@Override
			public boolean apply(ExampleServer server) {
				return server.getThisInstance().getName().endsWith(serviceName);
			}
		}, null);
		if (server == null) {
			System.err.println("No servers found named: " + serviceName);
			return;
		}
		servers.remove(server);
		CloseableUtils.closeQuietly(server);
		System.out.println("Removed a random instance of: " + serviceName);
	}
	private static void addInstance(String[] args, CuratorFramework client, String command, List<ExampleServer> servers) throws Exception {
		// simulate a new instance coming up
		// in a real application, this would be a separate process
		if (args.length < 2) {
			System.err.println("syntax error (expected add <name> <description>): " + command);
			return;
		}
		StringBuilder description = new StringBuilder();
		for (int i = 1; i < args.length; ++i) {
			if (i > 1) {
				description.append(' ');
			}
			description.append(args[i]);
		}
		String serviceName = args[0];
		ExampleServer server = new ExampleServer(client, PATH, serviceName, description.toString());
		servers.add(server);
		server.start();
		System.out.println(serviceName + " added");
	}
	private static void printHelp() {
		System.out.println("An example of using the ServiceDiscovery APIs. This example is driven by entering commands at the prompt:\n");
		System.out.println("add <name> <description>: Adds a mock service with the given name and description");
		System.out.println("delete <name>: Deletes one of the mock services with the given name");
		System.out.println("list: Lists all the currently registered services");
		System.out.println("random <name>: Lists a random instance of the service with the given name");
		System.out.println("quit: Quit the example");
		System.out.println();
	}
}
```
# 其它扩展
其它两个扩展Curator RPC Proxy（curator-x-rpc）扩展和Service Discovery Server（curator-x-discovery-server）是为了桥接非Java应用的扩展，本系列将不再介绍了。感兴趣的朋友可以看下面的文档。
**Curator Service Discovery**
**Curator RPC Proxy**