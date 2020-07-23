### 进程 线程 协程/纤程（quasar）

#### 进程

> 一个程序运行之后叫一个进程，程序是静态的概念，运行起来之后变成进程，是一个动态的概念

#### 线程

> 进程里的最小执行单元
>
> 一个进程里面的不同的执行路径

### 创建线程的方式

#### 1. 继承Thread类，重写run方法

// 1.创建
static class MyThread extends Thread {
  @Override
  public void run() { System.out.pringln("Hello MyThread!"); }
}
// 2.调用
new MyThread().start();

#### 2. 实现Runnable接口，重写run方法

```java
// 1.创建
statis class MyRun implements Runnable {
  @override
  public void run() { System.out.println("Hello MyRun!"); }
}
// 2.调用
new Thread(new MyRun()).start();
```

#### 3. Lambda表达式

```java
new Thread(() -> {
  System.out.println("Hello Lambda!")
}).start();
```

#### 4. Executors.newCachedThread线程池启动

### 线程的方法

#### Thread.sleep方法

```java
Thread.sleep(500); // 让该线程睡500ms，并让出CPU，不会释放锁，时间到自动唤醒并进入就绪状态
```

#### Thread.yield方法

```java
Thread.yield(); // 让出一下CPU，该线程进入等待队列（就绪状态），CPU取执行下一个线程（有可能下一个线程还是刚才的线程）
```

#### Thread.join方法

```java
// 在一个线程中插入另一个线程，通常用于等待某个线程的结束，也可以保证线程顺序执行
Thread t2 = new Thread(() -> {
  try {
    t1.join(); // 在此插入t1线程，等t1线程执行完毕后，t2继续执行
  } catch (InterruptedException e) {
    e.printStackTrace();
  }
})
```

---

#### sleep() VS wait()

> 1. sleep()方法是Thread的静态方法，而wait是Object实例方法。
> 2. wait()方法必须要在同步方法或者同步块中调用，也就是必须已经获得对象锁。而sleep()方法没有这个限制可以在任何地方使用。
> 3. wait()方法会释放占有的对象锁，使得该线程进入等待队列中，等待下一次获取资源。而sleep()方法只是会让出CPU并不会释放掉对象锁。
> 4. sleep()方法在休眠时间达到后如果再次获得CPU时间片就会继续执行，而wait()方法必须等待Object.notift/Object.notifyAll通知后，才会离开等待队列，并且再次获得CPU时间片才会继续执行。

### 线程的状态

![线程状态图](https://ultroncode.com/source/c2407e8f0d46a68f736366efa6ca0521.jpg)

#### NEW

初始状态，线程被构建，但是还没有调用`start()`方法

#### RUNNABLE

运行状态，Java线程将操作系统中的就绪(READY)和运行(RUNNING)两种状态笼统地称作“运行中”

#### WAITING

等待状态，表示线程进入等待状态，进入该状态表示当前线程需要等待其它线程作出一些特定动作（通知或中断）

#### TIME_WAITING

超时等待状态，该状态不同于WAITING，它是可以在指定的时间自行唤醒的

#### BLOCKED

阻塞状态，表示线程阻塞与锁

#### TERMINATED

终止状态，表示当前线程已经执行完毕

---

#### 参考资料

- 《Java并发编程的艺术》