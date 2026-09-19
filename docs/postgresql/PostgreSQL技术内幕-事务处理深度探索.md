# PostgreSQL技术内幕-事务处理深度探索

事务分类：

- 隐式事务：一个独立SQL语句，执行结束自动提交
- 显示事务：一组SQL语句，形成事务块

事务块 TransactionBlock

```sql
-- 事务块开始
begin

-- 事务块结束
end
commit
abort
rollback
```

示例

```sql
begin;
drop table if exists t1;
create table t1(id int, age int);
insert into t1(id, age) values(1, 1);
select * from t1 where id = 1;
update t1 set age = age + 1 where id = 1;
end;
```

事务中报错

```sql
begin;
drop table if exists t1;
create table t1(id int, age int);
create unique index t1_unique_id on t1(id);
insert into t1(id, age) values(1, 1);
insert into t1(id, age) values(1, 1);
-- ERROR:  duplicate key value violates unique constraint "t1_unique_id"
-- DETAIL:  Key (id)=(1) already exists.
end;
```

事务处理3个层次：

上层：处理显示的事务块状态

```cpp
// src/include/access/xact.h

extern void BeginTransactionBlock(void);
extern bool EndTransactionBlock(bool chain);
extern bool PrepareTransactionBlock(const char *gid);
extern void UserAbortTransactionBlock(bool chain);

extern void ReleaseSavepoint(const char *name);
extern void DefineSavepoint(const char *name);
extern void RollbackToSavepoint(const char *name);
```

中层：

```cpp
// src/include/access/xact.h

extern void StartTransactionCommand(void);
extern void CommitTransactionCommand(void);
extern void AbortCurrentTransaction(void);
```

底层：维护事务状态

```cpp
// src/backend/access/transam/xact.c

static void StartTransaction(void);
static void CommitTransaction(void);
static void CleanupTransaction(void);
static void AbortTransaction(void);

static void StartSubTransaction(void);
static void CommitSubTransaction(void);
static void AbortSubTransaction(void);
static void CleanupSubTransaction(void);
```

SAVEPOINT

```cpp
// src/backend/access/transam/xact.c

static void PushTransaction(void);
static void PopTransaction(void);
```

查看事务状态

```sql
select txid_current();

select txid_status(txid_current());
 txid_status 
-------------
 in progress
(1 row)
```

```cpp
// txid_current调用栈
pg_current_xact_id
GetTopFullTransactionId
AssignTransactionId // 给XactTopFullTransactionId 赋值
GetNewTransactionId
```
