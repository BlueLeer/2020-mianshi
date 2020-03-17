## 查询语句的基本格式

```mysql
#select 字段 1 , 字段 2 , 字段 3 , 表达式 , 函数 , ...
#from 表名
#where 条件
#group by 列名
#having 带组函数的条件
#order by 列名
```

例如:

`select  id,name from people where name = 'wangle';` 查询姓名为wangle的id和name信息

`select age,count(age) from people where age > 10 group by age having count(age) < 15;`对年龄大于10的人按照年龄来分组，同时统计出每组的人数，统计完以后，再对统计结果进行筛选，筛选出人数小于15的组。

> 巴拉巴拉说了这么大一堆，其实我隐含了 where子句和having子句的区别，那到底有什么区别呢？
>
> 1. where子句不能放在group by后面
> 2. having子句是和group by子句一起用的，放在group by后面，此时的作用相当于where。（啥，相当于where，懵了。。。可以这么说吧，group by分组用的，分完组以后，会得到一个分组后的结果，比如年龄相同的一个组，那么多个组，如果我们想知道那些分组人数小于15，这个时候就需要使用having子句了）
> 3. where后面不能跟聚合函数，例如 avg(age)，count(id)，sum(age)等，但是having可以。



## 字符函数

upper / lower / initcap/length / lpad / rpad / replace / trim

* upper  转换为大写，例如：`select id,upper(name) from people`
* lower  转换为小写
* length  取长度
* lpad 左补齐，例如：`select id,lpad(name,10,'*') from people;` name字段长度设置为10,不足的话在左边补 \*
* rpad  右补齐
* replace 字符替换
* trim 去除前后的空格
* concat 字符串连接，例如 `select id，concat(frist_name,concat(" ",last_name)) from people;`
* substring(str,a,b) 字符串提取，从字符串的第a位开始提取，提取b位，第一位是1。例如：`select substring(name,1,1) from people;`
* left(str,n)提取字符串最左边的n个字符，例如：`select id,left(name,2) from people;`
* right(str,n)