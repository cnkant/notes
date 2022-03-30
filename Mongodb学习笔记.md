# Mongodb学习笔记

## 一、安装

Linux：

​	命令安装	

​					sudo apt install -y mongodb-org

​	源码安装	

​					下载源码——解压——移动到/usr/local/目录下——在shell的初始化脚本.bashrc中添加mongodb可执行文件到环境变量path中。（在.bashrc文件的最后添加export PATH=/usr/local/mongodb/bin:$PATH）

## 二、简单使用

默认端口：27017

默认配置文件的位置：/etc/mongodb.conf

默认日志的位置：/var/log/mongodb/mongodb.log

启动方式：

​		本地测试方式启动，只具有本地数据增删改查的功能，主要是验证数据库是否能正常运行

​		生产环境启动，具有完整的全部功能，服务器部署启动

### 本地测试方式启动：

​		启动	sudo service mongodb start 

​		停止	sudo service mongodb stop

​		重启	sudo service mongodb restart

### 生产环境正式的启动方式：

​		sudo mongod 可选参数

​		只以sudo mongod 命令启动时，默认将数据存放在了/data/db目录下，需要手动创建

​		可选参数：

​				--dbpath=数据库存放路径（绝对路径）

​				--logpath=日志文件的存放路径，重启后日志会覆盖原来的日志，不是追加写入

​				在后面添加--append或者--logappend	可以设置日志的写入形式为追加模式

​				--fork或者-fork	在当前进程下开启新的子进程运行mongodb服务，一旦开启成功，父进程就会退出

​				--auth	以权限认证的方式启动

​				-f 配置文件名	可以将上述的配置信息写入到文件，然后通过该文件中的参数进行加载启动

​						例如配置文件这样写：

​									dbpath=数据库存放路径

​									logpath=日志文件的存放路径

​									logappend=true（注意不能写append）

​									fork=true

​									auth=true

### 启动mongodb的客户端：

​	启动本地客户端	mongo

​	退出	exit或者ctrl+c

​	查看帮助	mongo --help

### 数据库的操作

​	查看当前的数据库	db	（没有切换数据库的情况下默认使用test数据库）

​	查看所有的数据库	show dbs或者show databases	（看不到test数据库是因为此数据库不在磁盘中，而是在内存中，只能查看磁盘中存在的数据库）

​	切换数据库	use 数据库名，如果数据库不存在，则创建数据库，否则切换到指定数据库。创建之后show dbs并不能显示出来，需要往里面插入数据（db.集合名.insert({key:value})	集合若不存在会自动创建）或者创建一个集合才能将其显示出来

​	删除当前的数据库	dp.dropDatabase()

### 集合的操作

集合类似于mysql中的表

查看所有集合	show collections	当没有数据时是看不到集合的，需要先插入数据

显示的创建集合 db.createCollection(名字，可选参数)

​	db.createCollection("sub",{capped:true,size:10})	

​		capped默认为false，表示不设置上限；size表示集合所占用的字节数，当capped为true时需要设置此参数，当文档达到上限时，写入数据会将之前的数据覆盖，先覆盖最早插入的（size的值小于等于256时，实际上都为256）

​		有上限的集合数据不允许被修改，且查询速度也会相对较快

查看集合是否有上限	db.集合名.isCapped()	false无上限，true有上限

查看集合的数据	db.集合名.find()

删除集合	db.集合名.drop()

## 三、常见数据类型

Object ID：文档/数据的ID，是数据的主键，默认会创建索引

String：字符串，最常用，必须是有效的UTF-8

Boolean：布尔值，true或者false，必须是小写

Integer：整数，32或者64位，取决于服务器

Double：（双精度）浮点数

Arrays：数组、列表

Object：mongodb中的一条数据（文档），即文档嵌套文档

Null：空值

Timestamp：时间戳，表示从1970-1-1到现在的秒数

Date：存储当前日期或者时间的UNIX时间格式

注意：每个文档都有一个属性为_id，保证了每个文档的唯一性，mongodb默认使用其作为主键，其值可以手动设置，如果没有设置，MongoDB为每个文档提供了一个独特的值，类型为ObjectID

ObjectID是一个12字节的十六进制数，每个字节占两位，一共是24位（十六进制的数字字符串）

​		前四个字节（前八位）：当前时间戳

​		接下来的三个字节（六位）：机器的ID

​		接下来的两个字节（四位）：MongoDB的服务进程ID

​		最后三个字节（六位）：简单的增量值

## 四、增删改查

### 插入数据

方式一

db.集合名.insert({key:value})	key可以不加引号，终端会自动添加，value是数字不加引号，字符串必须加引号

db.集合名.insert([{},{},{}...])	批量插入

如果想修改_id的值，插入数据时将其作为key，值作为value，放到要插入的一条数据（即文档）前面即可

​		如：db.集合名.insert({_id:"值",key:value})

方式二

db.集合名.save(_id:值,key:value)	如果id值已经存在且其他数据不同则修改（会把原来数据全部删除，更新为新数据），如果id值不存在则添加新数据

db.集合名.save(key:value)	MongoDB会指定id

### 查询数据

db.集合名.find()	会查询全部数据

​	db.集合名.find({条件文档})

db.集合名.findOne()	只返回第一个

​	db.集合名.findOne({条件文档})

db.集合名.pretty()	将结果格式化，不能和findOne()一起使用

​	db.集合名.find({条件文档}).pretty()

条件文档：

#### 	比较运算符

​		等于：默认就是等于判断，没有运算符

​		小于：$lt	也就是less than

​		小于等于：$lte	也就是less than euqal

​		大于：$gt	也就是greater than

​		大于等于：$gte	也就是greater than equal

​		不等于：$ne	也就是no equal

​		db.stu.find({age:{$gte:18}})	查询年龄大于等于18的所有学生

#### 	逻辑运算符

​		and：$and

​				  或者直接在一个{}中写多个条件即可

​		or：$or	值为数组（列表），数组中的每个元素为json，代表一个条件

​		db.stu.find({age:{$lt:18},gender:'male'})	查询年龄小于18，性别为male的所有学生

​		db.stu.find({$or:[{age:{$ne:18}},{gende:'female'}]})	查询年龄不等于18或者性别为female的所有学生

​		db.stu.find({$or:[{age:{$gt:18}},{age:12}],gender:"male"})	查询年龄大于18或者年龄等于12并且性别为male的所有		学生

#### 	范围运算符

​		$in	查找的数据是在数组内

​		$nin	查找的数据不在数组内

​		db.stu.find({age:{$in:[12,23,34]}})	查询年龄为12、23、34的所有学生

#### 	正则表达式

​		$regex	一般存的值为字符串时使用

​		db.stu.find({name:{$regex:'^黄'}})	查询name以“黄”开头的所有学生

#### 	自定义查询

​		mongo shell 是一个js的执行环境，使用$where写一个js函数，返回满足条件的数据

​		db.stu.find({$where:function(){return this.age>30;}})	返回年龄大于30的所有学生，this类似于MySQL的游标

#### 	对查询结果进行操作

##### 		skip和limit

​		limit(数字)	读取指定数量的文档

​		skip(数字)	跳过指定数量的文档		

​		skip的优先级高于limit

​		db.stu.find().limit(5).skip(5)	跳过前五条数据，显示后面的前五条数据，改变skip和limit的前后顺序结果一样

##### 		投影

​		在查询到的结果中只显示选择的字段

​		db.集合名.find({},{字段名称:1,字段名称:1.....})	

​			前面的空括号填查询条件，不写就是查询条件为空值为1表示显示，值为0表示不显示

​			_id列是默认显示的，如果不显示需要明确设置为0

​			对于其他的字段不能既有设置为0的字段，还有设置为1的字段

​			PS：除了_id之外，想看哪个字段设置为1就行啦

##### 		排序

​		sort()

​		db.集合名.find().sort({字段名:1,...})	1代表升序排列，-1代表降序	排序

​		db.stu.find().sort({gender:-1,age:1})	根据性别降序排序，当性别相同时再根据年龄升序排序

##### 		统计个数

​		count()	统计查询结果集的文档条数

​		db.集合名.find({条件}).count()	等于db.集合名.count({条件})

##### 		去重

​		db.集合名.distinct("字段名")	只会显示此字段名的去重后的所有值，是一个列表

​		db.集合名.distinct("字段名",{查询条件})	显示根据查询条件查询出来的值的此字段名的去重后的所有值

##### 显示操作的详细信息

db.集合名.find().explain("executionStats")	可以查看操作的执行时间等各种信息

### 	更新数据

​		db.集合名.update({查询条件},{更新操作},{multi:true/false})

​		multi为可选参数，默认是false，表示只更新找到的第一条数据，true表示更新满足条件的全部数据

​		db.stu.update({name:"hr"},{name:"mc"})	会把找到的name为hr的第一条数据的name改为mc，并且会把其他的字		段值删除

​		db.stu.update({name:"hr"},{$set:{name:"mc"}})	会把找到的name为hr的第一条数据的name改为mc，其他值不变

​		db.stu.update({},{$set:{gender:"female"}},{multi:true})	把全部学生的性别改为female

​		PS：multi:true必须和$set一起使用！

​		upsert参数

​		db.stu.update({name:"liu"},{$set:{age:1000}},{upsert:true})	查询name为liu的第一条数据，如果其age值已经是1000		则不做任何操作，如果不是则更新age的值为1000，若不存在age字段则添加此字段，如果查找失败则插入此条数据

​		同时使用multi和upsert的话，要把它们写到一个大括号里面，逗号隔开

### 	删除数据

​		db.集合名.remove({查询条件},{justOne:true/false})

​		justOne可选，默认是false，表示删除全部，为true或1表示只删除查找到的第一条数据

​		db.集合名.remove({})	删除此集合中的全部数据，即使查询条件为空，第一个查询条件的{}也不能省略

## 五、聚合操作

聚合（aggregate）是基于数据处理的聚合管道，每个文档通过一个由多个阶段（stage）组成的管道，可以对每个阶段的管道进行分组、过滤等，然后经处理，输出相应的结果

db.集合名.aggregate({管道命令:{表达式}})

一个{}是一个管道

前一个管道的输出结果作为下一个管道的输入

### 常用管道命令

在mongodb中，文档处理完毕后，通过管道进行下一次处理

$group：将集合中的文档分组，可用于统计结果

$match：过滤数据，只输出符合条件的文档

$project：修改输入文档的结构，如重命名、增加、删除字段、创建计算结果

$sort：将输入文档排序后输出

$limit：限制聚合管道返回的文档数

$skip：跳过指定数量的文档，并返回余下的文档

$unwind：拆分指定的字段，爬虫用得不多

### 常用表达式

表达式用来处理输入文档并输出

语法：表达式:'$列名'

$sum：计算总和，$sum:1 表示以一倍进行个数的统计

$avg：计算平均值

$min：获取最小值

$max：获取最大值

$push：在结果的指定值插入到一个数组中

#### $group

db.stu.aggregate({$group:{_id:'$gender',sum_age:{$sum:‘$age’},name_list:{$push:'$name'}}})	_id表示以什么字段进行分组，其他的字段名可以	自己任意写，结果是将学生安装gender分组，并且计算每组的年龄总和，把每组的名字存到名为name_list的列表中

或者可以把整个文档分为一组进行统计，查询显示出来的是全部文档数据

db.stu.aggregate({$group:{_id:null,counter:{$sum:1}}})	相当于db.stu.find().count()

#### 数据透视

db.stu.aggregate({$group:{_id:null,name:{$push:'$name'}}})	把所有name放到一个列表里

db.stu.aggregate({$group:{_id:null,data_stu:{$push:'$$ROOT'}}})	将整个文档放入数组中

#### $match

和find的区别就是$match过滤出来的结果可以作为下一个管道的输入

$match的值可以是前面提到的五种查询方式

db.stu.aggregate({$match:{age:{$gt:20}}})	查询年龄大于20的学生

db.stu.aggregate({$match:{age:{$gt:20}}},{$group:{_id:"$gender",counter:{$sum:1}}})	查询年龄大于20的男生和女生的人数

#### $project

类似于投影操作，输出的结果每一条数据都在一个{}中

db.stu.aggregate({$project:{_id:0,name:1,age:1}})	仅输出学生的年龄和姓名

db.stu.aggregate({$group:{_id:"gender",counter:{$sum:1}}},{$project:{_id:0,counter:1}})	查询男生和女生的人数，仅输出人数

#### $sort

db.stu.aggregate({$sort:{age:1}})	查询学生信息，按照年龄升序输出

db.stu.aggregate({$group:{_id:"gender",counter:{$sum:1}}},{$sort:{counter:-1}})	查询男生和女生人数，按照人数降序输出

#### $limit和$skip

注意和在find()中使用时的区别

db.stu.aggregate({$limit:2},{$skip:1})	会只显示第二条数据

db.stu.aggregate({$skip:1},{$limit:2})	会只显示第二条和第三条数据

#### $unwind

拆分，了解即可

db.集合名.aggregate({$unwind:{path:"$你要拆分的字段名",preserveNullAndEmptyArrays:true})

## 六、索引

索引：加快查询速度；进行数据的去重

创建索引后会把原先的数据结构进行调整，调整后的结构类似于二叉排序树，查询效率是O(logn)

### 创建索引

db.集合名.ensureIndex({属性:1or-1})	1表示升序，-1表示降序

### 查看索引

默认情况下_id是集合的索引

db.集合名.getIndexes()

### 删除索引

db.集合名.dropIndex({"索引名称":1})

### 创建唯一索引

在插入数据时会检查一下数据是否已经存在，存在则插入失败

会降低插入速度

db.集合名.ensureIndex({"字段名":1},{"unique":true})

### 建立复合索引

db.collection_name.ensureIndex({字段1:1,字段2:1})	最左原则（用字段1查询使用索引，用字段1和字段2一起查询使用索引，但是用字段2查询不会使用索引）

### 注意点：

根据需要选择是否需要创建唯一索引

索引字段是升序还是降序在单个索引的情况下不影响查询效率，但是带复合索引的条件下会有影响

数据量巨大并且数据库的读出操作非常频繁时才需要创建索引，如果写入操作频繁，创建索引会影响写入速度

PS：在查询时如果字段1需要升序的方式排序输出，字段2需要降序的方式排序输出，那么此时复合索引的建立需要把字段1设置为1，字段2设置为-1

一般在读写操作都非常频繁且数据量巨大时采用读写分离的策略

## 七、权限管理

刚安装的MongoDB默认不使用权限认证方式启动，而且在安装的时候并没有设置权限，而公网运行时需要设置权限以保证数据安全。

MongoDB没有默认的管理员账号，所以要先添加管理员账号，并且MongoDB服务器需要在运行的时候开启验证模式。

用户只能在用户所在的数据库登录（创建用户的数据库），包括管理员账号。

管理员可以管理所有的数据库，但是不能直接管理其他数据库，要先认证后才可以。

### 创建超级用户

先杀死mongod进程

再以权限认证的方式启动mongodb服务端

sudo mongod --auth	或者将auth=true写入配置文件，sudo mongod -f 配置文件名，这样启动

打开客户端，mongo，此时执行数据库各种语句会报权限错误，需要认证才能执行操作

use admin	切换数据库（名字必须是admin）

db.createUser({"user":"用户名","pwd":"密码","roles":["root"]})	添加超级管理员

登录管理员

use admin	由于是在admin数据库上创建的用户，因此必须切换到此数据库上（用户创建在哪个数据库上就切换到哪个数据库）

db.auth("用户名","密码")	显示为1则登录成功，为0则登录失败

登录成功后就能正常执行语句啦

### 创建普通用户

#### 在使用的数据库上创建普通用户

若添加了超级管理员，且以权限认证方式登录，必须先认证后才能创建

use test1

db.createUser({"user":"user1","pwd":"user1","roles":["read"]})	如果是readWrite，则是读写权限

show users	查看当前数据库中的用户情况

#### 在admin数据库上创建普通用户

由于在每个数据库上面都创建一个用户有些繁琐，因此一般大多数情况下都把用户创建在admin数据库上面

use admin

先认证管理员权限	db.auth("python":"python")

添加普通用户

db.createUser({user:"user2",pwd:"user2",roles:[{db:"test2",role:"read"},{db:"test3",role:"readWrite"}]})

这里user2用户在test2数据库上的权限为读，在test3上的权限为读写

PS：登录user2用户时要在admin数据库下登录，因为是在这个数据库上面创建的此用户！

#### 删除用户

登录超级管理员的情况下才能删除其他用户

先进入账号数据所在的数据库

db.dropUser("用户名")	删除用户

