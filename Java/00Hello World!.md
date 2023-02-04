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

