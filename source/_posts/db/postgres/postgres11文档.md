---
title: postgres11文档
p: db/postgres/postgres11文档
date: 2019-12-05 23:38:52
categories:
tags:
---

# postgres v11 官方文档阅读笔记
## 特点
1. 表继承
2. 表分割



## 概览
### 安装
参考 安装postgres.md

### pg架构
pg采用cs模式架构。服务端程序叫做postgres，其中客服端连接后服务端会为每个连接fork一个进程。pg采用多进程模型实现并发连接。其中postgres进程是master进程，负责创建子进程，master进程将在服务器一直运行。

### 访问数据库
安装pg后会自带一个客户端叫做psql。后续操作就直接使用psql进行了。

```
psql mydb
```
连接指定数据库，如果不指定数据库的话，就默认连接的是当前用户同名的数据库。

使用psql连接时提示符会有两种：
```
mydb=>
mydb=#
```
第二种代表你是当前数据库的超级用户。

### 创建表
在psql中直接输入sql语句，而且可以折行，psql在没遇到 分号 之前是不会结束读取输入的。

```sql
CREATE TABLE 表名(
    列名 类型,
    ......
);
```
### 插入一条数据
```sql
INSERT INTO 表名 VALUES(列1数据,......);

INSERT INTO 表名(列名1......) VALUES(列1数据......)
```
插入数据还有一种方便的方式，就是使用copy来实现，直接将文件中的内容插入到表中
```
COPY 表名 FROM 文件路径
```
### 查询
```sql
SELECT * FROM 表名;

SELECT 列名... FROM 表名;
```
### 连接查询
实现多张表之间的连接查询。连接分为如下几类：
- 内连接 inner join
- 自连接 表与自身相连接
- 左外连接 左表所有记录加上右边满足条件的记录。如果右边没有满足条件的记录，那么右边记录则是empty(null)
- 右外连接 右表所有记录加上左表满足条件的记录。左边如果没有满足条件的，则为empty(null)
- 全外连接 左边和右表所有记录，满足连接条件的则成为一条记录，不满足条件的单独成为一行。

### 聚合函数
和其他关系型数据库一样，pg也支持聚合函数，常用的有count(),sum(),max(),avg(),min()......

```sql
SELECT count(*) FROM 表名;
```

### 更新
```sql
UPDATE 表名 SET 列名=值/表达式,...... WHERE 条件;
```
### 删除
```sql
DELETE FROM 表名 WHERE 条件;
```

### 高级特性
#### 视图
视图是一个虚拟表，在表之上创建的一个新的角度去组织原有的表结构，所以视图的数据来源于表，一般用视图来组织新的数据结构当成数据查询的源头，更新数据一般不直接操作视图。
```sql
CREATE VIEW 视图名 AS SELECT语句;
```
#### 外键
一个表中的某列(外键)必须参考另一个表中的某列(常常是主键)

```sql
CREATE TABLE a(
    id integer primary key,
    a text;
);

CREATE TABLE b(
    id integer,
    aid integer REFERENCES a(id),
    b text
);

```
#### 事务
事务是数据库的一个重要组成部分。它是实现原子操作(一项任务需要多个步骤实现，而多个步骤是一个整体，要成的话每个步骤都要成功，否则一旦中间某个步骤出现失败，那么所有步骤都失败，数据库必须回滚到没有执行该任务之前的样子)的基本方式。
```sql
BEGIN;
操作语句......

SAVEPOINT 临时保存点1名字;

ROLLBACK TO 需要回滚到哪个保存点的名字;

COMMIT;
```
在事务中可以设定保存点，rollback的时候可以指定回滚到指定保存点的地方。如果没有设置保存点，直接使用ROLLBACK 而不是 ROLLBACK TO 那么就会回滚到整个事务开始之前的状态。

#### 窗口函数
窗口函数功能类似group这种，但是又有所区别。窗口函数不会影响最终结果的记录条数，而group by这种分组函数是要影响最终记录条数的。窗口函数配合聚合函数使用。
```sql
SELECT depname,name,salary,avg(salary) OVER (PARTITION BY depname) FROM empSalary; 
```
结果:
depname | name | salary | avg(salary)
-------- | ----- | ---- | -----
develop | 张三 | 200 | 300
develop | 李四 | 300 | 300 
develop | 王五 | 400 | 300
sales | 小米 | 400 | 400
sales | 小明 | 400 | 400

如果使用的是group by，那么结果应该是下面这样:
```sql
select depname,avg(salary) FROM empSalary GROUP BY depname;
```
depname | avg(salary) 
---- | ---------
develop |  300 
salels | 400 

#### 继承
和面向对象一样，pg支持表继承。这样就可以省略一些表的重复字段的书写。在不支持继承的数据库中，如果要定义一个城市表和一个首都表，而首都又是城市的一种，所以他们有很多相同的字段。那么在不支持继承的数据库中，如何实现的呢？

```sql
CREATE TABLE capitals (
    name text,
    population real,
    altitude int,
    state char(2)
);

CREATE TABLE non_capitals (
    name text,
    population real,
    altitude int
);
CREATE VIEW cities AS
    SELECT name, population, altitude FROM capitals
    UNION
    SELECT name, population, altitude FROM non_capitals;
```
如果只是查询，那么这种方式没有什么不对的。但是如果要插入一个城市数据，那么就不是那么方便的了。

下面是pg实现这样需求的方式：
```sql
CREATE TABLE cities (
    name text,
    population real,
    altitude int
);
CREATE TABLE capitals (
    state char(2)
) INHERITS (cities);
```
这样captials就有了cities的字段。

## SQL
### 语法
- 所有SQL语句都是 关键字 + 标识符 + 值构成的。
- 关键字则是SQL语句中的 SELECT, UPDATE 等；标识符则是表名，列名这些带有某种指代的词；值就是人为填写的值了，比如 WHERE 条件中的常量值，UPDATE 中 SET 的值等等。
- 双引号在pg中代表将该引号阔起的部分看成一个整体。当表名或列名中含有空格时这是非常有用的。同时双引号中还允许unicode码转义，只需要在引号前面加上 "U&"即可。比如 U&"\0441\043B\043E\043D" 

#### 常量
- 字符常量。在pg中字符串常量要使用单引号阔起。如果字符串常量中含有单引号，那么就要使用两个 **连续[中间空格都不能有]** 的单引号，此时前一个单引号相当于转义符。比如 SELECT 's''',得到的结果是 s'。
    - 带有C风格的转义符（\）时，在单引号前添加一个E/e字母就可以让C风格转义生效
    - 带有unicode转义时，在单引号前加上 U/u& 即可
    - pg特有的使用$符号来充当引号。这种方式很像PHP的heredoc。例子$SomeTag$Dianne's horse$SomeTag$  使用$$开头$$结束。其中$$之间可以加上一个名字。
- 位字符串
    - B'0110'
    - X'1FF'
- 数值常量
    - digits
    - digits.[digits][e[+-]digits]
    - [digits].digits[e[+-]digits]
    - digitse[+-]digits
- 操作符
    - \+ \- \* \/ \< \> \= \~ \! \@ \# \% \^ \& \| \` \?
    - \-\- 和 \/\*是注释的开始
- 特殊符号
    - \$ 如果紧跟一个数字，那么在function内部表示的是位置参数。其他情况当成普通字符
    - () 用来将表达式分组和强制优先。
    - [] 用来选择数组的某一项值。
    - , 多项之间的分割
    - ; SQL语句结束
    - : 数组切片
    - 优先级， 由高到低
    
操作符 | 结合行 | 含义
--- | --- | ---
. | left | schema，表，列连接符
:: | left | 类型转换
[] | left | 数组选择符
+ - | right | 正，负
^ | left | 幂
* / %  | left | 乘 除 取余
+ - | left | 加 减
其他操作符 | left | 其他操作符或用户自定义操作符
BETWEEN IN LIKE ILIKE SIMILAR | - | 范围，集合，字符串匹配
> >= < <= = <> | - | 比较运算符
IS ISNULL NOTNULL |- |
NOT | right | 逻辑非
AND | left | 逻辑与
OR | left | 逻辑或

#### 注释
两种风格
- -- 单行注释
- /\*多行注释\*/

#### 类型转换
- CASE(表达式 AS 需要转换成的类型)-------------SQL标准用法
- 类型 表达式  但这种方式只能用于简单类型，不能作用于数组等复杂类型
- 表达式::需要转换成的类型-----------------------pg特有用法
- 需要转换成的类型(表达式)--------------------这种是使用一个function来完成的，前提是要有相应类型的转换function才行。

#### 值表达式
- 列引用
```sql
表.列
schema.表.列
表.*   => 所有列
```
- 位置参数
```sql
CREATE FUNCTION dept(text) RETURNS dept
    AS $$ SELECT * FROM dept WHERE name = $1 $$
LANGUAGE SQL;
```

- 下标 用于数组
```sql
数组[下标]
数组[下标1:下标2]   => 切片
```

- 运算式
```
表达式 运算符 表达式
一元运算符 表达式
```

- 函数调用
```
函数名(参数...)
```

- 排序规则
```
表达式 COLLATE 规则
```

- 数组构造器
```
ARRAY[1,2,3+4]
ARRAY[1,2,22.7]::integer[]
ARRAY[ARRAY[1,2], ARRAY[3,4]]

```

- 行构造器
用来将数据组成新的行记录
```sql
ROW(1,2.5,'this is a test');
SELECT ROW(t.*, 42) FROM t;
```
**注意**：在pg 8.2之前行构造器中是不能使用 .* 表示表的所有列的。


#### function的调用

调用方式为: function名字(参数...)
- 按位置传参  function的名字(1,2,3)
- 按名字传参  function的名字(a => 1 ,b => 2)
- 混合以上两种传参方式 function的名字(1,2,c => 3)

**注意**：混合传参方式不能在调用聚合函数使用，但是能够作用于配合窗口函数的聚合函数。

### 数据定义

#### 表
```sql
-- 创建表
CREATE TABLE 表名(
    列名 类型,
    ......
);

-- 删除表
DROP TABLE 表名;
```
**注意**：表格列数是有限制的，根据列类型不同，列数量限制在250~1600不等。

#### 默认值
创建表的时候可以给列赋予一个默认值
```sql
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric DEFAULT 9.99
);
```

####　生成列
生成列不能够直接写入，在insert，update语句中不能指定该列，但是在delete时可以指定该列作为删除条件。
生成列的一些限制条件：
1. 不能在表达式中使用子查询或者引用其他表
2. 表达式中不能引用其他生成列
3. 不能有默认值和标识定义
4. 生成列不能成为键的一部分

生成列有两种类型
- stored   占据一列，在数据插入或更新时重新计算
- virtual  不占据存储，在数据读取时重新计算
```sql
-- 在创建表时指定：
-- 列名 类型 GENERATED ALWAYS AS 计算表达式 STORED/VIRTUAL

CREATE TABLE people (
    ...,
    height_cm numeric,
    height_in numeric GENERATED ALWAYS AS (height_cm / 2.54) STORED
);
```


#### 约束
创建表时也可以指定列的约束。
- CHECK 约束
    - 示例
    ```sql
    -- 匿名
    CREATE TABLE products (
        product_no integer,
        name text,
        price numeric CHECK (price > 0)
    );
    -- 命名,挂在字段上
    CREATE TABLE products (
        product_no integer,
        name text,
        price numeric CONSTRAINT positive_price  CHECK (price > 0)
    );
    -- 单独指定字段
    CREATE TABLE products (
        product_no integer,
        name text,
        price numeric,
        CHECK(条件)
    );
    -- 单独指定字段且命名
    CREATE TABLE products (
        product_no integer,
        name text,
        price numeric,
        CONSTRAINT 名字 CHECK(条件)
    );
    ```
- 非空约束 not null
    - 示例
    ```sql
    CREATE TABLE products (
        product_no integer NOT NULL,
        name text NOT NULL,
        price numeric
    );
    ```
- 唯一约束 unique
    - 示例
    ```sql
    -- 直接在列上指定
    CREATE TABLE products (
        product_no integer UNIQUE,
        name text,
        price numeric
    );
    -- 最后再指定
    CREATE TABLE products (
        product_no integer,
        name text,
        price numeric,
        UNIQUE (product_no)
    );
    -- 命名
    CREATE TABLE products (
        product_no integer CONSTRAINT must_be_different UNIQUE,
        name text,
        price numeric
    );
    -- 组合唯一约束。多列组成一个唯一约束
    CREATE TABLE example (
        a integer,
        b integer,
        c integer,
        UNIQUE (a, c)
    );
    ```
- 主键约束。主键约束其实就是唯一约束加上非空约束。
    - 示例
    ```sql
    -- 组合主键。和组合唯一约束一样。ac作为一个整体唯一索引看待。且ac都不能为null
    CREATE TABLE example (
        a integer,
        b integer,
        c integer,
        PRIMARY KEY (a, c)
    );
    -- 单列主键
    CREATE TABLE products (
        product_no integer PRIMARY KEY,
        name text,
        price numeric
    );
    ```
- 外键约束。约束该列的值一定出现在另一张表的某列中，如果另一个表中那列没有该值，那么插入到这个表就会失败。外键一定是一个表的某列参照另一个表的主键列。
    - 示例
    ```sql
    -- 指定参照products表的product_no列
    CREATE TABLE orders (
        order_id integer PRIMARY KEY,
        product_no integer REFERENCES products (product_no),
        quantity integer
    );
    -- 组合外键
    CREATE TABLE t1 (
        a integer PRIMARY KEY,
        b integer,
        c integer,
        FOREIGN KEY (b, c) REFERENCES other_table (c1, c2)
    );
    ```
    - 注意点<br />
        - 被参照的数据在删除的时候 (ON DELETE) 会报错，说不能够删除。那么就应该在外键那个表添加删除被参照数据时应该执行什么动作。
            1. 删除被参考数据，那么外键对应的记录也应该被删除. (CASCADE)
            2. 删除被参考数据时，外键对应的记录保留，不做任何操作.
            3. 删除被参考数据时，外键对应的记录设置为null. (SET NULL)
            4. ......，外键对应的记录设置为默认值。(SET DEFAULT)
            5. 删除被参考数据时，不允许删除，提示该数据被xx作为外键。(默认情况 NO ACTION，即值设定外键，没有指定ON DELETE执行什么操作；RESTRICT 也是这样，不过RESTRICT不允许在事务结束后才检查约束)
        - 被参照数据在更新时(ON UPDATE).
            1. 更新被参考数据，那么同步更新到外键对应记录中 (CASCADE)
    ```sql
    CREATE TABLE order_items (
        product_no integer REFERENCES products ON DELETE RESTRICT,
        order_id integer REFERENCES orders ON DELETE CASCADE,
        quantity integer,
        PRIMARY KEY (product_no, order_id)
    );
    ```
#### 系统列
系统列指的是pg数据库系统自带的列，用户自定义数据的列名不能与之冲突。
- oid
    - 表示对象id(object id)。创建表的时候使用 WITH OIDS选项或者设置了oids变量的配置时生效。
- tableoid
    - 表的oid。此列对于从继承层次结构中选择的查询特别有用，因为没有它，很难判断出哪一个单独的表来自哪里。tableoid可以与pg_class的oid列连接以获得表名。
- xmin
    - 此行版本的插入事务的标识（事务ID）。行版本是行的单独状态；行的每一次更新都为同一逻辑行创建新行版本。
- cmin
    - 插入事务中的命令标识符（从零开始）
- xmax
    - 删除事务的标识（事务ID），或未删除行版本的零。此列在可见行版本中是非零的。这通常表明删除事务尚未提交，或者尝试删除被回滚。
- cmax
    - 删除事务中的命令标识符，或为零。
- ctid
    - 行版本在其表中的物理位置。注意，虽然CTID可以很快地定位行版本，但是如果它被真空填充更新或移动，行的CTID将改变。因此CTID作为长期行标识符是无用的。应该使用OID，甚至更好的用户定义的序列号来标识逻辑行。

#### 修改表结构
##### 增加列
```sql
ALTER TABLE 表名 ADD COLUMN 列名 列数据类型;

-- 同时制定一个约束
ALTER TABLE products ADD COLUMN description text CHECK (description<> '');
```
**注意**：如果增加的列指定了默认值。数据库会物理更新每一条数据，所以如果你添加的列有默认值，但是有要让某些记录不会被更新为默认值，那么你创建该列时就要创建一个没有默认值的列，然后手动update。

##### 移除某列
```sql
ALTER TABLE 表名 DROP COLUMN 列名;
```
如果该列被某个外键参考了，那么用上面这种方式是删不掉的。于是需要使用到CASCADE，强制删除。
```sql
ALTER TABLE products DROP COLUMN description CASCADE;
```
##### 添加约束
```sql
ALTER TABLE products ADD CHECK (name <> '');
ALTER TABLE products ADD CONSTRAINT some_name UNIQUE (product_no);
ALTER TABLE products ADD FOREIGN KEY (product_group_id) REFERENCES product_groups;
```
##### 删除约束

选表->选列->删约束
```sql
ALTER TABLE products ALTER COLUMN product_no DROP NOT NULL;
```
##### 修改列默认值
```sql
ALTER TABLE products ALTER COLUMN price SET DEFAULT 7.77;

ALTER TABLE products ALTER COLUMN price DROP DEFAULT;
```
##### 修改列类型定义
选表->选列->修改定义
```sql
ALTER TABLE products ALTER COLUMN price TYPE numeric(10,2);
```
##### 修改列名
```sql
ALTER TABLE products RENAME COLUMN product_no TO product_number;
```
##### 修改表名
```sql
ALTER TABLE products RENAME TO items;
```

#### 权限

- 修改表拥有者
```
ALTER TABLE table_name OWNER TO new_owner
```
- 该用户赋予权限
```
GRANT UPDATE ON accounts TO joe
```
- 移除权限
```
REVOKE ALL ON accounts FROM PUBLIC
```





#### schema
目的:
1. 允许在一个数据库中，不需要一一指定用户就能共享一张表。
2. 可按逻辑对数据表进行分组，增加管理便利性。
3. 可将第三方应用的数据库表放入另一个schema中，避免和自己的数据库表冲突。

schema做了除用户以外的相互隔离。可以将schema看成一个子数据库。

每个数据库中默认有一个public的schema，这个是在执行sql时不指定schema的默认schema。

```sql
CREATE SCHEMA myschema;
CREATE TABLE myschema.mytable (
...
);
-- 如果不指定schema那么创建的表就在默认的schema (public)中
CREATE TABLE products ( ... );
CREATE TABLE public.products ( ... );
```
##### 删除空schema
```sql
DROP SCHEMA myschema;
```
#### 强制删除schema
```sql
DROP SCHEMA myschema CASCADE;
```

#### 搜索路径
当执行一条sql语句时，系统如何查找到指定的对象。通过 show search_path 就可以看到当前语句中的对象(表等等)查找schema顺序。排在前的有限搜索，$user表示跟当前用户同名的schema。

#### 继承
pg提供了表继承的特性。
特点:
- 在子表插入数据时，默认情况会在父表中也插入一条数据。
- 查询父表中的所有数据时，默认将插入子表时插入的记录也会出现在结果集中。(就是普通的select)
- 如果只想在父表中查询到哪些非插入字表而插入的记录时，可以使用ONLY关键字。
- 有了继承关系，表就有了tableoid字段。
- 约束继承只会继承非空约束，默认是会继承非空约束的，除非特别指明。其他类型的约束是不会继承的。
- 继承可以继承一个或多个表。如果所有父表以及子表中字段名字冲突，则要合并这些同名列。要想成功合并，那么就需要相同的数据类型而且有相同的约束。这里的约束就单指非空约束了，因为继承只会继承非空约束。
- 两个表之间建立继承关系有两种方式。1 是在创建表的时候使用 INHERITS 指定。 2 是在两个表创建之后使用 ALTER TABLE 再建立继承关系。前提是这两个表构成继承关系条件。

示例：
```sql
CREATE TABLE "A"(
    "Name" text
);

CREATE TABLE "B"(
    "Describe" text
) INHERITS("A");
```
```sql
-- 查询A中所有的数据
SELECT * FROM "A";
-- 和上等价
SELECT * FROM "A"*;

-- 只查不是因为向B插入数据而增加的记录
SELECT * FROM ONLY "A";

-- 查询记录所属的tableoid。 这样你就能看到哪些数据只属于这个表。
select tableoid,"Name" from "A";

-- 查找记录所属表名，而不只是tableoid
select p.relname,a.tableoid,"Name" from "A" a,pg_class p where a.tableid=p.oid;
```

#### 表的分割


#### 依赖跟踪
在类似外键约束这种情况下，删除被依赖的数据默认情况下是不允许的。通过给出的错误信息中的DETAIL信息你就能看到该数据的依赖关系。如果想要强制删除也是可以的，那么在DROP最后加上CASCADE就可以了，几乎所有的DROP语句都支持CASCADE。你可以使用RESTRICT代替CASCADE，那么就会严格进行依赖检查，和默认不写CASCADE的行为一样。

**注意**：在SQL标准中CASCADE和RESTRICT是必须出现在DROP语句中的，只是默认情况(不填这两个单词)到底是CASCADE还是RESTRICT根据数据库系统而有所区别。

### 数据操纵

#### 插入数据
```sql

-- 所有列
INSERT INTO products VALUES (1, 'Cheese', 9.99);

-- 指定列
INSERT INTO products (product_no, name, price) VALUES (1, 'Cheese',9.99);

-- PG扩展格式。 从左往右赋值，右边没有赋值到的字段使用默认值。
INSERT INTO products VALUES (1, 'Cheese');

-- 多行插入
INSERT INTO products (product_no, name, price) VALUES
(1, 'Cheese', 9.99),
(2, 'Bread', 1.99),
(3, 'Milk', 2.99);

-- 用其他表数据作为插入数据源
INSERT INTO products (product_no, name, price)
SELECT product_no, name, price FROM new_products
WHERE release_date = 'today';

```

#### 更新数据
语句分为三部分:
- 要更新的表
- 要更新的列以及新值
- 要更新哪些记录(条件)

```sql
-- 常量值
UPDATE products SET price = 10 WHERE price = 5;
-- 表达式
UPDATE products SET price = price * 1.10;
-- 多列
UPDATE mytable SET a = 5, b = 3, c = 1 WHERE a > 0;
```

#### 删除数据
```sql
-- 删除满足条件的记录
DELETE FROM products WHERE price = 10;
-- 删除所有数据
DELETE FROM products;
```

#### 操纵数据的同时返回数据表(RETURNING)
通过标题理解起来肯定费解。通过例子你就能恍然大悟了。在我们insert数据到表后，往往我们需要我们刚刚插入的数据的id是好多。此时使用这个东西就能在插入数据后将id返回给我们。
```sql
-- 返回一个字段
INSERT INTO users (firstname, lastname) VALUES ('Joe', 'Cool') RETURNING id;
-- 返回多个字段
UPDATE products SET price = price * 1.10 WHERE price <= 99.99 RETURNING name, price AS new_price;
-- 返回所有字段
DELETE FROM products WHERE obsoletion_date = 'today' RETURNING *;
```
RETURNING * 指代表的所有字段，无需一个一个字段罗列出来


### 查询
```sql
[WITH with_queries] SELECT select_list FROM table_expression [sort_specification]
```
#### 表表达式
- FROM。  FROM 表名1 [,表名2......]
- JOIN。 表1 join类型 表2 [join条件]
    - join类型分为：
        1. [INNER] JOIN
        2. LEFT [OUTER] JOIN
        3. RIGHT [OUTER] JOIN
        4. FULL [OUTER] JOIN
    - 条件
        1。 ON 布尔表达式。  使用布尔表达式作为条件。结果中会出现来自两个表的所有列(**包含"同名" 列**)。
        2. USING (列名,....)。 使用两表相同列名做等值连接。但是结果里面**不会出现 "同名" 列**。
        3. 
        
    - 说明
        1. 内连接。只显示满足条件的记录
        2. 左外。显示左表所有记录，右表只显示满足条件的记录。不满足条件的字段为null值。最后结果记录数为左表的记录数。
        3. 右外。显示右表所有记录，左表只显示满足条件的记录。不满足条件的字段为null值。最后结果记录数为右表的记录数。
        4. 全外。 显示左右两表所有记录，满足条件的则成为结果的一条记录。不满足条件的则要么左表字段为null，要么右表字段为null。结果记录数为左右两表记录数总和减去左右两表满足条件的记录数。
        
- 带别名。 FROM 表名 [AS] 别名
- 子查询。 FROM (SELECT * FROM table1) AS alias_name
    - LATERAL关键字，能够让子查询能够引用到子查询以外的表
```sql
SELECT * FROM a, LATERAL (SELECT * FROM b WHERE b.id=a.id)
```

- 表函数 function
表函数返回一个标量或返回表记录。使用表函数时可以把函数当成一个子查询来使用。可以使用select，join，where条件等。

#### where条件句
```sql
WHERE search_condition
```
#### group by 和 having
```sql
SELECT select_list
FROM ...
[WHERE ...]
GROUP BY grouping_column_reference
[, grouping_column_reference]...

```
having是对分组后的数据再进行一次过滤。

Group By中可SELECT可以使用聚合函数，以及DISTINCT，和group by的字段名

~~在pg中group by还有一些可选的分组方式。比如 grouping sets 对多个字段分别进行分组，然后得到结果。 cube 多个字段进行组合，然后对组合结果进行grouping sets。 rollup  过于复杂，需要时请阅读官方文档~~

#### Grouping Sets,CUBE,ROLLUP
- GROUPING SETS
    - 将结果按照指定的多个字段集合group得到结果，然后将结果做个并集作为整条sql语句的结果
```sql
表：
brand | size | sales
-------+------+-------
Foo | L | 10
Foo | M | 20
Bar | M | 15
Bar | L | 5

SELECT brand, size, sum(sales) FROM items_sold GROUP BY GROUPING
SETS ((brand), (size), ());
结果集：
brand | size | sum
-------+------+-----
Foo | | 30
Bar | | 20
| L | 15
| M | 35
| | 50

分析：
grouping sets 有三个字段集合1. (brand) 2. (size) 3. ()
结果集为 group by brand   group by size  group by *  的结果集的并集
```
- ROLLUP
```
ROLLUP(a,b,c,d)=grouping sets(
    (a,b,c,d),
    (a,b,c),
    (a,b),
    (a),
    ()
)
```
- CUBE
    - 排列组合
```
CUBE(a,b,c)=grouping sets(
    (a,b,c),
    (a,c),
    (a,b),
    (b,c),
    (a),
    (b),
    (c),
    ()
)
```
#### select 列表
1. 字段名列表，使用逗号表达式。[表名/表别名.]字段名 AS 列别名
2. DISTINCT 
    - SELECT DISTINCT select_list ... 对填写的所有列做distinct
    - SELECT DISTINCT ON (expression [, expression ...]) select_list ... 对指定列做distince，并且指定结果集字段
3. 结果集合并
    - query1 UNION [ALL] query2  并集
    - query1 INTERSECT [ALL] query2 交集
    - query1 EXCEPT [ALL] query2 差集
    集合运算条件：左右有相同的列数并且对应的列的数据类型相同

#### 排序
```sql
SELECT select_list
FROM table_expression
ORDER BY sort_expression1 [ASC | DESC] [NULLS { FIRST | LAST }]
[, sort_expression2 [ASC | DESC] [NULLS { FIRST |
LAST }] ...]

NULLS FIRST| LAST 选可以指定null值排在最前还是最后
```
#### limit和offset
```sql
SELECT select_list
FROM table_expression
[ ORDER BY ... ]
[ LIMIT { number | ALL } ] [ OFFSET number ]

LIMIT ALL 相当于没有limit
```
#### 值列表
```
VALUES ( expression [, ...] ) [, ...]
```
```sql
SELECT 1 AS column1, 'one' AS column2
UNION ALL
SELECT 2, 'two'
UNION ALL
SELECT 3, 'three';
可以写成
SELECT * FORM (VALUES(1,'one'),(2,'two'),(3,'three'))

```

#### WITH AS 子查询
with让子查询可以直接放在最开始位置，让后续的语句可以多次复用。
```sql
WITH regional_sales AS (
    SELECT region, SUM(amount) AS total_sales FROM orders GROUP BY region
), top_regions AS (
    SELECT region FROM regional_sales WHERE total_sales > (SELECT SUM(total_sales)/10 FROM regional_sales)
)
SELECT region,
product,
SUM(quantity) AS product_units,
SUM(amount) AS product_sales
FROM orders
WHERE region IN (SELECT region FROM top_regions)
GROUP BY region, product;
```
WITH 子句中还可以写modify数据的sql

### 数据类型

Name | aliases | description
---- | ----------- | -----------------
bigint | int8 | 有符号8字节整数
bigserial | serial8 | 自增型8字节整数
bit[(n)] |  | 固定长度位
bit varying [(n)] | varbit [(n)] | 变长位
boolean | bool | 布尔值
box | | 矩形
bytea | | 二进制数据
character [(n)] | char([n]) | 定长字符串
character varying [(n)] | varchar[(n)] | 变长字符串
cidr | | ipv4/ipv6网络地址
circle | | 圆
date | | 日历时间(年月日)
double precision | float8 | 8字节双精度浮点型
decimal | - | 用户指定精度decimal(整数位数,小数位数)
numeric  | - | 用户指定精度numeric(整数位数,小数位数)
inet | | ipv4/ipv6主机地址
integer |int,int4 | 有符号4字节整数
internal [(p)] | | 时间间隔
json | | json字符串
jsonb | | 二进制json数据
line | | 平面上的线
lseg | | 平面上的线段
macaddr |  | mac地址
macaddr8 | | EUI-64格式的mac地址
money | | 货币
numeric[(p,s)] | decimal[(p,s)] | 可选择精度的精确数值
path | | 平面几何路径
pg_lsn | | pg日志序列号
point | | 平面几何点
polygon | | 平面几何封闭路径
real | float4 | 4 字节单精度浮点型
smallint | int2 | 有符号2字节整数
smallserial | serial2 | 自增型2字节整数
serial | serial4 | 自增型4字节整数
text | | 变长字符串
time [(p)] [with/without time zone] | 带时区的简写为timetz | 带/不带时区的日期
timestamp [(p)] [with/without time zone] | 带时区的简写为timestamptz |  带/不带时区的日期+时间
tsquery | | 全文检索查询
tsvector | | 全文检索文档
txid_snapshot | | 用户级别事务ID快照
uuid | | 通用唯一标识符
xml | | xml字符串数据

#### 类型
- 数值
```
Infinity    正无穷
-Infinity   负无穷
NaN         非数，计算机不能表示的数
```
- 时间
    - timestamp [(p)] [without time zone]
    - timestamp [(p)] with time zone
    - date 
    - time [(p)] [without time zone]
    - time [(p)] with time zone
    - interval [fields] [(p)] 用于指定时间间隔
```
精度 p = 0~6
interval的fields字段如下
YEAR
MONTH
DAY
HOUR
MINUTE
SECOND
YEAR TO MONTH
DAY TO HOUR
DAY TO MINUTE
DAY TO SECOND
HOUR TO MINUTE
HOUR TO SECOND
MINUTE TO SECOND
```
* 注意，在指定了精度之后，必须将时间信息指定到秒，因为时间精度是在秒基础上做的精度

```
time =>  时分秒
date => 年月日
timestamp => 年月日 时分秒

比较简单的时区表示方法 +、-数字
2019=07-26 10:10:00 +8
```
特殊值
符号 | 可用于类型 | 含义
--- | --- | ---
epoch | timestamp，date | 1970-01-01 00:00:00 (unix元年)
infinity | date,timestamp | 和正无穷含义相似，表示大于所有时间的时间
- infinity | date,timestamp | 表示小于所有时间的时间
now | date,time,timestamp | 当前时间
today | date,timestamp | 今日午夜
tomorrow | date,timestamp | 明日午夜
yesterday | date,timestamp | 昨日午夜
allballs | time | 00:00:00.00 UTC

- 布尔类型
```
表示true的值有
't' 'true' 'y' 'yes' 'on' '1'
表示false的值有
'f' 'false' 'n' 'no' 'off' '0'
```

- 枚举类型
1. 声明
```
CREATE TYPE 名字 AS ENUM (值...);
```
2. 使用
```
CREATE TABLE A1(
    col1 integer,
    col2 枚举类型名
); 
插入的时候就只能填定义的枚举类型的值了
WHERE的时候就不能只简单的使用=了，因为=不能适用于自定义类型，只有将自定义类型转换成 数据库内部的类型
所以where的是时候可以写成
WHERE a1.col2::text == a2.col2::text
```
通过ALTER TYPE可以再往enum类型增加值，但是不能减，也不能更改里面的顺序，所以要修改的话只有删除，然后重新创建

- 位类型
bit(n)
```
CREATE TABLE test (a BIT(3), b BIT VARYING(5));
INSERT INTO test VALUES (B'101', B'00');
INSERT INTO test VALUES (B'10', B'101');
ERROR: bit string length 2 does not match type bit(3)
INSERT INTO test VALUES (B'10'::bit(3), B'101');
SELECT * FROM test;
a | b
-----+-----
101 | 00
100 | 101
```

- 全文搜索类型
    - tsvector   分词并排序类型
    - tsquery   

- UUID
    
- XML
    - 如果需要xml类型的支持，需要在编译是指定--with-libxml 参数，否则默认不支持

- JSON
    - 分为json和jsonb类型，两种使用上没有什么区别，只是效率上的。json只是将输入作为字符串存在数据库，而jsonb则将输入解析完然后以二进制存储，所以在查询的时候，就不会再次解析。
    - jsonb上可以对其中的字段做索引

- 数组
    - 创建数组类型
```
CREATE TABLE a(
    x text[][], -- 二维
    y integer[] -- 一维
);
```
    - 输入
```
'{{"xxx","xxxx"}}','{1,2,3}'
```
    - 访问 同程序语言一样通过下标访问，下标从0开始。同时还可以像程序语言一样做切片
    
- 复合类型【用起来很复杂】
    - 创建。 和程序中结构体的定义差不多
```sql
CREATE TYPE 类型名 AS (
    字段名 类型,
    ....
);
```
```
-- 创建类型
CREATE TYPE inventory_item AS (
    name text,
    supplier_id integer,
    price numeric
);
-- 创建表
CREATE TABLE on_hand (
    item inventory_item,
    count integer
);
-- 插入数据
INSERT INTO on_hand VALUES (ROW('fuzzy dice', 42, 1.99), 1000);

```
- Range类型
postgres内建了一些range类型 所有的都是左闭右开区间
    - int4range (start,end,['区间类型'])     int4区间
    - int8range (start,end,['区间类型'])     int8 区间
    - numrange (start,end,['区间类型'])      数字区间
    - tsrange (start,end,['区间类型'])       时间戳区间
    - tstzrange (start,end,['区间类型'])     带time zone的时间戳区间
    - daterange (start,end,['区间类型'])     日期区间
区间类型：
    - (] [] [) () 对应数学上的区间符号


### function和操作符

#### 操作符
    - AND OR NOT
    - >,<, >=, <=, =, <>, !=
    - BETWEEN, NOT BETWEEN, IS NULL, IS NOT NULL, IS TRUE, IS NOT TRUE, IS UNKNOWN, IS NOT UNKNOWN
    - + - * / % ^(次方) |/(平方根) ||/(立方根) !(阶乘) @(绝对值) & | # ~  << >>(位操作) 

#### function
- abs(x) 绝对值
- cbrt(x) 立方根
- ceil(x) 取下整
- degrees(x) 弧度转度数
- div(y,x)  除
- exp(x)  e^x
- floor(x)  取上整
- ln(x)     对数log(x,e)
- log(x)    log(x,10)
- log(x,y)
- mod(y,x)  取模
- pi()      圆周率
- power(x,y)    x^y
- radians(x)    度数转弧度
- round(x,精度)      四舍五入
- trunc(x)          截去小数
- trunc(x,精度)        保留多少位小数
- random()      返回0.0-1.0的随机数
- setseed(x)    设置随机种子
- 三角函数...

#### 字符串函数

function | 返回值 | 说明
--- | --- | ---
s1 || s2 | text | 字符串连接
s1||非字符串 | text | 将其他类型转成字符串后再拼接
bit_length(s) | int | 字符串位长度
char_length(s) | int | 字符数
lower(s) | text | 转小写
upper(s) | text | 大写
position(sub in str) | int | sub在str中的位置
substring(str from x for y) | text | 截取[x,y)的字符串
substring(str form 模式 for 转义符号) | text | 支持模式的截取
trim(方向,'字符集合' from str) | text | 移除str两端存在 字符集合中的字符
ascii(s) | int | 字符转ascii
btrim(s,sub) | text | 移除s两端 sub子串
chr(int) | text | ascii 转 字符
concat(s,s1,s2...) | text | 字符串拼接
concat_ws(分隔符,s1,s2...) | text | 字符串之间的拼接连接符
format(fmt,....) |text | 字符串格式化，和c中的printf函数参数一样

- 模式匹配
    - LIKE _单个字符 %任意多个字符
    - 正则 字段 SMALLAR TO '正则'  这里的正则支持得还是比较全面的


#### 格式化函数
- to_char(timestamp,fmt)
- to_char(interval,fmt)
- to_char(int,fmt)
- to_date(text,)
- 
【后面补上】


#### 时间函数

#### 枚举函数

#### 网络类型函数

cidr，inet操作
操作 | 描述 
--- | ---
< <= > >= <> | 和普通意义一样
 << | 判断左某个地址是否属于右某个cidr块
 <<= | 同上，多一项可等
 >> | 判断右某个地址是否属于左某个cidr块
 && | 将<< >> 结合起来，不分左右
~ | 按位取反
& | 位与
| | 位或
+ - | 地址加减


#### 文本搜索类型
操作 | 描述 | 示例
--- | --- | ---
@@ | tsvector是否匹配tsquery | to_tsvector('fat catsa ate rats') @@ to_tsquery('cat & rat')  => t
@@@ | 
\|\| |连接多个tsvector| vector1 \|\| vector2
&& | AND连接多个tsquery | tsquery1 && tsquery2
\|\| | OR连接多个tsquery | tsquery1 || tsquery2
!! | NOT tsquery | !!tsquery
@> | tsquery包含 | tsquery1 @> tsquery2 
@< | tsquery被包含 | 



#### JSON

操作 | 说明
--- | ---
->  | 如果右边是数字，代表取数组的某个下标的值，如果是字符串，则代表取某个对象的成员
->> | 和->操作一样，只不过是返回值不再是json表示的类型，而是直接将取得的值转成text类型
#> | 直接获取json对象的深层成员，不需要多次调用->取 。'{"a": {"b":{"c":"foo"}}}'::json#>'{a,b}'  => {"c":"foo"}
#>> | 将#>获取的结果转换为text

jsonb操作符
- | -
--- | ---
@> | 左对象在顶层是否包含右对象的键和值
<@ | 左对象在顶层是否 被 包含右对象的键和值
? | 对象顶层是否包含某个键
?\| | 顶层对象是否包含某些键中的部分键
?& | 顶层对象是否包含所有指定的键
\|\| | 将两个jsonb连接成一个
- (text,text[],int)| 删除掉对象的键，数组中的第几个元素
#- | 删除指定path（可指定很深层次）的元素


#### 条件表达式

- CASE
```
CASE WHEN 条件 THEN 结果
    [WHEN ...]
    [ELSE 结果]
END
```
- COALESCE(val1,val2,val3...)
从左到右，返回第一个非null的值

- NULLIF(v1,v2)
如果v1==v2则返回null，否则返回v1
- GREATEST(v1....) 、 LEAST(v1...)
GREATEST 返回值中的最大值
LEAST 返回值中的最小值


#### 聚合函数


#### 窗口函数

### 子查询

- EXISTS
```
SELECT col1 FROM tab1 WHERE EXISTS (SELECT 1 FROM tab2 WHERE col2 = tab1.col2);
```
- IN
- NOT IN
- ANY/SOME  任意一个/其中一个
- ALL 所有


### 触发器

```
CREATE TRIGGER 名字
BEFOR UPDATE ON 表名
FOR ECHO ROW EXECUTE PROCEDURE
存储过程();
```


### 类型转换 












## 服务端编程
存储过程可以使用多种语言写，比如c，python等，但是要安装对应的模块才行。

### SQL
```sql
CREATE FUNCTION 名字(integer,text) RETURNS integer 
AS
'函数体 里面的特殊字符需要转义'
LANGUAGE SQL;

CREATE FUNCTION 名字(integer,text) RETURNS integer 
AS $[符号]$
    函数体  特殊字符不需要转义了
$[符号]$
LANGUAGE SQL;
```
- 参数只能用于数据值，不能用作标识符(表名，列名等)
- 参数可命名也可不命名。 使用参数也可以使用 $数字 来使用位置参数，也可以直接使用参数名。 也可以使用 函数名.参数名
- 实现可变参数，将参数前加 VARIADIC  关键字并设置为一个数组就可以了，函数内部按照数组的取值方法取即可
- 参数可以设置默认值  (参数名 类型 DEFAULT 默认值)
- 返回值为一个集合。 SETOF sometype。 返回多行某类型的数据
- 返回值为一个表。TABLE(列...)
- 方法可以重载。 函数名相同，只要参数个数，类型不同就行

```
基本类型
CREATE FUNCTION one() RETURNS integer AS $$
    SELECT 1 AS result;
$$ LANGUAGE SQL;

命名参数
CREATE FUNCTION add_em(x integer, y integer) RETURNS integer AS $$
    SELECT x + y;
$$ LANGUAGE SQL;
SELECT add_em(1, 2) AS answer;
位置参数
CREATE FUNCTION add_em(integer, integer) RETURNS integer AS $$
    SELECT $1 + $2;
$$ LANGUAGE SQL;
SELECT add_em(1, 2) AS answer;

使用函数名.参数名
CREATE FUNCTION tf1 (accountno integer, debit numeric) RETURNS
numeric AS $$
    UPDATE bank
    SET balance = balance - debit
    WHERE accountno = tf1.accountno
    RETURNING balance;
$$ LANGUAGE SQL;

复合类型
CREATE TABLE emp (
    name text,
    salary numeric,
    age integer,
    cubicle point
);
CREATE FUNCTION double_salary(emp) RETURNS numeric AS $$
    SELECT $1.salary * 2 AS salary;
$$ LANGUAGE SQL;
SELECT name, double_salary(emp.*) AS dream FROM emp WHERE emp.cubicle ~= point '(2,1)';

出参定义  (IN,OUT,INOUT)
CREATE FUNCTION add_em (IN x int, IN y int, OUT sum int)
    AS 'SELECT x + y'
LANGUAGE SQL;
多出参
CREATE FUNCTION sum_n_product (x int, y int, OUT sum int, OUT product int)
    AS 'SELECT x + y, x * y'
LANGUAGE SQL;

可变长参数
CREATE FUNCTION mleast(VARIADIC arr numeric[]) RETURNS numeric AS $$
    SELECT min($1[i]) FROM generate_subscripts($1, 1) g(i);
$$ LANGUAGE SQL;
SELECT mleast(10, -1, 5, 4.4);

参数带默认值
CREATE FUNCTION foo(a int, b int DEFAULT 2, c int DEFAULT 3)
RETURNS int
LANGUAGE SQL
AS $$
    SELECT $1 + $2 + $3;
$$;
SELECT foo(10, 20, 30);
SELECT foo(10, 20);
SELECT foo(10);

返回集合
CREATE TABLE foo (fooid int, foosubid int, fooname text);
CREATE FUNCTION getfoo(int) RETURNS SETOF foo AS $$
    SELECT * FROM foo WHERE fooid = $1;
$$ LANGUAGE SQL;
SELECT * FROM getfoo(1) AS t1;

返回表 这种情况下不允许指定OUT，INOUT参数
CREATE FUNCTION sum_n_product_with_tab (x int)
RETURNS TABLE(sum int, product int) AS $$
    SELECT $1 + tab.y, $1 * tab.y FROM tab;
$$ LANGUAGE SQL;


函数重载 参数格式，类型不同
CREATE FUNCTION test(int, real) RETURNS ...
CREATE FUNCTION test(smallint, double precision) RETURNS ...




```



### 自定义聚合函数

```
CREATE AGGREGATE sum (complex)
(
    sfunc = complex_add, -- 函数
    stype = complex,    -- 参数类型
    initcond = '(0,0)'  -- 初始值
);

```
### 操作符重载
```
CREATE OPERATOR + (
    leftarg = complex,  -- 左参
    rightarg = complex, -- 右参
    procedure = complex_add, -- 函数
    commutator = +
);

```

### 触发器





### 存储过程
#### PL/pgSQL

```
CREATE FUNCTION 名字(integer,text) RETURNS integer 
AS
'函数体 里面的特殊字符需要转义'
LANGUAGE plpgsql;

CREATE FUNCTION 名字(integer,text) RETURNS integer 
AS $[符号]$
    函数体  特殊字符不需要转义了
$[符号]$
LANGUAGE plpgsql;
```
函数体必须是一个block。而一个block结构如下
```
[<<label>>]
[DECLARE  declarations]
BEGIN
    statements;
END [label];
```
样例
```
CREATE FUNCTION somefunc() RETURNS integer AS $$
<< outerblock >>
DECLARE
    quantity integer := 30;
BEGIN
    RAISE NOTICE 'Quantity here is %', quantity; -- Prints 30
    quantity := 50;
--
-- Create a subblock
--
    DECLARE
        quantity integer := 80;
    BEGIN
        RAISE NOTICE 'Quantity here is %', quantity; -- Prints 80
        RAISE NOTICE 'Outer quantity here is %', outerblock.quantity; -- Prints 50
    END;
    RAISE NOTICE 'Quantity here is %', quantity; -- Prints 50
    RETURN quantity;
END;
```
##### 变量声明块
```
DECLARE
    name [ CONSTANT ] type [ COLLATE collation_name ] [ NOT NULL ] [ { DEFAULT | := | = } expression ];
```
参数别名
```
1.
CREATE FUNCTION sales_tax(subtotal real) RETURNS real AS $$
BEGIN
    RETURN subtotal * 0.06;
END;
$$ LANGUAGE plpgsql;

2.
CREATE FUNCTION sales_tax(real) RETURNS real AS $$
DECLARE
    subtotal ALIAS FOR $1;
BEGIN
    RETURN subtotal * 0.06;
END;
$$ LANGUAGE plpgsql;
```
从变量会表列名复制其类型。 这很方便的声明某变量类型去接收其值。
```
name variable%TYPE
```
行类型
```
name table_name%ROWTYPE;
```

##### 表达式
- 赋值
```
variable { := | = } expression;
```
- 无结果集的命令
```
PERFORM query;
```
- 动态命令
```
EXECUTE command-string [ INTO [STRICT] target ] [ USING expression [, ... ] ];
```
- 空表达式
```
BEGIN
    y := x / 0;
    EXCEPTION
    WHEN division_by_zero THEN
        NULL; -- ignore the error
    END;

BEGIN
    y := x / 0;
    EXCEPTION
    WHEN division_by_zero THEN
        -- ignore the error
END;

```


##### 流程控制

- return
```
RETURN 表达式
RETURN NEXT expression;
RETURN QUERY query;
RETURN QUERY EXECUTE command-string [ USING expression [, ... ] ];
```
- if
```
IF ... THEN ... END IF
IF ... THEN ... ELSE ... END IF
IF ... THEN ... ELSIF ... THEN ... ELSE ... END IF
```
- case
```
CASE search-expression
    WHEN expression [, expression [ ... ]] THEN
        statements
    [ WHEN expression [, expression [ ... ]] THEN
        statements... ]
    [ ELSE
        statements ]
END CASE;
```
- loop
```
[ <<label>> ]
LOOP
    statements
END LOOP [ label ];

退出循环
EXIT [ label ] [ WHEN boolean-expression ];

继续循环
CONTINUE [ label ] [ WHEN boolean-expression ];


示例：
LOOP
-- some computations
    IF count > 0 THEN
        EXIT; -- exit loop
    END IF;
END LOOP;

LOOP
    -- some computations
    EXIT WHEN count > 0; -- same result as previous example
END LOOP;
```

- while
```
[ <<label>> ]
WHILE boolean-expression LOOP
    statements
END LOOP [ label ];
```
- for
```
[ <<label>> ]
FOR name IN [ REVERSE ] expression .. expression [ BY expression ]
LOOP
    statements
END LOOP [ label ];

[ <<label>> ]
FOR target IN query LOOP
    statements
END LOOP [ label ];


Example:
FOR i IN 1..10 LOOP
-- i will take on the values 1,2,3,4,5,6,7,8,9,10 within the
loop
END LOOP;

```

- 遍历数组
```
[ <<label>> ]
FOREACH target [ SLICE number ] IN ARRAY expression LOOP
    statements
END LOOP [ label ];

```

- 错误处理
```
BEGIN
    UPDATE mytab SET firstname = 'Joe' WHERE lastname = 'Jones';
    x := x + 1;
    y := x / 0;
    EXCEPTION
        WHEN division_by_zero THEN
        RAISE NOTICE 'caught division_by_zero';
    RETURN x;
END;
```

##### 游标

1. 声明
2. 打开
3. 使用
4. 关闭
```
1. 声明
name [ [ NO ] SCROLL ] CURSOR [ ( arguments ) ] FOR query;
2.打开
OPEN unbound_cursorvar [ [ NO ] SCROLL ] FOR query;
OPEN unbound_cursorvar [ [ NO ] SCROLL ] FOR EXECUTE query_string [ USING expression [, ... ] ];
OPEN bound_cursorvar [ ( [ argument_name := ] argument_value [, ...] ) ];
3. 使用
FETCH [ direction { FROM | IN } ] cursor INTO target;
MOVE [ direction { FROM | IN } ] cursor;
UPDATE table SET ... WHERE CURRENT OF cursor;
DELETE FROM table WHERE CURRENT OF cursor;

4. 关闭
CLOSE curs1;



Example:

DECLARE
    curs1 refcursor;
    curs2 CURSOR FOR SELECT * FROM tenk1;
    curs3 CURSOR (key integer) FOR SELECT * FROM tenk1 WHERE
unique1 = key;

OPEN curs1 FOR SELECT * FROM foo WHERE key = mykey;
OPEN curs1 FOR EXECUTE format('SELECT * FROM %I WHERE col1 = $1',tabname) USING keyvalue;
OPEN curs2;
OPEN curs3(42);
OPEN curs3(key := 42);



FETCH curs1 INTO rowvar;
FETCH curs2 INTO foo, bar, baz;
FETCH LAST FROM curs3 INTO x, y;
FETCH RELATIVE -2 FROM curs4 INTO x;
MOVE curs1;
MOVE LAST FROM curs3;
MOVE RELATIVE -2 FROM curs4;
MOVE FORWARD 2 FROM curs4;
UPDATE foo SET dataval = myval WHERE CURRENT OF curs1;

[ <<label>> ]
FOR recordvar IN bound_cursorvar [ ( [ argument_name:= ] argument_value [, ...] ) ] LOOP
    statements
END LOOP [ label ];


```

- 返回游标   refcursor 表示游标类型
```
CREATE FUNCTION reffunc(refcursor) RETURNS refcursor AS '
BEGIN
    OPEN $1 FOR SELECT col FROM test;
    RETURN $1;
END;
' LANGUAGE plpgsql;

BEGIN;
    SELECT reffunc('funccursor');
    FETCH ALL IN funccursor;
COMMIT;

```

- 错误信息
```
RAISE [ level ] 'format' [, expression [, ... ]] [ USING option = expression [, ... ] ];
RAISE [ level ] condition_name [ USING option = expression [, ... ] ];
RAISE [ level ] SQLSTATE 'sqlstate' [ USING option = expression [, ... ] ];
RAISE [ level ] USING option = expression [, ... ];
RAISE ;

```





