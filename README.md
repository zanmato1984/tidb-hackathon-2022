# TiDB Hackathon 2022 Demo

## Redistributed Index

### Connection

```
mysql -h 172.16.4.99 -P 9025 -u root -Dtest --prompt="tidb> "

set tidb_broadcast_join_threshold_count=0;
set tidb_broadcast_join_threshold_size=0;
```

### Create Table

```
drop table t1;
drop table t2;

create table t1(id int primary key, c1 int, c2 char(255)) redistributed by (c1) to group g1;
create table t2(id int primary key, c1 int, c2 char(255)) redistributed by (c1) to group g1;

alter table t1 set tiflash replica 1;
alter table t2 set tiflash replica 1;
```

### Data & Plan

```
set tidb_mpp_enable_redistributed_index=OFF;
explain select count(*) from t1 group by c1;

insert into t1 values (1, 101, 'hello'), (2, 102, 'world');

set tidb_mpp_enable_redistributed_index=ON;
explain select count(*) from t1 group by c1;
select count(*) from t1 group by c1;

set tidb_mpp_enable_redistributed_index=OFF;
explain select * from t1 join t2 on t1.c1 = t2.c1;

insert into t2 values (1, 101, 'foo'), (2, 102, 'bar');

set tidb_mpp_enable_redistributed_index=ON;
explain select * from t1 join t2 on t1.c1 = t2.c1;
select * from t1 join t2 on t1.c1 = t2.c1;
```

## Performance

### Connection

* Original

```
mysql -h 172.16.4.99 -P 9025 -u root -Dtpch_100_multi_key --prompt="tidb> "

set tidb_mpp_enable_redistributed_index=OFF;
```

* Optimized
```
mysql -h 172.16.4.99 -P 9025 -u root -Dtpch_100_multi_key --prompt="tidb(opt.)> "

set tidb_mpp_enable_redistributed_index=ON;
```

### TPCH Q8

```
select
	o_year,
	sum(case
		when nation = 'INDIA' then volume
		else 0
	end) / sum(volume) as mkt_share
from
	(
		select
			extract(year from o_orderdate) as o_year,
			l_extendedprice * (1 - l_discount) as volume,
			n2.n_name as nation
		from
			part,
			supplier,
			lineitem,
			orders,
			customer,
			nation n1,
			nation n2,
			region
		where
			p_partkey = l_partkey
			and s_suppkey = l_suppkey
			and l_orderkey = o_orderkey
			and o_custkey = c_custkey
			and c_nationkey = n1.n_nationkey
			and n1.n_regionkey = r_regionkey
			and r_name = 'ASIA'
			and s_nationkey = n2.n_nationkey
			and o_orderdate between '1995-01-01' and '1996-12-31'
			and p_type = 'SMALL PLATED COPPER'
	) as all_nations
group by
	o_year
order by
	o_year;
```

### Hand-made Query

```
```
