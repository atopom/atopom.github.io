---
title: Java-CountDownLatch
date: 2014-04-05 13:08:16
tags:
- JDK5.0
- CountDownLatch
- Concurrent
- Thread
categories:
- Java
---

# CountDownLatch是什么？
> CountDownLatch 位于java.util.concurrent包下，是JDK5.0的新特性，根据包名便可知，它是用于处理多线程并发的工具类。

# CountDownLatch能干什么？
> CountDownLatch 能够使一个线程处于等待状态，直到它期望的其他线程达到预期，再恢复运行状态。

# CountDownLatch的运行原理
> CountDownLatch 是通过一个计数器来实现的，计数器的初始值为线程数。
每当一个线程完成任务后，计数器的值就会减1。当计数器的值为0时，它表示所线程已经完成任务，然后在闭锁上等待线程就恢复运行状态。

```
1. Main thread start
2. Create CountDownLatch for N threads
3. Create and start N threads
4. Main thread wait on latch
5. N threads completes there tasks are returns
6. Main thread resume execution
```

![CountDownLatch原理](/images/Java/CountDownLatch_Work_Mechanism.png)

# CountDownLatch主要API

1. 构造函数
```
//Constructs a CountDownLatch initialized with the given count.
public void CountDownLatch(int count) {…}
```

2. 让线程处于等待状态的函数
```
CountDownLatch.await()
```

3. 让线程恢复执行状态的函数（cnt=0时，线程恢复运行状态）
```
CountDownLatch.countDown()
```

# 示例
> 主线程启动三个子线程，完成各自的验证工作后，主线程再继续运行，使用CountDownLatch进行实现。

## 主程序入口
```
package com.atopom.jdk5.concurrent.countdownlatch;

public class _Main {
	public static void main(String[] args) {
		boolean result = false;
		try {
			result = ApplicationStartupUtil.checkExternalServices();
		} catch (Exception e) {
			e.printStackTrace();
		}
		System.out.println("External services validation completed !! Result was :: " + result);
	}
}
```

## 运行结果
```
Checking Cache Service
Checking Database Service
Checking Network Service
Database Service is UP
Cache Service is UP
Network Service is UP
External services validation completed !! Result was :: true
```

# 代码地址
> https://github.com/atopom/java_familiar_strange/tree/master/Code/CountDownLatchDemo
