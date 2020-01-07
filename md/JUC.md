# 1.JUC(java.util.concurrent)

1.1 进程/线程

进程就行运行中的程序。

多个线程组成一个进程，线程是进程的内部构成。

1.2 并发/并行

并发，在同一时间点，多个请求蜂拥而至，抢夺同一份资源(线程)，例如秒杀，抢票这种行为

并行，多件事情在同一时间可以同时执行。



### 包结构

java.util.concurrent

java.util.concurrent.atomic

java.until.concurrent.locks



引出：

题目：三个售票员 卖出  30张票

如何编写企业级的多线程代码？

固定的编程套路+模板是什么？

思考：

1.在高内聚，低耦合的前提下，  线程    操作     资源类

1.1 一言不合，先创建一个资源类

1.2 创建线程

1.3使用线程操作资源类

```java
    public static void main(String[] args) {
        Ticket ticket = new Ticket();
        //函数式,请参照1.8 lambda @FunctionInterface
        new Thread(()->{for(int i= 0;i<31;i++)ticket.sale_new();},"A").start();
        new Thread(()->{for(int i= 0;i<31;i++)ticket.sale_new();},"B").start();
        new Thread(()->{for(int i= 0;i<31;i++)ticket.sale_new();},"C").start();
    }

	class Ticket {

    private int number = 30;

    Lock lock = new ReentrantLock();

    public void sale_new() {
        lock.lock();
        try {	
            if (number > 0) {
                number--;
                System.out.printf(Thread.currentThread().getName() + "\t卖出第" + (30 - 	number) + "张票,剩余:" + number + "张票\n");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

}
```



#### 集合不安全

1.1请说出ArrayList 是否线程

1.2如果不安全请试着证明他,否则请说出理由

实例:

```java
  List<String> sList = new ArrayList<>();

        for (int i = 0; i < 30; i++) {
            new Thread(()->{sList.add(UUID.randomUUID().toString().substring(0,8));
                System.out.println(sList);
            },String.valueOf(i)).start();
        }
        Thread.sleep(1000);

        System.out.println(sList.size());
        }
	//会出现ConcurrentmodificationException(并发修盖异常)
```



1.3如何使其线程安全

```java
        List<String> sList = new Vector<>();
        List<String> sList = Collections.synchronizedList(new Vector<>());
        List<String> sList = new CopyOnWriteArrayList<>();
```





#### 如何精准控制线程

案例: 

现有3个线程, 需要线程按特定顺序执行 资源类的方法(run1[打印aaa],run2[打印bbbb],run3[打印ccccc])的顺序循环打印10次

```java
class Number3 {
    //初始化阈值
    private static int num = 0;
    //创建锁实例
    private Lock lock = new ReentrantLock();
    private Condition c1 = lock.newCondition();
    private Condition c2 = lock.newCondition();
    private Condition c3 = lock.newCondition();

    public void run1() {
        //为此方法加锁
        lock.lock();
        try {
            //条件判断
            while (num != 0) {
                //线程等待
                c1.await();
            }
            //打印
            System.out.println("aaa");
            //改变开关状态
            num = 1;
            c2.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //解锁
            lock.unlock();
        }
    }

    public void run2() {
        //为此方法加锁
        lock.lock();
        try {
            //条件判断
            while (num != 1) {
                //线程等待
                c2.await();
            }
            //打印
            System.out.println("bbbb");
            //改变开关状态
            num = 2;
            c3.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //解锁
            lock.unlock();
        }
    }

    public void run3() {
        //为此方法加锁
        lock.lock();
        try {
            //条件判断
            while (num != 2) {
                //线程等待
                c3.await();
            }
            //打印
            System.out.println("ccccc");
            //改变开关状态
            num = 0;
            c1.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //解锁
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        Number3 number3 = new Number3();
        new Thread(()->{for(int i = 0;i < 10;i++){number3.run1();}},"A").start();
        new Thread(()->{for(int i = 0;i < 10;i++){number3.run2();}},"B").start();
        new Thread(()->{for(int i = 0;i < 10;i++){number3.run3();}},"C").start();
    }
```

