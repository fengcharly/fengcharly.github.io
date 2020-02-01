---
title: Zookeeper实现分布式锁来获取唯一订单号
categories:
 - Zookeeper
tags:
 - 后端开发
description: 通常我们在购物的时候都会有一个订单号，那如果在高并发的情况下如何保证订单号的唯一性呢？比如秒杀抢购我我们既要保证性能的可靠(分布式)又要保证不生成重复的订单号...
---

#### 1.业务场景

​		通常我们在购物的时候都会有一个订单号，那如果在高并发的情况下如何保证订单号的唯一性呢？比如秒杀抢购我我们既要保证性能的可靠(分布式)又要保证不生成重复的订单号，这个时候我们就需要使用到分布式锁，这里我们介绍的分布式锁的实现方式是使用Zookeeper，关于Zookeeper我们这里就不做过多介绍，主要来看它如何实现分布式锁。

#### 2.什么是分布式锁

​		分布式锁是控制分布式系统之间同步访问共享资源的一种方式。在分布式系统中，常常需要协调他们的动作。如果不同的系统或是同一个系统的不同主机之间共享了一个或一组资源，那么访问这些资源的时候，往往需要互斥来防止彼此干扰来保证一致性，在这种情况下，便需要使用到分布式锁。（引用自百度百科）

#### 3.使用Zookeeper实现分布式锁

> 使用zookeeper创建临时序列节点来实现分布式锁，适用于顺序执行的程序，大体思路就是创建临时序列节点，找出最小的序列节点，获取分布式锁，程序执行完成之后此序列节点消失，通过watch来监控节点的变化，从剩下的节点的找到最小的序列节点，获取分布式锁，执行相应处理，依次类推……(这里我们简单起见,只创建临时节点,不做顺序的判断)

#### 4.实战项目

##### Maven依赖:

```java
	<dependencies>
		<dependency>
			<groupId>com.101tec</groupId>
			<artifactId>zkclient</artifactId>
			<version>0.10</version>
		</dependency>
	</dependencies>
```

##### OrderNumGenerator:

```java
//生成订单类
public class OrderNumGenerator {
    //全局订单id
    public static int count = 0;

    public String getNumber() {
        SimpleDateFormat simpt = new SimpleDateFormat("yyyy-MM-dd-HH-mm-ss");
        return simpt.format(new Date()) + "-" + ++count;
    }
}
```

##### Lock:

```java
/**
 * 自定义分布式锁
 */
public interface Lock {

    // 获取锁
    void getLock();

    // 释放锁
    void unlock();
}
```

##### ZookeeperAbstractLock:

```java
/**
 * 重构重复代码 将重复代码交给子类执行
 */
public abstract class ZookeeperAbstractLock implements Lock {

    // zk连接地址
    private static final String CONNECTSTRING = "127.0.0.1:2181";
    // 创建zk连接
    protected ZkClient zkClient = new ZkClient(CONNECTSTRING);
    protected static final String PATH = "/lock";


    @Override
    public void getLock() {
        if (tryLock()) {
            System.out.println("获取zk锁成功");
        } else {
            // 等待
            waitLock();
            // 重新获取锁
            getLock();
        }
    }

    // 等待锁
    protected abstract void waitLock();

    // 是否获取锁成功 如果成功返回true 失败返回false
    protected abstract boolean tryLock();

    @Override
    public void unlock() {
        if (zkClient != null) {
            zkClient.close();
            System.out.println("关闭连接 释放锁资源...");
        }
    }
}
```

##### ZookeeperDistrbuteLock:

```java
public class ZookeeperDistrbuteLock extends ZookeeperAbstractLock {

    private CountDownLatch countDownLatch = null;

    @Override
    protected boolean tryLock() {
        try {
            zkClient.createEphemeral(PATH);
            return true;
        } catch (RuntimeException e) {
            return false;
        }
    }

    @Override
    protected void waitLock() {
        // 使用事件监听 获取到节点被删除的通知
        IZkDataListener iZkDataListener = new IZkDataListener() {
            // 当节点被删除的时候
            @Override
            public void handleDataDeleted(String s) throws Exception {
                System.out.println("节点被删除了:" + s);
                if (countDownLatch != null) {
                    // 唤醒
                    countDownLatch.countDown();
                }
            }
            // 当节点发生改变的时候
            @Override
            public void handleDataChange(String s, Object o) throws Exception {

            }

        };
        // 注册节点信息 获取事件通知
        zkClient.subscribeDataChanges(PATH, iZkDataListener);
        if (zkClient.exists(PATH)) {
            // 创建信号量
            countDownLatch = new CountDownLatch(1);
            try {
                countDownLatch.await();
            } catch (Exception e) {
            }
        }
        // 删除事件通知
        zkClient.unsubscribeDataChanges(PATH, iZkDataListener);
    }
}
```

##### OrderSerice:

```java
// 订单生成调用业务逻辑
public class OrderSerice implements Runnable {

    // 生成订单号
    OrderNumGenerator orderNumGenerator = new OrderNumGenerator();

    private ZookeeperDistrbuteLock lock = new ZookeeperDistrbuteLock();

    @Override
    public void run() {
        getnNumber();
    }

    private void getnNumber() {
        try {
            lock.getLock();
            // 模拟用户生成订单号
            String number = orderNumGenerator.getNumber();

            System.out.println("当前线程:" + Thread.currentThread().getName()
                    + "  生成的订单号:" + number);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            // 释放锁
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        System.out.println("模拟生成订单号");
        for (int i = 0; i < 100; i++) {
            new Thread(new OrderSerice()).start();
        }
    }

}
```





 



