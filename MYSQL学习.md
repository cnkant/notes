# MYSQL学习

# 安装：sudo apt install mysql-server

https://blog.csdn.net/qq_38505969/article/details/109957055

mysql -h 主机名 -p端口 -u 用户名 -p密码（注意p后面无空格）

或mysql -u 用户名 -p密码

或mysql -u 用户名 -p

-u：用户名，-p：密码

退出：exit、quit、ctrl+d

查看mysql进程：ps -aux | grep mysql

查看mysql服务状态：sudo service mysql status

停止mysql服务：sudo service mysql stop

启动mysql服务：sudo service mysql start

重启mysql服务：sudo service mysql restart

mysql配置文件路径：/etc/mysql/mysql.conf.d/mysqld.cnf

查看当前使用的数据库：select database();

删除数据库：drop database 库名;

查看数据库创建信息：show create database 库名;

修改数据库字符集：alter database 库名 charset=utf8;

简单创建数据表：create table user(id int,name char(7),age tinyint);

创建标准的数据表：create table person(id int(5) unsigned zerofill primary key auto_increment,name char(5) not null,height float(5,2) default 0.0,gender enum('男','女') default '男');

查看表结构：desc 表名;

查看创建表的信息：show creat table 表名;

为表添加字段：alter table 表名 add weight float(5,2) not null;

为表修改类型：alter table 表名 modify weight int(5);

修改表的字段名和约束：alter table 表名 change weight tizhong float(5,3) default 99.99;

当字段类型发生变化时，如果原字段中有数据，那么数字字符串和数字之间可以转换

可同时修改多个字段和约束，中间用逗号分隔。

删除表字段 alter table 表名 drop tizhong;

删除表：drop table 表名;

## 查询：

​	select * from person;

## 插入：

​	insert into 表名 values(1,'tom',default,male)，(2,'Jack',178,male);	如果不想提供数据，也要使用default占位

​	insert into 表名 (name) values('Lucy'),('Tom') ;

​	insert into 表名 (id,name,gender) values(5,'Bulke',1);

## 更新数据：

​	update 表名 set 字段名=值;

​	update 表名 set 字段名=值 where 条件;

## 删除数据：

​	delete from 表名;

​	delete from 表名 where 条件;

## 别名：

​	select id as 序号,name as 姓名，gender as 性别 from person;(as 可以省略)

## 数据去重：

​	select distinct name from person;

## 模糊查询：like，%(0或多个字符)，_(一个字符)，

​	select * from person where name like '孙%';

​	select * from person where name like '孙_';

​	select * from person where name like '%孙%';

## 范围查询：between...and...(连续范围内)，in(非连续的范围)

​	select * from person where height between 170 and 180;

​	select * from person where id in (1,7,9);

## 空判断：is null，is not null

​	select * from person where height is null;

​	注意：where height is not null;和where not height is null;结果一样但是执行逻辑不一样，后者是先把空数据取出来再反选。

## 排序：order by

​	select * from 表名 order by 列名 asc：升序排列，不写asc默认升序

​	select * from 表名 order by 列名 desc：降序排列

​	select * from 表名 order by 列1 desc,列2 asc：当列一中出现相同值情况无法排序时使用列二排序

​	where 条件 要写在order by 的前面

## 分页查询：limit，start——索引起始位置，count——条数

​	select * from 表名 limit start,count	从start+1开始读取count条数据，start不写默认为0，count不够时有多少就显示多少

​	select * from student limit (n-1)*m,m	获取第n页数据

​	select * from person where gender='女' order by gender desc limit 5;	查询性别为女的年龄最大的五个人

​	where 条件——order by——limit

## 聚合函数：用来对数据进行统计和计算

​	count(col)：求指定列的总行数(不统计空值的行)、max(col)、min(col)、sum(col)、avg(col)：求指定列的平均值

​	round()，指定保留小数点的位数，如round(avg(height),2)

​	select count(height),max(height),min(height),sum(height),avg(height) from person;

​	一般都是使用count(*)，可以将空值也统计进去

ifnull(height,0)：如果height为空，则用0代替，可与聚合函数连用，如count(ifnull(height,0));

## 分组查询：group by

​	select gender from person group by gender;	通过什么分组就查询什么——select A from 表名 group by A，此方法只能看到分组的类别，看不到各组的成员情况

​	group by 与group_concat()连用可以查看各组成员情况：select gender ,group_concat(name) from person group by gender;

​	having过滤分组数据（having只能用于group by）：select gender,group_concat(name) from person where age>50 group by gender having gender='女';	显示年龄大于50的女生成员姓名

​	与聚合函数连用：select gender,count(age),max(age),min(age),sum(age),avg(age) from person where age is not null group by gender;

​	with rollup：select gender,count(age),max(age),min(age),sum(age),avg(age) from person where age is not null group by gender with rollup;	在最后记录后面新增一行，显示select查询时聚合函数的统计和计算的结果

内连接、左连接、右连接、自连接查询时where必须写到连接语句后面

## 内连接：inner join on	根据on后面的条件取两个表的“交集”，on可省略

​	select * from student inner join class;	这样的话查询出来的条数是student的条数于class条数之积
​	select * from student inner join class on student.class_id=class.id;	on 后跟条件

​	select s.id,s.name,c.class_name class from student as s inner join class c on s.class_id=c.id;	as可以省略

## 左外连接：left join on	以左表为主，根据条件查询右表数据，如果右表数据不存在则以null填充，on不可省略

​	select 字段 from 左表 left join 右表 on 左表.字段1=右表.字段2;

## 右外连接：right join on	以右表为主，根据条件查询左表数据，如果左表数据不存在则以null填充，on不可省略

​	select 字段 from 左表 right join 右表 on 左表.字段1=右表.字段2;

​	将左表和右表互换一下，right和left替换一下，可实现左连接查询的结果和右连接查询的结果相同

## 自连接：左表和右表是同一张表，内连接、左连接、右连接都可以实现自连接

​	select p.name 省名 c.name 市名 from areas p inner join areas c on p.id=c.pid;	pid表示改地名的上级地名id

子查询：子查询是一条完整的select语句，嵌入到主查询中

​	select * from student where age > (select avg(age) from student);	查询年龄大于平均值的学生

​	select name from class where id in (select distinct class_id from student where class_id is not null);	查询学生都归属于哪些班级

​	select * from student where (age,height) = (select max(age),max(height) from student);	查找年龄最大且身高最高的学生

## 外键约束：foreign key(col) reference 

​	对外键字段的值进行更新和插入时，会引用表中的字段进行数据验证，如果数据不合法则会失败，保证数据的有效性

​	一个表可以有多个外键

​	创建表时直接写入：create table Test(id int(5) not null primary key auto_increment,字段1 char(5),foreign key(字段1) references 表2(地段2));	字段1和字段2类型要一致，被参考的字段，即字段2一定是表2的主键

​	注意：定义外键后，要是想删除被参考的那张表或者表中数据，需要先把定义外键的表或者表中数据删除，就好比如果想解散一家公司，要先解散员工。或者先把外键约束删除了。

​	删除外键约束：	

​		1.获取外键名称（系统自动生成）：show create table 表名;	查看外键名

​		2.删除外键约束：alter table 表名 drop foreign key 外键名;

​	添加外键约束：如果字段1中存在无法与字段2匹配的数据，则添加外键会失败。

​		alter table 表名 add constraint fk_class_id foreign key(字段1) references 表2(字段2);	fk_class_id为外键名，constraint fk_class_id可以省略，省略后系统自动生成外键名，外键名一般为fk_+字段名

## 将查询结果作为数据插入到另一张表：

​	insert into ... select...;	不需要写values()

## 使用连接更新表中数据：

​	update 表1 inner/left/right join 表2 on...set...;

建表时通过查询加入数据：create table...select...

​	create table 表名(id int primary key auto_increment,name char(10) not null) select name from 表名;	或者select 字段名 as name from 表名。id可以省略不写，默认从第二个字段开始匹配，注意：select后跟的字段名必须和创建表时添加的字段名相同。

## 导入数据

​	source 文件名.sql	需要提前进入sql文件目录，否则还需要带上绝对路径名

## 事务：	

​	事务是用户定义的一系列执行SQL语句的操作（增、删、改），在一个事务中的所有操作要么完全执行，要么完全不执行（执行到一半意外退出则会发生回滚），它是一个不可分割的工作执行单元。

​	开启事务后执行修改指令，变更的数据会保存到mysql服务端的缓存文件中，而不维护到物理表中，只有执行commit之后才会将本地缓存文件中的数据提交到原物理表中，完成数据的更新。

​	注意：如果没有显示的开启一个事务，每条sql语句都会被当做一个事务，并且由于mysql默认采用自动提交模式(autocommit)，所以会自动执行提交事务的操作。

​	set autocommit=0;	表示取消自动提交事务模式，需要手动commit完成事务的提交（当开启一个事务后，会自动设置autocommit值为0）。

​	显示的执行commit或rollback表示该事务的结束

​	四大特性：

​			原子性(Atomicity)：强调一个事务中的多个操作是一个整体

​			一致性(Consistency)：强调数据库中保存一致的状态，例如ATM取钱时你取多少钱，银行卡中余额就要减少多少

​			隔离性(Isolation)：强调数据库中的多个事务之间的操作互不可见

​			持久性(Durability)：强调数据库能永久保存数据，一旦提交就不可撤销

​	只有InnoDB引擎可使用事务，常用的表的存储引擎是InnoDB和MyISAM，MyISAM不支持事务，优势是访问速度快。

​	修改表的存储引擎：alter  table 表名 engine='MyISAM';

## 事务的操作：

​	sql语句：

​		开启事务begin;或start transaction;

​		提交事务commit;

​		回滚事务rollback;

​		提交或回滚事务后，当前事务会结束

pymysql：

​		连接数据库对象.cursor()	创建游标对象，会自动隐式的开启一个事务

​		提交事务：连接数据库对象.commit()

​		回滚事务：连接数据库对象.rollback()

## 索引：

​	在mysql中也叫做”键“，是一个特殊的文件，保存着数据表里所有记录的位置信息，就像一本书的目录，能加快数据库的查询速度。

​	show index from 表名;	查看索引（主键列会自动创建索引）

​	alter table test_index add index idx_title(title);	创建索引（非常耗费时间），非唯一性索引的名称一般设置为idx_+字段名，不写默认是字段名，添加索引后，使用where查询数据时只需要对比一次。

​	在查询语句前面加desc可以通过查看rows属性值来查看对比次数，Extra属性值查看查询方法

​	alter table test_index drop index idx_title;	删除索引

​	set profiling=1;	开启时间监测

​	show profiles;	查看时间监测的记录数据

## 联合索引（复合索引）：一个索引覆盖表中两个或者多个字段，一般用在多个字段一起查询的时候

​		alter table 表名 add index [索引名] (字段1,字段2,...);

​		最左原则：如果给(字段1,字段2,字段3)创建联合索引，那么同时会给(字段1)、(字段1,字段2)创建索引

​		好处是减少了磁盘空间的开销，缺点是耗费时间长

## 索引的使用原则：

​	1.对经常更新的表避免创建较多的索引，因为更新索引非常耗时，对经常查询的字段应该创建索引

​	2.数据量小的表最好不要创建索引

​	3.在同一字段相同值比较多时不要建立索引，例如“性别”字段

​	4.索引并不是越多越好，要在查询速度和建立索引所占用的时间和空间之间综合权衡



​	



​	

​	

​	
