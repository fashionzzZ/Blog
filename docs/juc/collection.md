### 同步容器类

1. Vector Hashtable ：早期使用synchronized实现
2. ArrayList HashSet ：未考虑多线程安全（未实现同步）
3. HashSet vs Hashtable StringBuilder vs StringBuffer
4. Collections.synchronizedXXX工厂方法使用的也是synchronized

### Vector HashTable

- 自带锁
- HashTable所有方法都加了Synchronized
- 基本不用

```java
static Hashtable<UUID, UUID> m = new Hashtable<>();
```

### ArrayList

### HashMap

#### 初始容量

16

#### 负载因子

0.75

#### 底层实现

- 数组+链表
- 数组+红黑树

#### 元素数据类型

- 1.7及以前：Entry
- 1.8及以后：Node

#### 元素插入到链表中的位置

- 1.7及以前：头插法：新值插入到链表的头部

  为啥要这样，这是之前开发者觉得后面插入的数据会先用到，因为要使用这些Node是要遍历这个链表，在前面的遍历的会更快。

- 1.8及以后：尾插法：新值插入到链表的尾部

  为啥要这样，链表成环的问题，使用头插法会改变链表的顺序，如果扩容的话，由于原本链表顺序有所改变，扩容之后重新hash，可能导致的情况就是扩容转移后前后链表顺序倒置，在转移过程中修改了原来链表中节点的引用关系。

  这样的话在多线程操作下就会出现死循环，而使用尾插法，在相同的前提下就不会出现这样的问题，因为扩容前后链表顺序是不变的，他们之间的引用关系也是不变的。

#### hash算法

扰动函数：让高位参与哈希运算，为了降低哈希碰撞几率。

`h = key.hashCode()) ^ (h >>> 16)`

![哈希算法](https://ultroncode.com/source/bc23adf4c762587967f7c92f1b91e293.png)

#### 计算下标算法

`i = (n - 1) & hash`

`(n - 1) & hash = n % hash`

总结起来就是使用位与运算可以实现和取模运算相同的效果，而且位与运算性能更高。

#### 扩容

条件：比如HashMap的容量是100，负载因子是0.75，乘以100就是75，所以当你增加第76个的时候就需要扩容了。

步骤：首先是创建一个新的数组，容量是原来的二倍，为啥是2倍，因为容量要是2的整数次幂。

#### 为什么扩容为原来的2倍

因为之前的容量是2的整数次幂，扩容为原来的2倍，得到的容量也是2的整数次幂。

#### 为什么容量要是2的整数次幂

因为2的整数次幂的数-1得到的二进制数后面几位都是1，方便计算元素下标时的位与运算。

#### 链表转红黑树

`泊松分布`

当链表长度为8时会将链表转换为红黑树，为6时又会转换成链表，这样是提高了性能，也可以防止哈希碰撞攻击。

#### 增加元素的主要步骤

1. 根据key值，通过哈希算法得到value应该放在底层数组中的下标位置
2. 根据这个下标定位到底层数组中的元素，可能是链表或红黑树
3. 拿到当前位置上的key值，与要放入的key值比较，判断是否==或equals，成立就替换并返回旧值
4. 如果是树的话，就遍历树中的节点，判断是否==或equals，成立替换，否则添加到树里
5. 如果是链表的话，遍历链表的节点，判断是否==或equals，成立替换，否则添加到链表的尾部

### SynchronizedHashMap

```java
static Map<UUID, UUID> m = Collections.synchronizedMap(new HashMap<UUID, UUID>());
```

### ConcurrentHashMap

无序

```java
public class TestConcurrentMap {
	public static void main(String[] args) {
		Map<String, String> map = new ConcurrentHashMap<>();
		//Map<String, String> map = new ConcurrentSkipListMap<>(); //高并发并且排序
		
		//Map<String, String> map = new Hashtable<>();
		//Map<String, String> map = new HashMap<>(); //Collections.synchronizedXXX
		//TreeMap
		Random r = new Random();
		Thread[] ths = new Thread[100];
		CountDownLatch latch = new CountDownLatch(ths.length);
		long start = System.currentTimeMillis();
		for(int i=0; i<ths.length; i++) {
			ths[i] = new Thread(()->{
				for(int j=0; j<10000; j++) map.put("a" + r.nextInt(100000), "a" + r.nextInt(100000));
				latch.countDown();
			});
		}
		
		Arrays.asList(ths).forEach(t->t.start());
		try {
			latch.await();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		long end = System.currentTimeMillis();
		System.out.println(end - start);
		System.out.println(map.size());
	}
}
```

### ConcurrentSkipListMap

跳表

有序

![跳表原理图](https://ultroncode.com/source/2771b38902507fd11f97f0b8088c6139.jpg)

从上往下遍历

### CopyOnWrite

#### CopyOnWriteList

#### CopyOnWriteSet

写时复制

写的时候加锁复制原list，然后把长度加1，把要写入的值放到末尾，把指针指向新复制的list；读的时候不加锁

适用场景：读特别多，写比较少

```java
public class TestCopyOnWriteList {
	public static void main(String[] args) {
		List<String> lists = 
				//new ArrayList<>(); //这个会出并发问题！
				//new Vector();
				new CopyOnWriteArrayList<>();
		Random r = new Random();
		Thread[] ths = new Thread[100];
		
		for(int i=0; i<ths.length; i++) {
			Runnable task = new Runnable() {
				@Override
				public void run() {
					for(int i=0; i<1000; i++) lists.add("a" + r.nextInt(10000));
				}
			};
			ths[i] = new Thread(task);
		}
		runAndComputeTime(ths);
		System.out.println(lists.size());
	}
	
	static void runAndComputeTime(Thread[] ths) {
		long s1 = System.currentTimeMillis();
		Arrays.asList(ths).forEach(t->t.start());
		Arrays.asList(ths).forEach(t->{
			try {
				t.join();
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		});
		long s2 = System.currentTimeMillis();
		System.out.println(s2 - s1);
	}
}
```

---

### BlockingQueue

阻塞队列

Queue提供了一些方法：offer、poll、peek

> offer：添加数据，返回布尔值（是否添加成功）
> poll：取数据，并且remove掉
> peek：取数据，不remove

BlockingQueue在Queue的基础上增加了两个方法：put、take

> put：添加数据，如果满了，线程阻塞
> take：取数据，如果空了，线程阻塞

```java
public class TestConcurrentQueue {
	public static void main(String[] args) {
		Queue<String> strs = new ConcurrentLinkedQueue<>();
		
		for(int i=0; i<10; i++) {
			strs.offer("a" + i);  //add
		}
		
		System.out.println(strs);
		
		System.out.println(strs.size());
		
		System.out.println(strs.poll());
		System.out.println(strs.size());
		
		System.out.println(strs.peek());
		System.out.println(strs.size());
	}
}
```

### Queue和List的区别

Queue相较于List，添加了offer、peek、poll、put、take这些对线程友好的或者阻塞，或者等待的方法。

### LinkedBlockingQueue

最大容量为Integer的最大值

```java
public class TestLinkedBlockingQueue {
	static BlockingQueue<String> strs = new LinkedBlockingQueue<>();
	static Random r = new Random();
	public static void main(String[] args) {
		new Thread(() -> {
			for (int i = 0; i < 100; i++) {
				try {
					strs.put("a" + i); //如果满了，就会等待
					TimeUnit.MILLISECONDS.sleep(r.nextInt(1000));
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}, "p1").start();

		for (int i = 0; i < 5; i++) {
			new Thread(() -> {
				for (;;) {
					try {
						System.out.println(Thread.currentThread().getName() + " take -" + strs.take()); //如果空了，就会等待
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}, "c" + i).start();
		}
	}
}
```

### ArrayBlockingQueue

有界队列

```java
public class TestArrayBlockingQueue {
	static BlockingQueue<String> strs = new ArrayBlockingQueue<>(10);
	static Random r = new Random();
	public static void main(String[] args) throws InterruptedException {
		for (int i = 0; i < 10; i++) {
			strs.put("a" + i);
		}
		
		//strs.put("aaa"); //满了就会等待，程序阻塞
		//strs.add("aaa");
		//strs.offer("aaa");
		strs.offer("aaa", 1, TimeUnit.SECONDS);
		System.out.println(strs);
	}
}
```

### PriorityQueue

PriorityQueue是从AbstractQueue继承的。
PriorityQueue特点是往里面添加数据的时候，它内部进行了排序，按照最小的优先。它内部实现的结构是一个二叉树，这个二叉树可以认为是堆排序里面的那个最小堆（小顶堆）。

```java
public class TestPriorityQueque {
    public static void main(String[] args) {
        PriorityQueue<String> q = new PriorityQueue<>();

        q.add("c");
        q.add("e");
        q.add("a");
        q.add("d");
        q.add("z");

        for (int i = 0; i < 5; i++) {
            System.out.println(q.poll());
        }
    }
}
```

### DelayQueue

按照等待的时间进行排序，让等待时间短的排在前面。
本质上用的是一个PriorityQueue。

```java
public class TestDelayQueue {
	static BlockingQueue<MyTask> tasks = new DelayQueue<>();
	static Random r = new Random();
	static class MyTask implements Delayed {
		String name;
		long runningTime;
		
		MyTask(String name, long rt) {
			this.name = name;
			this.runningTime = rt;
		}

		@Override
		public int compareTo(Delayed o) {
			if(this.getDelay(TimeUnit.MILLISECONDS) < o.getDelay(TimeUnit.MILLISECONDS))
				return -1;
			else if(this.getDelay(TimeUnit.MILLISECONDS) > o.getDelay(TimeUnit.MILLISECONDS)) 
				return 1;
			else 
				return 0;
		}

		@Override
		public long getDelay(TimeUnit unit) {
			return unit.convert(runningTime - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
		}
		
		@Override
		public String toString() {
			return name + " " + runningTime;
		}
	}

	public static void main(String[] args) throws InterruptedException {
		long now = System.currentTimeMillis();
		MyTask t1 = new MyTask("t1", now + 1000);
		MyTask t2 = new MyTask("t2", now + 2000);
		MyTask t3 = new MyTask("t3", now + 1500);
		MyTask t4 = new MyTask("t4", now + 2500);
		MyTask t5 = new MyTask("t5", now + 500);
		
		tasks.put(t1);
		tasks.put(t2);
		tasks.put(t3);
		tasks.put(t4);
		tasks.put(t5);
		
		System.out.println(tasks);
		
		for(int i=0; i<5; i++) {
			System.out.println(tasks.take());
		}
	}
}
```

### SynchronousQueue

SynchronousQueue容量为0，它不是用来装内容的，是专门用来两个线程之间传内容的，给线程下达任务的。

```java
public class TestSynchronusQueue { //容量为0
	public static void main(String[] args) throws InterruptedException {
		BlockingQueue<String> strs = new SynchronousQueue<>();
		
		new Thread(()->{
			try {
				System.out.println(strs.take());
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}).start();

		strs.put("aaa"); //阻塞等待消费者消费
		//strs.put("bbb");
		//strs.add("aaa");
		System.out.println(strs.size());
	}
}
```

### TransferQueue

它添加了一个方法叫transfer，如果我们用put添加数据，线程添加完就走了。transfer就是装完在这等着，阻塞等有线程把它取走我这个线程才回去干我自己的事情。

```java
public class TestTransferQueue {
	public static void main(String[] args) throws InterruptedException {
		LinkedTransferQueue<String> strs = new LinkedTransferQueue<>();
		
		new Thread(() -> {
			try {
				System.out.println(strs.take());
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}).start();
		
		strs.transfer("aaa");
		
		//strs.put("aaa");

		/*new Thread(() -> {
			try {
				System.out.println(strs.take());
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}).start();*/
	}
}
```

---

### 总结

#### 对于map/set的选择使用

- HashMap
- TreeMap
- LinkedHashMap

- Hashtable
- Collections.sychronizedXXX

- ConcurrentHashMap
- ConcurrentSkipListMap 

#### 队列

- ArrayList
- LinkedList
- Collections.synchronizedXXX
- CopyOnWriteList
- Queue
  - ConcurrentLinkedQueue
  - ConcurrentArrayQueue
  - BlockingQueue
    - LinkedBlockingQueue
    - ArrayBlockingQueue
    - PriorityQueue
    - DelayQueue
    - SynchronusQueue
    - TransferQueue