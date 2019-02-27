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


设置用户可以在本地访问mysql：grant all privileges on *.* to username@localhost identified by "password" ;
