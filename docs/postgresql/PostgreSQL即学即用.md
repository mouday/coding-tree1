# 《PostgreSQL即学即用》学习笔记

## 配置文件

- postgresql.conf 默认设置
- pg_hba.conf 用户连接方式
- postgresql.auto.conf `ALTER SYSTEM`修改后记录在这个文件

```sql
select name, setting from pg_settings where category = 'File Locations';
       name        |    setting
-------------------+-------------------------
 config_file       | /Users/wang/local/postgres-data/postgresql.conf
 data_directory    | /Users/wang/local/postgres-data
 external_pid_file |
 hba_file          | /Users/wang/local/postgres-data/pg_hba.conf
 ident_file        | /Users/wang/local/postgres-data/pg_ident.conf
(5 rows)
```

使用命令行执行语句

```shell
psql db_test -c 'select version();'
```

postgresql.conf

配置项

```sql
select
name,
-- context:
-- postmaster 重启生效
-- user 重新加载生效
context,
-- 单位
unit,
-- 当前值
setting,
-- 默认值
boot_val,
-- 重启服务器或者重新加载设置之后新设置值
reset_val
from pg_settings

where name in(
    'listen_addresses',
    'port',
    'max_connections',
    'shared_buffers',
    'effective_cache_size',
    'maintenance_work_mem',
    'work_mem'
) order by context, name;

 name   |  context   | unit |  setting  | boot_val  | reset_val
----+------------+------+-----------+-----------+-----------
 listen_addresses     | postmaster |      | localhost | localhost | localhost
 max_connections      | postmaster |      | 100       | 100       | 100
 shared_buffers       | postmaster | 8kB  | 16384     | 16384     | 16384
 effective_cache_size | user       | 8kB  | 524288    | 524288    | 524288
 maintenance_work_mem | user       | kB   | 65536     | 65536     | 65536
 work_mem             | user       | kB   | 4096      | 4096      | 4096
(6 rows)
```

常见配置项

| 设置                 | 说明                                                   |
| -------------------- | ------------------------------------------------------ |
| listen_addresses     | 监听IP地址：localhost或`*`                             |
| port                 | 监听端口，默认5432 环境变量：PGPORT                    |
| max_connections      | 最大并发连接数                                         |
| shared_buffers       | 缓存最近访问过的数据页内存区大小，`系统内存*25%`-`8GB` |
| effective_cache_size | 一个查询执行过程中最大缓存                             |
| maintenance_work_mem | 用于vaccum操作的内存总量                               |
| work_mem             | 最大内存量                                             |

更改配置

```sql
ALTER SYSTEM SET work_mem = 8192;

-- 重新加载设置
SELECT pg_reload_conf();
```

## pg_hba.conf

自上而下匹配规则

常见：

- trust
- reject
- md5
- password
- ident
- peer

重新加载

```shell
# 方式1：
pg_ctl reload -D /Users/wang/local/postgres-data/

# 方式2：如果以服务形式安装
service postgres reload

# 方式3：
SELECT pg_reload_conf();
```

## 连接管理

```sql
-- 模拟耗时查询
select pg_sleep(10000);

-- 查询活动链接
select * from pg_stat_activity where datname = 'db_test' ;

-- 取消查询pid
select pg_cancel_backend(35892);

-- 终止链接
select pg_terminate_backend(35892);

-- 终止多个连接
select pg_terminate_backend(pid) from pg_stat_activity where datname = 'db_test' ;
```

## 角色

- 用户 `create user`
- 组 `create group`
- 角色 `create role` (推荐)
  - 成员角色 member role
  - 组角色 group role

创建角色

```sql
-- 创建登录用户
create role leo login password 'king' createdb valid until 'infinity';
\x
select * from pg_roles where rolname = 'leo';

-- 创建超级用户
create role regina login password 'queen' superuser valid until 'infinity';
\x
select * from pg_roles where rolname = 'regina';
```

组角色

```sql
-- 创建组角色 超级用户权限无法被继承
create role royalty inherit;
\x
select * from pg_roles where rolname = 'royalty';

-- 将组角色的权限授权给成员角色
grant royalty to leo;
grant royalty to regina;
```

会话级授权

```sql
-- 成员角色授予父角色身份
set role royalty;

-- superuser用户设置为任意角色身份
set session authorization royalty;
```

查看用户身份

```sql
-- 登录用户身份
select session_user;

-- 当前用户身份
select current_user;
```

区别

| 设置角色  | session_user | current_user |
| ------------------ | ------------ | ------------ |
| psql登录用户 tom                                 | tom          | tom          |
| `set role royalty`                  | tom          | royalty      |
| `set session authorization royalty` | royalty      | royalty      |

## 创建database

```sql
-- 默认模板：template1
create database db_test2;

-- 显示指定模板：template1
create database db_test3 template template1;
```

- template0 系统模板，不建议修改
- template1 默认模板
 
```sql
-- 标记为模板数据库，禁止编辑或删除
update pg_database set datistemplate = true where datname = 'db_test3';

\x
select * from pg_database where datname='db_test3';
```

## schema

逻辑分组

```sql
-- 当前登录用户名
select user;
```

技巧：角色和schema同名

```sql
-- 搜索顺序
show search_path;
 "$user", public
```

将扩展的schema加入搜索路径
```sql
alter database db_test set search_path="$user", public, extension_name;
```

## 权限管理

```sql
-- 创建用户
create role db_admin login password 'passwd';
-- 创建数据库
create database db_admin with owner = 'db_admin';
```

授权
```sql
grant some_privilege to some_role;
```

示例
```sql
grant All  on database db_admin to db_admin;

-- 被授权者可以建得到的权限再次授予别人
grant All on all tables in schema public to db_admin with grant option;

-- 授予所有权限
set search_path='public';
drop table if exists t_test;
create table t_test(id int, age int);
grant all on public.t_test to db_admin;

-- all指代所有对象
grant select, update on all sequences in schema public to public;

-- public指代所有角色
grant all on schema public to public;

-- 取消授权
revoke all on database db_admin from public;

-- 修改默认权限
alter default privileges in schema public
grant select, update on sequences to public;
```

## 扩展包

网址: pgxn.org

常见扩展包

- btree_gist GIST索引
- btree_gin GIN索引
- postgis 空间数据库
- fuzzystrmatch 字符串模糊匹配
- hstore 键值数据库
- pg_trgm 字符串模糊搜索
- dblink 访问另一台数据库
- pgcrypto 加密工具
- tsearch 全文搜索
- xml XML支持

```sql
-- 查看可用扩展包
select * from pg_available_extensions;

-- 查看详细信息
\dx+ plpgsql

-- 安装扩展
create extension fuzzystrmatch schema fuzzystrmatch;
```

## 备份恢复

pg_dump不支持命令行密码,两个方案：

- `~/.pgpass`
- PGPASSWORD

```sql
CREATE TABLE t1 (
    id integer,
    age integer
);

INSERT INTO t1 VALUES (1, 10);
INSERT INTO t1 VALUES (2, 20);
INSERT INTO t1 VALUES (3, 30);
```

```shell
pg_dump --help

pg_dump -h localhost -p 5432 -U wang -F c -b -v -f db_test.backup db_test

pg_dump --verbose --column-inserts --dbname=db_test --table=t1 --file=t1.sql 

# 全库备份
pg_dumpall

# 恢复
psql -U postgres -f database.sql

pg_restore --dbname --verbose mydb.backup
```

表空间

```sql
-- 创建表空间
create tablespace secondary location '/usr/data/pgdata_secondary';

-- 迁移表空间
alter database mydb set tablespace secondary;
alter table t1 set tablespace secondary;
alter tablespace pg_default move all to secondary;
```

默认表空间
- pg_default 存储所有用户级数据
- pg_global 存储所有系统级数据

## PSQL工具

环境变量

```shell
PGHOST 主机
PGPORT 端口号
PGUSER 用户
PGPASSWORD 密码
PSQL_HISTORY psql命令日志 默认：~/.psql_history
PSQLRC 配置文件 默认：linux: ~/.psqlrc / windows: psqlrc.conf
```

示例

```shell
# ~/.psqlrc
\timing on
```

非交互式

```shell
# 执行脚本文件
psql -f test.sql

# 执行SQL语句
psql -c "sql语句"
```

交互式

```sql
-- 交互式支持的命令
\?

-- 查看帮助
\h 命令关键字

\a -- 关闭对齐模式
\t -- 忽略标题栏输出
\g test.sql -- 查询内容输出到指定文件
\i test.sql  -- 执行指定脚本文件，等价于-f
\timing on/off -- 语句执行时间
\set AUTOCOMMIT off -- 关闭事务自动提交
\! -- 执行shell命令

select datname, query from pg_stat_activity
 where state = 'avtive' and pid != pg_backend_pid();
-- 每3秒执行一次命令
\watch 3

-- 每5秒记录一次系统负载
select * into log_activity from pg_stat_activity;
insert into log_activity select * from pg_stat_activity;
\watch 5;

-- 查询表信息
\dt+  pg_catalog.pg_t*

-- 查询对象详细信息
\d+ pg_stat_activity

-- 切换目录
\cd dir

-- 导出数据
\copy (select * from t1) to t1.csv with csv header;

-- 导入数据
\copy table_name from t1.csv delimiter ',' null as '';

-- 导入文件夹列表
create table t_file_list (filename text);
\copy t_file_list from program 'ls $(pwd)'


-- 导出html
psql db_test -c 'select * from t1' --html  -o t1.html

-- 导出csv
psql db_test -c 'select * from t1' --csv  -o t1.csv
```

界面提示符

```sql
\set PROMPT1 '%n@%M:%>%x %/# '
%n 登录角色
@ 字符
%M 主机名
: 字符
%> 侦听端口
%x 事务状态
%/ 当前database
# 字符

效果：
wang@[local]:5432 db_test# 
```

工具
- pgAdmin 可视化管理工具
- pgScript 脚本
- pgAgent 定时任务

serial 类型

```sql
create table t3(
    id serial, -- autonumber
    age int
)

insert into t3(age) values(10);
insert into t3(age) values(11);

select * from t3;
 id | age 
----+-----
  1 |  10
  2 |  11
(2 rows)
```

generate_series 生成数组序列

```sql

select x from generate_series(1, 3, 1) as x;
 x 
---
 1
 2
 3
(3 rows)
```

repeat

```sql
select repeat('x', 3);
 repeat 
--------
 xxx
(1 row)
```

split_part 按分隔符取值

```sql
select split_part('127.0.0.1', '.', '4');
 split_part 
------------
 1
(1 row)
```

string_to_array 将字符串拆分

```sql

select unnest(string_to_array('127.0.0.1', '.'));
 unnest 
--------
 127
 0
 0
 1
(4 rows)
```

regexp_replace 正则表达式

```sql
select regexp_replace(
    '0123456789',
    '([0-9]{3})([0-9]{3})([0-9]{4})',
    E'\(\\1\) \\2-\\3'
) as x;
       x        
----------------
 (012) 345-6789
(1 row)
```

数组

```sql
select array[0, 1, 2] as arr;
   arr   
---------
 {0,1,2}
(1 row)
```

json

```sql
create table t_person(
    id serial primary key,
    profile json
);

insert into t_person (profile)
values('{"name": "Tom", "age": 18}');

select
json_extract_path_text(profile, 'name') as name
from t_person;
 name 
------
 Tom
(1 row)

select
profile->>'name' as name
from t_person;
 name 
------
 Tom
(1 row)
```

xml

```sql
-- ./configure的时候需要加：--with-libxml
-- This functionality requires the server to be built with libxml support.
drop table if  exists t_person;
create table t_person(
    id serial primary key,
    profile xml
);

insert into t_person (profile)
values('<person><name>Tom</name><age>18</age></person>');

select
(xpath('person/name/text()', profile))[1]::text as name
from t_person;
 name 
------
 Tom
(1 row)
```

所有的表都是一个自定义类型
```sql
drop table if exists t_school;
create table t_school(
    id serial primary key,
    name text
);

drop table if exists t_person;
create table t_person(
    id serial primary key,
    name text,
    school t_school
);

insert into t_person (name, school)
values('Tom', ROW(1, 'PKU')::t_school);

select * from t_person;
 id | name | school  
----+------+---------
  1 | Tom  | (1,PKU)
(1 row)
```

自定义类型和运算符

```sql
create type type_point as (x int, y int);

drop table if exists t_circle;
create table t_circle(
    center type_point,
    radius int
)

insert into t_circle(center, radius)
values(ROW(1, 3)::type_point, 4);

select center, radius from t_circle;
 center | radius 
--------+--------
 (1,3)  |      4
(1 row)

select (center).*, radius from t_circle;
 x | y | radius 
---+---+--------
 1 | 3 |      4
(1 row)

create or replace function add_point(a type_point, b type_point)
returns type_point as
$$
    select ((a.x + b.x), (a.y + b.y))::type_point;
$$
language sql;

create operator +(
    procedure=add_point,
    leftarg=type_point,
    rightarg=type_point,
    commutator=+
);

select (1, 3)::type_point + (2, 4)::type_point;
 ?column? 
----------
 (3,7)
(1 row)
```

建表
- 基本建表 create table
- 继承表 inherits
- 无日志表 unlogged
- type of

约束：
- 外键约束 foreign key
- 唯一约束 unique
- check约束 check
- 排他性约束 

索引
- B-树索引
- GiST索引 Generalized Search Tree 通用搜索树
- GIN索引 Generalized Inverted Index 通用逆序索引
- SP-GiST索引 Space-Partitioning Trees 空间分区树
- 哈希索引
- 函数索引
- 部分索引
- 多列索引

```sql
-- 查询支持的语言列表
select lanname from pg_language;
```

支持：
- sql
- plsql
- c
- python
- javascript
- R
- java
- sh
- tsql
- perl

explain查看执行计划

表格显示执行计划：http://explain.depesz.com

```sql
drop table if exists t1;
create table t1(
    id serial, -- autonumber
    age int
);

insert into t1(age) values(10);
insert into t1(age) values(11);
insert into t1(age) values(12);

explain (analyze, verbose, buffers)
select * from t1 where id = 1;

explain (analyze, verbose, buffers)
select * from t1 where age = 10;
Seq Scan on public.t1  
    -- 该步骤起始执行时间..该步骤总执行时间
    -- cost 估算执行时间
    -- actual 实际执行时间
    (cost=0.00..38.25 rows=11 width=8) 
    (actual time=0.091..0.095 rows=1 loops=1)
   Output: id, age
   Filter: (t1.age = 10)
   Rows Removed by Filter: 2
   Buffers: shared hit=1
 Planning:
   Buffers: shared hit=30
 Planning Time: 1.743 ms
 Execution Time: 1.237 ms
(9 rows)

```

pg_stat_statements执行统计信息

```sql
# postgresql.conf

#shared_preload_libraries = ''      # (change requires restart)
shared_preload_libraries = 'pg_stat_statements'
pg_stat_statements.max = 10000
pg_stat_statements.track = all

-- 安装扩展
create extension pg_stat_statements;

-- 视图
select * from pg_stat_statements;
-- 函数
select pg_stat_statements_reset();
```

查看索引是否被用到
```sql
drop table if exists t1;
create table t1(
    id serial, -- autonumber
    age int
);

insert into t1(age) values(10);
insert into t1(age) values(11);
insert into t1(age) values(12);

create index idx_t1_age on t1(age);
set enable_seqscan=off;

explain select * from t1 where id = 1;
select * from t1 where id = 1;
                    QUERY PLAN                    
--------------------------------------------------
 Seq Scan on t1  (cost=0.00..1.04 rows=1 width=8)
   Disabled: true
   Filter: (id = 1)
(3 rows)

explain select * from t1 where age = 1;
select * from t1 where age = 1;
                QUERY PLAN      
----------------------------------
 Index Scan using idx_t1_age on t1  (cost=0.13..8.15 rows=1 width=8)
   Index Cond: (age = 1)
(2 rows)

\x
select * from pg_stat_user_indexes where relname = 't1' limit 1;

\x
select * from pg_stat_user_tables where relname = 't1' limit 1;
```

pg_stats 表的统计信息

```sql
-- vacuum: 将已删除的记录永久性的从表中移除
-- analyze: 更新表的统计信息
vacuum analyze;

select attname, n_distinct, most_common_vals, most_common_freqs
from pg_stats where tablename = 't1';
 attname | n_distinct | most_common_vals | most_common_freqs 
---------+------------+------------------+-------------------
 id      |         -1 |                  | 
 age     |         -1 |                  | 
(2 rows)
```

random_page_cost 随机页访问成本比RPC
物理磁盘速度越快，该比率就越小

```sql
show random_page_cost;
 random_page_cost 
------------------
 4
(1 row)
```

pg_buffercache 数据缓存扩展
pg_prewarm 预先缓存到内存扩展
shared_buffers 缓存数据内存大小

```sql
select * from pg_buffercache limit 1;
```

超大尺寸属性存储技术
TOAST the oversized-attribute storage techenique
