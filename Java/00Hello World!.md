# Hello World!

这里的第一个`java`程序我们用记事本来写，并且用`cmd`来编译并运行

1. 首先在记事本中编写代码

	```java
	public class Welcome{
	
		public static void main(String[] args){
			System.out.println("hello world!");
		}
	}

2. 再打开`cmd`来到当前目录下，用`javac 文件.java`来编译刚刚写好的源文件，会生成一个`文件.class`的字节码文件

	**这里需要注意的是：每个类都会生成一个字节码文件（.class）**

3. 最后用`java 文件名`来运行生成的`.class`文件

**注意：尽量不要使用多个Scanner对象读入数据**

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        System.out.println("Hello World!");
        Scanner scanner = new Scanner(System.in);
        Scanner scanner2 = new Scanner(System.in);

        int a = scanner.nextInt();

        int b = scanner2.nextInt();

        System.out.println("a = " + a);
        System.out.println("b = " + b);
    }
}
```

> 输入：
>
> 1 2
>
> 3
>
> 输出：
>
> a = 1
>
> b = 3

[具体解释看某大佬的]([(181条消息) [Java\] [缓冲区] 不要在同一个输入流使用多个Scanner_小米煎蛋的博客-CSDN博客](https://blog.csdn.net/twllx/article/details/81739142))

## 命令行参数

一般在用`cmd`运行`java`程序的时候可以使用，在编译成功后，运行的时候可以添加命令行参数

具体如下：

源程序为：

```java
public class abc {
	public static void main(String[] args) {
		for (String x : args) {
			System.out.println(x);
		}	
	}	
}
```

命令行使用的指令为：

先编译：`javac abc.java`

后运行：`java abc 我 也 不是 很懂`

> 输出：
>
> 我
> 也
> 不是
> 很懂

传入的各个又空格分开的字符串被当做多个字符串参数传入字符串数组，输出正确！

---

## 不定数输入

### 读一行数字

可以使用一个字符串先读入整行，然后按空格分割给新的字符串数组，然后通过`Integer`类的`parseInt`方法转成整形（当然也可以按照需要转换成其他的），举例：

```java
import java.util.Scanner;

public class Test {
    public static void main(String[] args) {
    	Scanner s = new Scanner(System.in);
    	String ssr = s.nextLine();
    	String[] arrs = ssr.split(" ");
    	int[] a = new int[arrs.length];
    	for (int i = 0; i < a.length; ++i) {
    		a[i] = Integer.parseInt(arrs[i]);
    	}
    	for (int i = 0; i < a.length; ++i) {
    		System.out.println("第" + i + "个：" + a[i]);
    	}
    }
}
```

> 输入：1 23 15 123
>
> 输出：
>
> 第0个：1
> 第1个：23
> 第2个：15
> 第3个：123

---

### 读入多组数据

可以用`Scanner`对象的`hasNext`方法即可判断后面是否还有数据，举例：

```java
import java.util.Scanner;
public class Test {
    public static void main(String[] args) {
        Scanner s = new Scanner(System.in);
        while (s.hasNext()) {
            int n = s.nextInt();
            int[] a = new int[n];
            for (int i = 0; i < n; ++i) {
                a[i] = s.nextInt();
            }
            for (int i = 0; i < a.length; ++i) {
                for (int j = 0; j < a.length - i - 1; ++j) {
                    if (a[j] > a[j + 1]) {
                        int t = a[j];
                        a[j] = a[j + 1];
                        a[j + 1] = t;
                    }
                }
            }
            for (int i = 0; i < n - 1; ++i) {
                System.out.printf("%d,", a[i]);
            }
            System.out.println(a[n - 1]);
        }
    }
}
```

---

## 总结

1. `Java`对大小写敏感

2. `Java`是面向对象的语言，所有代码都写在类里面

3. 源文件编译后，得到相应的字节码文件，编译器为每个类生成独立的字节码文件

4. `main`方法是`Java`应用程序的入口方法，格式固定：

	`public static void mian(String[] args){...}`

	其中`args`的名字不用固定

5. 一个源文件可以包含多个类

6. 每个语句必须以分号结束，回车不是语句结束的标志，所以一个语句可以跨多行

	```java
	public class Welcome{
	
		public static void main(String[] args){
			System.
	            out
	            .println("hello world!");
		}
	}
	```

	像这么抽象的代码居然都是可以运行的，且和原结果一样，离谱！

