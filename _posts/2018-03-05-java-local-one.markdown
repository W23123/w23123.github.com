---
layout: post
title: Java中锁的分类（1）
date: 2018-03-05 00:00:20 +0300
description: 介绍了公平锁、自旋锁、可重入锁、悲观锁、锁消除等等
tags: [Lock,JAVA]
---

- [前言](#前言)
- [公平锁和非公平锁](#公平锁和非公平锁)
- [自旋锁](#自旋锁)
- [锁消除](#锁消除)
- [锁粗化](#锁粗化)
- [可重入锁](#可重入锁)
- [悲观锁和乐观锁](#悲观锁和乐观锁)
- [共享锁和排它锁](#共享锁和排它锁)
- [读写锁](#读写锁)
- [互斥锁](#互斥锁)

---

#### 前言
在平时工作中总是会遇到很多的锁，基本对这些锁都只停留在用的阶段，而且用的比较笼统，故而决定花些时间来总结下java中的锁。在接下来先看看java中的锁的概念，或者说分类也可以。

---

#### 公平锁和非公平锁
公平锁：是指多个线程在等待同一个锁时，必须按照申请锁的顺序来获得锁。
公平锁能避免有些线程饿死的情况，但是相对效率会低一点。非公平锁整体效率高一点，但是极有可能有本分线程一直没有执行的机会（获取不到锁）。

在java的ReentrantLock 的构造方法（后面分析）
```java
/**
     * Creates an instance of {@code ReentrantLock}.
     * This is equivalent to using {@code ReentrantLock(false)}.
     */
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    /**
     * Creates an instance of {@code ReentrantLock} with the
     * given fairness policy.
     *
     * @param fair {@code true} if this lock should use a fair ordering policy
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }

```
---

#### 自旋锁

自旋锁不是一种锁，而是一种锁的优化技术。它有2个值，‘解锁’和‘锁定’。当获取临界资源的锁，能获取，则进入，‘锁定’锁。否则，忙循环重新检查这个资源锁(并非睡眠或者阻塞，这是重要的区别，单处理器，自旋操作未空操作)

自旋锁的特点：

- 用于临界区互斥
- 任何时刻只能一个执行单元(线程)获得锁
- 要求持有锁处理器锁占时间尽量段
- 等待锁的线程进入忙循环(并不是阻塞或睡眠)

自旋锁在轻量级锁中，重量级锁中不会使用。

获取自旋锁自旋次数：
如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也是很有可能再次成功，进而它将允许自旋等待持续相对更长的时间，比如100次循环。另外，如果对于某个锁，自旋很少成功获得过，那在以后要获取这个锁时将可能省略掉自旋过程，以避免浪费处理器资源。

---

#### 锁消除

锁消除是Java虚拟机在JIT编译是，通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁，通过锁消除，可以节省毫无意义的请求锁时间。

看下面这段代码：

```java
public String getString(String s1, String s2, String s3){
      StringBuffer sb = new StringBuffer();
      sb.append(s1);
      sb.append(s2);
      sb.append(s3);
      return sb.toString();
}
```
StringBuffer的append方法：

```java
 @Override
    public synchronized StringBuffer append(Object obj) {
        toStringCache = null;
        super.append(String.valueOf(obj));
        return this;
    }
```
在getString方法中请求了同步方法，但是Stringbuffer在一个方法里，是局部变量。其他线程不可能访问到，不会出现`逃逸`的情况，所以锁是没必要的。在JIT编译的时候会去掉锁。

---

#### 锁粗化

原则上，我们在编写代码的时候，总是推荐将同步块的作用范围限制的尽量小——只在共享数据的实际作用域中才进行同步，这样是为了使得需要同步的操作数量尽可能变小，如果存在锁禁止，那等待的线程也能尽快拿到锁。大部分情况下，这些都是正确的。但是，如果一些列的联系操作都是同一个对象反复加上和解锁，甚至加锁操作是出现在循环体中的，那么即使没有线程竞争，频繁地进行互斥同步操作也导致不必要的性能损耗。

看下面的代码：
``` java
    Object object = new Object();
        for(int i=0;i<1000;i++){
            synchronized (object){
                doSomething();
            }
        }

```
锁粗话后：
```java
    Object object = new Object();
        synchronized (object){
            for(int i=0;i<1000;i++){
                doSomething();
            }
        }
```
虽然JVM能帮我们优化，但是这些语法要知道，而且尽量不要编写这样类似的代码。

---

### 可重入锁

可重入锁，也叫做递归锁，指的是同一线程外层函数获得锁之后，内层递归函数仍然有获取该锁的代码，但不受影响。

在JAVA环境下 ReentrantLock 和synchronized 都是可重入锁
```java
class RunTest implements Runnable {

    private CountDownLatch countDownLatch;
    private int index;

    public RunTest(CountDownLatch countDownLatch,int index) {
        this.countDownLatch = countDownLatch;
        this.index = index;
    }

    @Override
    public void run() {
        try {
            countDownLatch.await();
            get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public synchronized void get() {
        System.out.println("this is get method"+index);
        set();
    }

    public synchronized void set() {
        System.out.println("this is set method"+index);
    }
}
```
ReentrantLock

```java
class RunTest implements Runnable{
    private ReentrantLock reentrantLock = new ReentrantLock();

    private CountDownLatch countDownLatch;
    private int index;

    public RunTest(CountDownLatch countDownLatch,int index) {
        this.countDownLatch = countDownLatch;
        this.index = index;
    }

    @Override
    public void run() {
        try {
            countDownLatch.await();
            get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public  void get() {
        reentrantLock.lock();
        System.out.println("this is get method"+index);
        set();
        reentrantLock.unlock();
    }

    public  void set() {
        reentrantLock.lock();
        System.out.println("this is set method"+index);
        reentrantLock.unlock();
    }
}
```

可重入锁的最大好处，不会引起死锁。

---

#### 悲观锁和乐观锁

悲观锁：假定会发生并发冲突，屏蔽一切可能违反数据完整性的操作。
悲观锁每次获取资源都会上锁，这样别的执行单元(线程)就会被阻塞掉。synchronized 就是悲观锁的一种实现，每次线程修改数据都必须先获取锁，确保在有且仅有一个线程在操作资源。

乐观锁：假设不会发生并发冲突，只在提交操作时检查是否违反数据完整性。（使用版本号或者时间戳来配合实现）

---

#### 共享锁和排它锁

共享锁：如果事务T对数据A加上共享锁后，则其他事务只能对A再加共享锁，不能加排它锁。获准共享锁的事务只能读数据，不能修改数据。

排它锁：如果事务T对数据A加上排它锁后，则其他事务不能再对A加任何类型的锁。获得排它锁的事务即能读数据又能修改数据。

---

#### 读写锁

读写锁是一个资源能够被多个读线程访问，或者被一个写线程访问但不能同时存在读线程。Java当中的读写锁通过ReentrantReadWriteLock实现。

---

#### 互斥锁

互斥锁是一个二元变量，其状态为开锁(允许0)和上锁(禁止1)，将某个共享资源与某个特定互斥锁在逻辑上绑定(要申请该资源必须先获取锁)。




