# 注意点：

1. MySQL不区分大小写

2. MySQL对于引号的区分和Python一样，没有双引号和单引号的区别，但嵌套的时候要注意

3. 何时用having何时用where？

	作为条件的列如果是表中原来就有的，就用where

	如果是原来没有的列，新产生的列作为条件的时候，就用having加条件
	
4. 查询出来单个数据是可以作为值进行判断的：

  ```mysql
  #4、查询选修了全部课程的学生的信息!!!
  # 4.1、查询课程数
  select count(*) from course;
  # 4.2、实现
  select * from student where s in (select s from sc group by s having count(*) = (select count(*) from course));
  ```

5. 查询结果的末尾可以通过逗号隔开来增加列

  **注意：这里还需要加上and表示对应学生的课程加到对应学生的行中**

  ```mysql
  #8、查询每个同学01课程的成绩，包括学生信息!!!
  # 8.1、查询每个同学01课程的成绩
  select score from sc where c = '01';
  # 8.2、实现
  select *, (select score from sc where c = '01' and sc.S = student.S) score01 from student;
  ```

6. 上面提到过的表中未出现的属性判断要用`having`

  这里的表指的是`from`后面跟着的表

  ```mysql
  #9、查询01课程分数>02课程分数的学生的信息!!!
  select *, (select score from sc where c = '01' and sc.S = student.S) score01, 
  (select score from sc where c = '02' and sc.S = student.S) score02
  from student having score01 > score02;
  ```

7. 属性值判空可以用`is not null`

  ```mysql
  #10、查询同时学习01课程和02课程的学生的信息
  select *, (select score from sc where c = '01' and sc.S = student.S) score01, 
  (select score from sc where c = '02' and sc.S = student.S) score02
  from student having score01 is not null and score02 is not null;
  ```

8. MySQL中的除法是遵循四舍五入的

	比如下面的代码查询结果为3：

	```mysql
	DROP FUNCTION IF EXISTS YYDS;
	DELIMITER //
	CREATE FUNCTION YYDS(a INT, b INT)
	RETURNS INT
	BEGIN
		RETURN a / b;
	END //
	DELIMITER ;
	
	SELECT YYDS(5, 2);
	```

	可以使用`FLOOR`函数来向下取整

	```mysql
	DROP FUNCTION IF EXISTS YYDS;
	DELIMITER //
	CREATE FUNCTION YYDS(a INT, b INT)
	RETURNS INT
	BEGIN
		RETURN FLOOR(a / b);
	END //
	DELIMITER ;
	
	SELECT YYDS(5, 2); # 2
	```

	也可以使用`CEIL`函数来向上取整

	```mysql
	DROP FUNCTION IF EXISTS YYDS;
	DELIMITER //
	CREATE FUNCTION YYDS(a INT, b INT)
	RETURNS INT
	BEGIN
		RETURN CEIL(a / b);
	END //
	DELIMITER ;
	
	SELECT YYDS(5, 4);  
	```


---

# 字符串

MySQL中字符串有三种类型：

1. `char(n)`：如果存的字符串为'hello'，则实际存的是'hello0000……'总长为n个字节
2. `varchar(n)`：如果存'hello'，则实际存的也是'hello'，实际长度为多少就存多少
3. `nvarchar(n)`：如果存'hello'，实际存的是'hello'，但存的是宽字节字符，一个字符占两个字节

**注意：这里的n指的是字符数，而不是字节数，所以不用管中文字符还是英文字符都可以输入n个**

MySQL中可以使用`CONCAT`函数将多个字符串拼接成一个字符串，例如：

```mysql
SET @x = CONCAT('Hello', ' ', 'World', '!');
SELECT @x;
```

---

# 语法部分

## 授权

语法为：`grant 权限 on 表名 to 用户`，例如：

```mysql
# 临时授权： grant 权限 on 表名 to 用户
grant insert on first to max;
```

---

## 取消授权

语法为：`revoke 权限 on 表名 from 用户`，例如：

```mysql
# 取消授权： revoke 权限 on 表名 from 用户
revoke insert on first from max;
```

---

# 规范

1. 关键字和函数名称全部大写；

2. 数据库名、表名、表别名、字段名、字段别名等全部小写；

3. SQL 语句必须以分号结尾。
