---
layout: post
title: Transactional注解的传播机制
date: 2018-03-29 00:00:20 +0300
description: 又简单的总结了一遍
img: i-rest.jpg # Add image post (optional)
tags: [事务]
---

- [前言](#前言)
- [Transactional使用](#Transactional使用)
- [传播机制](#传播机制)

---
#### 前言

最近在写一个数据同步的时候用到了事务，具体的描述如下：
- 读取一个账号信息，查看状态是否再同步中，如果再同步中，则返回。如果没有同步中，则将原来的状态改变为同步状态
- 接下来再同步数据，如果异常，这回滚数据。

就是这么一个简单的任务。它们在同一个事务里，要求是在更改账号状态的时候要先入库，因为在页面或其他的地方需要获取即时状态，然后是再整个同步过程中要记录joblog到数据库，出问题时方便查找，但是同步过程中出现异常时joblog不能回滚，这也就是说它耶必须先入库，或者说不再同一个事务里，和同步的事务互不影响。

其实是很简单的 就是新开启一个事务或者挂起当前事务，已非事务的方式执行(事务的传播属性)。再此过程中，感觉对有些属性或是具体的情况不是很了解，因此在此处再总结一次。

---
#### Transactional使用

从上一篇转载的文章拷一段：

@Transactional 可以作用于接口、接口方法、类以及类方法上。当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。

虽然 @Transactional 注解可以作用于接口、接口方法、类以及类方法上，但是 Spring 建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。另外， @Transactional 注解应该只被应用到 public 方法上，这是由 Spring AOP 的本质决定的。如果你在 protected、private 或者默认可见性的方法上使用 @Transactional 注解，这将被忽略，也不会抛出任何异常。

默认情况下，只有来自外部的方法调用才会被AOP代理捕获，也就是，类内部方法调用本类内部的其他方法并不会引起事务行为，即使被调用方法使用@Transactional注解进行修饰。

#### 传播机制

*事务传播行为*
所谓事务的传播行为是指，如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为。在TransactionDefinition定义中包括了如下几个表示传播行为的常量：
- TransactionDefinition.PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。这是默认值。
- TransactionDefinition.PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
- TransactionDefinition.PROPAGATION_MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
- TransactionDefinition.PROPAGATION_NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

1.TransactionDefinition.PROPAGATION_REQUIRED
  B方法设置传播机制为PROPAGATION_REQUIRED，不设置也是默认这个值。
  事务A调用B，<span style="color:blue">无论A或B发生异常,A和B都会回滚</span>
 。

2.TransactionDefinition.PROPAGATION_REQUIRES_NEW
  B方法设置传播机制为PROPAGATION_REQUIRES_NEW
  如果 事务A 调用 B,B会开启一个新的事务。区别在于，<span style="color:blue">如果B发生异常,B会回滚，若抛出的异常A未处理也会回滚；若果A发生异常，B不会回滚（无论B执不执行），A会回滚</span>。再前言里面的joblog 使用的方法久可以用这个传播机制

3.TransactionDefinition.PROPAGATION_SUPPORTS
有没有事务和调用方法一样

4.TransactionDefinition.PROPAGATION_NOT_SUPPORTED
不支持事务，和PROPAGATION_REQUIRES_NEW区别再于被调用的方法需不需要事务，如果被调用的方法不需要事务，则用PROPAGATION_NOT_SUPPORTED比较适宜

5.TransactionDefinition.PROPAGATION_NEVER
调用方法不能有事务，目前好像没有遇到过类是的场景。我觉得应该时被调用的方法的数据太多，以至于一次commit会有太大的压力，还有就是当前的对数据的操作必须马上立即进入数据库，无论成功与否的场景。

6.TransactionDefinition.PROPAGATION_MANDATORY
感觉这个和PROPAGATION_NEVER 刚好相反，调用的方法必须有事务，否则会抛出，可能在对数据的一致性要求比较高的系统会使用（对即时性没有太大要求）

7.TransactionDefinition.PROPAGATION_NESTED
就是说当前被调用的方法一定存在自己的事务中，调用反方不会影响到它。