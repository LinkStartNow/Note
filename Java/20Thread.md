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

由于