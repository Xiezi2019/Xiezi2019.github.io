---
layout:     post
title:      MVC设计模式
subtitle:   springMVC、适配器模式
date:       2018-10-29
author:     xiezi
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Spring
---
Sychronized 和 ReetrantLock

- 相似点

  - 加锁同步，阻塞式同步
  - 可重入锁

- 功能区别

  - Sychronized:java语言关键字，悲观锁，jvm实现，jvm保证锁的加锁和释放，线程异常自动释放锁，不会造成死锁
  - ReetrantLock:类，JDK 1.5后提供的API层面互斥锁，手动lock()和unlock(),配合try/finally语句完成，最好在finally中声明释放锁。等待可中断、默认公平锁（也可以非公）、锁绑定多个条件（Condition类：conditon.await(),conditon.signal()）
  - 锁的粒度和灵活度，Reentrant更优

  ```
  //创建一个非公平锁，默认是非公平锁
  Lock lock = new ReentrantLock();
  Lock lock = new ReentrantLock(false);
   
  //创建一个公平锁，构造传参true
  Lock lock = new ReentrantLock(true);
  ```

  

- 性能区别

  - Sychronized在引入偏向锁和轻量级锁（自旋锁）后，两者性能差不多。



### Sychronized实现原理

- 实现基础：Mark word + Monitor  对象头和监视器

```
synchronized的实现原理和应用总结
（1）synchronized同步代码块：synchronized关键字经过编译之后，会在同步代码块前后分别形成monitorenter和monitorexit字节码指令，在执行monitorenter指令的时候，首先尝试获取对象的锁，如果这个锁没有被锁定或者当前线程已经拥有了那个对象的锁，锁的计数器就加1，在执行monitorexit指令时会将锁的计数器减1，当减为0的时候就释放锁。如果获取对象锁一直失败，那当前线程就要阻塞等待，直到对象锁被另一个线程释放为止。指令
（2）同步方法：方法级的同步是隐式的，无须通过字节码指令来控制，JVM可以从方法常量池的方法表结构中的ACC_SYNCHRONIZED访问标志得知一个方法是否声明为同步方法。当方法调用的时，调用指令会检查方法的ACC_SYNCHRONIZED访问标志是否被设置，如果设置了，执行线程就要求先持有monitor对象，然后才能执行方法，最后当方法执行完（无论是正常完成还是非正常完成）时释放monitor对象。在方法执行期间，执行线程持有了管程，其他线程都无法再次获取同一个管程。
————————————————
版权声明：本文为CSDN博主「lzcWHUT」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/jinjiniao1/article/details/91546512
```





- wait和notify的原理
  调用wait方法，首先会获取监视器锁，获得成功以后，会让当前线程进入等待状态进入等待队列并且释放锁。当其他线程调用notify后，会选择从等待队列中唤醒任意一个线程，而执行完notify方法以后，并不会立马唤醒线程，原因是当前的线程仍然持有这把锁，处于等待状态的线程无法获得锁。必须要等到当前的线程执行完按monitorexit指令以后，也就是锁被释放以后，处于等待队列中的线程就可以开始竞争锁了。

- wait和notify为什么需要在synchronized里面？
  wait方法的语义有两个，一个是释放当前的对象锁、另一个是使得当前线程进入阻塞队列，而这些操作都和监视器是相关的，所以wait必须要获得一个监视器锁。而对于notify来说也是一样，它是唤醒一个线程，既然要去唤醒，首先得知道它在哪里，所以就必须要找到这个对象获取到这个对象的锁，然后到这个对象的等待队列中去唤醒一个线程。
  ————————————————
  版权声明：本文为CSDN博主「KeepMoving++」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
  原文链接：https://blog.csdn.net/u011212394/article/details/82228321

### ReentrantLock实现原理

- CAS+CLH队列实现： 先通过CAS尝试获取锁。如果此时已经有线程占据了锁，那就加入CLH队列并且被挂起。当锁被释放之后，排在CLH队列队首的线程会被唤醒，然后CAS再次尝试获取锁。 

- CAS:Compare and Swap，比较并交换。CAS有3个操作数：内存值V、预期值A、要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。该操作是一个原子操作，被广泛的应用在Java的底层实现中。在Java中，CAS主要是由sun.misc.Unsafe这个类通过JNI调用CPU底层指令实现。

   ![img](https://img-blog.csdnimg.cn/20190116010722130.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01pY2hhZWxlcw==,size_8,color_FFFFFF,t_70) 

-  CLH队列：CLH队列锁是一个[自旋锁](https://www.baidu.com/s?wd=自旋锁&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)。 基于FIFO双端链表 ，能确保无饥饿性。提供先来先服务的公平性。（n个线程有n个myNode。L个锁有L个tail） 前驱引用

  -  当一个线程需要获取锁时，会创建一个新的QNode，将其中的locked设置为true表示需要获取锁，然后线程对tail域调用getAndSet方法，使自己成为队列的尾部，同时获取一个指向其前趋的引用myPred,然后该线程就在前趋结点的locked字段上旋转，直到前趋结点释放锁； 
  -  当一个线程需要释放锁时，将当前结点的locked域设置为false

- ————————————————
  版权声明：本文为CSDN博主「Michaeles」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
  原文链接：https://blog.csdn.net/Michaeles/article/details/86501327
