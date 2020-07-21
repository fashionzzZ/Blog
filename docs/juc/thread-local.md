### ThreadLocal

```java
public class ThreadLocalTest {
	static ThreadLocal<Person> tl = new ThreadLocal<>();
	
	public static void main(String[] args) {
				
		new Thread(()->{
			try {
				TimeUnit.SECONDS.sleep(2);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			
			System.out.println(tl.get()); // 从tl中get值，拿不到
		}).start();
		
		new Thread(()->{
			try {
				TimeUnit.SECONDS.sleep(1);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			tl.set(new Person()); // 往tl中set一个Person对象
		}).start();
	}
	
	static class Person {
		String name = "zhangsan";
	}
}
```

#### set方法

map(ThreadLocal,person)

是把set的值放在Thread类下的ThreadLocalMap中，让其成为线程私有的。

#### 应用场景

- 声明式事务，Spring的@Transactional事务，保证线程用同一个Connection

---

### 引用类型

```java
// 当M对象被垃圾回收器回收时，会调用finalize方法。
public class M {
    @Override
    protected void finalize() throws Throwable {
        System.out.println("finalize");
    }
}
```

#### 强引用

强引用就是我们经常用的普通引用，例如`M m = new M()`，这个就是强引用。只要有一个引用指向这个对象，那么垃圾回收器一定不会回收它。

#### 软引用

```java
public class SoftReferenceTest {
    public static void main(String[] args) {
        // 栈内存里有一个m，指向堆内存里的SoftReference软引用对象，软引用里又有一个引用指向了一个10MB大小的字节数组
        SoftReference<byte[]> m = new SoftReference<>(new byte[1024*1024*10]);
        //m = null;
        System.out.println(m.get());
        System.gc();
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(m.get());

        //再分配一个数组，heap将装不下，这时候系统会垃圾回收，先回收一次，如果不够，会把软引用干掉
        byte[] b = new byte[1024*1024*15];
        System.out.println(m.get());
    }
}
```

当有一个对象被一个软引用所指向的时候，只有系统内存不够用的时候，才会回收它。

**应用场景**

- 非常适合做缓存

#### 弱引用

弱引用的意思是，只要遭遇GC就会回收。

```java
public class WeakReferenceTest {
    public static void main(String[] args) {
        WeakReference<M> m = new WeakReference<>(new M());

        System.out.println(m.get()); // 可以get到
        System.gc(); // m指向的对象被回收
        System.out.println(m.get()); // get不到，输出null

        ThreadLocal<M> tl = new ThreadLocal<>();
        tl.set(new M());
        tl.remove();
    }
}
```

弱引用最典型的一个场景就是ThreadLocal

#### 虚引用

对于虚引用它就干一件事，他就是管理堆外内存的。

虚引用的构造方法至少是两个参数，第二个参数还必须是一个队列，一旦虚引用被回收，这个虚引用会装到这个队列里，这个时候你只需要监听这个队列，如果里面有虚引用，说明虚引用被回收了，然后执行相应操作（手动回收堆外内存）。

NIO里边有一个比较新的Buffer叫DirectByteBuffer（直接内存），是可以指向堆外内存的。

Java里，Unsafe类里面有两个方法，allocateMemory方法可以直接分配堆外内存，freeMemory方法可以手动回收内存，但是普通开发人员用不了这个类。

```java
/**
 *     一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，
 *     也无法通过虚引用来获取一个对象的实例。
 *     为一个对象设置虚引用关联的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。
 *     虚引用和弱引用对关联对象的回收都不会产生影响，如果只有虚引用活着弱引用关联着对象，
 *     那么这个对象就会被回收。它们的不同之处在于弱引用的get方法，虚引用的get方法始终返回null,
 *     弱引用可以使用ReferenceQueue,虚引用必须配合ReferenceQueue使用。
 *
 *     jdk中直接内存的回收就用到虚引用，由于jvm自动内存管理的范围是堆内存，
 *     而直接内存是在堆内存之外（其实是内存映射文件，自行去理解虚拟内存空间的相关概念），
 *     所以直接内存的分配和回收都是有Unsafe类去操作，java在申请一块直接内存之后，
 *     会在堆内存分配一个对象保存这个堆外内存的引用，
 *     这个对象被垃圾收集器管理，一旦这个对象被回收，
 *     相应的用户线程会收到通知并对直接内存进行清理工作。
 *
 *     事实上，虚引用有一个很重要的用途就是用来做堆外内存的释放，
 *     DirectByteBuffer就是通过虚引用来实现堆外内存的释放的。
 */
public class PhantomReferenceTest {
    private static final List<Object> LIST = new LinkedList<>();
    private static final ReferenceQueue<M> QUEUE = new ReferenceQueue<>();

    public static void main(String[] args) {

        PhantomReference<M> phantomReference = new PhantomReference<>(new M(), QUEUE);

        new Thread(() -> {
            while (true) {
                LIST.add(new byte[1024 * 1024]);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    Thread.currentThread().interrupt();
                }
                System.out.println(phantomReference.get());
            }
        }).start();

        new Thread(() -> {
            while (true) {
                Reference<? extends M> poll = QUEUE.poll();
                if (poll != null) {
                    System.out.println("--- 虚引用对象被jvm回收了 ---- " + poll);
                }
            }
        }).start();

        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

