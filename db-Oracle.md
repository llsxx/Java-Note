## 基本概念

### 数据库

> Oracle 数据库是数据的物理存储。 这就包括（数据文件 ORA 或者 DBF、控制文件、联机日志、参数文件）。
>
> 这里的数据库是一个操作系统只有**一个库**    

### 实例

> 一个 Oracle 实例（Oracle Instance）有一系列的后台进程（Backguound Processes)和内存结构（Memory Structures)组成。 
>
> **一个数据库可以有 n 个实例。**  

### 用户

> 用户是在实例下建立的。  

### 表空间

> 表空间是 Oracle 对物理数据库上相关数据文件（ORA 或者 DBF 文件） 的逻辑映射。
>
> 一个数据库在逻辑上被划分成一到若干个表空间，每个表空间包含了在逻辑上相关联的一组结构。
>
> 每个数据库至少有一个表空间(称之为 system 表空间)。  
>
> 
>
> 每个表空间由同一磁盘上的一个或多个文件组成，这些文件叫数据文件(datafile)。一个数据文件
> 只能属于一个表空间  

<img src="../java-note/image/image-20210128132419195.png" alt="image-20210128132419195" style="zoom:67%;" />

> 总结：一个数据库下可以建立多个表空间，一个表空间可以建立多个用户、一个用户下可以建立多个表。  

```sql
-- 创建表空间
create tablespace table1
datafile 'F:\table1.dbf'  -- 指定表空间对应的数据文件
size 100m  -- 初始大小
autoextend on  -- 自动增长 ，当表空间存储都占满时，自动增长
next 10m;  -- 一次自动增长的大小

-- 删除表空间
drop tablespace table1;
```

```sql
-- 创建用户
create user ox
identified by ox   -- 密码
default tablespace table1;  -- 表空间

-- oracle常用角色
-- connect-连接角色，基本角色
-- resource-开发者角色
-- dba-超级管理员角色
-- 给用户授予权限
grant dba to ox;
```

### SQL常用操作

增删改需手动提交

#### 删除表

```sql
----三个删除
--删除表中全部记录
delete from person;
--删除表结构
drop table person; .
--先删除表，再次创建表。效果等同于删除表中全部记录。
--在数据量大的情况下，尤其在表中带有索引的情况下，该操作效率高。
--索引可以提供查询效率，但是会影响增删改效率。
truncate table person;
```

#### 序列

```sql
----序列不真的属于任何一张表，但是可以逻辑和表做绑定。
----序列:默认从1开始，依次递增，主要用来给主键赋值使用。
----dual: 虚表，只是为了补全语法（查询语句必须有from），没有任何意义。
create sequence s_person;
select s_person.nextval from dual;
添加一条 记录
insert into person (pid,pname) values (s_person.nextval,'小明')；
commit;

```

#### scott用户

```sql
----scott用户， 密码tiger。
-- 解锁scott用户
alter user scott account unlock;
--解锁scott用户的密码[此句也可以用来重置密码]
alter user scott identified by tiger;
--切换到scott用户下
```

#### 视图

​	