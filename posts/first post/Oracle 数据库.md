## Oracle 用户管理
切换用户：conn sys as sysdba
创建用户： create user xiaoming identified by 0000;【 具有dba的权限可以创建用户 】
更改密码：password  用户名;   【 dba 可以更改任何用户的密码】 
自己改自己密码：直接password
删除用户：drop user --- 如果要删除的用户已经创建了表，需要带上“cascade”参数，删除用户会连带用户创建的表一起被删除 [级联删除]
## Oracale 权限管理
Oracle中大概有140多种权限。
权限分为两种：**系统权限**(描述了用户对数据库的相关权限)和对象权限(25个)用户对其他用户的数据对象操作的权限)
数据对象：用户创建的存储过程、表、视图、触发器......
角色：事先定义一些角色，将一些权限批量赋给角色；
角色分两种：自定义角色和预定义角色【connect，dba，resource(在任意表空间建表)】 
授权：grant connect to xiaoming;
注：新建的xiaoming用户是没有权限创建表的，先要赋给xiaoming角色 grant resource to xiaoming;
对象权限，如：select,insert,update,delete,all,create,index
希望xiaoming用户可以查询scott的emp表，只允许查询不允许修改：grant select on emp to xiaoming;
方案：
//数据的组织方式是以用户为单位的，在一个实例中可以用两张同名的表；
收回权限：revoke

xiaoming用户可以查询emp表还让小明可以把这个权限继续给别人
对权限的维护：grant select on emp to xiaoming with grant option;

sys给xiaoming用户权限时还让小明可以把这种权限继续给别人(权限传递)
grant connect to xiaoming with admin option

如果scott把xiaoming对emp的查询权限回收，那么xiaohong会怎样？
xiaohong也会被回收权限
## profile
profile管理用户口令：profile是口令限制，资源限制的命令集合，当建立数据库时，oracle会自动建立名称为default的profile。当建立用户没有指定profile选项，那oracle就会将default分配给用户。
### 账户锁定
指定改账户登录时最多可以输入密码的次数，也可以指定用户锁定的时间(天)一般用dba的身份去执行该命令。
eg.指定scott这个用户最多只能尝试3次登录，锁定时间为2天
实现： 创建profile文件
create profile lock_account limit failed_login_attempts 3 password_lock_time 2;
alter user scott profile lock_account;
### 账户解锁
alter user scott account unlock;
### 终止口令
create profile myprofile limit password_life_time 10 password_grace_time 2;
alter user scott profile myprofile;
### 口令历史
### 用户在修改密码时，不能使用以前使用过的密码。
create profile myprofile limit password_life_time 10 password_grace_time 2 password_reuse_time 10;
// password_reuse_time 指定口令可重用时间即10天后就可以重用 
### 删除profile
drop profile password_history 【cascade】【级联】

Oracle中日期类型默认是'18-10月-1997' ： 更改SQL命令alter SESSION set nls_date_format='yyyy-mm-dd';

### 删除表中数据 
delete from student
## 简单查询
select 大小不区分，里面的内容大小写区分
### EMP表
![图片](https://uploader.shimo.im/f/cJBPDZOC5bo0RMfq.png!thumbnail)

### DEPT表
![图片](https://uploader.shimo.im/f/c2w8IcrntHcldFmq.png!thumbnail)

### SALGRADE表
![图片](https://uploader.shimo.im/f/3sPZYQcfRgUPnKrg.png!thumbnail)

### like 操作符
```
select ename,sal from emp where ename like 'S%';
```

第三个字母为大写O的员工姓名和工资
```
select ename,sal from emp where ename like '__O%';
```
### where 中 in
```
select * from emp where empno in(7844,234,456);

select * from emp where (sal > 500 or job = 'MANAGER') and ename like 'J%';
```

### is null 操作符
```
select * from emp where mgr is null; 
```

### order by 子句
 默认的是按从低到高排序
```
select * from emp sal order by sal;
```
desc 按从高到低排序
```
select * from emp sal order by sal desc;
```
查询emp表的结果按部门号升序并且工资降序排列
```
select * from emp ORDER BY deptno,sal desc;
```
使用列的别名排序，按照年薪排
```
select ename,(sal+nvl(comm,0))*12 "年薪" from emp ORDER BY "年薪";
```

## 复杂查询
~~复杂查询 子查询多表查询是难点~~
~~oracle的分页是最难的...~~

--max(sal) 列中最大的

如何显示所有员工中最高工资和最低工资
```
select max(sal),min(sal) from emp;
```
如何显示所有员工中最高工资和最低工资，并把最高工资的名字打出来
```
select ename,sal from emp where sal=(select max(sal) from emp);
```
计算总共有多少员工
```
select count(*)  from emp;
```
请显示工资高于平均工资的员工信息
```
select * from emp where sal > (select avg(sal) from emp);
```

### GROUP BY 对查询的结果进行分组统计
如何显示每个部门的平均工资和最高工资
```
select avg(sal),max(sal),deptno from emp group by deptno;
```
显示每个部门的每种岗位的平均工资和最高工资
```
select avg(sal),max(sal),deptno,job from emp group by deptno,job;
```

###  having子句用于限制分组显示结果
显示平均工资低于2000的部门号和它的平均工资
```
select avg(sal),deptno from emp GROUP BY deptno having avg(sal) < 2000;
```

###  对数据分组的总结
1. 分组函数只能出现在选择列表、having、order by子句中。
2. 如果在select语句中同时包含有group by、having 、order by。那它们的顺序一定是：group by, having,order by
```
select avg(sal),deptno from emp GROUP BY deptno having avg(sal)>2000 order by deptno;
```
3. 在选择列中如果有列、表达式和分组函数，那么这些列和表达式必须有一个出现在group by子句中，否则就会出错

     如select deptno,avg(sal),max(sal) from emp group by deptno having avg(sal) < 2000；这里deptno就一定要出现在group by中

## 多表查询
多表查询的条件是 至少不能少于表的个数-1【避免笛卡尔集的出现】
eg.
显示雇员名，雇员工资及所在部门的名字
```
select a1.ename,a1.sal,a2.dname from emp a1,dept a2  where a1.DEPTNO=a2.DEPTNO;
```

如何显示部门号为10的部门名、员工名和工资
```
select a2.DNAME,a1.ENAME,a1.SAL from emp a1,dept a2 where a1.DEPTNO=a2.DEPTNO and a1.DEPTNO=10;
```

显示各个员工的姓名，工资，及其工资的级别

```
select * from SALGRADE;
```

```
select a1.ENAME,a1.SAL,a2.GRADE from emp a1,SALGRADE a2 where a1.SAL BETWEEN a2.LOSAL and a2.HISAL;
```

显示雇员名，雇员工资及所在部门的名字，并按部门排序
```
select a1.ENAME,a1.SAL,a2.dname from emp a1,dept a2 where a1.DEPTNO=a2.DEPTNO ORDER BY a2.DNAME;
```

### 自连接
把一张表当作两张相同的表
显示FORD的上级
```
select worker.ENAME ,boss.ENAME from emp worker,emp boss where worker.MGR = boss.EMPNO and worker.ENAME='FORD';
```

## SQL 99 语法标准
```
SELECT [DISTINCT] *| 列[别名]
FROM 表名称1
    [CROSS JOIN 表名称2]
    [NATURAL JOIN 表名称2]
    [JOIN 表名称2 ON(条件)|USING(字段)]
    [LEFT|RIGHT|FULL OUTER JOIN 表名称2 ON(条件)|USING(字段)]
```
1.交叉连接 cross join，主要的功能是产生笛卡尔积，简单是实现多表查询
select * from emp cross join dept;
2.自然连接 natural join，自动使用关联字段消除笛卡尔积，属于内连接的概念
select * from emp natural join dept; 
在返回查询结果的时候，默认情况下会将关联字段设置在第一列上，重复的内容列不显示
3.using 子句：如果说现在要一张表里面有多个关联字段存在，那么可以使用using子句指定一个字段
select * from emp join dept using(deptno)；
1. ON 子句：如果现在没有关联字段，则可以使用on子句设置条件：

select * from emp e join salgrade s on(e.sal between s.losal and s.hisal);
## 子查询
### 单行子查询
(子查询语句)只返回一行数据的子查询语句
如何显示与SMITH同一部门的所有员工
```
select ename,deptno from emp where deptno= (select deptno from emp where ename = 'SMITH');
```

### 多行子查询
 (子查询语句)返回多行数据的子查询语句

--如何查询和部门10号的工作相同的雇员的名字、岗位、工资、部门号
--子查询返回的数据不是一行时，要用in关键字
```
select ename,job,sal,deptno from emp where job in (select job from emp where deptno=10);
```

### 在多行子查询中使用all操作符
-如何显示工资比部门30的所有员工的工资高的员工的姓名、工资和部门号?
-- 第一种方法
```
select ename,sal,deptno from emp where sal > all (select sal from emp where deptno = 30);
```
--第二种方法
```
select ename,sal,deptno from emp where sal > (select max(sal) from emp where deptno=30);
```
### any 操作符
```
select ename,sal,deptno from emp where sal > any (select sal from emp where deptno = 30);
```
### 多列子查询
找出与SMITH相同部门相同岗位的人
```
select * from emp where (job,deptno)=(select job,deptno from emp where ename='SMITH');
```
### 在from子句中使用子查询 
--如何显示高于自己部门平均工资的员工信息
-- 1.先算出自己部门的平均工资
```
select a2.ename,a2.sal,a2.deptno,avg_sal from emp a2,(select deptno,avg(sal) avg_sal from emp GROUP BY deptno) a1 where a2.deptno=a1.deptno and a2.sal > a1.avg_sal;
```
### 对在from子句中使用子查询的总结
这里需要说明：当在from子句中使用子查询时，该子查询会被作为一个视图来对待，因此叫作内嵌式图，当在from子句中使用子查询时，必须给子查询指定别名。
## 分页查询
oracle数据库中需要通过多次子查询来完成分页
```
select * from (select a1.*,rownum rn from (select * from emp) a1 where rownum<=10) where rn>=6;
```

在需要只查询部分字段时，只需更改最里面的内嵌视图的选择列即可(select * from emp)，不管是排序还什么操作。
```
select * from (select a1.*,rownum rn from (select * from emp order by sal) a1 where rownum<=10) where rn>=6;
```

### Oracle三种分页方式对比
![图片](https://uploader.shimo.im/f/YLARn1PunY4FaTyk.png!thumbnail)

## 合并查询
有时在实际应用中，为了合并多个select语句的结果，可以使用集合操作符号 
union,union all,intersect,minus
### union
该操作符用于取得两个结果集的并集。当使用该操作符时会自动去掉结果集中重复行。 
```
select ename,sal,job from emp where sal>2500 union select ename,sal,job from emp where job='MANAGER';
```
返回结果：
![图片](https://uploader.shimo.im/f/5B5Gg5EvF6krZqV1.png!thumbnail)
### union all
该操作符与union相似，但是它不会取消重复行，而且不会排序。 
```
select ename,sal,job from emp where sal>2500 union all select ename,sal,job from emp where job='MANAGER';
```
## 小技巧
### to_date
to_date('1987-12-01','yyyy-mm-dd');

### 使用子查询插入数据
当使用values子句时，一次能插入一行数据，当使用子查询插入数据时，一条values语句可以插入大量的数据。当处理行迁移或者装载外部表的数据到数据库时，可以使用子查询来插入数据。
eg.
```
create table emp_temp (
	id number(8),
	rname varchar2(8),
	rno number(5)
);
```

```
insert into EMP_TEMP (id,rname,rno)
select empno,ename,deptno from emp  where
deptno=10;
```

### 使用子查询更新数据
使用update语句更新数据时，既可以使用表达式或者数值直接修改数据，也可以使用子查询修改数据。
希望员工scott的岗位、工资、奖金与SMITH员工一样
```
update emp set (job,sal,comm) = (select job,sal,comm from emp where ename='SMITH') where ename = 'SCOTT';
```
## Oracle事务
### 概念
事务用于保证数据的一致性，它由一组相关的DML语句(增、删、改)组成，该组DML语句要么全部成功，要么全部失败。
如：网上转账就是典型的要用事务来处理的案例，用以保证数据的一致性。
### 事务和锁
当执行事务操作时(DML语句)，oralce会在被作用的表上加锁，防止其它用户改变表的结构，这里对我们用户来讲是非常重要的。
### 提交事务
使用commit语句可以提交事务，当执行了commit语句后，会确认事务的变化、结束事务、删除保存点、释放锁，当使用commit语句结束事务后，其它会话将可以查看到事务变化后的新数据。
### 只读事务
只读事务是指只允许执行查询的操作，而不允许执行任何其它DML操作的事务，使用只读事务可以确保用户只能取得某时间点的数据。假定机票代售点每天18点开始统计今天的销售情况，这时可以使用只读事务，在设置了只读事务后，尽管其它会话可能会提交新的事务，但是只读事务将不会取得最新数据的变化，从而可以保证取得特定时间点的数据信息。
```
set transaction read only;
```

## Oracle sql函数
### 字符函数
eg.列出emp表中员工的姓名，首字母大写，其余小写

substr(ename,1,2) ---- (要截取的字段,从哪个字符开始，一共截几个字母) 

1. 首字母大写
```
 select Upper(substr(ename, 1, 1)) from emp;
```
2. 其余字母小写
```
select LOWER(substr(ename, 2, length(ename)-1)) from emp;
```
 3.  合并
```
select Upper(substr(ename, 1, 1)) || LOWER(substr(ename, 2, length(ename)-1)) from emp;
```

### 系统函数

