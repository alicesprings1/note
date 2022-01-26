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

  <img src="SQL语句总结.assets/image-20220113003754426.png" style="zoom:50%;" />

- 日期和时间处理函数

  ```mysql
  select order_num
  from Orders
  where YEAR(order_date)=2012;
  ```

- 数值处理函数

  <img src="SQL语句总结.assets/image-20220113004042516.png" style="zoom:50%;" />

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

### 数据分组

- 创建分组

  ```mysql
  select vend_id,count(*) as num_products
  from Products
  GROUP BY vend_id;
  ```

  - GROUP BY子句指示DBMS分组数据，然后对每个组而不是整个结果集进行聚集
  - GROUP BY子句可以包含任意数目的列，因而可以对分组进行嵌套，更细致地进行数据分组
  - 如果在GROUP BY子句中嵌套了分组，数据将在最后指定的分组上进行汇总
  - GROUP BY子句中列出的每一列都必须是检索列或有效的表达式（但不能是聚集函数）。如果在SELECT中使用表达式，则必须在GROUP BY子句中指定相同的表达式。不能使用别名
  - 大多数SQL实现不允许GROUP BY列带有长度可变的数据类型（如文本或备注型字段）
  - 除聚集计算语句外，SELECT语句中的每一列都必须在GROUP BY子句中给出
  - 如果分组列中包含具有NULL值的行，则NULL将作为一个分组返回。如果列中有多行NULL值，它们将分为一组
  - GROUP BY子句必须出现在WHERE子句之后，ORDER BY子句之前

- 过滤分组

  ```mysql
  select cust_id,count(*) as orders
  from Orders
  GROUP BY cust_id
  HAVING COUNT(*)>=2;
  ```

  - HAVING和WHERE类似，不同点在于HAVING作用于分组，WHERE作用于行

    ```mysql
    select vend_id,count(*) as num_prods
    from Products
    where prod_price>=4
    group by vend_id
    having cound(*) >=2;
    ```

  - 若不指定group by，大多数DBMS会同等对待having和select。但使用HAVING时应该结合GROUP BY子句，而WHERE子句用于标准的行级过滤。

- 分组和排序 ORDER BY

  <img src="SQL语句总结.assets/image-20220120211618343.png" alt="image-20220120211618343" style="zoom:70%;" />

  - 一般在使用GROUP BY子句时，应该也给出ORDER BY子句。这是保证数据正确排序的唯一方法。千万不要仅依赖GROUP BY排序数据。

    ```mysql
    select order_num,count(*) as items
    from OrderItems
    GROUP BY order_num
    having count(*)>=3
    ORDER BY items,order_num;
    ```

- select子句顺序

  <img src="SQL语句总结.assets/image-20220120212604956.png" alt="image-20220120212604956" style="zoom:67%;" />

### 使用子查询

- 利用子查询进行过滤
  - 作为子查询的SELECT语句只能查询单个列。企图检索多个列将返回错误。

即嵌套在其它查询中的查询。

```mysql
select cust_name,cust_contact
from Customers
where cust_id IN(
    select cust_id 
    from Orders
    where order_num IN(
        select order_num
        from OrderItems
        where prod_id='RGAN01'
    )
);
```

- 作为计算字段使用子查询

  ```mysql
  select cust_name,cust_state,(
      select count(*) 
      from Orders
      where Orders.cust_id=Customer.cust_id
  ) as orders
  from Customers
  order by cust_name;
  ```

  - 如果在SELECT语句中操作多个表，就应使用完全限定列名来避免歧义。

### 联结表

join操作

- 创建联结

  ```mysql
  select vend_name,prod_name,prod_price
  from Vendors,Products
  where Vendors.vend_id=Products.vend_id;
  ```

  - 由没有联结条件的表关系返回的结果为笛卡儿积。检索出的行的数目将是第一个表中的行数乘以第二个表中的行数。
  - 有时，返回笛卡儿积的联结，也称叉联结（cross join）。

- 内联结

  - 目前为止使用的联结称为等值联结（equijoin），它基于两个表之间的相等测试。这种联结也称为内联结（inner join）。其实，可以对这种联结使用稍微不同的语法，明确指定联结的类型。下面的SELECT语句返回与前面例子完全相同的数据：

  ```mysql
  select vend_name,prod_name,prod_price
  from Vendors INNER JOIN Products
  ON Vendors.vend_id=Products.vend_id;
  ```

- 联结多个表

  ```mysql
  select prod_name,vend_name,prod_price,quantity
  from OrderItems,Products,Vendors
  where Products.vend_id=Vendors.vend_id
  AND OrderItems.prod_id=Products.prod_id
  AND order_num=20007;
  ```

  - 不要联结不必要的表。联结的表越多，性能下降越厉害。

### 创建高级联结

- 使用表别名

  ```mysql
  select cust_name,cust_contact
  from Customers as C, Orders as O, OrderItems as OI
  where C.cust_id=O.cust_id
  AND OI.order_num=O.order_num
  AND prod_id='RGAN01';
  ```

  - 表别名只在查询执行中使用。与列别名不一样，表别名不返回到客户端。

- 使用不同类型的联结

  - 自联结 self join

    ```mysql
    select c1.cust_id,c1.cust_name,c1.cust_contact
    from Customers as c1, Customers as c2
    where c1.cust_name=c2.cust_name
    and c2.cust_contact='Jim Jones';
    ```

    - 自联结通常作为外部语句，用来替代从相同表中检索数据的使用子查询语句。虽然最终的结果是相同的，但许多DBMS处理联结远比处理子查询快得多。应该试一下两种方法，以确定哪一种的性能更好。

  - 自然联结 natural join

    - 自然联结排除多次出现，使每一列只返回一次。自然联结要求你只能选择那些唯一的列，一般通过对一个表使用通配符（SELECT ＊），而对其他表的列使用明确的子集来完成。

    ```mysql
    select C.*, O.order_num, O.order_date, OI.prod_id, OI.quantity, OI.item_price
    from Customers as C, Orders as O, OrderItems as OI
    where C.cust_id=O.cust_id
    AND OI.order_num=O.order_num
    AND prod_id='RGAN01';
    ```

  - 外联结 outer join

    - 许多联结将一个表中的行与另一个表中的行相关联，但有时候需要包含没有关联行的那些行。联结包含了那些在相关表中没有关联行的行。这种联结称为外联结。在使用OUTER JOIN语法时，必须使用RIGHT或LEFT关键字指定包括其所有行的表（RIGHT指出的是OUTER JOIN右边的表，而LEFT指出的是OUTER JOIN左边的表）。

      ```mysql
      select Customers.cust_id, Orders.order_num
      from Customers 	LEFT OUTER JOIN Orders
      ON Customers.cust_id=Orders.cust_id;
      ```

  - 使用带聚集函数的联结

    ```mysql
    select Customers.cust_id,
    		count(Orders.order_num) as num_ord
    from Customers INNER JOIN Orders
    	ON Customers.cust_id=Orders.cust_id
    group by Customers.cust_id;
    ```

    ### 组合查询

    多数SQL查询只包含从一个或多个表中返回数据的单条SELECT语句。但是，SQL也允许执行多个查询（多条SELECT语句），并将结果作为一个查询结果集返回。这些组合查询通常称为并（union）或复合查询（compound query）。任何具有多个WHERE子句的SELECT语句都可以作为一个组合查询

  - 创建组合查询

    - 使用UNION

      ```mysql
      select cust_name,cust_contact,cust_email
      from Customers
      where cust_state in('IL','IN','MI')
      UNION
      select cust_name,cust_contact,cust_email
      from Customers
      where cust_name='Fun4All';
      ```

      ```mysql
      select cust_name,cust_contact,cust_email
      from Customers
      where cust_state in('IL','IN','MI') or cust_name='Fun4All'; 
      ```

      在这个简单的例子中，使用UNION可能比使用WHERE子句更为复杂。但对于较复杂的过滤条件，或者从多个表（而不是一个表）中检索数据的情形，使用UNION可能会使处理更简单

  - UNION规则

    - UNION必须由两条或两条以上的SELECT语句组成，语句之间用关键字UNION分隔
    - UNION中的每个查询必须包含相同的列、表达式或聚集函数（不过，各个列不需要以相同的次序列出）。
    - 列数据类型必须兼容：类型不必完全相同，但必须是DBMS可以隐含转换的类型（例如，不同的数值类型或不同的日期类型）。

  - 包含或消除重复的行

    UNION从查询结果集中自动去除了重复的行；换句话说，它的行为与一条SELECT语句中使用多个WHERE子句条件一样。这是UNION的默认行为，如果愿意也可以改变它。事实上，如果想返回所有的匹配行，可使用UNION ALL而不是UNION。

    ```mysql
    select cust_name,cust_contact,cust_email
    from Customers
    where cust_state in('IL','IN','MI')
    UNION ALL
    select cust_name,cust_contact,cust_email
    from Customers
    where cust_name='Fun4All';
    ```

  - 对组合查询结果排序

    SELECT语句的输出用ORDER BY子句排序。在用UNION组合查询时，只能使用一条ORDER BY子句，它必须位于最后一条SELECT语句之后。对于结果集，不存在用一种方式排序一部分，而又用另一种方式排序另一部分的情况，因此不允许使用多条ORDER BY子句。

    ```mysql
    select cust_name,cust_contact,cust_email
    from Customers
    where cust_state in('IL','IN','MI')
    UNION
    select cust_name,cust_contact,cust_email
    from Customers
    where cust_name='Fun4All'
    order by cust_name,cust_contact;
    ```

    UNION在需要组合多个表的数据时也很有用，即使是有不匹配列名的表，在这种情况下，可以将UNION与别名组合，检索一个结果集。

### 插入数据

- 插入完整的行

  ```mysql
  insert into Customers
  values('10000006','Toy Land','NY');
  ```

  存储到表中每一列的数据在VALUES子句中给出，必须给每一列提供一个值。如果某列没有值，如上面的cust_contact和cust_email列，则应该使用NULL值（假定表允许对该列指定空值）。各列必须以它们在表定义中出现的次序填充。

  **虽然这种语法很简单，但并不安全，应该尽量避免使用。上面的SQL语句高度依赖于表中列的定义次序，还依赖于其容易获得的次序信息。即使可以得到这种次序信息，也不能保证各列在下一次表结构变动后保持完全相同的次序。因此，编写依赖于特定列次序的SQL语句是很不安全的，这样做迟早会出问题。最好明确给出列名**

```mysql
insert into Customers(
    cust_id,
    cust_name,
    cust_city
)
values('10000006','Toy Land','NY');
```

不要使用没有明确给出列的INSERT语句。给出列能使SQL代码继续发挥作用，即使表结构发生了变化。不管使用哪种INSERT语法，VALUES的数目都必须正确。如果不提供列名，则必须给每个表列提供一个值；如果提供列名，则必须给列出的每个列一个值。

- 插入部分行

  ```mysql
  insert into Customers(
  	cust_id,
      cust_name,
      cust_address,
      cust_city,
      cust_state,
      cust_zip,
      cust_country
  )
  values(
  	'10000006',
      'Toy Land',
      '123 Any Street',
      'New York',
      'NY',
      '11111',
      'USA'
  );
  ```

  在本课前面的例子中，没有给cust_contact和cust_email这两列提供值。这表示没必要在INSERT语句中包含它们。因此，这里的INSERT语句省略了这两列及其对应的值。

  - 省略列
    - 该列定义为允许NULL值（无值或空值）。
    - 或在表定义中给出默认值。这表示如果不给出值，将使用默认值。

- 插入检索出的数据 insert select

  ```mysql
  insert into Customers(
  	cust_id,
      cust_name,
      cust_address,
      cust_city,
      cust_state,
      cust_zip,
      cust_country
  )
  select cust_id,
      cust_name,
      cust_address,
      cust_city,
      cust_state,
      cust_zip,
      cust_country
  from CustNew;
  ```

  这个例子使用INSERT SELECT从CustNew中将所有数据导入Customers。为简单起见，这个例子在INSERT和SELECT语句中使用了相同的列名。但是，不一定要求列名匹配。事实上，**DBMS一点儿也不关心SELECT返回的列名。它使用的是列的位置**，因此SELECT中的第一列（不管其列名）将用来填充表列中指定的第一列，第二列将用来填充表列中指定的第二列，如此等等。

  INSERT通常只插入一行。要插入多行，必须执行多个INSERT语句。INSERTSELECT是个例外，它可以用一条INSERT插入多行，不管SELECT语句返回多少行，都将被INSERT插入。

- 从一个表复制到另一个表 

  ```mysql
  create table CustCopy as
  select * from Customers;
  ```

  这条语句创建一个名为CustCopy的新表，并把Customers表的整个内容复制到新表中。
  - 任何SELECT选项和子句都可以使用，包括WHERE和GROUP BY
  - 可利用联结从多个表插入数据
  - 不管从多少个表中检索数据，数据都只能插入到一个表中

### 更新和删除数据

- 更新数据 UPDATE

  - 组成

    - 要更新的表
    - 列名和它们的新值
    - 确定更新哪些行的过滤条件

    ```mysql
    update Customers
    set cust_email='aaa@bb.com'
    	cust_contact='Sam'
    where cust_id='1000000';
    ```

    UPDATE语句中可以使用子查询，使得能用SELECT语句检索出的数据更新列数据。

- 删除数据 DELETE

  ```mysql
  delete from Customers
  where cust_id='10000000';
  ```

  如果想从表中删除所有行，不要使用DELETE。可使用TRUNCATE TABLE语句，它完成相同的工作，而速度更快

### 创建和操纵表

- 创建表CREATE TABLE

  ```mysql
  create table Products
  (
      prod_id CHAR(10) NOT NULL,
      vend_id CHAR(10) NOT NULL,
      prod_name CHAR(254) NOT NULL
  );
  ```

  在创建新的表时，指定的表名必须不存在，否则会出错。防止意外覆盖已有的表，SQL要求首先手工删除该表（请参阅后面的内容），然后再重建它，而不是简单地用创建表语句覆盖它。

- 使用NULL值

  NULL值就是没有值或缺值。允许NULL值的列也允许在插入行时不给出该列的值。不允许NULL值的列不接受没有列值的行。

  多数DBMS中NULL为默认设置，不指定NOT NULL时，就认为指定的是NULL。

- 指定默认值 DEFAULT

  ```mysql
  create table OrderItems
  (
      order_num INTEGER NOT NULL,
      quantity INTEGER NOT NULL DEFAULT 1
  );
  ```

  默认值常用于时间或时间戳列。MYSQL用户指定 DEFAULT CURRENT_DATE()

- 更新表 ALTER TABLE

  ```mysql
  # 给已有表增加列
  alter table Vendors
  add vend_phone CHAR(20);
  
  # 删除列
  alter table Vendors
  drop column vend_phone;
  ```

  - 复杂表的结构更改一般需要手动删除过程，它涉及以下步骤
    - 用新的列布局创建一个新表
    - 使用INSERT SELECT语句从旧表复制数据到新表。有必要的话，可以使用转换函数和计算字段
    - 检验包含所需数据的新表
    - 重命名旧表（如果确定，可以删除它）
    - 用旧表原来的名字重命名新表
    - 根据需要，重新创建触发器、存储过程、索引和外键

- 删除表 DROP TABLE

- 重命名表 RENAME

### 使用视图

- 视图

  视图是虚拟的表。与包含数据的表不一样，视图只包含使用时动态检索数据的查询。

  ```mysql
  select cust_name,cust_contact
  from ProductCustomers
  where prod_id='RGAN01';
  ```

  ProductCustomers是一个视图，它不包含任何列或数据，包含的是一个查询（与上面用以正确联结表的查询相同）。

  - 视图的常见应用
    - 重用SQL语句。
    - 简化复杂的SQL操作。在编写查询后，可以方便地重用它而不必知道其基本查询细节。
    - 使用表的一部分而不是整个表。
    - 保护数据。可以授予用户访问表的特定部分的权限，而不是整个表的访问权限。
    - 更改数据格式和表示。视图可返回与底层表的表示和格式不同的数据。

  创建视图之后，可以用与表基本相同的方式使用它们。可以对视图执行SELECT操作，过滤和排序数据，将视图联结到其他视图或表，甚至添加和更新数据。

  视图仅仅是用来查看存储在别处数据的一种设施。视图本身不包含数据，因此返回的数据是从其他表中检索出来的。在添加或更改这些表中的数据时，视图将返回改变过的数据。

- 创建视图 CREATE VIEW

- 视图重命名 必须先删除它然后再重新创建 DROP VIEW viewname

- 利用视图简化复杂的联结

  ```mysql
  CREATE VIEW ProductCustomers AS
  select cust_name,cust_contact,prod_id
  from Customers,Orders,OrderItems
  where Customers.cust_id=Orders.cust_id
  and OrderItems.order_num=Orders.order_num;
  
  select cust_name,cust_contacnt
  from ProductCustomers
  where prod_id='RGAN01';
  ```

- 用视图重新格式化检索出的数据

  ```mysql
  create view VendorLocations as
  select RTRIM(vend_name)+' ('+RTRIM(vend_country)+' )'
  as vend_title
  from Vendors;
  ```

- 用视图过滤不想要的数据

  ```mysql
  create view CustomerEMailList as
  select cust_id,cust_name,cust_email
  from Customers
  where cust_email is not null;
  ```

- 使用视图与计算字段

  ```mysql
  create view OrderItemsExpanded as 
  select order_num,prod_id,quantity,item_price,
  		quantity*item_price as expanded_price
  from OrderItems;
  ```

### 使用存储过程

- 存储过程

  简单来说，存储过程就是为以后使用而保存的一条或多条SQL语句。可将其视为批文件，虽然它们的作用不仅限于批处理。

- 执行存储过程

  ```mysql
  EXECUTE AddNewProduct(
      'JTS01',
      'Stuffed Eifffel Tower',
      6.49,
      'Plush stuffed toy with the text La Tour Eiffel in red white and blue'
  );
  ```

- 创建存储过程

  ```mysql
  CREATE PROCEDURE MailingListCount(
      ListCount OUT INTEGER
  )
  IS
  v_rows INTEGER;
  BEGIN
  	select count(*) INTO v_rows
  	from Customers
  	where not cust_email is null
  	ListCount := v_rows;
  END;
  ```

  这个存储过程有一个名为ListCount的参数。此参数从存储过程返回一个值而不是传递一个值给存储过程。关键字OUT用来指示这种行为。Oracle支持IN（传递值给存储过程）、OUT（从存储过程返回值，如这里）、INOUT（既传递值给存储过程也从存储过程传回值）类型的参数。存储过程的代码括在BEGIN和END语句中，这里执行一条简单的SELECT语句，它检索具有邮件地址的顾客。然后用检索出的行数设置ListCount（要传递的输出参数）。

### 管理事务处理

事务处理是一种机制，用来管理必须成批执行的SQL操作，保证数据库不包含不完整的操作结果。利用事务处理，可以保证一组操作不会中途停止，它们要么完全执行，要么完全不执行（除非明确指示）。如果没有错误发生，整组语句提交给（写到）数据库表；如果发生错误，则进行回退（撤销），将数据库恢复到某个已知且安全的状态。

> 事务（transaction）指一组SQL语句
>
> 回退（rollback）指撤销指定SQL语句的过程
>
> 提交（commit）指将未存储的SQL语句结果写入数据库表
>
> 保留点（savepoint）指事务处理中设置的临时占位符（placeholder），可以对它发布回退（与回退整个事务处理不同）

事务处理用来管理INSERT、UPDATE和DELETE语句。不能回退SELECT语句（回退SELECT语句也没有必要），也不能回退CREATE或DROP操作。事务处理中可以使用这些语句，但进行回退时，这些操作也不撤销。

管理事务的关键在于将SQL语句组分解为逻辑块，并明确规定数据何时应该回退，何时不应该回退。

```mysql
START TRANSACTION
...
```

- 使用ROLLBACK

  ```mysql
  delete from Orders;
  ROLLBACK;
  ```

  在事务处理块中，DELETE操作（与INSERT和UPDATE操作一样）并不是最终的结果。

- 使用COMMIT

  一般的SQL语句都是针对数据库表直接执行和编写的。这就是所谓的**隐式提交**（implicit commit），即提交（写或保存）操作是自动进行的。在事务处理块中，提交不会隐式进行。不过，不同DBMS的做法有所不同。有的DBMS按隐式提交处理事务端，有的则不这样。

  ```mysql
  START TRANSACTION
  ...
  COMMIT TRANSACTION
  ```

- 使用保留点

  要支持回退部分事务，必须在事务处理块中的合适位置放置占位符。这样，如果需要回退，可以回退到某个占位符。

  ```mysql
  SAVEPOINT deletel;
  ```

  每个保留点都要取能够标识它的唯一名字，以便在回退时，DBMS知道回退到何处。

  ```mys
  ROLLBACK TO deletel;
  ```

### 游标

游标（cursor）是一个存储在DBMS服务器上的数据库查询，它不是一条SELECT语句，而是被该语句检索出来的结果集。在存储了游标之后，应用程序可以根据需要滚动或浏览其中的数据。游标主要用于交互式应用，其中用户需要滚动屏幕上的数据，并对数据进行浏览或做出更改。

- 创建游标

  ```mysql
  DECLARE CustCursor CURSOR
  FOR
  select * from Customers
  where cust_email is null
  ```

- 使用游标

  ```mysql
  OPEN CURSOR CustCursor
  ```

- 关闭游标

  ```mysql
  CLOSE CustCursor
  ```

  一旦游标关闭，如果不再次打开，将不能使用。第二次使用它时不需要再声明，只需用OPEN打开它即可。

### SQL高级特性

- 约束

  - 主键 PRIMARY KEY
  - 外键 FOREIGN KEY REFERENCES
  - 唯一约束 UNIQUE
  - 检查约束 CHECK

- 索引

  - 建立索引

    ```mysql
    CREATE INDEX prod_name_ind
    ON Products(prod_name);
    ```

- 触发器

  触发器是特殊的存储过程，它在特定的数据库活动发生时自动执行。触发器可以与特定表上的INSERT、UPDATE和DELETE操作（或组合）相关联。

  与存储过程不一样（存储过程只是简单的存储SQL语句），触发器与单个的表相关联。与Orders表上的INSERT操作相关联的触发器只在Orders表中插入行时执行。类似地，Customers表上的INSERT和UPDATE操作的触发器只在表上出现这些操作时执行。

