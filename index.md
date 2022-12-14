### Prepare

```
drop table t1;
drop table t2;

create table t1(id int primary key, c1 int, c2 char(255), redistributed key (c1) to group g1);
create table t2(id int primary key, c1 int, c2 char(255), redistributed key (c1) to group g1);

alter table t1 set tiflash replica 1;
alter table t2 set tiflash replica 1;
```

### Query

```
show create table t1;

insert into t1 values (1, 101, 'hello'), (2, 102, 'world');
insert into t2 values (1, 101, 'foo'), (2, 102, 'bar');

set tidb_mpp_enable_redistributed_index=OFF;
explain select * from t1 join t2 on t1.c1 = t2.c1;
```
