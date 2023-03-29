# 字符串类型的区别

MySQL中字符串有三种类型：

1. `char(n)`：如果存的字符串为'hello'，则实际存的是'hello0000……'总长为n个字节
2. `varchar(n)`：如果存'hello'，则实际存的也是'hello'，实际长度为多少就存多少
3. `nvarchar(n)`：如果存'hello'，实际存的是'hello'，但存的是宽字节字符，一个字符占两个字节

**注意：这里的n指的是字符数，而不是字节数，所以不用管中文字符还是英文字符都可以输入n个**

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

## 删除表

语法为：`drop table 表名`，例如：

```mysql
drop table first;
```

---

## 创建表

语法为：`create table 表名 (列名1 数据类型 约束, 列明2 数据类型 约束, ...)`，例如：

**注意：这里是为了观察清楚于是分行了，实际上不强制分行**

```mysql
create table test (
id int primary key auto_increment, 
name varchar(45) not null, 
sex enum('男', '女') default '男', 
age int
);
```

> 约束：
>
> 主键： primary key， 只能一列是主键列，该列的数据不能重复，且不能为空
>
> 唯一： unique， 每个表中可以有多个唯一列，数据不能重复，可以为空
>
> 默认值 default
>
> 非空 not null
>
> 自增： auto_increment
>
> 外键约束：...

---

## 修改表

语法为：`alter table 表名 具体操作`

接下来细分一些操作：

### 增加表

语法为：`alter table 表名 add column 列名 数据类型 约束`，例如：

```mysql
alter table test add column 学校 int;
```

### 修改列的属性

语法为：`alter table 表名 modify 列名 数据类型 约束`，例如：

```mysql
alter table test modify 学校 varchar(100);
```

### 删除列

语法为：`alter table 表名 drop 列名`，例如：

```mysql
alter table test drop 学校;
```

---

## 插入数据

语法为：`insert 表名 values (值1, 值2, ...)`

可以理解为：把所有的列都给选中了，所以是要按照建表的顺序插入数据，而且数目要一一对应，例如：

```mysql
insert test values (1, 'zly', '女', 19);
```

这里还有一种可以指定列添加数据的方法：

语法为：`insert into 表名 (表名1, 表名2, ...) values (值1, 值2, ...)`，例如：

例1：

```mysql
insert test (name, age) values ('hd', 20);
```

> 解释：因为id在约束中加了自增于是是有默认值的，sex在约束中加了默认值所以也有默认值。因此，这两个可以不填

例2：

```mysql
insert test (name) values ('why');
```

> 解释：因为age没有说不能为空，所以这里不给值也没事