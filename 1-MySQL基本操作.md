# MySQL基本操作
```
$ mysql -u root -p
```
`-u` 用户名，`-p` 输入密码， `-h` 主机名， `-P` 端口，
`mysql --help` 命令行选项和参数列表

连接到数据库需要：主机名（本地为localhost）、端口（如果使用默认端口3306之外的端口）、合法的用户名、用户口令（如果需要）

可能需要root权限

## 使用MySQL
下载create.sql和populate.sql两个sql脚本文件，其中，create.sql包含创建6个数据库表的MySQL语句，populate.sql包含用来填充这些表的INSERT语句。执行下列操作：
```
--创建数据库
CREATE DATABASE crashcourse；
--使用数据库
--必须先使用USE打开数据库，才能读取其中的数据。
USE crashcourse;
--执行sql脚本
--需要指定文件的完整路径
--先执行create.sql，再执行populate.sql
SOURCE ~/create.sql;
SOURCE ~/populate.sql
```
以上为准备工作。

### 了解数据库和表
```
-- 显示可用的数据库列表
SHOW DATABASES;

-- 获得一个数据库内的表的列表
SHOW TABLES;

-- 用来显示表列
SHOW COLUMNS FROM customers;
DESCRIBE customers;

-- 其他SHOW语句
SHOW STATUS -- 用于显示广泛的服务器状态信息
SHOW CREATE DATABASE -- 显示创建特定数据库的MySQL语句
SHOW CREATE TABLE -- 显示创建特定表的语句
SHOW GRANTS -- 显示授予用户（所有用户或特定用户）的安全权限
SHOW ERRORS -- 显示服务器错误
SHOW WARNINGS -- 警告信息
```

## 检索数据
### SELECT语句
```
-- 检索单个列
-- 检索products表中的prod_name列
SELECT prod_name FROM products;

-- 检索多个列
-- 检索products表中的prod_id，prod_name和prod_price列
SELECT prod_id, prod_name, prod_price FROM products;

-- 检索所有列
--检索products表中的所有列
SELECT * FROM products;

-- 检索不同的行
-- DISTINCT关键字必须直接放在列名的前面，不能部分使用DISTINCT，DISTINCT关键字应用于所有列而不仅是前置它的列。
SELECT DISTINCT vend_id FROM products;

-- 限制结果，指定返回前几行
-- 返回不多于5行
SELECT prod_name FROM products LIMIT 5;
-- 返回从第5行开始的5行
SELECT prod_name FROM products LIMIT 5,5;
```
**检索出来的第一行为行0**，因此`LIMIT 1,1`检索出来的是第二行而不是第一行
MySQL 5 支持LIMIT的另一种替代语法
`LIMIT 4 OFFSET 3`为从行3开始取4行，同`LIMIT 3,4`

```
-- 使用完全限定的表名
SELECT products.prod_name FROM products;
SELECT products.prod_name FROM crashcoures.products;
```
## 排序检索数据
```
-- 排序数据
SELECT prod_name
FROM products
ORDER BY prod_name;
-- 按多个列排序
SELECT prod_id, prod_price, prod_name
FROM products
ORDER BY prod_price, prod_name;
```
对于上述例子中的输出，仅在多个行具有相同的prod_price 值时才对产品按prod_name进行排序。如果prod_price列中所有的值都是唯一的，则不会按prod_name排序。

```
-- 指定排序方向
-- 默认升序排序，降序使用DESC关键字
SELECT prod_id, prod_price, prod_name
FROM products
ORDER BY prod_price DESC;

SELECT prod_id, prod_price, prod_name
FROM products
ORDER BY prod_price DESC, prod_name;
```
DESC关键字只应用到直接位于其前面的列名。上例中，只对prod_price列指定DESC，对prod_name列不指定。
升序关键字ASC，可省略

### 找出一列中最高或最低的值
```
SELECT prod_proce FROM products
ORDER BY prod_price DESC LIMIT 1;
```
给出ORDER BY句子时，应保证位于FROM句子之后，如果使用LIMIT，应位于ORDER BY之后。

## 过滤数据
### 使用WHERE子句
```
-- 返回prod_price为2.50的行
SELECT prod_name, prod_price FROM products WHERE prod_price = 2.50
```
### WHERE子句操作符

| 符号 | 说明 |
|--------|--------|
|=|等于|
|<>|不等于|
|！=|不等于|
|<|小于|
|<=|小于等于|
|>|大于|
|>=|大于等于|
|BETWEEN|在指定的两个值之间|

```
-- 检查单个值
-- 返回prod_name为Fuses的一行（匹配时默认不区分大小写）
SELECT prod_name, prod_price FROM products WHERE prod_name = 'fuses';
-- 列出小于10美元的所有产品
SELECT prod_name, prod_price FROM products WHERE prod_price < 10;
-- 列出小于等于10美元的所有产品
SELECT prod_name, prod_price FROM products WHERE prod_price <= 10;

-- 不匹配检查
-- 列出不是1003的所有产品
SELECT vend_id, prod_name FROM products WHERE vend_id <> 1003;
SELECT vend_id, prod_name FROM products WHERE vend_id ！= 1003;

-- 范围值检查
-- 检索价格在5-10美元之间的所有产品
SELECT prod_name, prod_price FROM products
WHERE prod_price BETWEEN 5 AND 10;

-- 空值检查
-- 返回价格为空的所有产品
SELECT prod_name FROM products WHERE prod_price IS NULL;
```


## 创建和操纵表

### 创建表
#### 创建表基础
CREATE TABLE
- 新表的名字，在关键字CREATE TABLE之后给出
- 表列的名字和定义，用逗号分隔。
例：
```
CREATE TABLE customers
(
    cust_id int NOT NULL AUTO_INCREMENT,
    cust_name char(50) NOT NULL,
    cust_address char(50) NULL,
    cust_city char(50) NULL,
    cust_state char(5) NULL,
    cust_zip char(10) NULL,
    cust_country char(50) NULL,
    cust_contact char(50) NULL,
    cust_email char(255) NULL,
    PRIMARY KEY (cust_id)
) ENGINE=InnoDB;
```

在创建新表时，指定的表名必须不存在，否则将出错。如果要防止意外覆盖已有的表，SQL要求首先手工删除该表，然后再重建它，而不是简单地用创建表语句覆盖它。

如果你仅想在一个表不存在时创建它，应该在表名后给出IF NOT EXISTS。这样做不检查已有表的模式是否与你打算创建的表模式相匹配。它只是查看表名是否存在，并且仅在表名不存在时创建它。

#### 使用NULL值
每个表列或者是NULL列或者是NOT NULL列，这种状态在创建时由表的定义规定
例：
```
CREATE TABLE orders
(
    order_num int NOT NULL AUTO_INCREMENT,
    order_date datetime NOT NULL,
    cust_id int NOT NULL,
    PRIMARY KEY (order_num)
) ENGINE-InnoDB;
```

例：混合了NULL和NOT NULL列的表
```
CREATE TABLE vendors
(
    vend_id int NOT NULL AUTO_INCREMENT,
    vend_name char(50) NOT NULL,
    vend_address char(50) NULL,
    vend_city char(50) NULL,
    vend_state char(5) NULL,
    vend_zip char(10) NULL,
    vend_country char(50) NULL,
    PRIMARY KEY(vend_id)
) ENGINE = InnoDB;
```

#### 主键
主键值必须唯一。如果主键使用单个列，则它的值必须唯一。如果使用多个列，则这些列的组合值必须唯一。

例：创建多个列组成的主键
```
CREATE TABLE orderitems
(
    order_num int NOT NULL,
    order_item int NOT NULL,
    prod_id char(10) NOT NULL,
    quantity int NOT NULL,
    item_price decimal(8,2) NOT NULL,
    PRIMARY KEY (order_num, order_item)
)ENGiNE = InnoDB;
```

#### 使用AUTO_INCREMENT
AUTO_INCREMENT告诉MySQL，本列每当增加一行时自动增量。每次 执行一个INSERT操作时，MySQL自动对该列增量（从而才有这个关键字AUTO_INCREMENT），给该列赋予下一个可用的值。这样给每个行分配一个唯一的cust_id，从而可以用作主键值。

覆盖AUTO_INCREMENT:如果一个列被指定为AUTO_INCREMENT，则它需要使用特殊的值吗？你可以简单地INSERT语句中指定一个值，只要它是唯一的（至今尚未使用过）即可，该值将被用来替代自动生成的值。后续的增量将开始使用该手工插入的值。

#### 指定默认值
```
CREATE TABLE orderitems
(
    order_num int NOT NUL,
    order_item int NOT NULL,
    prod_id char(10) NOT NULL,
    quantity int NOT NULL DEFAULT 1,
    item_price decimal(8,2) NOT NULL,
    PRIMARY KEY (order_num,order_item)
) ENGINE = InnoDB;
```
MySQL不允许使用函数作为默认值，只支持常量

#### 引擎类型
- InnoDB是一个可靠的事务处理引擎，它不支持全文本搜索；
- MEMORY在功能等同于MyISAM，但由于数据存储在内存（不是磁盘） 中，速度很快（特别适合于临时表）；
- MyISAM是一个性能极高的引擎，它支持全文本搜索，但不支持事务处理。
引擎类型可以混用。
外键不能跨引擎 混用引擎类型有一个大缺陷。外键（用于强制实施引用完整性）不能跨引擎，即使用一个引擎的表不能引用具有使用不同引擎的表的外键。

### 更新表
使用ALTER TABLE更改表的结构，必须给出以下信息：
- 在ALTER TABLE之后给出要更改的表名（该表必须存在，否则将出错）；
- 所做更改的列表。

例：
```
ALTER TABLE vendors
ADD vend_phone CHAR(20);
```
例：删除刚增加的列
```
ALTER TABLE vendors
DROP COLUMN vend_phone;
```


为了对单个表进行多个更改，可以使用单条ALTER TABLE语句，每个更改用逗号分隔

复杂的表结构更改一般需要手动删除过程，它涉及以下步骤：
- 用新的列布局创建一个新表；
- 使用INSERT SELECT语句从旧表复制数据到新表。如果有必要，可使用转换函数和计算字段；
- 检验包含所需数据的新表；
- 重命名旧表（如果确定，可以删除它）；
- 用旧表原来的名字重命名新表；
- 根据需要，重新创建触发器、存储过程、索引和外键。

使用ALTER TABLE要极为小心，应该在进行改动前做一个完整的备份（模式和数据的备份）。数据库表的更改不能撤销，如果增加了不需要的列，可能不能删除它们。类似地，如果删除了不应该删除的列，可能会丢失该列中的所有数据。

### 删除表
```
DROP TABLE customers2;
```

### 重命名表
```
RENAME TABLE customers2 TO customers;

#对多个表重命名
RENAME TABLE backup_customers TO customers,
	         backup_vendors TO vendors,
             backup_products TO products;
```

## 插入数据
INSERT
- 插入完整的行
- 插入行的一部分
- 插入多行
- 插入某些查询的结果

### 插入完整的行

```
INSERT INTO Customers
VALUES(NULL,
    'Pep E. LaPew',
    '100 Main Street',
    'Los Angles',
    'CA',
    '90046',
    'USA',
    NULL,
    NULL);
```
语法简单但不安全。更安全的方法为：
```
INSERT INTO customers(cust_name,
    cust_address,
    cust_city,
    cust_state,
    cust_zip,
    cust_country,
    cust_contact,
    cust_email)
VALUES('Pep E. LaPew',
    '100 Main Street',
    'Los Angeles',
    'CA',
    '90046'
    'USA'
    NULL,
    NULL);
#下面的INSERT语句填充所有列（与前面的一样），但以一种不同的次序填充。
#因为给出了列名，所以插入结果仍然正确：
INSERT INTO customers(cust_name,
    cust_contact,
    cust_email,
    cust_address,
    cust_city,
    cust_state,
    cust_zip,
    cust_country)
VALUES('Pep E. LaPew',
    NULL,
    NULL,
    '100 Main Street',
    'Los Angles',
    'CA',
    '90046',
    'USA');
```
不管哪种INSSERT语法，都必须给出VALUES的正确数目，如果不提供列名，则必须给每个表提供一个值。如果提供列名，则必须对每个列出的列值给出一个值。

列名被明确列出时，可以省略列,如果表的定义允许则可以省略列

- 该列定义为允许NULL值（无值或空值）
- 在表定义中给出默认值。

### 插入多个行
```
INSERT INTO customers(cust_name,
    cust_address,
    cust_city,
    cust_state,
    cust_zip,
    cust_country)
VALUES('Pep E. LaPew',
    '100 Main Street'
    'Los Angeles',
    'CA',
    '90046',
    'USA');
INSERT INTO customers(cust_name,
    cust_address,
    cust_city,
    cust_state,
    cust_zip,
    cust_country)
VALUES('M. Martian',
    '42 Galaxy Way'
    'New York',
    'NY',
    '11213',
    'USA');

#使用组合句
INSERT INTO customers(cust_name,
    cust_address,
    cust_city,
    cust_state,
    cust_zip,
    cust_country)
VALUES('Pep E. LaPew',
    '100 Main Street'
    'Los Angeles',
    'CA',
    '90046',
    'USA')，

    ('M. Martian',
    '42 Galaxy Way'
    'New York',
    'NY',
    '11213',
    'USA');

#单条INSERT语句有多组值，每组值用一对圆括号括起来，用逗号分隔。
```
### 插入检索出的数据

```
INSERT INTO customers(cust_id,
    cust_contact,
    cust_email,
    cust_name,
    cust_address,
    cust_city,
    cust_state,
    cust_zip,
    cust_country)
SELECT cust_id,
    cust_contact,
    cust_email,
    cust_name,
    cust_address,
    cust_city,
    cust_state,
    cust_zip,
    cust_country
FROM custnew;
```

## 更新和删除数据
### 更新数据
UPDATE
- 更新表中特定行
- 更新表中所有行
例：客户10005更新电子邮件
```
UPDATE customers
SET cust_email = 'elmer@fudd.com'
WHERE cust_id = 10005;
```
例：更新多个列
```
UPDARTE customers
SET cust_name = 'The Fudds',
cust_email = 'elmer@fudd.com'
WHERE cust_id = 10005;
```
在更新多个列时，只需要使用单个SET命令，每个“列=值”对之间 用逗号分隔（最后一列之后不用逗号）。在此例子中，更新客户10005的cust_name和cust_email列。

IGNORE关键字:如果用UPDATE语句更新多行，并且在更新这些 行中的一行或多行时出一个现错误，则整个UPDATE操作被取消 （错误发生前更新的所有行被恢复到它们原来的值）。为即使是发生错误，也继续进行更新，可使用IGNORE关键字，如下所示：`UPDATE IGNORE customers…`

为了删除某列的值，可以设置为NULL
```
UPDATE customers
SET cust_email = NULL
WHERE cust_id = 10005;
```

### 删除数据
使用DELETE语句
- 从表中删除特定的行
- 从表中删除所有的行

```
DELETE FROM customers
WHERE cust_id = 10006;
```
### 更新和删除的指导原则
下面是许多SQL程序员使用UPDATE或DELETE时所遵循的习惯。
- 除非确实打算更新和删除每一行，否则绝对不要使用不带WHERE子句的UPDATE或DELETE语句。
- 保证每个表都有主键，尽可能像WHERE子句那样使用它（可以指定各主键、多个值或值的范围）。
- 在对UPDATE或DELETE语句使用WHERE子句前，应该先用SELECT进行测试，保证它过滤的是正确的记录，以防编写的WHERE子句不正确。
- 使用强制实施引用完整性的数据库，这样MySQL将不允许删除具有与其他表相关联的数据的行。

## 补充
MySQL的注释方法
一共有三种，分别为
```
#单行注释可以使用"#"
-- 单行注释也可以使用"--"，注意与注释之间有空格
/*
用于多行注释
*/
```
