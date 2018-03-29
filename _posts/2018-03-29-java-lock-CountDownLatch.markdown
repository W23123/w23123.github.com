---
layout: post
title: CountDownLatch
date: 2018-03-29 00:20:20 +0300
description: 又简单的总结了一遍
img: i-rest.jpg # Add image post (optional)
tags: [Java,Lock]
---

- [简介](#简介)
- [示例](#示例)
- [源码分析](#源码分析)

---

#### 简介

CountDownLatch是同步协助，允许一个或多个线程等待，直到在其他线程中执行的一组操作完成。

CountDownLatch用给定的计数进行初始化。该await方法被阻塞，直到当前的计数达到零(countDown)，所有被阻塞(await)的线程将执行。一个CountDownLatch初始化为N可以用来做一个线程等待，直到N线程完成一些动作，或某些动作已经完成N次。
<p style='color:red'>注意: count不能重置，如果想重置 可以用CyclicBarrier</p>

---

#### 示例

```java
/**
 * Created by dongbin on 2018/3/13.
 * 代码来源 java特种兵
 */
public class CountDownLatchTest {

    private final static int GROUP_SIZE = 5;
    private final static Random RANDOM = new Random();

    private static void processOneGroup(final String groupName) throws InterruptedException {
        //等待所有人就绪
        final CountDownLatch preDownLatch = new CountDownLatch(GROUP_SIZE);
        final CountDownLatch startDownLatch = new CountDownLatch(1);
        final CountDownLatch endDownLatch = new CountDownLatch(GROUP_SIZE);

        System.out.println("=========>\n分组：" + groupName + "比赛开始：");

        for (int i = 0; i < GROUP_SIZE; i++) {
            new Thread(String.valueOf(i)) {
                public void run() {
                    preDownLatch.countDown();
                    System.out.println("我是线程组【" + groupName + "】,我是第" + this.getName() + "线程，准备就绪");
                    try {
                        startDownLatch.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    try {
                        Thread.sleep(Math.abs(RANDOM.nextInt()) % 1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("我是线程组【" + groupName + "】,我是第" + this.getName() + "线程，执行完成");
                    endDownLatch.countDown();
                }
            }.start();
        }

        preDownLatch.await();//等待所有选手就位
        System.out.println("各就各位,预备跑！！！");
        startDownLatch.countDown();//开跑
        endDownLatch.await();//等待所有选手跑完
        System.out.println(groupName+"已跑完！");

    }

    public static void main(String[] args) throws InterruptedException {
        processOneGroup("线程组1");
        processOneGroup("线程组2");
    }

}

```
执行结果：
```
=========>
分组：线程组1比赛开始：
我是线程组【线程组1】,我是第0线程，准备就绪
我是线程组【线程组1】,我是第1线程，准备就绪
我是线程组【线程组1】,我是第2线程，准备就绪
我是线程组【线程组1】,我是第3线程，准备就绪
我是线程组【线程组1】,我是第4线程，准备就绪
各就各位,预备跑！！！
我是线程组【线程组1】,我是第3线程，执行完成
我是线程组【线程组1】,我是第0线程，执行完成
我是线程组【线程组1】,我是第4线程，执行完成
我是线程组【线程组1】,我是第1线程，执行完成
我是线程组【线程组1】,我是第2线程，执行完成
线程组1已跑完！
=========>
分组：线程组2比赛开始：
我是线程组【线程组2】,我是第0线程，准备就绪
我是线程组【线程组2】,我是第1线程，准备就绪
我是线程组【线程组2】,我是第2线程，准备就绪
我是线程组【线程组2】,我是第3线程，准备就绪
我是线程组【线程组2】,我是第4线程，准备就绪
各就各位,预备跑！！！
我是线程组【线程组2】,我是第2线程，执行完成
我是线程组【线程组2】,我是第3线程，执行完成
我是线程组【线程组2】,我是第0线程，执行完成
我是线程组【线程组2】,我是第4线程，执行完成
Disconnected from the target VM, address: '127.0.0.1:64955', transport: 'socket'
我是线程组【线程组2】,我是第1线程，执行完成
线程组2已跑完！
```
---

#### 源码分析

先看看countdown()方法：
```java
public void countDown() {
        sync.releaseShared(1);
    }
```
调用还是AQS的releaseShared方法，真正实现释放的实现再CountDownLatch中，源码：
```java
protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
```
这个地方用的时典型的CAS模式，一直自旋到state减1为止。

await源码在上一篇中已分析，主要用的是AQS提供的公共方法。当调用await后，获取state 是否等于0，如果等于0直接继续执行接下来的代码，否者返回-1 进入到自旋获取state，直到state为0时。


