# -
MySQL 是最流行的关系型数据库管理系统之一,属于 Oracle 旗下产品

安装启动操作
1.安装mysql命令 ：$ sudo apt-get install -y mysql-server
2.查看mysql的版本命令（注意-V是大写，不然会出现如下错误）：$ mysql -V
3.启动mysql命令(关闭，重启等只需将start换成stop,restart等即可)：$sudo service mysql start
4.登录mysql命令为：$ mysql -u用户名 -p密码
5.连接远程数据库：$ mysql -h <host> -P <port> -u<username> -p<password>

数据库操作
1.查看数据库：> show databases; （注意分号“；”不要落下）
2.新建一个数据库命令：> create database 数据库名称;
   删除一个数据库命令：> drop database 数据库名称;
3.使用某个数据库：> use 数据库名称;

表操作
1.查看表命令：> show tables;
2.建立一个新表：> create table 表名 （字段参数）; 或 >create table if not exists 表名（字段参数）；
   删除一个旧表：> drop table 表名； 或 >drop table if exists 表名；
3.查看表结构：> desc 表名称; 或 >show columns from 表名称;
4.对表数据的操作：
   增：>insert into 表名称 (字段名1，字段名2，字段名3......) values(字段名1的值，字段名2的值，字段名3的值......);
   删：>delete from 表名称 where 表达式;
   改：>update 表名称 set 字段名=“新值” where 表达式；
   查：>select 字段名1,字段名2,字段名3..... from 表名称;
5.增加字段：>alter table 表名称 add 字段名 数据类型 其它; (其它包括默认初始值的设定等等)
6.删除字段：>alter table 表名称 drop 字段名;

用户相关操作
注：以下命令均需先以root身份登录mysql：mysql -uroot -p
1.添加新用户
（1）创建新用户：> insert into mysql.user(Host,User,Password) values("localhost","user1",password("password1"));
（2）为用户分配权限：
            设置用户可以在本地访问mysql：grant all privileges on *.* to username@localhost identified by "password" ;
            设置用户只能访问指定数据库：grant all privileges on 数据库名.* to username@localhost identified by "password" ;
（3）刷新系统权限表：>flush privileges;
2.查看MySql当前所有的用户：>SELECT DISTINCT User FROM mysql.user;
3.删除用户及其数据字典中包含的数据：>drop user 'xbb'@'localhost';

联合查询：多条select语句共同构成，每条select语句获取的字段数必须严格一致，但是字段类型无关
select 语句1
Union [union选项]
select 语句2 ...

Union 选项与select选项一样有两个
All 保留所有（不管重复）
Distinct 去重 默认的
保留第一张表的字段

子查询：
查询是在某一个查询结果之上进行的查询（一条select语句内部包含了另外一条select语句）按位置分类，按结果分类
标量子查询：select * from my_class where c_id=(select id from my_class where c_name='class81')
行子查询（返回的可以是多行多列，也可以是一行多列）
数据源：select * from my_student where age=? and high=?;
select max(age),max(high) from my_student;
构造行列数->select * from my_student where (age,height)=select max(age),max(height) from my_student; 

列子查询：查询所有在读班级的学生
select * from my_student where  c_id in(?);
select id from my_class;
-> select * from my_student where c_id in (selece id from my_class);
列子查询返回的结果会比较，一列多行，需要使用in作为条件匹配，其实在mysql中还有几个类似的条件，all，some，any
=any ==== in
any ====some ；--any跟some是一样

表子查询：子查询返回的结果是一个多行多列的二维表；子查询返回的结果是当作一个二维表来使用
需求：找出每个班中最高的一个学生
数据源：将学生按照升高排序：select * from my_student order by height desc;
从每个班选出第一个学生：
select * from my_student group_by c_id;
->select * from (select * from my_student orderby height desc) as student group by c_id; --from后面只能跟表名

exists子查询：判断某些条件是否存在（跨表）
where之后 exist返回的值是0和1
需求：查询所有有班级存在的学生
数据源：select * from my_student where ?;
确定条件是否满足：exists（select * from my_class）
-> select * from my_student where exists(select * from my_class)

视图：view	
是一种有结构（有行有列）没结果（结果中不真实存放数据）的虚拟表
虚拟表的结构来源不是自己定义，而是从对应的基表中产生

创建视图：
create view 视图名称 as select 语句；--select语句可以是普通查询；可以是链接查询；可以是联合查询；可以是子查询
创建单表视图：基表只有一个
create view v1 as select * from my_student;

创建多表视图：基表有N个
create view v2 as select * from my_class;
create view v3 as select * from my_student as s lift join my_class c on s.c_id=c_id;  --id重复

create view v3 as select s.*,c.name,c.room from my_student as s lift join my_class c on s.c_id=c_id;  --视图基表有多张的时候，字段名不能重复

查看视图：查看视图的结构
视图是一张虚拟表，表的所有查看方式都适用于视图，但是视图与表有一个关键字的区别（table与view）
查看视图创建语句：show create view v3\G;   --G：旋转表
视图一旦创建，系统会在对应的数据库文件下自动创建一个.frm文件

使用视图：
使用视图主要是为了查询，将视图当成表一样查询即可
视图的执行的本质就是执行的封装的select语句

修改&删除视图：
视图本身不可修改，但是视图的来源可以修改
修改视图：修改视图本身的来源语句（select语句）
alter view 视图名字 as 新的select语句；
-> alter view v1 as select id,name,age,sex,height,c_id from my_student;

删除视图：drop view 视图名字；

视图的意义：
1.视图可以节省sql语句，将一条复杂的查询语句用视图进行保存，以后可以直接针对视图进行操作
2.数据安全，视图操作是针对查询的，如果对视图结果进行处理（删除），不会影响基表数据（相随安全）
3.视图往往是在大项目中使用，而且是在多系统中使用，可以对外提供有用的数据，但是隐藏关键（无用）的数据--数据安全
4.视图可以对外提供友好型，不同的视图提供不同的数据，对外好像是专门设计的
5.视图可以更好更好（更容易的）进行权限控制

视图数据操作：
视图是可以进行写操作，但是有很多限制
将数据直接在视图上操作

新增数据：
直接对视图进行新增：
多表视图不能进行新增数据；
可以向单表视图中插入数据，但是视图中包含的字段必须有基表中所有不能为空（或者没有默认值）的字段

删除数据：多表视图不能删除数据
delete from v1 where ?;

更新数据：无论单表视图还是多表视图都可以更新数据
update v3 set c_id=3 where id=5;
更新限制：with check option如果视图在更新数据的时候，限定了某个字段，那么视图在进行更新操作的时候，系统会进行验证，要保证更新之后，数据依然可以被查询出来，否则不让更新。

视图算法：
系统对视图及外部查询视图的select语句的一种解析方式
视图算法分为三种：
1.Undefined：这不是一种实际的算法，是一种推卸责任的算法，告诉系统视图没有定义算法，系统自己看着办
2.Temptable：临时表算法，系统应该先定义视图的select语句，后执行外部查询语句
3.Merge：系统应该先将视图对应的select语句与外部执行的select语句进行合并，然后执行
算法指定：在创建视图的时候指定算法：
create algorithm= 指定算法 view 视图名字 as select语句

数据备份与还原：
备份：数据表备份，单表数据备份，sql备份，增量备份

事务（transaction）：一系列要发生的连续的操作，要么都成功要么都不成功	---存储引擎必须为InnoDB
事务安全：一种保护连续操作同时满足（实现）的一种机制
事务安全的意义：保证数据操作的完整性

事务操作：自动事务（默认的），手动事务

手动事务：
1.开启事务，告诉系统以下操作不要直接操作，先放到事务日志
	start transaction；
2.进行事务操作
3.关闭事务：选择性的将日志文件中的操作结果保存到数据表（同步）或者直接清空事务
日志（原来操作全部清空）
	1.提交事务：同步数据表（操作成功）：commit；
	2.回滚事务：直接清空日志表（操作失败）：rollback；

事务原理：事务开启之后，所有的操作都会存到事务日志，事务日志只有在得到commit命令的时候才会同步，如果断开链接，临时事务日志自动清空
回滚点：在某一个成功的操作完成之后，后续的操作有可能成功也有可能失败，但不管成功还是失败还是成功，前面的操作都已经成功，可以在成功的位置设置一个回滚点，后续步骤操作失败的时候回到该点，而不是返回所有操作

设置回滚点：savepoint 回滚点名字
回到回滚点名字：rollback to 回滚点名字

自动事务：在mysql中，默认的都是自动事务，用户操作完成之后会立即同步到数据库中
系统通过autocommit变量控制
关闭自动提交：set sutocomm 
关闭自动提交之后，commit提交，rollback回滚；
注意：通常我们都会使用自动事务

事务特性：
1.a--atonic原子性：事务操作是一个整体，要么都执行要么都不执行
2.Consistency，一致性：事务操做的前后，数据表中的数据没有变化
3.Isolatio，隔离型，事务操作是相互隔离不受影响的
4.Durability，持久性，数据一旦提交不可改表，永久改变数据表中的数据

锁机制：InnoDB默认是航锁，但是如果在事务操作的过程中，没有使用到索引，那么系统会自动全表检索数据，自动升级为表锁
	 行锁：只有当前行被锁住
	 表锁：整张表都被锁住，别的用户不能操作

Sql变量：
系统变量：定义好的变量，我们只需要使用，大部分时候用户不需要使用系统变量，系统变量是用来控制服务器的表现的
查看系统变量：show variables；
查看具体变量值：select @@系统变量名  ->select @@version
修改系统变量：修改会话级别 set （@@）变量名 = value；
永久修改，对所有客户端都修改：set global 变量名 = value；

自定义变量：
定义变量：为了区分系统变量，规定用户自定义的变量必须使用一个@符号  set @变量名=value；
自定义变量也是类似系统变量查看 select @变量名；
在mysql中‘=’会默认的当作比较符号来使用，mysql为了区分比较和赋值，mysql重新定义了一个赋值符号：=； 如set @age：=18；
mysql允许从数据表中获取数据然后赋值给变量：
1.边赋值边查看结果 set @变量名：=字段名 from 数据源；从字段中取数据赋值给变量，如果使用‘=’会变成比较
2.只有赋值不看结果：数据记录最多只允许获取一条，mysql不支持数组
select 字段名 from 表名 where id=1 into @name，@age
所有自定义的变量都是会话级别，当次链接当次有效
所有自定义变量不区分数据库，用户级别

触发器：
事先为某张表绑定好一段代码，当表中的某些内容发生改变的时候，系统自动触发代码，执行
事件类型：增删改，三种类型insert，delete和update
触发时间：前后：before和after
触发对象：表中的每一条数据
一张表中只能拥有一种触发时间的一种类型的触发器，最多一张表中能有6个触发器

创建触发器：在mysql高级结构中没有{}和【】

触发器创建基本语法：
delimiter 自定义符号；后续代码中只有碰到自定义符号才算结束
create trigger 触发器名字 触发时间 事件类型 on 表名 for each row
begin    --代表左大括号：开始
		--里面就是触发器的内容，每行内容必须用语句结束符：；
end
	--语句结束符
	 自定义符号
	 
	 --将临时修正过来
	 delimiter；
	 

触发器--订单生成一个，库存商品少一个
临时修改语句结束符
delimiter $$
create trigger after_order after insert on order for each row
begin
	update my_goods set inv=inv-1 where id=2;
end
--结束触发器
$$
--修改临时语句结束符
delimiter ；

查看触发器：
show triggers； 查看所有触发器
查看触发器创建语句：
show create trigger 触发器名字

代码执行结构：顺序结构，分支结构，循环结构
分支结构：在mysql中只有if分支
语法：
if 条件判断 then
	--满足条件要执行的代码
else
	--不满足条件要执行的代码
	
触发器结合if分支：
delimiter %%
create trigger befor_order before insert on my_order for each row
begin
	--判断商品是否足够
	select inv from my_goods where id=new.g_id into @inv
	--比较库存
	if @inv <new.g_number then
	--库存不够；触发器没有提供一个阻止时间发生的能力（暴力报错）
		insert into xxx values(xxx);
	end if;
		
end
%%
delimiter ;

while循环：
语法：
whlie 条件：
	--满足条件要执行的语句
	--变更循环条件
end while；

循环控制：在循环内部进行判断和控制，在mysql中没有continue和break，但是有替代品
iterator：迭代
leave：离开

使用方式：
--定义循环名字
循环名字：whlie 条件 do
	--循环体
	--循环控制
	 leave/iterate 循环名字；
end while；

函数：





存储过程（procedure）：是一种处理数据的方式，存储过程是一种没有返回值的函数
创建过程：
create procedure 过程名字([参数名字])
begin
	--过程体
end

create procedure porl()
	select * from xxx;



函数的查看过程完全适用于所有过程
show procedure status[like 'por%']

调用过程：
过程没有返回值，select不能访问
过程有一个专门的调用关键字 call
call prol（）；

修改过程&删除过程：
过程只能先删除后新增；
drop procedure 过程名；

过程参数：
函数的参数需要数据类型指定，过程比函数还严格
过程有自己的类型限定：
in：数据只是从外部传入给内部使用，可以是值也可以是变量
out：只允许过程内部使用（不用外部数据），给外部使用的（引用传递，外部的数据先清空才会进入到内部），只能是变量
inout：外部可以在内部使用，内部修改也可以给外部使用，典型的引用传递，只能传变量

使用：
create procedure 过程名 (in 形参名字 数据类型，out 形参名字 数据类型，inout 形参名字 数据类型)
调用：
out和inout类型的参数必须传入变量而不是赋值













设置用户可以在本地访问mysql：grant all privileges on *.* to username@localhost identified by "password" ;
