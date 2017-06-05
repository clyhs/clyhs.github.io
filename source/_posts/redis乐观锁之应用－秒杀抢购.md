---
title: redis乐观锁之应用－秒杀抢购
date: 2017-06-05 15:05:39
category: redis
tags: redis
---
## 乐观锁
    基于数据版本（version）的记录机制实现的。即为数据增加一个版本标识，在基于数据库表的版本解决方案中，一般是通过为数据库表增加一个”version”字段来实现读取出数据时，将此版本号一同读出，之后更新时，对此版本号加1。此时，将提交数据的版本号与数据库表对应记录的当前版本号进行比对，如果提交的数据版本号大于数据库当前版本号，则予以更新，否则认为是过期数据。
    redis中可以使用watch命令会监视给定的key，当exec时候如果监视的key从调用watch后发生过变化，则整个事务会失败
    
## redis事务
    Redis中的事务(transaction)是一组命令的集合。事务同命令一样都是Redis最小的执行单位，一个事务中的命令要么都执行，要么都不执行。Redis事务的实现需要用到 MULTI 和 EXEC 两个命令，事务开始的时候先向Redis服务器发送 MULTI 命令，然后依次发送需要在本次事务中处理的命令，最后再发送 EXEC 命令表示事务命令结束。Redis的事务是下面4个命令来实现 
1.multi，开启Redis的事务，置客户端为事务态。 
2.exec，提交事务，执行从multi到此命令前的命令队列，置客户端为非事务态。 
3.discard，取消事务，置客户端为非事务态。 
4.watch,监视键值对，作用时如果事务提交exec时发现监视的监视对发生变化，事务将被取消。

```java
package com.github.distribute.lock.redis;

import java.util.List;
import java.util.Set;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.Transaction;

/**
 * redis乐观锁实例 
 * @author linbingwen
 *
 */
public class OptimisticLockTest {

	public static void main(String[] args) throws InterruptedException {
		 long starTime=System.currentTimeMillis();
		
		 initPrduct();
		 initClient();
		 printResult();
		 
		long endTime=System.currentTimeMillis();
		long Time=endTime-starTime;
		System.out.println("程序运行时间： "+Time+"ms");   

	}
	
	/**
	 * 输出结果
	 */
	public static void printResult() {
		Jedis jedis = RedisUtil.getInstance().getJedis();
		Set<String> set = jedis.smembers("clientList");

		int i = 1;
		for (String value : set) {
			System.out.println("第" + i++ + "个抢到商品，"+value + " ");
		}

		RedisUtil.returnResource(jedis);
	}

	/*
	 * 初始化顾客开始抢商品
	 */
	public static void initClient() {
		ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
		int clientNum = 10000;// 模拟客户数目
		for (int i = 0; i < clientNum; i++) {
			cachedThreadPool.execute(new ClientThread(i));
		}
		cachedThreadPool.shutdown();
		
		while(true){  
	            if(cachedThreadPool.isTerminated()){  
	                System.out.println("所有的线程都结束了！");  
	                break;  
	            }  
	            try {
					Thread.sleep(1000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}    
	        }  
	}

	/**
	 * 初始化商品个数
	 */
	public static void initPrduct() {
		int prdNum = 100;// 商品个数
		String key = "prdNum";
		String clientList = "clientList";// 抢购到商品的顾客列表
		Jedis jedis = RedisUtil.getInstance().getJedis();

		if (jedis.exists(key)) {
			jedis.del(key);
		}
		
		if (jedis.exists(clientList)) {
			jedis.del(clientList);
		}

		jedis.set(key, String.valueOf(prdNum));// 初始化
		RedisUtil.returnResource(jedis);
	}

}

/**
 * 顾客线程
 * 
 * @author linbingwen
 *
 */
class ClientThread implements Runnable {
	Jedis jedis = null;
	String key = "prdNum";// 商品主键
	String clientList = "clientList";//// 抢购到商品的顾客列表主键
	String clientName;

	public ClientThread(int num) {
		clientName = "编号=" + num;
	}

	public void run() {
		try {
			Thread.sleep((int)(Math.random()*5000));// 随机睡眠一下
		} catch (InterruptedException e1) {
		}
		while (true) {
			System.out.println("顾客:" + clientName + "开始抢商品");
			jedis = RedisUtil.getInstance().getJedis();
			try {
				jedis.watch(key);
				int prdNum = Integer.parseInt(jedis.get(key));// 当前商品个数
				if (prdNum > 0) {
					Transaction transaction = jedis.multi();
					transaction.set(key, String.valueOf(prdNum - 1));
					List<Object> result = transaction.exec();
					if (result == null || result.isEmpty()) {
						System.out.println("悲剧了，顾客:" + clientName + "没有抢到商品");// 可能是watch-key被外部修改，或者是数据操作被驳回
					} else {
						jedis.sadd(clientList, clientName);// 抢到商品记录一下
						System.out.println("好高兴，顾客:" + clientName + "抢到商品");
						break;
					}
				} else {
					System.out.println("悲剧了，库存为0，顾客:" + clientName + "没有抢到商品");
					break;
				}
			} catch (Exception e) {
				e.printStackTrace();
			} finally {
				jedis.unwatch();
				RedisUtil.returnResource(jedis);
			}

		}
	}

}

```


