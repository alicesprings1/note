[TOC]

# SQL总结

### 选择数据

- select from
- select * from 
- 注意尽量少用通配符，使用*会先在字典中查找表中所有的列，降低性能

### 排序

- order by 
- order by desc （默认asc）
- order by 应该在where子句之后

### 过滤数据

- where condition
- 常见的条件判断
  - < > = !=
  - BETWEEN AND 包括边界
  - IS NULL 
  - 未知（unknown）有特殊的含义，数据库不知道它们是否匹配，所以在进行匹配过滤或非匹配过滤时，不会返回对应值为NULL的结果

### 高级数据过滤

- **组合where子句**
  - AND OR 操作符
  - where condition1 AND condition2 AND condition3
  - where condition1 OR condition2
  - 注意在SQL中AND运算符优先级高于OR，为确保正确逻辑请常用()
- **IN操作符**
  - IN 的作用与OR相当
  - IN操作符一般比一组OR操作符执行得更快
  - IN的最大优点是可以包含其他SELECT语句，能够更动态地建立WHERE子句

```mysql
select prod_name,prod_price
from Products
where vend_id IN ('DLL01','BSR01');
```

- **NOT操作符**

```mysql
select prod_name
from Products
where NOT vend_id='DLL01';
```

### 利用通配符过滤

- LIKE操作符

- % 通配符

  - 在搜索串中，%表示任何字符出现任意次数。例如，为了找出所有以词Fish起头的产品，可发布以下SELECT语句

  - %不会匹配NULL

  - 注意空格对匹配的影响，许多DBMS用空格填满字符串长度不足的部分

    ```mysql
    select prod_id,prod_name
    from Products
    # 'Fish%'表示搜索模式
    where prod_name LIKE 'Fish%'; 
    ```

    ```mysql
    select prod_id,prod_name
    from Products
    # %可以放在任意位置
    where prod_name LIKE '%bean bag%'; 
    ```

- _ 通配符

  - 只匹配单个字符

    ```mysql
    select prod_id,prod_name
    from Products
    where prod_name LIKE '__ inch teddy bear';# __匹配刚好两个字符
    ```

  - 不要过度使用通配符
  - 在确实需要使用通配符时，也尽量不要把它们用在搜索模式的开始处。把通配符置于开始处，搜索起来是最慢的

### 创建计算字段

- 拼接字段

  ```mysql
  select Concat(vend_name,' (',vend_country,')') # 拼接的字段、字符串用逗号分隔 Concat函数
  from Vendors;
  ```

  - 使用RTRIM()函数去除字符串右边多余的空格

    ```mysql
    select Concat(RTRIM(vend_name),' (',RTRIM(vend_country),')') # 拼接字段并去除字符串右边多余的空格
    from Vendors;
    ```

  - LTRIM() TRIM() 函数

  - 列别名（导出列）

    ```mysql
    select Concat(RTRIM(vend_name),' (',RTRIM(vend_country),')') 
    AS vend_title
    from Vendors;
    ```

  - 算数计算 + - * /

### 使用函数处理数据

- 文本处理函数

  <img src="D:\学习\note\MySQL\SQL语句总结.assets\image-20220113003754426.png" alt="image-20220113003754426" style="zoom:50%;" />

- 日期和时间处理函数

  ```mysql
  select order_num
  from Orders
  where YEAR(order_date)=2012;
  ```

- 数值处理函数

  <img src="D:\学习\note\MySQL\SQL语句总结.assets\image-20220113004042516.png" alt="image-20220113004042516" style="zoom:50%;" />

### 汇总数据

- 聚集函数

  - AVG()

    ```mysql
    select AVG(prod_price) as avg_price 
    from Products;
    ```

    - AVG忽略NULL值
    - AVG只能作用于单列

  - COUNT()

    ```mysql
    select COUNT(*) as num_cust # COUNT(*)对表中行数进行计数，不忽略NULL值
    from Customers;
    ```

    ```mysql
    select COUNT(cust_email) as num_cust # 只对cust_email列有值的行进行计数
    from Customers;
    ```

  - MAX() MIN()

    ```mysql
    select MAX(prod_price) as max_price
    from Products;
    ```

    - 虽然MAX()一般用来找出最大的数值或日期值，但许多（并非所有）DBMS允许将它用来返回任意列中的最大值，包括返回文本列中的最大值。在用于文本数据时，MAX()返回按该列排序后的最后一行。MAX()函数忽略列值为NULL的行。
    - 虽然MIN()一般用来找出最小的数值或日期值，但许多（并非所有）DBMS允许将它用来返回任意列中的最小值，包括返回文本列中的最小值。在用于文本数据时，MIN()返回该列排序后最前面的行。

  - SUM()

    ```mysql
    select SUM(item_price*quantity) as total_price
    from OrderItems;
    ```

    - SUM()函数忽略列值为NULL的行

- 聚集不同值

  - DISTINCT

    ```mysql
    select AVG(DISTINCT prod_price) as avg_price
    from Products;
    ```

    - DISTINCT 不能用于COUNT(*)

