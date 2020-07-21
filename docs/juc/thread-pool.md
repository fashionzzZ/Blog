### Executor

```java
/* The {@code Executor} implementations provided in this package
 * implement {@link ExecutorService}, which is a more extensive
 * interface.  The {@link ThreadPoolExecutor} class provides an
 * extensible thread pool implementation. The {@link Executors} class
 * provides convenient factory methods for these Executors.
 *
 * <p>Memory consistency effects: Actions in a thread prior to
 * submitting a {@code Runnable} object to an {@code Executor}
 * <a href="package-summary.html#MemoryVisibility"><i>happen-before</i></a>
 * its execution begins, perhaps in another thread.
 *
 * @since 1.5
 * @author Doug Lea
 */
public interface Executor {

    /**
     * Executes the given command at some time in the future.  The command
     * may execute in a new thread, in a pooled thread, or in the calling
     * thread, at the discretion of the {@code Executor} implementation.
     *
     * @param command the runnable task
     * @throws RejectedExecutionException if this task cannot be
     * accepted for execution
     * @throws NullPointerException if command is null
     */
    void execute(Runnable command);
}
```

### ExecutorService

继承Executor

除了去实现Executor可以去执行一个任务之外，还完善了整个任务执行器的一个生命周期。

定义了如下一些方法：

```java
void shutdown(); // 结束
List<Runnable> shutdownNow(); // 马上结束
boolean isShutdown(); // 是否结束了
boolean isTerminated(); // 是不是整体都执行完了
boolean awaitTermination(long timeout, TimeUnit unit)
    throws InterruptedException; // 等着结束，等多长时间，时间到了还不结束的话就返回false
```

等等，所以它实现了一些线程的生命周期的东西，扩展了Executor的接口，真正的线程池是在ExecutorService的基础上实现的。

### Callable

它和Runnable类似，call方法相当于Runnable的run方法。不同的是，Callable有返回值。

```java
/**
 * A task that returns a result and may throw an exception.
 * Implementors define a single method with no arguments called
 * {@code call}.
 *
 * <p>The {@code Callable} interface is similar to {@link
 * java.lang.Runnable}, in that both are designed for classes whose
 * instances are potentially executed by another thread.  A
 * {@code Runnable}, however, does not return a result and cannot
 * throw a checked exception.
 *
 * <p>The {@link Executors} class contains utility methods to
 * convert from other common forms to {@code Callable} classes.
 *
 * @see Executor
 * @since 1.5
 * @author Doug Lea
 * @param <V> the result type of method {@code call}
 */
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

### Future

用来存储Callable的返回值

```java
public class TestCallable {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Callable<String> c = new Callable() {
            @Override
            public String call() throws Exception {
                return "Hello Callable";
            }
        };

        ExecutorService service = Executors.newCachedThreadPool();
        Future<String> future = service.submit(c); //异步

        System.out.println(future.get());//阻塞

        service.shutdown();
    }
}
```

### FutureTask

即是一个Future同时又是一个Task，因为它实现了RunnableFuture，而RunnableFuture实现了Runnable和Future。

```java
public class TestFuture {
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		
		FutureTask<Integer> task = new FutureTask<>(()->{
			TimeUnit.MILLISECONDS.sleep(500);
			return 1000;
		}); //new Callable () { Integer call();}
		
		new Thread(task).start();
		
		System.out.println(task.get()); //阻塞
	}
}
```

### CompletableFuture

管理多个Future的结果

它是各种任务的管理类

底层用的是ForkJoinPool

```java
/**
 * 假设你能够提供一个服务
 * 这个服务查询各大电商网站同一类产品的价格并汇总展示
 */
public class TestCompletableFuture {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        long start, end;

        /*start = System.currentTimeMillis();
        priceOfTM();
        priceOfTB();
        priceOfJD();
        end = System.currentTimeMillis();
        System.out.println("use serial method call! " + (end - start));*/

        start = System.currentTimeMillis();

        CompletableFuture<Double> futureTM = CompletableFuture.supplyAsync(()->priceOfTM());
        CompletableFuture<Double> futureTB = CompletableFuture.supplyAsync(()->priceOfTB());
        CompletableFuture<Double> futureJD = CompletableFuture.supplyAsync(()->priceOfJD());

        CompletableFuture.allOf(futureTM, futureTB, futureJD).join();

        /*CompletableFuture.supplyAsync(()->priceOfTM())
                .thenApply(String::valueOf)
                .thenApply(str-> "price " + str)
                .thenAccept(System.out::println);*/

        end = System.currentTimeMillis();
        System.out.println("use completable future! " + (end - start));

        try {
            System.in.read(); //阻塞住main线程
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static double priceOfTM() {
        delay();
        return 1.00;
    }

    private static double priceOfTB() {
        delay();
        return 2.00;
    }

    private static double priceOfJD() {
        delay();
        return 3.00;
    }

    /**
    * 模拟延迟
    **/
    private static void delay() {
        int time = new Random().nextInt(500);
        try {
            TimeUnit.MILLISECONDS.sleep(time);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.printf("After %s sleep!\n", time);
    }
}
```

---

### 线程池的处理流程

![线程池的处理流程](https://www.ultroncode.com/source/afe557ab4de5f05c503770000a063248.jpg)

### ThreadPoolExecutor

ThreadPoolExecutor的父类是AbstractExecutorService，AbstractExecutorService实现了ExecutorService，ExecutorService的父类是Executor

#### 七个参数

1. **corePoolSize：核心线程数**

   当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其它空闲的核心线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池的核心线程数时就不再创建。如果调用了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有核心线程。

2. **maximumPoolSize：最大线程数**

   线程池允许创建的最大线程数。如果任务队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是，如果使用了无界的任务队列这个参数就没什么效果。 

3. **keepAliveTime：空闲存活时间**

    线程池的工作线程空闲后，保持存活的时间。所以，如果任务很多，并且每个任务的执行时间比较短，可以调大时间，提高线程的利用率。

4. **TimeUnit：空闲存活时间的单位**

    可选的单位有天（DAYS）、小时（HOURS）、分钟（MINUTES）、秒（SECONDS）、毫秒（MILLISECONDS）等。

5. **BlockingQueue：任务队列**

    用于保存等待执行的任务的阻塞队列，可以选择以下几种阻塞队列：

   - ArrayBlockingQueue：是一个基于数组结构的有界阻塞队列，此队列按FIFO（先进先出）原则对元素进行排序。
   - LinkedBlockingQueue：一个基于链表结构的阻塞队列，此队列按FIFO排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()使用了这个队列。
   - SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue，静态工厂方法Executors.newCachedThreadPool()使用了这个队列。
   - PriorityBlockingQueue：一个具有优先级的阻塞队列。

6. **ThreadFactory：线程工厂**

    用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字。使用开源框架guava提供的ThreadFactoryBuilder可以快速给线程池里的线程设置有意义的名字，代码如下：

   ```java
   new ThreadFactoryBuilder().setNameFormat("XX-task-%d").build();
   ```

7. **RejectedExecutionHandler：拒绝策略**

   当任务队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理新提交的任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。Java线程池框架提供了以下4种策略：

   - AbortPolicy：直接抛出异常。
   - CallerRunsPolicy：只用调用者所在的线程来运行任务。
   - DiscardOldestPolicy：丢弃队列里最老的任务，把新任务添加到队列。
   - DiscardPolicy：不处理，丢弃掉。

   当然，也可以根据应用场景需要来实现RejectedExecutionHandler接口自定义策略，如记录日志或持久化存储不能处理的任务。

```java
public class TestThreadPool {
    static class Task implements Runnable {
        private int i;
        
        public Task(int i) {
            this.i = i;
        }

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + " Task " + i);
            try {
                System.in.read();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        @Override
        public String toString() {
            return "Task{" +
                    "i=" + i +
                    '}';
        }
    }

    public static void main(String[] args) {
        ThreadPoolExecutor tpe = new ThreadPoolExecutor(2, 4,
                60, TimeUnit.SECONDS,
                new ArrayBlockingQueue<Runnable>(4),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.CallerRunsPolicy());

        for (int i = 0; i < 8; i++) {
            tpe.execute(new Task(i));
        }

        System.out.println(tpe.getQueue());

        tpe.execute(new Task(100));

        System.out.println(tpe.getQueue());

        tpe.shutdown();
    }
}
```

### Executors

### SingleThreadExecutor

任务队列是LinkedBlockingQueue

这个线程池里面只有一个线程，可以保证我们扔进去的任务是顺序执行的。

```java
// 源码
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

```java
public class TestSingleThreadExecutor {
	public static void main(String[] args) {
		ExecutorService service = Executors.newSingleThreadExecutor();
		for(int i=0; i<5; i++) {
			final int j = i;
			service.execute(()->{
				System.out.println(j + " " + Thread.currentThread().getName());
			});
		}
	}
}
```

为什么会有单线程的线程池？

线程池是有任务队列的；生命周期管理线程池是可以帮你提供的。

### CachedThreadPool

它没有核心线程，最多可以有Integer.MAX_VALUE个线程

任务队列是SynchronousQueue

**新来一个任务必须马上执行，没有空闲的线程就new一个线程。**

```java
// 源码
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

```java
public class TestCachedThreadPool {
	public static void main(String[] args) throws InterruptedException {
		ExecutorService service = Executors.newCachedThreadPool();
		System.out.println(service);
		for (int i = 0; i < 2; i++) {
			service.execute(() -> {
				try {
					TimeUnit.MILLISECONDS.sleep(500);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println(Thread.currentThread().getName());
			});
		}
		System.out.println(service);
		TimeUnit.SECONDS.sleep(80);
		System.out.println(service);
	}
}
```

### FixedThreadPool

固定线程数的线程池

任务队列是LinkedBlockingQueue

FixedThreadPool指定一个参数，核心线程数和最大线程数都是指定的这个参数，所以这个线程池里全是核心线程。

```java
// 源码
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

```java
public class TestFixedThreadPool {
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		long start = System.currentTimeMillis();
		getPrime(1, 200000); 
		long end = System.currentTimeMillis();
		System.out.println(end - start);
		
		final int cpuCoreNum = 4;
		
		ExecutorService service = Executors.newFixedThreadPool(cpuCoreNum);
		
		MyTask t1 = new MyTask(1, 80000); //1-5 5-10 10-15 15-20
		MyTask t2 = new MyTask(80001, 130000);
		MyTask t3 = new MyTask(130001, 170000);
		MyTask t4 = new MyTask(170001, 200000);
		
		Future<List<Integer>> f1 = service.submit(t1);
		Future<List<Integer>> f2 = service.submit(t2);
		Future<List<Integer>> f3 = service.submit(t3);
		Future<List<Integer>> f4 = service.submit(t4);
		
		start = System.currentTimeMillis();
		f1.get();
		f2.get();
		f3.get();
		f4.get();
		end = System.currentTimeMillis();
		System.out.println(end - start);
	}
	
	static class MyTask implements Callable<List<Integer>> {
		int startPos, endPos;
		
		MyTask(int s, int e) {
			this.startPos = s;
			this.endPos = e;
		}
		
		@Override
		public List<Integer> call() throws Exception {
			List<Integer> r = getPrime(startPos, endPos);
			return r;
		}
	}
	
	static boolean isPrime(int num) {
		for(int i=2; i<=num/2; i++) {
			if(num % i == 0) return false;
		}
		return true;
	}
	
	static List<Integer> getPrime(int start, int end) {
		List<Integer> results = new ArrayList<>();
		for(int i=start; i<=end; i++) {
			if(isPrime(i)) results.add(i);
		}
		return results;
	}
}
```

#### 并发和并行有什么区别

concurrent vs parallel

并发是指任务提交，并行是指任务执行；并行是并发的子集。

并发是多个任务同时被提交，不一定同时处理；并行是多个cpu同时进行处理多个任务。

#### Cache vs Fixed

什么时候用Cache，什么时候用Fixed?

需要预估并发了控制线程数，如果线程池中线程数过多，最终它们会竞争稀缺的处理器和内存的资源，浪费大量的时间在上下文切换上；反之，如果线程数过少，处理器的一些核心可能就无法充分利用。

线程池大小与处理器的利用率之比可以使用公式进行估算：

**线程数=cpu核心数 * cpu期望利用率 * (1+W/C)**
W是wait等待时间，C是conputation计算时间

### ScheduledThreadPool

定时任务线程池

任务队列是DelayedWorkQueue

scheduleAtFixedRate间隔多长时间在一个固定频率上来执行这个任务，有三个参数：

1. Delay：第一个任务在多长时间后开始执行
2. Period：间隔时间
3. TimeUnit：时间单位

```java
// 源码
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
public ScheduledThreadPoolExecutor(int corePoolSize) {
    // super是ThreadPoolExecutor，所以本质上还是ThreadPoolExecutor
    super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
          new DelayedWorkQueue());
}
```

```java
public class TestScheduledThreadPool {
	public static void main(String[] args) {
		ScheduledExecutorService service = Executors.newScheduledThreadPool(4);
		service.scheduleAtFixedRate(()->{
			try {
				TimeUnit.MILLISECONDS.sleep(new Random().nextInt(1000));
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println(Thread.currentThread().getName());
		}, 0, 500, TimeUnit.MILLISECONDS);
	}
}
```

### ForkJoinPool

把大任务切分成一个一个的小任务去运行，任务执行完了要汇总

需要定义它特定的任务类型，叫ForkJoinTask，一般用ForkJoinTask的两个子类：

1. RecursiveAction：递归执行，没有返回值
2. RecurisiveTask：递归执行，有返回值

```java
public class TestForkJoinPool {
	static int[] nums = new int[1000000];
	static final int MAX_NUM = 50000;
	static Random r = new Random();
	
	static {
		for(int i=0; i<nums.length; i++) {
			nums[i] = r.nextInt(100);
		}
		System.out.println("---" + Arrays.stream(nums).sum()); //stream api
	}

	static class AddTask extends RecursiveAction {
		int start, end;

		AddTask(int s, int e) {
			start = s;
			end = e;
		}

		@Override
		protected void compute() {
			if(end-start <= MAX_NUM) {
				long sum = 0L;
				for(int i=start; i<end; i++) sum += nums[i];
				System.out.println("from:" + start + " to:" + end + " = " + sum);
			} else {
				int middle = start + (end-start)/2;

				AddTask subTask1 = new AddTask(start, middle);
				AddTask subTask2 = new AddTask(middle, end);
				subTask1.fork();
				subTask2.fork();
			}
		}
	}
	
	static class AddTaskRet extends RecursiveTask<Long> {
		private static final long serialVersionUID = 1L;
		int start, end;
		
		AddTaskRet(int s, int e) {
			start = s;
			end = e;
		}

		@Override
		protected Long compute() {
			if(end-start <= MAX_NUM) {
				long sum = 0L;
				for(int i=start; i<end; i++) sum += nums[i];
				return sum;
			} 
			
			int middle = start + (end-start)/2;
			
			AddTaskRet subTask1 = new AddTaskRet(start, middle);
			AddTaskRet subTask2 = new AddTaskRet(middle, end);
			subTask1.fork();
			subTask2.fork();
			
			return subTask1.join() + subTask2.join();
		}
	}
	
	public static void main(String[] args) throws IOException {
		/*ForkJoinPool fjp = new ForkJoinPool();
		AddTask task = new AddTask(0, nums.length);
		fjp.execute(task);*/

		TestForkJoinPool temp = new TestForkJoinPool();

		ForkJoinPool fjp = new ForkJoinPool();
		AddTaskRet task = new AddTaskRet(0, nums.length);
		fjp.execute(task);
		long result = task.join();
		System.out.println(result);
		
		//System.in.read();
	}
}
```

### WorkStealingPool

线程池中的每一个线程都有自己单独的任务队列，当某一个线程执行完自己的任务队列时，它会去另外一个线程的任务队列的队尾取任务，添加到自己的任务队列的队尾，然后执行。

原理：

1. 多个work queue
2. 采用work stealing算法

从源码可以看出，本质上是一个ForkJoinPool

```java
// 源码
public static ExecutorService newWorkStealingPool() {
    return new ForkJoinPool
        (Runtime.getRuntime().availableProcessors(),
         ForkJoinPool.defaultForkJoinWorkerThreadFactory,
         null, true);
}
```

```java
public class TestWorkStealingPool {
	public static void main(String[] args) throws IOException {
		ExecutorService service = Executors.newWorkStealingPool();
		System.out.println(Runtime.getRuntime().availableProcessors());

		service.execute(new R(1000));
		service.execute(new R(2000));
		service.execute(new R(2000));
		service.execute(new R(2000)); //daemon
		service.execute(new R(2000));
		
		//由于产生的是精灵线程（守护线程、后台线程），主线程不阻塞的话，看不到输出
		System.in.read(); 
	}

	static class R implements Runnable {
		int time;

		R(int t) {
			this.time = t;
		}

		@Override
		public void run() {
			try {
				TimeUnit.MILLISECONDS.sleep(time);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println(time  + " " + Thread.currentThread().getName());
		}
	}
}
```

### ParallelStream

并行流

底层用的也是ForkJoinPool，也是ForkJoinPool的算法来实现的

```java
public class TestParallelStream {
	public static void main(String[] args) {
		List<Integer> nums = new ArrayList<>();
		Random r = new Random();
		for(int i=0; i<10000; i++) nums.add(1000000 + r.nextInt(1000000));
		
		//System.out.println(nums);
		
		long start = System.currentTimeMillis();
		nums.forEach(v->isPrime(v));
		long end = System.currentTimeMillis();
		System.out.println(end - start);
		
		//使用parallel stream api
		
		start = System.currentTimeMillis();
		nums.parallelStream().forEach(T13_ParallelStreamAPI::isPrime);
		end = System.currentTimeMillis();
		
		System.out.println(end - start);
	}
	
	static boolean isPrime(int num) {
		for(int i=2; i<=num/2; i++) {
			if(num % i == 0) return false;
		}
		return true;
	}
}
```

---

### 总结

线程池总体上分两种：

1. ThreadPoolExecutor
   1. SingleThreadExecutor
   2. CachedThreadPool
   3. FixedThreadPool
   4. ScheduledThreadPool
2. ForkJoinPool
   1. WorkStealingPool
   2. ForkJoinPool

它们两个的区别：

1. ThreadPoolExecutor：多个线程共用同一个任务队列
2. ForkJoinPool：每个线程都有自己的任务队列。

**Tips：在日常开发中用到线程池时，尽量根据业务场景自定义线程池，不建议用Executors提供的线程池。**