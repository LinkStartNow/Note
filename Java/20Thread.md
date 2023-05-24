# 创建

## 扩展Thread类

可以用子类继承Thread类，并重写Thread类的run方法（run方法就是线程函数）

```java
class MyThread extends Thread {
	MyThread(String s) {
		super(s);
	}

	@Override
	public void run() {
		for (int i = 0; i < 4; ++i) {
			System.out.println(getName() + " " + i);
		}
	}
}

public class tst {
    public static void main(String[] args) {
        MyThread t1 = new MyThread("t1");
        MyThread t2 = new MyThread("t2");
        MyThread t3 = new MyThread("t3");
		t1.start();
		t2.start();
		t3.start();
    }
}
```

> 这里构造函数也使用了父类的构造函数，传入一个字符串当做线程名
>
>  要启动一个线程就调用start方法

## 实现Runnable接口

由于java不支持多继承，因此在已有一个父类的情况下不能再对Thread进行扩展，但是可以使用接口

### 具体实现

1. 首先得有自己的一个类实现了接口`Runnable`，并实现了它的run函数

	**注意：run函数的修饰一定是 public void**

	```java
	class one implements Runnable {
		public void run() {
			System.out.println("Hello World");
		}
	}
	```

2. 用自己定义的类创建一个对象，然后再用这个对象创建一个Thread类对象

	```java
	public class tst {
		public static void main(String[] args) {
			one o = new one();
			Thread t = new Thread(o);
			t.start();
		}
	}
	```

完整代码就是：

```java
class one implements Runnable {
	public void run() {
		System.out.println("Hello World");
	}
}

public class tst {
	public static void main(String[] args) {
		one o = new one();
		Thread t = new Thread(o);
		t.start();
	}
}
```

---

# 线程调度

## 优先级

优先级有1-10，越大的表示越优先，默认为5

可以通过`setPriority(int)`方法设置优先级

通过`getPriority()`方法来获取一个线程的优先级

有三个常数表示优先级：

1. Thread.MIN_PRIORITY：1
2. Thread.MAX_PRIORITY：10
3. Thread.NORM_PRIORITY：5

# 常用方法

- `currentThread()`：返回当前正在执行的线程对象的引用，为**静态函数**
- `getName()`：获得名字
- `join()`：等待该线程终止
- `setName(String name)`
- `sleep(long millis)`：让当前的线程休眠
- `yield()`：暂停当前的线程，执行其他线程

**注意：对于休眠操作，一定要用try**

```java
// 休眠一秒
try {
    Thread.sleep(2000);
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

# 线程联合

**注意：还是要通过try使用**

可以通过`join`方法实现线程联合

```java
class myThread extends Thread {
	private String s;

	myThread(String s) {
		this.s = s;
		super.setName(s);
	}

	public void run() {
		// System.out.println(getName());
		for (int i = 0; i < 5; i++) {
			// 休眠一秒
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println(s + " " + i);
		}
	}
}

public class tst {
	public static void main(String[] args) {
		myThread t1 = new myThread("t1");
		t1.start();
		try {
			t1.join();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("main");
	}
}
```

> 结果：
>
> t1 0
> t1 1
> t1 2
> t1 3
> t1 4
> main
>
> 主线程在执行的过程中join了线程t1
>
> 于是主线程会等待t1结束了再运行

这里使用匿名类就可以实现子线程互相等待

```java
class myThread extends Thread {
	private String s;

	myThread(String s) {
		this.s = s;
		super.setName(s);
	}

	public void run() {
		// System.out.println(getName());
		for (int i = 0; i < 5; i++) {
			// 休眠一秒
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			System.out.println(s + " " + i);
		}
	}
}

public class tst {
	public static void main(String[] args) {
		myThread t1 = new myThread("t1");
		myThread t2 = new myThread("t2") {
			public void run() {
				for (int i = 0; i < 5; i++) {
					// 休眠一秒
					try {
						t1.join();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					System.out.println(getName() + " " + i);
				}
			}
		};
		t1.start();
		t2.start();
		try {
			t1.join();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("main");
	}
}
```

> 输出：
>
> t1 0
> t1 1
> t1 2
> t1 3
> t1 4
> main
> t2 0
> t2 1
> t2 2
> t2 3
> t2 4

# 守护线程

在某个线程内可以让其他线程作为自己的守护线程，例如：

```java
class myThread extends Thread {
	private String s;

	myThread(String s) {
		this.s = s;
		super.setName(s);
	}

	public void run() {
		while (true) {
			System.out.println(getName() + " " + s);
			try {
				sleep(1000);
			} catch (InterruptedException e) {
				System.out.println(e);
			}
			if (s.equals("t2")) {
				break;
			}
		}
	}
}

public class tst {
	public static void main(String[] args) {
		myThread t1 = new myThread("t1");
		myThread t2 = new myThread("t2");
		t1.setDaemon(true);
		t1.start();
		t2.start();
		System.out.println("main start");
		// while (true) {
		// 	System.out.println("runing...");
		// 	try {
		// 		Thread.sleep(1000);
		// 	} catch (InterruptedException e) {
		// 		System.out.println(e);
		// 	}
		// }
	}
}
```

> t1是main线程的守护线程，main线程一结束t1也会结束

# 线程同步与锁机制

## synchronized关键字

### 修饰实例方法

如果**一个对象有多个synchronized方法**，只要**其中一个synchronized方法**被访问了，其他线程就不能同时访问这个对象中的**任何一个synchronized方法**

**注意：相同类的另一个对象的synchronized方法依旧可以被同时访问**

**注意：若其中一个线程进入wait了，那么synchronized又可以被其他对象调用了**

实践代码：

```java
class syn {

    synchronized void hello() {
        System.out.println("Hello");
        try {
            Thread.sleep(2000);
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}

class MyThread extends Thread {
    static syn t = new syn();

    MyThread() {
        
    }

    public void run() {
        t.hello();
    }
}

public class tst {
    public static void main(String[] args) {
        MyThread t1 = new MyThread();
        MyThread t2 = new MyThread();

        t1.start();
        t2.start();
    }
}
```

> 两个线程的主函数都只是调用了t对象的hello方法，但是由于t是静态属性，于是他俩调用的是同一个对象t，加了同步关键字后，很明显感受到第一个hello输出后登了2s
>
> 这是因为第一个线程输出完后又睡了2秒才退出，第二个线程才能调用这个方法
>
> 在不加同步关键字的情况下是没有时间差的

```java
class syn {

    synchronized void hello() {
        System.out.println("Hello");
        try {
            Thread.sleep(2000);
        } catch (Exception e) {
            System.out.println(e);
        }
    }

    synchronized void world() {
        System.out.println("World");
    }
}

class MyThread extends Thread {
    static syn t = new syn();

    MyThread() {
        
    }

    public void run() {
        t.hello();
        t.world();
    }
}

public class tst {
    public static void main(String[] args) {
        MyThread t1 = new MyThread();
        MyThread t2 = new MyThread();

        t1.start();
        t2.start();
    }
}
```

> 输出：
>
> Hello
> World
> Hello
> World
>
> 在第一个线程睡完后，马上调用了world，world输出完没有睡，于是第二个线程瞬间就接着输出了hello，再睡了2s后输出了world

### 修饰静态方法

作用于整个类，任一`synchronized static`方法执行中，当前类**所有对象的所有**`synchronized static`方法都会被卡住

```java
class syn {

    synchronized static void hello() {
        System.out.println("Hello");
        try {
            Thread.sleep(2000);
        } catch (Exception e) {
            System.out.println(e);
        }
    }

    synchronized static void world() {
        System.out.println("World");
    }
}

class MyThread extends Thread {
    syn t = new syn();

    MyThread() {
        
    }

    public void run() {
        t.hello();
        t.world();
    }
}

public class tst {
    public static void main(String[] args) {
        MyThread t1 = new MyThread();
        MyThread t2 = new MyThread();

        t1.start();
        t2.start();
    }
}
```

> 代码中每个线程对象都有一个独立的属性t（是不同的对象哦），但是在其中一个静态同步函数运行的时候依旧会卡住其他的

### 修饰方法中的某个代码块

```java
class syn {
    int a = 0;
}

class MyThread extends Thread {
    static syn s = new syn();

    MyThread() {
        
    }

    public void run() {
        for (int i = 0; i < 100; i++) {
            synchronized(s) {
                s.a++;
                System.out.println(s.a);
            }
        }
    }
}

public class tst {
    public static void main(String[] args) {
        MyThread t1 = new MyThread();
        MyThread t2 = new MyThread();

        t1.start();
        t2.start();

        try {
            t1.join();
            t2.join();
        }
        catch (InterruptedException e) {
            System.out.println("Interrupted");
        }
        System.out.println("res = " + MyThread.s.a);
    }
}
```

### 修饰类

对这个类的所有对象加锁

## 信号量

### 实践1

```java
package Problem4;

import java.util.concurrent.Semaphore;

class Me {
    private boolean onPhone = false;
    Semaphore semaphore = new Semaphore(1);

    void Phone(People man) {
        try {
            semaphore.acquire();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        onPhone = true;
        System.out.println("我在接" + man.getName() + "打来的电话，他对我说：" + man.str);
        int time = (int) (Math.random() * 2000 + 1000);
        // time = 1000;
        try {
            Thread.sleep(time);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("我已经挂断了" + man.getName() + "打来的电话");
        onPhone = false;
        semaphore.release();
    }
}

class People extends Thread {
    Me me;
    String str;

    public People(Me me, String name, String str) {
        this.me = me;
        this.setName(name);
        this.str = str;
    }

    @Override
    public void run() {
        me.Phone(this);
    }
}

public class ssr {
    public static void main(String[] args) {
        Me me = new Me();
        People p1 = new People(me, "张三", "你好");
        People p2 = new People(me, "李四", "你好吗");
        People p3 = new People(me, "王五", "我不太好呢");
        p1.start();
        p2.start();
        p3.start();
    }
}
```

### 实践2

```java
package Problem5;

import java.util.concurrent.Semaphore;

class student extends Thread{
    int cnt = 0;
    Semaphore sem = new Semaphore(0);

    student(String name) {
        setName(name);
    }

    synchronized public void run()  {
        try {
            sem.acquire();
        }
        catch (InterruptedException e) {
            return;
        }
        System.out.println(getName() + "被老师叫醒了");
        System.out.println(getName() + "开始听课");
    }
}

class teacher extends Thread {
    student stu;

    public teacher(student stu) {
        this.stu = stu;
    }

    @Override
    public void run() {
        while (true) {
            System.out.println("上课！");
            stu.cnt++;
            if (stu.cnt == 3) {
                stu.sem.release();
                break;
            }
            else {
                System.out.println(stu.getName() + "正在睡觉，不听课");
            }
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

public class ssr {
    public static void main(String[] args) {
        student stu = new student("张三");
        teacher tea = new teacher(stu);
        stu.start();
        tea.start();
    }
}
```

# 线程交互

## 实践1

```java
public class tst {
    public static void main(String[] args) {
        MyThread t = new MyThread();
        t.start();
        synchronized (t) {
            try {
                t.wait();
                System.out.println(t.a);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

class MyThread extends Thread {
    int a = 0;

    public void run() {
        synchronized (this) {
            for (int i = 0; i < 1000000; i++) {
                a++;
            }
            notify();
        }
    }
}
```

## 实践2

```java
class GunFight {
    private int bullets = 40;

    synchronized public void fire(int bulletsToBeFired) {
        for (int i = 1; i <= bulletsToBeFired; i++) {
            if (bullets == 0) {
                System.out.println(i - 1 + " bullets fired and " + bullets + " remains");
                System.out.println("Invoking the wait() method");
                try {
                    wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("Continuing the fire after reloading");
            }
            bullets--;
        }
        System.out.println("The firing process is complete");
        System.out.println("The number of bullets left in the magazine is " + bullets);
    }

    synchronized public void reload() {
        System.out.println("Reloading the magazine and resuming the thread using notify()");
        bullets += 40;
        notify();
    }
}

public class tst extends Thread {
    public static void main(String[] args) {
        GunFight gf = new GunFight();
        new Thread() {
            @Override
            public void run() {
                gf.fire(60);
            }
        }.start();
        new Thread() {
            @Override
            public void run() {
                gf.reload();
            }
        }.start();
    }
}
```

> 在上面的wait中若收不到后面的notify则不会继续运行下去，而是一直等待

# 经典问题

## 消费者生产者问题

**注意：这里最细节的是用while循环来做等待，而不是一次的if判断**

> 每次notify放开之后，会继续往下执行，若只是if则不会二次判定，于是可能存在放过错误的可能
>
> 但如果用while则会再次判断，直到真正满足条件才会放出去

```java
package Problem1;

class Buffer {
    int num;
    private int size;

    public Buffer(int size) {
        num = 0;
        this.size = size;
    }

    public void put(int value) {
        synchronized (this) {
            while (num >= size) {
                System.out.println("缓冲区已满，生产者等待");
                try {
                    wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            num++;
            notify();
        }
    }

    public void take() {
        synchronized (this) {
            while (num <= 0) {
                System.out.println("缓冲区为大小为" + num + "，消费者等待");
                try {
                    wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            num--;
            notify();
        }
    }
}

class Consumer extends Thread {
    private Buffer buffer;
    private int value;
    private String name;

    public Consumer(Buffer buffer, int value, String name) {
        this.buffer = buffer;
        this.value = value;
        this.name = name;
    }

    @Override
    public void run() {
        while (true) {
            buffer.take();
            System.out.println(name + "取产品使用");
            try {
                sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

class Producer extends Thread {
    private Buffer buffer;
    private int value;
    private String name;

    public Producer(Buffer buffer, int value, String name) {
        this.buffer = buffer;
        this.value = value;
        this.name = name;
    }

    @Override
    public void run() {
        while (true) {
            buffer.put(value);
            System.out.println(name + " 放一个产品");
            try {
                sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Buffer buffer = new Buffer(4);

        Producer producerA = new Producer(buffer, 1, "生产者A");
        Producer producerB = new Producer(buffer, 1, "生产者B");
        Consumer consumer1 = new Consumer(buffer, 1, "消费者1");
        Consumer consumer2 = new Consumer(buffer, 1, "消费者2");
        Consumer consumer3 = new Consumer(buffer, 1, "消费者3");

        producerA.start();
        producerB.start();
        consumer1.start();
        consumer2.start();
        consumer3.start();
    }
}
```

