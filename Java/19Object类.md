# 说明

`Object`类是所有类的直接、间接父类，也就是说：只要是个对象就能用它的方法

# getClass方法

该方法可以获取某个对象属于的类

**注意：是成员函数，不是静态函数，所以记得实例化对象**

```java
A x = new A();
B y = new B();
System.out.println(x.getClass()); // class A
System.out.println(y.getClass()); // class B
```

# equals方法

## 与`==`符号的区别

- `==`符号用于比较两个对象是否指向同一个地址
- `equals`用于判断两个对象的内容是否相同

例如：

```java
public class tst {
    public static void main(String[] args) {
        String s1 = "123";
		String s2 = "123";
		String s3 = new String("123");
		System.out.println(s1 == s2);
		System.out.println(s1 == s3);
		System.out.println(s1.equals(s3));
		System.out.println(s1.equals(s2));
    }
}
```

> 结果：
>
> true
> false
> true
> true