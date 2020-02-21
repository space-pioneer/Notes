**sql语句后加分号**

## 操作

### 插入

1. 插入单值

   - insert into 表名(列名) values(值)

   - insert into 表名 value(值)

   - insert into (select 语句) values(值)

   - 利用临时表 

     insert into 表名(列名) select语句

2. 利用子查询插入数据(批量)

   insert into 表名 select语句

   insert into mysql select 1100,job from emp where empno=7788

   针对大量数据，最好采用不写入日志文件中，这样可以提高执行速率， “/*+APPEND*/”

   ```sql
   create table myemp as select empno,job from emp where 1=2
   select * from myemp
   insert /*+APPEND*/ into myemp(empno,job) select empno,job from emp
   commit
   ```

3. 向多个表中插入数据

   - 无条件多表插入

     ```sql
     insert ALL
     into 表1 values(列)
     into 表2 values(列)
     select语句
     ```

   - 有条件多表插入

     ```sql
     insert all(两个表都插[多条件满足])/first(只插入第一个表)
     when 条件1 then into 表1(列)
     when 条件2 then into 表2(列)
     else into 表3
     select语句
     ```

   - 多表插入的应用

     ```sql
     列转换成行
     insert all
     into sale_info values(emp_id, week_id,sal_mon)
     into sale_info values(emp_id,week_id,sal_tus)
     into sale_info values(emp_id,week_id,sal_wed)
     select语句
     ```

### 修改

#### 全部修改

```sql
update 表名 set 列名=值，列名=值
```



#### 局部修改

```sql
update 表名 set 列名=值,列名=值 where 条件
临时表：
update myemp set(ename,sal) =('ella',7000 from dual) where empno=7000
将30号部门的员工工资修改为部门20员工工资加300
update myemp set sal=300+(select avg(sal) from myemp where deptno=20) where deptno=30
```



### 删除

#### 全部删除

delete (from) 表名

#### 局部删除

delete from 表名 where 条件

```sql
删除和7788岗位相同的员工所有信息
delete from myemp where job=(select job from myemp where empno=7788 )
```



### 复制表

create table 表名 as  数据来源(select语句)



数据操作增删改查后必须加上事务处理操作commit

truncate table 表名        删除并提交(关系型数据库中‘table’不能省略，其它数据库可以省略)



rownum：表示行号，但它是一个伪列

一般用来做分页的操作

```sql
select * from emp where rownum<=5

如果是5-10行
select * from where rownum>=5 and rownum<=10  （未选定行）
select * from where rownum>=5(未选定行，伪列，作为条件只能作为小的那一方)

用临时表来
selct * from (select rownum m,empno from emp where rownum<=10) temp
where temp.m>5
```



格式化查询结果：

- 格式化列

  只针对sqlplus

  column命令(heading[设置列标题],justify[格式],clear,format[定义显示的格式])

  ```sql
  colunm ename heading 员工名 justify center format a7(a：代表宽度，7代表字符)
  ```

  column 列名      显示格式

  clear column 清除所有列

- 压缩列(重复列值的显示)

  break on 列名 (skip 值)

  ```mysql
  break on deptno skip 1(分组间距)
  select deptno,ename from emp order by deptno
  ```

  实现组函数的分组统计操作

  compute 组函数 label 标签内容 of 列 on 分组条件

  ```mysql
  break on deptno
  compute avg label 'avg sal' of sal on deptno
  select deptno,ename from emp order by deptno
  ```

- 设置标题和页脚

  ttitle(标题) btitle(页脚)

  ```mysql
  ttitle center 'emplyees'
  btitle right 'made by you'
  select语句
  
  ttitle
  btitle
  显示样式
  
  关闭标题和页脚
  ttitle off
  btitle off
  ```

### 层次查询

select (level), 列名

from 表名

(where 条件)

(start with 列名=值)

(connect by prior 关系)

```mysql
select empno,ename,mgr from emp
start with empno=78
connect by prior empno=mgr

select lpad(' ',5*level-1)||empno EMPNO,lpad(' ',5*level-1)||ename ENAME
from emp
start with empno=78
connect by prior empno=mgr
```

### 表的操作

#### 建表

建表前先删表

char

varchae2

date

number (6)

number (6,2)

clob

blob

```mysql
create table 表名(
	列1 数据类型 [default 默认值],
);
create table student(
	sname varchar2(10),
    sid varchar2(16),
);	
```

#### 删除表

```mysql
把表放在回收站里
drop table 表名;    

从回收站里恢复表(恢复最近的，不能全部恢复)
flashback table 表名 to before drop;

彻底删除
drop table 表名 purge;

从回收站里删除表
purge table 表名;

查看回收站里的表
select object_name,original_name,operation,type from recyclebin

清空回收站
purge recyclebin
```

#### 修改表

1. 增加列

   ```mysql
   alter table 表名 add(列名 数据类型 默认值, 列名 数据类型 默认值)
   ```

2. 修改列的数据类型和长度

   如果有数据，长度改小时是失败的

   ```mysql
   alter table 表名 modify(列名 数据类型 默认值, 列名 数据类型 默认值)
   ```

3. 修改列的名字

   ```mysql
   alter table 表名 rename colnum 原 to 改
   ```

4. 删除列

   ```mysql
   alter table 表名 drop column 列名
   ```

5. 修改列的无效状态，效果等同于删除

   ```mysql
   alter table 表名 set unused column 列名
   ```

6. 删除无效列

   ```mysql
   alter table 表名 drop unused columns;
   ```

#### 表的重命名

```mysql
rename 原 to 改
```

### 约束

对表的强制规定

分类：(只有非空约束只能列级约束，其它都有列级约束和表级约束)，建议采用表级约束

- 主键约束	不能重复，不能为空，可以是多个

  ```mysql
  primary key   列级别约束
  constraint person_pid_pk primary key(pid)  表级别约束
  自定义错误提示
  ```

- 唯一约束        不能重复

  ```mysql
  unique
  constraint person_name_uk unique(name)
  ```

- 非空约束        不能为空，只有列级别，没有表级别

  ```mysql
  not null
  ```

- 检查约束	是否合法

  ```mysql
  check(age between 1 and 150)
  check(sex in('男'，'女'))
  
  constraint person_sex_ck check(sex in('男','女'))
  ```

- 外键约束        多表关联(其它表的主键)

  ```mysql
  deptno number(3) references department(deptno)
  
  constraint employee_deptno_fk foreign key(deptno) references department(deptno)
  ```

追加约束

```mysql
alter table 表名 add 约束
表约束可以，非空约束不能
若要添加非空约束，只能通过修改列
alter table person add primary key(pid);
```

删除约束

```mysql
alter table 表名 drop 约束
删除列级约束
alter table person drop primary key
删除表级约束
alter table person drop constraint person_pid_pk;
删除非空约束采用修改列的方式
```

约束的禁用和启动

```mysql
禁用
alter table 表名 disable 约束
alter table person disable primary key
alter table person disable constraint person_pid_pk

启动
alter table 表名 enable 约束
```

查看约束

```mysql
select * from user_constraints where table_name='emp'
```



## 视图和同义词

### 视图

来自一个或多个表上的数据集合

```mysql
create view 视图名 as select语句
creat view v_myemp as select empno eid from scott.emp

create [or replace] [force unforce] view 视图名 
as 
select语句 
[with check option 约束]
[with read only]
```

对表的操作同样适用于视图

```mysql
查看表的结构
desc 表名
```

```mysql
删除视图
drop view 视图名
```

```mysql
查看视图
select * from user_views;
```

### 同义词

只有管理员才有创建和删除同义词的权限

```mysql
create synonym 同义词名 for 对象名
create synonym s_emp for scott.emp

drop synonym 同义词
drop synonym s_emp
```



## 序列和索引

### 序列

按照一定的规则能够自动增加减少数字的一种数据库对象

#### 创建序列

```mysql
N代表数字
create sequence 序列名
[increment by N]
[start with N]
[maxvalue N]
[minvalue N]
[cycle nocycle]
[cache N nocache]  默认内存是20
```

```mysql
创建序列test_seq 初始值10，每次增长2，最大值100.最小值9，循环序列，每次存储10
create sequence test_seq
start with 10
increment by 2
maxvalue 100
minvalue 9
cycle
cache 10
```

#### 伪列

currval 表示序列返回的当前值

nextval 表示序列返回的下一个值

在currval在被引用之前必须先使用nextsal来产生一个序列值

#### 修改序列

```mysql
alter sequence 序列名
[increment by N]
[start with N]
[maxvalue N]
[minvalue N]
[cycle nocycle]
[cache N nocache] 
```

#### 删除序列

```mysql
drop sequence 序列名;
```

#### ROWID

伪列，系统自动产生，rowid能唯一标识出每一条数据库记录的物理地址，通过rowid可以快速定位到一条行记录

数据对象编号(OOOOOO) 相关文件编号(FFF) 块编号(BBBBBB) 行编号(RRR)



### 索引

可以加速对表的查询速率

- 单类索引    索引建立在一列上
- 复合索引    索引建立在某几列上

#### 创建索引

- 自动创建

  当在建表时，使用了primary key或者unique约束时，数据库自动创建一个索引

- 手工创建

  ```mysql
  create index 索引名
  on 表名(列名,列名)
  
  索引名的命名规范：idx_表名_列名
  
  create index idx_emp_deptnojob
  on emp(deptno,job);
  ```

单行函数，频繁修改数据，DML不建议使用索引

#### 删除索引

```
drop index 索引名;
```



## 用户与权限授予

### Oracle数据库的初始用户

- sys 超级管理员，权限最大，可以启动、修改、关闭数据库
- system 一般管理员(辅助管理员)，不可以启动和关闭数据库，主要进行一些管理操作：创建用户、删除用户等
- scott 用来测试网络连接
- public 本质上是一个用户组，数据库中的任何一个用户都属于该组，因为要为数据库中的每个用户授权，则可以把授权直接授予public组

### 用户属性

- 身份认证
  1. 数据库身份认证 账号密码保存在数据库中
  2. 外部身份认证 账号数据库保存，密码web保存
  3. 全局身份认证 
- 默认表空间
- 临时表空间
- 表空间配额
- **概要文件**
- 账号状态 加密解密/密码失效

### 创建用户

```mysql
create user 用户名 identified
[by 密码 externally|globally as 'external_name']
[default tablespace tablespace_name]
[temporary tablespace temp_tablespace_name]
[quote n K|M|UNLIMITED on tablespace_name]
[profile profile_name]
[password expire]
[account lock|unlock]
```

```mysql
创建一个用户user3,口令为user3,默认表空间为users,该表空间配额10MB,初始状态为锁定
create user user3 identified by user3
default tablespace users quota 10M on users account lock;
```

### 修改账号

```mysql
alter user 用户名 [identidied] 
[by 密码 externally|globally as 'external_name']
[default tablespace tablespace_name]
[temporary tablespace temp_tablespace_name]
[quote n K|M|UNLIMITED on tablespace_name]
[profile profile_name]
[default ROLE role_list|all [except role_list] none]
[password expire]
[account lock|unlock]
```

```mysql
修改user3的密码为newuser3，同时将该账号解锁
alter user user3 identified by newuser3 account unlock;
```

```mysql
修改用户的默认空间为orcltbs1，该表空间配额为20MB，在users表空间的配额为5MB
alter user user3 default tablespace orcltbs1 QUOTA 20M on orcltbs1 QUOTA 5M on users;
```

### 删除用户

```mysql
drop user 用户名 [casacde]
```

### 查询用户

```mysql
select * from 关键词
```

- ALL_USERS：包含数据库所有用户的用户名、用户ID和创建时间
- DBA_USERS: 包含数据库所有用户的详细信息
- USER_USERS： 包含当前用户的详细信息
- DBA_TS_QUOTAS：包含所有用户的表空间、配额信息
- USERS_TS_QUOTA：包含当前用户的表空间、配额信息
- V$SESSION：包含用户会话信息
- V$OPEN_CURSOR：包含用户执行的sql语句信息

### 权限管理

- 对象权限：访问或操作数据库对象，增删改查

  1. 对象权限的授予

     ```mysql
     grant 对象权限列表|all on 对象 to 用户列表|角色列表 [with grant option];
     
     grant select,update,insert on scott.emp to user1;
     
     将emp表的select授予user2，user2再将emp表的select授予user3;
     grant select on scott.emp to user2 with grant option;
     grant select on scott.emp to user3;
     ```

  2. 对象权限回收

     ```mysql
     revoke 对象权限列表|all on 对象 from 用户列表|角色列表;
     
     revoke select on scott.emp from user2;
     ```

     

- 系统权限：许可对各种特性的访问或许可Oracle数据库中的特定任务，建表删表建视图等

  1. 注意事项

     - 只有DBA才能拥有ALTER DATABASE系统权限
     - 应用程序开发者一般需要拥有的是CREATE TABLE, CREATE VIEW, CREATE INDEX等系统权限
     - 普通用户一般只具有CREATE SESSION系统权限
     - 只有授权时带有WITH ADMIN OPTION子句时，用户才可以将获得的系统权限再授予给其它用户(系统权限的传递性)

  2. 语法格式

     ```
     GRANT 系统权限 TO 用户|角色|PUBLIC [WITH ADMIN OPTION]
     
     如果授予多个系统权限，之间用逗号分隔
     如果授予多个用户，之间用逗号分隔
     ```

     ```
     权限的传递性
     GRANT CREATE TABLE,CREATE VIEW TO user3;
     GRANT CREATE SESSION,CREATE TABLE VIEW TO user2 WITH ADMIN OPTION;
     GRANT CREATE TABLE TO user1;
     ```

  3. 权限回收

     ```mysql
     revoke 系统权限 from 用户列表|角色列表|PUBLIC;
     
     revoke create table,create view from user1;
     ```

- 查询权限信息

  DBA_TAB_PRIVS 授权信息    select * from DBA_TAB_PRIVS

  ALL_TAB_PRIVS   对象授权信息

  USER_TAB_PRIVS 当前用户

  DBA_COL_PRIVS  

  ALL_COL_PRIVS

  USER_COL_PRIVS  

  DBA_SYS_PRIVS 系统权限信息

  USER_SYS_PRIVS   当前用户的系统权限信息

### 角色管理

角色：一系列相关权限的集合

#### 角色分类

- 系统预定义角色

  ```mysql
  select * from DBA_ROLES;
  ```

- 用户自定义角色

  1. 创建角色

     ```mysql
     create ROLE 角色名称 [NOT IDENTIFIED] [IDENTIFIED BY 密码]
     create role high_tiger_role IDENTIFIED BY high;
     ```

  2. 角色权限的授予与回收

     ```mysql
     grant connect,create table,create view to low_tiger_role;
     grant select on scott.emp to high_tiger_role;
     revoke connect from low_tiger_role;
     ```

     一个角色可以被授权另外一个角色，但不能授予其本身，不能产生循环授权

  3. 修改角色

     生效或失效

     ```mysql
     alter ROLE 角色名称 [NOT IDENTIFIED] [IDENTIFIED BY 密码]
     
     为high_tiger_role 角色添加口令,取消middle_tiger_role的角色口令
     alter role high_tiger_role IDENTIFIED BY highrole;
     alter role middle_tiger_role NO IDENTIFIED;
     ```

     修改角色必须具有alter any role系统权限，以及with admin option 权限；如果时角色的创建者，自动具有修改角色的修改权限

  4. 角色的生效失效

     ```mysql
     set ROLE [角色名称 [IDENTIFIED BY 密码]] [ALL [EXCEPT 角色名称]] [NONE];
     
     设置所有角色失效
     set ROLE NONE
     设置某一个角色生效
     set ROLE high_tiger_role IDENTIFIED BY highrole;
     除了某个角色之外，其它角色都生效
     set ROLE ALL EXCEPT low_tiger_role;
     ```

  5. 删除角色

     ```mysql
     drop role 角色名称;
     drop role low_tiger_role;
     ```

#### 利用角色进行权限管理

- 给用户或角色授予角色

  ```
  grant 角色列表 to 用户列表|角色列表;
  grant connect,high_tiger_role to user1;
  grant resource,connect to middle_tiger_role;
  ```

- 从用户或角色回收角色

  ```
  revoke 角色列表 from 用户列表|角色列表;
  revoke resource,connect from middle_tiger_role;
  ```

- 用户角色的激活或屏蔽

  ```
  alter user 用户名 default role [角色名称] [ALL [except 角色列表]] [NONE];
  屏蔽user1所有角色
  alter user user1 default role none;
  ```

- 查询角色信息

  DBA_ROLES

  DBA_ROLE_PRIVS

  USER_ROLE_PRIVS

  ROLE_ROLE_PRIVS

  ROEL_SYS_PRIVS

  ROLE_TAB_PRIVS

  SESSION_PRIVS

  SESSION_ROLES

## 概要文件

概要文件是数据库和系统资源限制的集合，是Oracle数据库安全策略的重要组成部分。利用概要文件，可以限制用户对数据库和系统资源的使用，同时还可以对用户口令进行管理

每个数据库用户必须用一个概要文件。通常DBA将用户分为几种类型，然后每种类型的用户创建一个概要文件。

### 系统资源的限制包括

- CPU使用时间
- 每个用户的并发会话数
- 用户连接数据库的时间
- 用户连接数据库的空闲时间
- 是有SQL区和PL SQL区的使用

### 创建概要文件

```
create profile 概要文件名称 limit 系统资源限制参数|口令管理参数;

创建一个名为pwd_profile的概要文件，用户连续4次登陆失败，该账号被锁定，10天后解锁
create profile pwd_file limit failed_login_attempts 4 password_lock_time 10;
创建一个名为res_profile的概要文件，每个用户最多可以创建4个并发会话，每个会话最多持续60分钟；若会话连续20分钟内空闲，则结束会话；每个SQL区大小为100KB，每个SQL语句占用的CPU时间总量不超过10秒
create profile res_profile limit
session_per_user 4 connect_time 60 idle_time 20 private_sga 100k CPU_PER_CALL 1000;
```

### 将概要文件分配给用户

- 在创建用户的时候指定概要文件

  ```
  create user user5 IDENTIFIED BY user5 PROFILE res_profile;
  ```

- 为用户指定概要文件

  ```
  alter user user5 profile pwd_profile;
  ```

### 删除概要文件

```
drop profile 概要文件名称 cascade;
```

