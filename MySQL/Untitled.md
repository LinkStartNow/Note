# 例题——确认率

## 法1——sum求和

### 1、求出每个人的总action数

> 通过分组和count函数可以直接求出每个用户的总action数
>
> 这里通过左连接可以让没有action的用户的action数变成0

```mysql
select
    s.user_id, count(c.action)
from
    Signups as s
left join
    Confirmations as c
on
    s.user_id = c.user_id
group by
    s.user_id
```

---

### 2、求出每个人的确认数

> sum函数可以用来求和，我们传入表达式为action值为confirmed，这样可以将action为confirmed的行返回1，其他返回0，再求和便可以巧妙计算出confirmed行数

```mysql
select
    s.user_id, count(c.action), sum(c.action = 'confirmed')
from
    Signups as s
left join
    Confirmations as c
on
    s.user_id = c.user_id
group by
    s.user_id
```

---

### 3、求出最终答案

> 有了confirmed数和action总数，相除就可以得出最终的答案，但是由于前面用sum求confirmed时没有action的记录也就根本没有计算的数量默认给了null最后除的时候我们需要用ifnull函数给这个空值赋值
>
> 同时要注意四舍五入用round函数来保留两位小数

```mysql
# Write your MySQL query statement below
select
    s.user_id, 
    ifnull(round(sum(action = 'confirmed') / count(s.user_id), 2), 0) as confirmation_rate 
from
    Signups as s
left join
    Confirmations as c
on
    s.user_id = c.user_id
group by
    s.user_id
```

----

## 法2——avg求平均

> 将action为confirmed看做值1，其余看做0，那么直接用avg求平均就可以非常巧妙的算出confirmed率

```mysql
SELECT
    s.user_id,
    ROUND(IFNULL(AVG(c.action='confirmed'), 0), 2) AS confirmation_rate
FROM
    Signups AS s
LEFT JOIN
    Confirmations AS c
ON
    s.user_id = c.user_id
GROUP BY
    s.user_id
```

