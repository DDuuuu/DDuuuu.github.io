# DotNet基础 Sql Ado.Net




# 数据库

## 数据库概述

&#43; DBMS(DataBase Management System,数据库管理系统)和数据库。平时谈到“数据库”可能有两种含义：MSSQLServer、Oracle等某种DBMS；存放一堆数据表的一个分类(Catalog)
&#43; 数据库的构成-管理软件/服务/数据文件(表,视图...)
&#43; 不同品牌的DBMS有自己的不同的特点：MYSQL、MSSQLServer、DB2、Oracle、Access、Sybase等。对于开发人员来讲，大同小异
&#43; 除了Access、SQLServerCE等文件型数据库之外，大部分数据库都需要数据库服务器才能运行。学习\开发时是连接本机的数据库，上线运行时是数据库运行在单独的服务器
&#43; 为什么要用数据库：我们平时把数据以文件的方式存放在硬盘里，但当数据量庞大的时候：文件大，操作效率很低下。所以，便有了很多种数据库软件(Mssql,Ora,DB2…)，它们代替我们做数据文件的操作(mdf,ndf,ldf)并提供高效的存储和检索等操作。还提供了很多接口给其他程序语言调用。

## 为什么要使用数据库

用文件保存数据与用数据库的优劣：

&#43; 高效维护大量数据-检索/增/删/改
&#43; 处理各个表之间的关系
&#43; 压缩表数据
&#43; 安全

## 数据库的概念

&#43; Catalog(分类)(又叫数据库DataBase,表空间TableSpace),不同类的数据应该放到不同的数据库中
  &#43; 便于对各个Catalog进行个性化管理 
  &#43; 避免命名冲突3安全性更高
&#43; Table(表):书都放到书架上,碗都放到橱柜中,不同类型的资料放到不同的“格子”中，将这种区域叫做“表”(Table)。不同的表根据放的数据不同进行空间的优化，找起来也方便。
&#43; 列(Column)、字段(Field)
&#43; 主键(Primary Key)：主键就是一个表中每个数据行的唯一标识。不会有重复值的列才能当主键。一个表可以没有主键，但是会非常难以处理，因此没有特殊理由表都要设定主键
&#43; 主键有两种选用策略：业务主键和逻辑主键。业务主键是使用有业务意义的字段做主键，比如身份证号、银行账号等；逻辑主键是使用没有任何业务意义的字段做主键，完全给程序看的，业务人员不会看的数据。因为很难保证业务主键不会重复（身份证号重复）、不会变化（帐号升位），因此推荐用逻辑主键。
&#43; 外键(Foreign Key)—记录表与表的关联

## SQLSERVER管理

&#43; 需要安装SQLServer2005或者SQLServer2008，若要使用SQLServer管理工具进行开发还要安装SQL Server Management Studio，还可以使用VisualStudio进行管理
&#43; 使用免费的SQL Server Express版本，Express版本的服务器名称. \SQLEXPRESS，对于开发人员来讲和其他版本没有区别。
  SQLServer的两种验证方式：用户名验证和Windows验证，开发时用Windows验证就行。
&#43; 开发人员关注点在开发上,而不是配置/备份等之上,那是DBA做的事情。
&#43; 创建数据库，创建表，设置主键
&#43; SQLServer2008中：编辑200行；SQLServer2005中：打开表。
  常用字段类型：bit(可选值0、1)、datetime、int、varchar、nvarchar（可能含有中文用nvarchar）
  Nvarchar(50)、Nvarchar(MAX)
&#43; varchar、nvarchar 和char(n)的区别： char(n)不足长度n的部分用空格填充。Var：Variable，可变的。

![](/dotnet/qLostNM.png)



## SQL语句入门

&#43; SQL语句是和DBMS“交谈”专用的语句，不同DBMS都认SQL语法
&#43; SQL语句中字符串用单引号。
&#43; SQL语句是大小写不敏感的,不敏感指的是SQL关键字,字符串值还是大小写敏感的
&#43; 创建表、删除表不仅可以手工完成,还可以执行SQL语句完成,在自动化部署、数据导入中用的很多,`CREATE TABLE T_Person(Id int NOT NULL,Name nvarchar(50),Age int NULL)`、Drop table T_Person1
&#43; 简单的Insert语句。`INSERT INTO T_Person(Id,Name,Age) VALUES(1,&#39;Jim&#39;,20)`
&#43; （*） SQL主要分DDL（数据定义语言）和DML（数据操作语言）两类。Create Table、Drop Table、Alter Table等属于DDL，Select、Insert、Update、Delete等属于DML

## 主键的选择

&#43; SQLServer中两种常用的主键数据类型：int（或bigint）&#43;标识列(又称自动增长字段);uniqueidentifier(又称Guid、UUID)
  用标识列实现字段自增可以避免并发等问题，不要开发人员控制自增。用标识列的字段在Insert的时候不用指定主键的值。将字段的“是标识列”设置为“是”，一个表只能有一个标识列。
&#43; Guid算法是一种可以产生唯一标识的高效算法，它使用网卡MAC、地址、纳秒级时间、芯片ID码等算出来的，这样保证每次生成的GUID永远不会重复，无论是同一个计算机上还是不同的计算机。在公元3400年以前产生的GUID与任何其他产生过的GUID都不相同。SQLServer中生成GUID的函数newid()，.Net中生成Guid的方法：Guid.NewGuid()，返回是Guid类型。
&#43; （*）Int自增字段的优点：占用空间小、无需开发人员干预、易读；缺点：效率低；数据导入导出的时候很痛苦。
&#43; （*）Guid的优点：效率高、数据导入导出方便；缺点占用空间大、不易读。
&#43; 业界主流倾向于使用Guid。

&#43; Globally Unique Identifier(全球唯一标识符),也称作 UUID(Universally Unique IDentifier) GUID：用于指示产品的唯一性安装。是通过特定算法产生的一个二进制长度为128位的数字。在空间上和时间上具有唯一性，保证同一时间不同地方产生的数字不同。在公元3400年以前产生的UUID/GUID与任何其他产生过的UUIDs/GUIDs都不相同
  GUID的长度固定，并且相对而言较短小，非常适合于排序、标识和存储。

## 新增

&#43; 新增表和参数

```sql
CREATE TABLE T_Employee (FNumber VARCHAR(20),FName VARCHAR(20),FAge INT,FSalary NUMERIC(10,2),PRIMARY KEY (FNumber));
INSERT INTO T_Employee(FNumber,FName,FAge,FSalary) VALUES(&#39;DEV001&#39;,&#39;Tom&#39;,25,8300);
INSERT INTO T_Employee(FNumber,FName,FAge,FSalary) VALUES(&#39;DEV002&#39;,&#39;Jerry&#39;,28,2300.80);
INSERT INTO T_Employee(FNumber,FName,FAge,FSalary) VALUES(&#39;SALES001&#39;,&#39;John&#39;,23,5000);
INSERT INTO T_Employee(FNumber,FName,FAge,FSalary) VALUES(&#39;SALES002&#39;,&#39;Kerry&#39;,28,6200);
INSERT INTO T_Employee(FNumber,FName,FAge,FSalary) VALUES(&#39;SALES003&#39;,&#39;Stone&#39;,22,1200);
INSERT INTO T_Employee(FNumber,FName,FAge,FSalary) VALUES(&#39;HR001&#39;,&#39;Jane&#39;,23,2200.88);
INSERT INTO T_Employee(FNumber,FName,FAge,FSalary) VALUES(&#39;HR002&#39;,&#39;Tina&#39;,25,5200.36);
INSERT INTO T_Employee(FNumber,FName,FAge,FSalary) VALUES(&#39;IT001&#39;,&#39;Smith&#39;,28,3900);
INSERT INTO T_Employee(FNumber,FAge,FSalary) VALUES(&#39;IT002&#39;,27,2800);
```


&#43; 插入一列 `ALTER TABLE T_Employee ADD FSubCompany VARCHAR(20);`
  &#43; `UPDATE T_Employee SET FSubCompany=&#39;Beijing&#39;,FDepartment=&#39;Development&#39; 
    WHERE FNumber=&#39;DEV001&#39;;`

&#43; Insert语句可以省略表名后的列名，但是不推荐
&#43; 如果插入的行中有些字段的值不确定，那么Insert的时候不指定那些列即可。
&#43; 可以给字段默认值，如果Guid类型主键的默认值设定为newid()就会自动生成，很少这么干
  主键：

```sql
insert into Person3(Name,Age) values(&#39;lily&#39;,38);
insert into Person4(Id,Name,Age)   
values(newid(),&#39;tom&#39;,30);
```

## 更新

&#43; 更新一个列:UPDATE T_Person Set Age=30
&#43; 更新多个列:UPDATE T_Person Set Age=30,Name=‘tom’
&#43; 更新一部分数据： UPDATE T_Person Set Age=30 where Name=‘tom’，用where语句表示只更新Name是’tom’的行，注意SQL中等于判断用单个=，而不是==
&#43; Where中还可以使用复杂的逻辑判断U`PDATE T_Person Set Age=30 where Name=‘tom’ or Age&lt;25`，or相当于C#中的||（或者）
&#43; `update Person1 set NickName=N&#39;二十岁&#39; 
  where (Age&gt;20 and Age&lt;30) or(Age=80)`
&#43; Where中可以使用的其他逻辑运算符：or、and、not、&lt;、&gt;、&gt;=、&lt;=、!=（或&lt;&gt;）等

## 删除

&#43; 删除表中全部数据：DELETE FROM T_Person。
&#43; Delete只是删除数据，表还在，和Drop Table不同。
&#43; Delete 也可以带where子句来删除一部分数据：`DELETE FROM T_Person WHERE FAge &gt; 20`
&#43; Truncate  删除所有行并重置标识

## 检索

&#43; 执行备注中的代码创建测试数据表。
&#43; 简单的数据检索 ：`SELECT * FROM T_Employee`
&#43; 只检索需要的列 ：`SELECT FNumber FROM T_Employee` 、`SELECT FName,FAge FROM T_Employee`
&#43; 列别名：SELECT FNumber AS 编号,FName AS 姓名,FAge AS Age111 FROM T_Employee 
&#43; 使用where检索符合条件的数据：`SELECT FName FROM T_Employee WHERE FSalary&lt;5000`。故事：新员工的数据检索噩梦。
&#43; 还可以检索不与任何表关联的数据：select 1&#43;1;select newid();select getdate();

## 数据汇总

&#43; SQL聚合函数：MAX（最大值）、MIN（最小值）、AVG （平均值）、SUM （和）、COUNT（数量）
&#43; 大于25岁的员工的最高工资 ：
  `SELECT MAX(FSalary) FROM T_Employee WHERE FAge&gt;25` 
&#43; 最低工资和最高工资：
  `SELECT MIN(FSalary),MAX(FSalary) FROM  T_Employee `

## 排序

&#43; ORDER BY子句位于SELECT语句的末尾，它允许指定按照一个列或者多个列进行排序，还可以指定排序方式是升序（从小到大排列，ASC）还是降序（从大到小排列，DESC）。 
&#43; 按照年龄升序排序所有员工信息的列表：
  `SELECT * FROM  T_Employee ORDER BY FAge ASC `
&#43; 按照年龄从大到小排序，如果年龄相同则按照工资从大到小排序 ：`SELECT * FROM  T_Employee ORDER BY FAge DESC,FSalary DESC`（多个排序条件）
  ORDER BY子句要放到WHERE子句之后 ：`SELECT * FROM T_Employee WHERE FAge&gt;23 ORDER BY FAge DESC,FSalary DESC `

## 通配过滤符

&#43; 通配符过滤关键字使用LIKE 。
&#43; 单字符匹配的通配符为半角下划线“_”，它匹配单个出现的字符。
  &#43; eg:以任意字符开头，剩余部分为“erry”
  &#43; `SELECT * FROM T_Employee WHERE FName LIKE &#39;_erry&#39; `
&#43; 多字符匹配的通配符为半角百分号“%”，它匹配任意次数（零或多个）出现的任意字符。 “k%”匹配以“k”开头、任意长度的字符串
  &#43; eg:检索姓名中包含字母“n”的员工信息 
  &#43; `SELECT * FROM T_Employee WHERE FName LIKE &#39;%n%&#39; `

![](/dotnet/tae76ts.png)

## 空值处理

&#43; 数据库中，一个列如果没有指定值，那么值就为null，这个null和C#中的null，数据库中的null表示“不知道”，而不是表示没有。
&#43; 因此select null&#43;1结果是null，因为“不知道”加1的结果还是“不知道”。
  &#43; `SELECT * FROM T_Employee WHERE FNAME=null` ； `SELECT * FROM T_Employee WHERE FNAME!=null `；
    都没有任何返回结果，因为数据库也“不知道”。
&#43; SQL中使用is null、is not null来进行空值判断： 
  &#43; `SELECT * FROM T_Employee WHERE FNAME is null` ； `SELECT * FROM T_Employee WHERE FNAME is not null` ；

## 多值匹配

删除多条数据

`Delete T_Employee where FId in (21,22)`

`SELECT FAge,FNumber,FName FROM T_Employee WHERE FAge IN (23,25,28)` 

范围值：

&#43; SELECT * FROM T_Employee WHERE FAGE&gt;=23 AND FAGE &lt;=27 
&#43; SELECT * FROM T_Employee WHERE FAGE BETWEEN 23 AND 27 

## 数据分组

&#43; 按照年龄进行分组统计各个年龄段的人数：`SELECT FAge,Count(*) FROM T_Employee GROUP BY Fage`
&#43; GROUP BY子句必须放到WHERE语句的之后 
  没有出现在GROUP BY子句中的列是不能放到SELECT语句后的列名列表中的 (**聚合函数**中除外)
  错误：
  ` SELECT FAge,FSalary FROM T_Employee GROUP BY FAge `
  正确：
  ` SELECT FAge,AVG(FSalary) FROM T_Employee GROUP BY FAge`

### 对分组进行过滤

&#43; **在Where中不能使用聚合函数，必须使用Having，Having要位于Group By之后**：

```sql
SELECT FAge,COUNT(*) AS 人数 FROM T_Employee 
GROUP BY FAge 
HAVING COUNT(*)&gt;1 
```

在分组的时候对分组后的成员进行过滤

**注意Having中不能使用未参与分组的列，Having不能替代where。作用不一样，Having是对组进行过滤。having和where不会同时出现**

![](/dotnet/YNaOMzS.png)

## 限制结果集行数

&#43; `SELECT top 5 * FROM T_Employee order by FSalary Desc` 
&#43; （*）检索按照工资从高到低排序检索从第六名开始一共三个人的信息 ：

```sql
SELECT top 3 * FROM T_Employee 
WHERE FNumber NOT IN (SELECT TOP 5 FNumber FROM T_Employee ORDER BY FSalary DESC)  
ORDER BY FSalary DESC 
```

该语句用来分页语句。

## 去掉重复数据

&#43; 执行备注中的SQL语句，Alter和Insert单独执行

```sql
SELECT FDepartment FROM T_Employee
SELECT DISTINCT FDepartment FROM T_Employee 
```

&#43; DISTINCT是**对整个结果集进行数据重复处理的**，而不是针对每一个列，因此下面的语句并不会只保留Fdepartment进行重复值处理：`SELECT DISTINCT FDepartment,FSubCompany FROM T_Employee `

## 联合结果集

```sql
CREATE TABLE T_TempEmployee (FIdCardNumber VARCHAR(20),FName VARCHAR(20),FAge INT, PRIMARY KEY (FIdCardNumber));
INSERT INTO T_TempEmployee(FIdCardNumber,FName,FAge) VALUES(&#39;1234567890121&#39;,&#39;Sarani&#39;,33);
INSERT INTO T_TempEmployee(FIdCardNumber,FName,FAge) VALUES(&#39;1234567890122&#39;,&#39;Tom&#39;,26);
INSERT INTO T_TempEmployee(FIdCardNumber,FName,FAge) VALUES(&#39;1234567890123&#39;,&#39;Yalaha&#39;,38);
INSERT INTO T_TempEmployee(FIdCardNumber,FName,FAge) VALUES(&#39;1234567890124&#39;,&#39;Tina&#39;,26);
INSERT INTO T_TempEmployee(FIdCardNumber,FName,FAge) VALUES(&#39;1234567890125&#39;,&#39;Konkaya&#39;,29);
INSERT INTO T_TempEmployee(FIdCardNumber,FName,FAge) VALUES(&#39;1234567890126&#39;,&#39;Fotifa&#39;,46);
```

&#43; 简单的结果集联合：
  `SELECT FNumber,FName,FAge FROM T_Employee  UNION  SELECT FIdCardNumber,FName,FAge FROM T_TempEmployee `
&#43; 基本的原则：每个结果集必须有**相同的列数**；每个结果集的列必须类型相容。 `SELECT FNumber,FName,FAge,FDepartment FROM T_Employee  UNION  SELECT FIdCardNumber,FName,FAge,‘临时工，无部门&#39; FROM T_TempEmployee` 

### union all

&#43; `SELECT FName FROM T_Employee UNION  SELECT FName FROM T_TempEmployee`
  UNION 合并两个查询结果集，并且将其中完全重复的数据行合并为一条

&#43; `SELECT FName FROM T_Employee UNION ALL SELECT FName FROM T_TempEmployee `Union因为要进行重复值扫描，所以效率低，因此如果确定**不要合并重复行**，那么就用UNION ALL

## 数字函数

&#43; ABS() ：求绝对值。
&#43; CEILING()：舍入到最大整数 。3.33将被舍入为4、2.89将被舍入为3、-3.61将被舍入为-3。 Ceiling→天花板
&#43; FLOOR()：舍入到最小整数。3.33将被舍入为3、2.89将被舍入为2、-3.61将被舍入为-4。 Floor→地板。
&#43; ROUND()：四舍五入。舍入到“离我半径最近的数” 。Round→“半径”。Round(3.1425,2)。

### string

&#43; LEN() ：计算字符串长度，求字符
&#43; Datalength():求字节
&#43; LOWER() 、UPPER () ：转小写、大写
&#43; LTRIM()：字符串左侧的空格去掉 
&#43; RTRIM () ：字符串右侧的空格去掉 
  &#43; LTRIM(RTRIM(&#39;         bb        &#39;))
&#43; SUBSTRING(string,start_position,length)
  &#43; 参数string为主字符串，start_position为子字符串在主字符串中的起始位置，length为子字符串的最大长度。`SELECT  SUBSTRING(&#39;abcdef111&#39;,2,3)` 

### 日期

![](/dotnet/WsWZEIu.png)

&#43; GETDATE() ：取得当前日期时间 
&#43; DATEADD (datepart , number, date )，计算增加以后的日期。参数date为待计算的日期；参数number为增量；参数datepart为计量单位，可选值见备注。DATEADD(DAY, 3,date)为计算日期date3天后的日期，而DATEADD(MONTH ,-8,date)为计算日期date8个月之前的日期 
&#43; DATEDIFF ( datepart , startdate , enddate ) ：计算两个日期之间的差额。 datepart 为计量单位，可取值参考DateAdd。

统计不同工龄的员工的个数：

`select DateDiff(year,FInDate,getdate()),count(*) from T_Employee group by DateDiff(year,FInDate,getdate())`

&#43; DATEPART (datepart,date)：返回一个日期的特定部分 
  统计员工的入职年份个数：

`select DatePart(year,FInDate),count(*) from T_Employee group by DatePart(year,FInDate)`

## 类型转换函数

&#43; CAST ( expression AS data_type)
&#43; CONVERT ( data_type, expression) 

```sql
SELECT FIdNumber,
 RIGHT(FIdNumber,3) as 后三位,
 CAST(RIGHT(FIdNumber,3) AS INTEGER) as 后三位的整数形式,
 CAST(RIGHT(FIdNumber,3) AS INTEGER)&#43;1 as 后三位加1,
 CONVERT(INTEGER,RIGHT(FIdNumber,3))/2 as 后三位除以2
FROM T_Person
```



## 空值处理函数

&#43; ISNULL(expression,value) ：
  如果expression不为空则返回expression，否则返回value
  `SELECT ISNULL(FName,&#39;佚名&#39;) as 姓名 FROM
  T_Employee `

## case

&#43; 单值判断，相当于switch case

```sql
CASE expression 
WHEN value1 THEN returnvalue1 
WHEN value2 THEN returnvalue2 
WHEN value3 THEN returnvalue3 
ELSE defaultreturnvalue 
END 
```

例子SELECT  

```sql
     SELECT  FName, 
     (CASE FLevel   WHEN 1 THEN &#39;VIP客户&#39;  
     WHEN 2 THEN &#39;高级客户&#39;  
     WHEN 3 THEN &#39;普通客户&#39; 
     ELSE &#39;客户类型错误&#39;  
     END) as FLevelName 
     FROM T_Customer

     select FName,
     (
     case
     when FSalary&lt;2000 then &#39;低收入&#39;
     when FSalary&gt;=2000 and FSalary&lt;=5000 then &#39;中等收入&#39;
     else &#39;高收入&#39;
     end
     ) as 收入水平
     from T_Employee

     SELECT  FName, FWeight, 
     (CASE 
     WHEN FWeight&lt;40 THEN &#39;瘦瘦&#39;  
     WHEN FWeight&gt;50 THEN &#39;肥肥&#39;  
     ELSE &#39;ok&#39;  
     END) as isnormal 
     FROM T_Person
```



## 索引index

&#43; 分为聚集索引和非聚集索引，
  &#43; 聚集索：聚集索引就相当于使用字典的拼音查找，因为聚集索引存储记录是物理上连续，即拼音a过后一定是b.
  &#43; 非聚集索引，就相当于使用字典的首部查找，逻辑连续，物理不连续。
&#43; 全表扫描：对数据进行检索（select）效率最差的是全表扫描，就是一条条的找。
&#43; 如果没有目录，查汉语字典就要一页页的翻，而有了目录只要查询目录即可。为了提高检索的速度，可以为经常进行检索的列添加索引，相当于创建目录。
&#43; 创建索引的方式，在表设计器中点击右键，**选择“索引/键”→添加→在列中选择索引包含的列**。不能为空
  使用索引能提高查询效率，但是索引也是占据空间的，而且添加、更新、删除数据的时候也需要同步更新索引，因此会降低Insert、Update、Delete的速度。只在经常检索的字段上(Where)创建索引。
&#43; （*）即使创建了索引，仍然有可能全表扫描，比如like、函数、类型转换等。


## 表连接

&#43; 有客户表（T_Customers）和订单表（T_Orders）两个表，客户表字段为：Id、Name、Age，订单表字段为：Id、BillNo、CustomerId，订单表通过CustomerId关联客户表。测试数据见备注。
  `SELECT o.BillNo,c.Name,c.Age from T_Orders as o
  JOIN T_Customers as c on o.CustomerId=c.Id`
&#43; join是和哪个表连接，on后是连接的关系是什么。(多表)
  要求显示所有年龄大于15岁的顾客购买的订单号、客户姓名、客户年龄。
  要求显示年龄大于平均年龄的顾客购买的订单
  （*）Inner Join、Left Join、Right Join


&#43; ：T_Orders![](/dotnet/8Qto2Tb.png)
&#43; ：T_Customers![](/dotnet/dK17gcQ.png)


&#43; `select o.BillNo,c.Name,c.Age
  from T_Orders as o
  join T_Customers as c on o.CustomerId=c.Id`这个语句是以 T_Orders表为准，在其后添加join表的内容。

![](/dotnet/zM7Z3aE.png)

&#43; `select o.BillNo,c.Name
  from T_Orders as o
  join T_Customers as c
  on o.CustomerId=c.Id
  where c.Age&gt;15`

![](/dotnet/rHWhpgi.png)

&#43; `select o.BillNo,c.Name,c.Age
  from T_Orders as o
  join T_Customers as c on o.CustomerId=c.Id
  where c.Age&gt;(select AVG(Age) from T_Customers)`

![](/dotnet/AdT3IBz.png)

## 子查询

&#43; 将一个查询语句做为一个结果集供其他SQL语句使用，就像使用普通的表一样，被当作结果集的查询语句被称为子查询。所有可以使用表的地方几乎都可以使用子查询来代替。`SELECT * FROM(SELECT * FROM T2 where FAge&lt;30) `
&#43; 单值做为子查询：`SELECT 1 AS f1,2,(SELECT MIN(FYearPublished) FROM T_Book),(SELECT MAX(FYearPublished)  FROM T_Book) AS f4`
&#43; 只有返回且仅返回一行、一列数据的子查询才能当成单值子查询。下面的是错误的：`SELECT 1 AS f1,2,(SELECT FYearPublished FROM T_Book)`
  `SELECT * FROM T_ReaderFavorite WHERE FCategoryId=(SELECT FId FROM T_Category WHERE FName=&#39;Story&#39;)`

&#43; 如果子查询是多行单列的子查询，这样的子查询的结果集其实是一个集合。
  `SELECT * FROM T_Reader 
  WHERE FYearOfJoin IN
  (
  select FYearPublished FROM T_Book
  )`
&#43; 限制结果集。返回第3行到第5行的数据（ ROW_NUMBER 不能用在where子句中，所以将带行号的执行结果作为子查询，就可以将结果当成表一样用了）： 

```sql
SELECT * FROM 
( 
SELECT ROW_NUMBER() OVER(ORDER BY FSalary DESC) AS rownum, 
FNumber,FName,FSalary,FAge FROM T_Employee 
) AS a 
WHERE a.rownum&gt;=3 AND a.rownum&lt;=5
```



## 视图概述

&#43; 视图是一张虚拟表，它表示一张表的部分数据或多张表的综合数据，其结构和数据是建立在对表的查询基础上
&#43; 视图在操作上和数据表没有什么区别，但两者的差异是其本质是不同:数据表是实际存储记录的地方，然而视图并不保存任何记录，它存储的实际上是查询语句
&#43; 相同的数据表，根据不同用户的不同需求，可以创建不同的视图（不同的查询语句）
&#43; 优点：
  &#43; 筛选表中的行
  &#43; 防止未经许可的用户访问敏感数据
  &#43; 降低数据库的复杂程度

## 局部变量

&#43; 声明局部变量

```sql
DECLARE @变量名  数据类型
DECLARE @bookName varchar(20)
DECLARE @bId int
```

&#43; 赋值 
  &#43; SET @变量名 =值      --set用于普通的赋值
  &#43; SELECT @变量名 = 值  --用于从表中查询数据并赋值
&#43; 例如：
  &#43; SET @ bookName =‘家宝’
  &#43; SELECT @ bookName=b_title FROM Book WHERE b_id=2

```sql
declare @money money --声明变量
set @money = 2000 –赋值
select @money – 查询变量值
```

## 变量种类

变量分为：

&#43; 局部变量：局部变量必须以标记@作为前缀 ，如@Age int
  &#43; 局部变量：先声明，再赋值 
&#43; 全局变量（系统变量）：
  &#43; 全局变量必须以标记@@作为前缀，如@@version
  &#43; 全局变量由系统定义和维护，我们只能读取，不能修改全局变量的值 

![](/dotnet/h6e0okw.png)

## if else

```sql
     IF(条件表达式)
       BEGIN --相当于C#里的{
       语句1 ……
       END --相当于C#里的}
     ELSE
      BEGIN
     语句1
     ……
       END

     declare @money money
     select @money=AVG(b_money) from Book
     if @money &gt;50
     begin
     	select &#39;A&#39;
     	select top 2 * from Book order by b_money desc
     end
     else
     begin
     	select &#39;B&#39;
     	select top 2 * from Book order by b_money asc
     end
```



## while循环

```sql
WHILE(条件表达式)
  BEGIN --相当于C#里的{
   语句
   ……
   BREAK
  END --相当于C#里的}
  
  declare @a int 
set @a=1
while(@a&lt;50 )
begin 
print str(@a)
set @a=@a&#43;1
end 

DECLARE @num int
WHILE(1=1) --条件永远成立
  BEGIN
   SELECT @num=COUNT(*) FROM T_Book 
    WHERE FYearPublished&lt;2000--统计不达标本数
   IF (@num&gt;0)
     UPDATE T_Book  --每本加2元
     SET FYearPublished=FYearPublished&#43;2 
   ELSE
     BREAK--退出循环(只有一行语句可省begin-end)
   END
SELECT * FROM T_Book
```



## 事务

&#43; 借钱问题：

假定钱从A转到B，至少需要两步：

A的资金减少

然后B的资金相应增加    

```sql
UPDATE bank SET uMoney=uMoney-1000 
 WHERE uName=&#39;家宝‘ 
 @@error
UPDATE bank SET uMoney=uMoney&#43;1000
 WHERE uName=&#39;奥巴马‘
 @@error
```


查看结果： `SELECT * FROM bank`

会出问题：不好回滚

&#43; 指访问并可能更新数据库中各种数据项的一个程序执行单元(unit)--也就是由多个sql语句组成，必须作为一个整体执行
&#43; 这些sql语句作为一个整体一起向系统提交，要么都执行、要么都不执行 
&#43; 语法步骤：
  &#43; 开始事务：BEGIN TRANSACTION
  &#43; 事务提交：COMMIT TRANSACTION
  &#43; 事务回滚：ROLLBACK TRANSACTION
&#43; 判断**某条语句执行是否出错**：
  &#43; 全局变量@@ERROR；
  &#43; @@ERROR只能判断**当前一条T-SQL语句执行是否有错**，为了判断事务中所有T-SQL语句是否有错，我们需要对错误进行累计；
    &#43; 例：SET @errorSum=@errorSum&#43;@@error

```sql
BEGIN TRANSACTION 
/*--定义变量，用于累计事务执行过程中的错误--*/
DECLARE @errorSum INT 
SET @errorSum=0  --初始化为0，即无错误
/*--转账：张三的账户少1000元，李四的账户多1000元*/
UPDATE bank SET currentMoney=currentMoney-1000
   WHERE customerName=&#39;张三&#39;
SET @errorSum=@errorSum&#43;@@error
UPDATE bank SET currentMoney=currentMoney&#43;1000
   WHERE customerName=&#39;李四&#39;
SET @errorSum=@errorSum&#43;@@error  --累计是否有错误
If @errorSum&gt;0
Begin
	rollback transaction
	 select ‘失败’
End
Else
Begin
	commit transaction
	select ‘成功’
End
```



## 存储过程

&#43; 存储过程---就像数据库中运行方法(函数)
&#43; 和C#里的方法一样，由存储过程名/存储过程参数组成/可以有返回结果。
&#43; 前面学的if else/while/变量 等，都可以在存储过程中使用

优点：

&#43; 执行速度更快，在数据库中保存的语句是编译过的
&#43; 允许模块化程序设计 ，方法的复用
&#43; 提高系统安全性，防止SQL注入 
&#43; 减少网络流通量，只要传输存储过程的名称

系统存储过程

&#43; 由系统定义，存放在master数据库中 
&#43; 名称以“sp_”开头或”xp_”开头

自定义存储过程

&#43; 由用户在自己的数据库中创建的存储过程

## 系统存储过程

![](/dotnet/eldeKm1.png)

```sql
EXEC sp_databases
EXEC  sp_renamedb &#39;Northwind&#39;,&#39;Northwind1&#39;
EXEC sp_tables
EXEC sp_columns stuInfo  
EXEC sp_help stuInfo
EXEC sp_helpconstraint stuInfo
EXEC sp_helpindex stuMarks
EXEC sp_helptext &#39;view_stuInfo_stuMarks&#39; 
EXEC sp_stored_procedures
```

  

## 创建存储过程

&#43; 定义存储过程的语法

```sql
CREATE  PROC[EDURE]  存储过程名 
@参数1  数据类型 = 默认值 OUTPUT,
@参数n  数据类型 = 默认值 OUTPUT
AS
  SQL语句
```




&#43; 参数说明：
  &#43; 参数可选
  &#43; 参数分为输入参数、输出参数 
  &#43; 输入参数允许有默认值
&#43; EXEC  过程名  [参数]

&#43; 例子--编写分页存储过程

```sql
create procedure proGetPageData
@pageIndex int,
@pageSize int
as
declare @sqlStr varchar(300)
set @sqlStr=&#39;select top &#39;&#43;str(@pageSize)&#43;&#39; * from Category where c_id not in(select top &#39;&#43;str((@pageIndex-1)*@pageSize)&#43;&#39; c_id from Category order by c_addtime)order by c_addtime&#39;
print @sqlStr
EXEC(@sqlStr)

execute proGetPageData 3,3
```




## 调用带参数的存储过程

&#43; 无参数的存储过程调用：
  Exec pro_GetAge
&#43; 有参数的存储过程两种调用法：
  &#43; EXEC proGetPageData 60,55 ---按次序
  &#43; EXEC proGetPageData @labPass=55,@writtenPass=60 --参数名
    参数有默认值时：
  &#43; EXEC proGetPageData --都用默认值 
  &#43; EXEC proGetPageData 1  --页容量(@pageSize)默认值 
  &#43; EXEC proGetPageData 1,5   --不用默认值

### 存储过程中使用输出参数

```sql
create procedure [dbo].[proGetPageData2] –带输出参数的存储过程
@pageIndex int=1,
@pageSize int=3,
@pageCount int output, --总页数
@rowCount int output -- 总行数
as
declare @sqlStr nvarchar(300),@sqlCount nvarchar(300)
SET @sqlCount = &#39;SELECT @rowCount=COUNT(b_id),@pageCount=CEILING((COUNT(b_id)&#43;0.0)/&#39;&#43; CAST(@pageSize AS VARCHAR)&#43;&#39;) FROM Book&#39;
print @sqlCount
EXEC SP_EXECUTESQL @sqlCount,N&#39;@rowCount INT OUTPUT,@pageCount INT OUTPUT&#39;,@rowCount OUTPUT,@pageCount OUTPUT
set @sqlStr=&#39;select top &#39;&#43;str(@pageSize)&#43;&#39; * from Category where c_id not in(select top &#39;&#43;str((@pageIndex-1)*@pageSize)&#43;&#39; c_id from Category order by c_addtime)order by c_addtime&#39;
print @sqlStr
EXEC(@sqlStr)
```



```sql
declare @pc int
declare @rc int
exec [proGetPageData2] 1,3,@pc output,@rc output
select @pc,@rc
```



## 触发器

&#43; 触发器是一种特殊类型的存储过程，它不同于前面介绍过的一般的存储过程。
&#43; 一般的存储过程通过存储过程名称被直接调用，而触发器主要是通过事件进行触发而被执行。
&#43; 触发器是一个功能强大的工具，在表中数据发生变化时自动强制执行。触发器可以用于SQL Server约束、默认值和规则的完整性检查，还可以完成难以用普通约束实现的复杂功能。
&#43; 那究竟何为触发器？在SQL Server里面也就是对某一个表的一定的操作，触发某种条件，从而执行的一段程序。
&#43; 触发器是一个特殊的存储过程。 
&#43; 常见的触发器有三种：
  &#43; 分别应用于Insert , 
  &#43; Update , 
  &#43; Delete 事件 

### 常用语法

```sql
CREATE TRIGGER triggerName ON Table
for UPDATE|INSERT|DELETE
AS
begin
…
end
```

### 更新

```SQL
CREATE TRIGGER testForFun ON  dbo.T_Category
for UPDATE
AS
begin
select * from dbo.T_Book
end
update dbo.T_Category set FName = &#39;Android2&#39; where FId=3
```

### 删除

```sql
CREATE TRIGGER testForDel ON dbo.Category 
for delete 
AS
begin
select * from book
end
delete Category set c_name = &#39;Android2&#39; where c_id=3
```

### 在触发器中获取值

&#43; `SELECT * FROM INSERTED`
&#43; 对于更新触发器，旧记录值在DELETED中可用，新记录值在INSERTED中

```SQL
DECLARE @OldValue int，@ NewValue int
SELECT @OldValue = Column1 FROM DELETED
SELECT @NewValue = Column1 FROM INSERTED
通过保持旧值和新值，您可以比较它们的状态。
```

# ADO.NET

程序要和数据库交互要通过ADO.Net 进行
通过ADO.Net就能在程序中执行SQL了。
ADO.Net中提供了对各种不同数据库的统一操作接口。

## ADO.NET的组成

![](/dotnet/zgdeIPR.png)

![](/dotnet/lMz89ZA.jpg)
![](/dotnet/lnJKZSp.jpg)

&#43; 如果要执行增删改和单个值查询的时候，可以直接让【车间工人】去【中央仓库】做。
&#43; 如果要从【中央仓库】查询多行货物的时候，有两种方式：
  &#43; 可以选择叫一辆【货运卡车】去搬，卡车可以一次性的都搬过来，但【生产车间】一下子用不了，所以卡车就把货先放在【车间临时仓库】，这样车间需要的时候直接拿就可以了。
  &#43; 可以让【车间工人】把自己的【摩托车】拿来，骑【摩托车】去仓库拿货，但每次只能拿一行货物，所以需要往返的拿很多次才能拿完。但因为每次只拿一行货物过来，车间就直接使用了，不必存到【车间临时仓库】里。

## Connection 类

&#43; 和数据库交互，你必须连接它。连接帮助指明数据库服务器、数据库名字、用户名、密码，和连接数据库所需要的其它参数。Connection对象会被Command对象使用，这样就能够知道是在哪个数据源上面执行命令。
&#43; 与数据库交互的过程意味着你必须指明想要执行的操作。这是依靠Command对象执行的。你使用Command对象来发送SQL语句给数据库。Command对象使用Connection对象来指出与哪个数据源进行连接。你能够单独使用Command对象来直接执行命令，或者将一个Command对象的引用传递给DataAdapter，它保存了一组能够操作下面描述的一组数据的命令。

## Command对象

&#43; 成功于数据建立连接后,就可以用Command对象来执行查询、修改、插入、删除等命令; Command对象常用的方法有ExecuteReader方法、ExecuteScalar()方法和ExecuteNonQuery()方法;插入数据可用ExecuteNonQuery()方法来执行插入命令。

## DataReader类

&#43; 许多数据操作要求你只是读取一串数据。DataReader对象允许你获得从Command对象的SELECT语句得到的结果。考虑性能的因素，从DataReader返回的数据都是快速的且只是“向前”的数据流。这意味着你只能按照一定的顺序从数据流中取出数据。这对于速度来说是有好处的，但是如果你需要操作数据，更好的办法是使用DataSet。

## DataSet对象

&#43; DataSet对象是数据在内存中的表示形式。它包括多个DataTable对象，而DataTable包含列和行，就象一个普通的数据库中的表。你甚至能够定义表之间的关系来创建主从关系（parent-child relationships）。DataSet是在特定的场景下使用――帮助管理内存中的数据并支持对数据的断开操作的。DataSet是被所有Data Providers使用的对象，因此它并不像Data Provider一样需要特别的前缀。

## DataAdapter类

&#43; 某些时候你使用的数据主要是只读的，并且你很少需要将其改变至底层的数据源。同样一些情况要求在内存中缓存数据，以此来减少并不改变的数据被数据库调用的次数。DataAdapter通过断开模型来帮助你方便的完成对以上情况的处理。当在一单批次的对数据库的读写操作的持续的改变返回至数据库的时候，DataAdapter 填充（fill）DataSet对象。DataAadapter包含对连接对象以及当对数据库进行读取或者写入的时候自动的打开或者关闭连接的引用。另外，DataAdapter包含对数据的SELECT、INSERT、UPDATE和DELETE操作的Command对象引用。你将为DataSet中的每一个Table都定义DataAadapter，它将为你照顾所有与数据库的连接。所有你将做的工作是告诉DataAdapter什么时候装载或者写入到数据库。

## DataTable类

&#43; DataTable 是一个数据网格控件,理解成一张表就可以了。
&#43; DataTable的实例化以及添加列：

```sql
DataTable dt = new DataTable();
dt.Columns.Add(&#34;ID&#34;);
dt.Columns.Add(&#34;Name&#34;);
DataRow dr = dt.NewRow();
object[] objs = { 1, &#34;Name&#34; };
dr.ItemArray = objs;
dt.Rows.Add(dr);
this.dataGridView1.DataSource = dt;
```



## ADO.Net基础

&#43; 直接在项目中内嵌mdf文件的方式使用SQLServer数据库（新建→数据→基于服务的数据库）。mdf文件随着项目走，用起来方便，和在数据库服务器上创建数据库没什么区别，运行的时候会自动附加（Attach）
&#43; 双击mdf文件会在“服务器资源管理器”中打开，管理方式和在Management Studio没有什么本质不同。要拷贝mdf文件需要关闭所有指向mdf文件的连接。
&#43; 正式生产运行的时候附加到SQLServer上、修改连接字符串即可，除此之外没有任何的区别，在“数据库”节点上点右键“附加”；在数据库节点上→任务→分离就可以得到可以拷来拷去mdf文件。
&#43; 用的时候要在控制台、WinForm项目中在Main函数最开始的位置加入备注中的代码。ASP.Net项目中不需要。

&#43; 连接字符串：程序通过连接字符串 指定要连哪台服务器上的、哪个实例的哪个数据库、用什么用户名密码等。
&#43; 项目内嵌mdf文件形式的连接字符串&#34;DataSource=.\SQLEXPRESS;AttachDBFilename=|DataDirectory|\Database1.mdf;Integrated Security=True;User Instance=True&#34;。“.\SQLEXPRESS”表示“本机上的SQLEXPRESS实例”，如果数据库实例名不是SQLEXPRESS，则需要修改。“Database1.mdf”为mdf的文件名。
&#43; ADO.Net中通过SqlConnection类创建到SQLServer的连接，SqlConnection代表一个数据库连接，ADO.Net中的连接等资源都实现了IDisposable接口，可以使用using进行资源管理。执行备注中的代码如果成功了就ok。

```sql
using (SqlConnection conn = new SqlConnection(@&#34;Data Source=.\SQLEXPRESS;AttachDBFilename=|DataDirectory|\Database1.mdf;Integrated Security=True;User Instance=True&#34;))
{
  conn.Open();
  Console.WriteLine(&#34;连接成功！&#34;);
}
```



&#43; 增删改查

![](/dotnet/YkrwU5H.png)

&#43; SqlCommand表示向服务器提交的一个命令（SQL语句等） , CommandText属性为要执行的SQL语句，ExecuteNonQuery方法执行一个非查询语句（Update、Insert、Delete等）

```sql
using (SqlCommand cmd = conn.CreateCommand())
{
cmd.CommandText = &#34;Insert into T_Users(UserName,Password) values(&#39;admin&#39;,&#39;888888&#39;)&#34;;
cmd.ExecuteNonQuery();
}
```


ExecuteNonQuery返回值是执行的影响行数
常犯错：

```sql
string username=&#39;test&#39;;
....
cmd.CommandText = &#34;Insert into T_Users(UserName,Password) values(username,&#39;888888&#39;)&#34;;
```



## 查询单个值

![](/dotnet/mXdmcSz.png)

&#43; SqlCommand的ExecuteScalar方法用于执行查询，并返回查询所返回的结果集中第一行的第一列，因为不能确定返回值的类型，所以返回值是object类型。

```sql
cmd.CommandText = &#34;select count(*) from T_Users&#34;;
int i = Convert.ToInt32(cmd.ExecuteScalar())
cmd.CommandText = &#34;select getdate()&#34;; 
DateTime dt = Convert.ToDateTime(cmd.ExecuteScalar());
```



&#43; 得到自动增长字段的主键值，用@@identity(**目前工作阶段任何资料表中所产生的最后一个识别值** )，用ExecuteScalar执行最方便

```sql
cmd.CommandText = “Insert into T_Users(UserName,Password) values(‘admin’,‘888888’);
select @@identity;&#34;;
int i = Convert.ToInt32(cmd.ExecuteScalar());
```



## 查询多行值

![](/dotnet/RCNwZxQ.png)

![](/dotnet/j5ryUB3.png)

&#43; 执行有多行结果集的用ExecuteReader

```sql
SqlDataReader reader = cmd.ExecuteReader();
while (reader.Read())
{  
Console.WriteLine(reader.GetString(1));
}
```

&#43; reader的GetString、GetInt32等方法只接受整数参数，也就是序号，用GetOrdinal方法根据列名动态得到序号
&#43; 为什么用using。Close：关闭以后还能打开。
&#43; Dispose：直接销毁，不能再次使用。
&#43; using在出了作用域以后调用Dispose，
&#43; SqlConnection、FileStream等的Dispose内部都会做这样的判断：判断有没有close，如果没有Close就先Close再Dispose。

## 注入漏洞攻击

```sql
登录判断：select * from T_Users where UserName=... and Password=...，将参数拼到SQL语句中。
构造恶意的Password：&#39; or &#39;1&#39;=&#39;1
if (reader.Read())
{
    Console.WriteLine(&#34;登录成功&#34;);
}
else
 {
    Console.WriteLine(&#34;登录失败&#34;);
}
```

防范注入漏洞攻击的方法：不使用SQL语句拼接，通过参数赋值

```sql
using (SqlCommand cmd = conn.CreateCommand())
{
string password = &#34;&#39; or &#39;1&#39;=&#39;1&#34;;
cmd.CommandText = &#34;select * from T_Users where UserName=&#39;admin&#39; and Password=&#39;&#34; &#43; password&#43;&#34;&#39;&#34;;
using (SqlDataReader reader = cmd.ExecuteReader())
{
if (reader.Read())
{
Console.WriteLine(&#34;登录成功&#34;);
}
else
{
Console.WriteLine(&#34;登录失败&#34;);
}
}
}
```



## 查询参数

&#43; SQL语句使用@UserName表示“此处用参数代替”，向SqlCommand的Parameters中添加参数

```sql
cmd.CommandText = &#34;select * from T_Users where UserName=@UserName and Password=@Password&#34;;
cmd.Parameters.Add(new SqlParameter(&#34;UserName&#34;,&#34;admin&#34;));
cmd.Parameters.Add(new SqlParameter(&#34;Password&#34;,password));
```

&#43; 参数在SQLServer内部不是简单的字符串替换，SQLServer直接用添加的值进行数据比较，因此不会有注入漏洞攻击。


## 案例

&#43; ComboBox的显示值：Items.Add的参数是Object类型，也就是可以放任意数据类型的数据，可以设置DisplayMember属性设定显示的属性，通过SelectedItem属性取得到就是选择的条目对应的对象。例子。疑问：取出来的是Object，怎么能转换为对应的类型？变量名只是“标签”。显示的值和实际的对象不一样，在ASP.Net中也有相同的东西
&#43; 创建一个ProvinceItem类，将数据填充在这个对象中添加到ComboBox中。
&#43; 将连接字符串写在代码中的缺点：多次重复，违反了DRY（Don&#39;t Repeat Yourself）原则；如果要修改连接字符串就要修改代码。将连接字符串写在App.Config中：
&#43; 添加App.config文件（文件名不能改）：添加→新建项→常规→应用程序配置文件。App.config是.Net的通用配置文件，在ASP.Net中也能同样使用。
&#43; 在App.config中添加connectionStrings段，添加一个add项，用name属性起一个名字（比如DbConnStr），connectionString属性指定连接字符串。
&#43; 在“引用”节点上点右键“添加引用”，找到System.configuration。不是所有.Net中的类都能直接调用，类所在的Assembly要被添加到项目的引用中才可以。
&#43; ConfigurationManager.ConnectionStrings[&#34; DbConnStr &#34;].ConnectionString得到连接字符串。
  如何在部署的程序中修改配置。

```sql
&lt;?xml version=&#34;1.0&#34; encoding=&#34;utf-8&#34; ?&gt;
&lt;configuration&gt;
  &lt;connectionStrings&gt;
&lt;add name=&#34;conStr&#34; connectionString =&#34;server=.;database=SimpleArticle;Integrated Security=True;&#34;/&gt;
  &lt;/connectionStrings&gt;
&lt;/configuration&gt;
```



## 多行查询

![](/dotnet/b9fcKtc.png)

&#43; SqlDataReader是连接相关的， SqlDataReader中的查询结果并不是放到程序中的，而是放在数据库服务器中，SqlDataReader只是相当于放了一个指针（游标），只能读取当前游标指向的行，一旦连接断开就不能再读取。这样做的好处就是无论查询结果有多少条，对程序占用的内存都几乎没有影响。
&#43; SqlDataReader对于小数据量的数据来说带来的只有麻烦，优点可以忽略不计。ADO.Net中提供了数据集的机制，将查询结果填充到本地内存中，这样连接断开、服务器断开都不影响数据的读取。

```sql
DataSet dataset = new DataSet();
SqlDataAdapter adapter = new SqlDataAdapter(cmd); 
adapter.Fill(dataset);
```




&#43; SqlDataAdapter是DataSet和数据库之间沟通的桥梁。数据集DataSet包含若干表DataTable，DataTable包含若干行DataRow。`foreach (DataRow row in dataset.Tables[0].Rows) row[&#34;Name&#34;]。`


## SQL Helper

&#43; 封装一个SQLHelper类方便使用，提供ExecuteDataTable(string sql,params SqlParameter[] parameters)、ExecuteNonQuery(string sql,params SqlParameter[] parameters)、ExecuteScalar(string sql,params SqlParameter[] parameters)等方法。 网上有微软提供的最全的SQLHelper类，是Enterprise Library中的一部分。

&#43; sqlconnection在程序中一直保持它open可以吗？对于数据库来说，连接是非常宝贵的资源，一定要用完了就close、dispose。

## DataSet

&#43; 可以更新行row[&#34;Name&#34;] = &#34;yzk&#34;、删除行datatable.Rows.Remove()、新增行datatable. NewRow()。这一切都是修改的内存中的DataSet，没有修改数据库。
&#43; 可以调用SqlDataAdapter的Update方法将对DataSet的修改提交到数据库，Update方法有很多重载方法，可以提交整个DataSet、DataTable或者若干DataRow。但是需要为SqlDataAdapter提供DeleteCommand、UpdateCommand、InsertCommand它才知道如何将对DataSet的修改提交到数据库，由于这几个Command要求的格式非常苛刻，因此开发人员自己写非常困难，可以用SqlCommandBuilder自动生成这几个Command，用法很简单：new SqlCommandBuilder(adapter)。查看生成的Command（没有直接赋值给SqlDataAdapter ，看SqlCommandBuilder的）。SqlCommandBuilder要求表必须有主键。
&#43; (*)通过DataRow的RowState可以获得行的状态（删除、修改、新增等）；调用DataSet的GetChanges()方法得到变化的结果集，降低传递的资源占用。

## 可空数据

&#43; C#中值类型（int、Guid、bool等）是不可以为空的，int i=null是错误的，因此int、bool等这些类型不能表示数据库中的“Null” 。因此C#提供了“可空类型”这种语法，只要在类型后加?就构成了可空的数据类型，比如int?、bool?，这样int? i=null 就可以了。解决数据库中int可以为null，而C#中int不能为null的问题。
&#43; 判断可空类型是否为空，i==null或者i.HasValue；得到可空变量的值，int i1=(int)i.Value或者int i1=i.Value。
&#43; 类型转换：不可空类型赋值给可空类型无需显式转换（一定成功），可空类型赋值给不可空类型则需显式转换（不一定成功）。

## 弱类型DataSet的缺点

&#43; 只能通过列名引用，dataset.Tables[0].Rows[0][“Age”]，如果写错了列名编译时不会发现错误，因此开发时必须要记着列名。
&#43; int age = Convert.ToInt32(dataset.Rows[0][“Age”])，取到的字段的值是object类型，必须小心翼翼的进行类型转换，不仅麻烦，而且容易出错。
&#43; 将DataSet传递给其他使用者，使用者很难识别出有哪些列可以供使用
&#43; 运行时才能知道所有列名，数据绑定麻烦，无法使用Winform、ASP.Net的快速开发功能。
&#43; 自己动手写强类型DataSet(类型化DataSet，TypedDataSet)，创建继承自DataSet的PersonDataSet类，封装出int? Age等属性和bool IsAgeNull等方法，向PersonDataSet中填充。

## VS自动生成强类型DataSet

&#43; 添加→新建项→数据集
&#43; 将表从服务器资源管理器拖放到DataSet中。注意拖放过程是自动根据表结构生成强类型DataSet等类，没有把数据也拖过来，程序还是连的那个数据库，自动将数据库连接字符串写在了App.Config中。
&#43; 代码中使用DataSet示例：CC_RecordTableAdapter adapter = new CC_RecordTableAdapter(); 如何得知Adapter的类名？选中DataSet中下半部分的Adapter，Name属性就是类名。需要右键点击类名→解析
  取得所有的数据：adapter.GetData()，例子程序：遍历显示所有数据，i&lt;adapter.GetData().Count;adapter.GetData()[i].Age。
&#43; 常见问题：类名敲不对，表名&#43;TableAdapter，表名&#43;DataTable，表名&#43;Row，然后用“解析”来填充类名，别照着我的代码敲。
&#43; 常见问题：类的内部定义的类要通过包含namespace的全名来引用，不能省略。类的内部定义的类就能避免同一个namespace下类不能重名的问题。
&#43; 强类型DataSet其实就是一种代码生成器的实现机制（DataSetPersons.Designer.cs），
&#43; 调用的 ***TableAdapter等类都是VS自动生成的，可以看到的，不要手动改生成的类代码，改xsd即可。
  GetData和Fill的区别。

## 强类型强在哪

&#43; 像使用类的属性一样使用列名，dsPerson[0].Age，可以使用VS的自动提示功能，绝对不会写错列名，写错了编译通不过。
&#43; 将强类型DataSet传递给其他人，使用者可以轻松确定有哪些列
&#43; int age = dsPerson[0].Age，列名的类型是明确的，避免类型转换的麻烦。
  编译时就可以确定
&#43; 名词：强类型DataSet（类型化DataSet），英文：Typed DataSet。
&#43; DataSet包含DataTable、DataTable包含DataRow，强类型DataSet同样如此。查看源代码看看VS帮我们做了什么
&#43; GetData返回是什么类型？每一行是什么类型？看类型定义即可得知。一般规律：表类型名：表名&#43;DataTable，行类型名：表名&#43;row，忘了也没关系：“转到定义”。

## 更新DataSet

&#43; 调用Adapter的Update方法就可以将DataSet的改变保存到数据库。adapter.Update(datatable)
  要调用Update方法更新必须设置数据库主键，后面的Delete也是如此。
&#43; 常见错误：“当传递具有已修改行的 DataRow 集合时，更新要求有效的 UpdateCommand”，要为表设置主键。“谁都变了，唯有主键不会变”，程序要通过主键来定位要更新的行。忘了设主键怎么办？先到数据库中设置主键，然后在DataSet的对应DataTable上点右键，选择“配置”，在对话框中点击【完成】。好习惯：所有表都要设置主键！！！看看为什么会自动帮我们GetData、Update、Delete。

## 其他问题

&#43; 插入新行，调用Insert方法。
&#43; 增加字段怎么办？DataSet设计器中点【配置】，对话框中点【查询生成器】，勾选新增加的字段即可。删除字段同样如此。如果是高手也可以直接手改SQL语句。
&#43; 要修改字段就要重新配置生成，这就是强类型DataSet的弱点，因此强类型DataSet不一定真的就是“强”，还是叫“类型化DataSet”(Typed DataSet)吧
&#43; 常见错误：报错：数据为空。判断列的值为空的方法：Is**Null
&#43; 为什么Select方法会填充、Update方法会更新，Insert方法会插入？没有多么神奇，看看Adapter的SelectCommand等属性，是那些SQL语句在起作用，如果有需要完全可以手工调整

## 增加新的SQL语句

&#43; 设计器的Adapter中点右键，选择“添加查询”→“使用SQL语句”，就可以添加多种类型的SQL语句。如果是“SELECT（返回行）”则SQL语句的列必须是对应DataSet类的父集合，生成两个方法：FillBy*和GetBy*，方法名根据查询语句的意义定，比如FillByAge，FillBy是将结果填充到现有DataSet，GetBy是将结果以DataSet方式返回，建议两个都生成，方便以后用。看看默认生成的GetData就明白了
&#43; GetDataById、IncAge
&#43; “SELECT（返回单个值）”就是ExecuteScalar
&#43; 对于增加的SQL语句在代码中是以方法的形式使用的。方法的参数类型、顺序就是VS猜测的，如果不正确或者需要调整只要选中对应的语句，然后在【属性】窗口中修改Parameters属性即可
&#43; 增加新的SQL语句本质论，探寻源码：不能并发调用。
&#43; 像使用普通类的方法一样使用Adapter。SQL语句不用再写在界面代码中。这就是一种数据访问层（DAL：Data Access Layer）


## 强类型DataSet其他

&#43; 通过查看生成的源代码的值，生成的强类型TableAdapter默认每次调用方法都是打开连接、执行、关闭连接，而如果操作之前连接已经打开则不会自动帮我们连接、关闭，因此如果想批量操作提高效率可以操作之前先自己Open，操作完毕再Close。经测试：插入三千条数据，不优化用了45秒，优化后只用一两秒。回答面试问题：如何优化访问数据库的效率。
&#43; 常见错误：DataSet ds = new DataSet()；ds = GetData();变量名和对象。

## 数据绑定

&#43; DataGridView绑定。拖放TableAdapter、DataSet、bindingSource，将bindingSource的DataSource设定为DataSet，设定DataMember属性，然后DataGridView绑定到bindingSource。在Load的时候调用TableAdapter的Fill方法将数据填充到DataSet。绑定：双方能同步感知对方的变化。
&#43; DataGridView绑定到BindingSource， BindingSource绑定到DataSet，所以DataGridView显示的是DataSet中的数据。
&#43; 修改列标题。
&#43; 将保存提交到数据库，在DataGridView中修改会同步反应到DataSet中，这样只要将DataSet Update到数据库就是“保存修改”，Update，保存前要dataGridView1.EndEdit(); dataGridView1.CommitEdit(DataGridViewDataErrorContexts.Commit);bindingsource1.EndEdit()已提交正在编辑的修改。
&#43; 删除当前选择行：cCRecordBindingSource.RemoveCurrent()，只是删除DataSet中的数据，需要Update才能提交到数据库。
&#43; 绑定单独控件，在控件属性的DataBindings中将属性绑定到BindingSource 的指定字段，这样控件中的值就会显示这个字段的值了

## 补充

&#43; 拖过来的控件是什么？控件就是控件类的对象，Winform中从Component类继承的类都可以拖到窗口中以控件的形式出来，本质上和new出来的对象没区别。控件的id就是变量名。
&#43; 新建的强类型DataSet只有“生成”以后才会在工具箱中出现
&#43; 并不是控件的所有属性都能绑定，只有显示在DataBindings节点下的属性才能绑定（*）只有标记了[Bindable(true)]的属性才能绑定。
&#43; 只有移开焦点才会同步，并不是实时同步。
&#43; 刷新查询窗口中的数据“执行SQL”

&#43; BindingSource是做什么的？维持当前项。这就是为什么详细控件和DataGridView会联动。试试控件绑定到不同的BindingSource。
&#43; Adapter的作用是负责DataSet和数据库之间的数据传递。
&#43; 绑定到ComboBox。给Person增加一个TypeId字段（表示是黄种人、白种人、黑种人还是其他人种）。ComboBox的绑定分为显示数据项的绑定、选中值的绑定两个，DataSource属性设定要数据项绑定的数据源，DisplayMember属性为显示的属性、ValueMember为值（通过SelectedValue取得）的属性；然后绑定SelectedValue属性到表的字段。
&#43; DataGridView中的ComboBox列：设定列的ColumnType为DataGridViewComboBoxColumn为，然后其他绑定和普通ComboBox一样，由于BindingSource是维持当前项，所以记住“专BindingSource专用”


---

> 作者: [AndrewDu](https://github.com/DDuuuu)  
> URL: https://DDuuuu.github.io/2018/10/dotnetbase2-sql-adonet/  

