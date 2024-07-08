# ZID with PostgreSQL

Zero-Information-Data with PostgreSQL database.


## Start psql

Start a typical PostgreSQL server and connect to it.

Example:

```sh
psql
```


## Load pgcrypto

Load the PostgreSQL cryptography extension:

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
```


## Generate random bytes

Generate some random bytes by using the pgcrypto function gen_random_bytes:

```sql
select gen_random_bytes(16);
```

Output:

```sql
          gen_random_bytes          
------------------------------------
 \xedc264fffe7976adf042fff2fb8b1b06
(1 row)
```


## Create zid function

Create a function name `zid` with an integer parameter that indicates the number of bits:


```sql
CREATE FUNCTION zid(int) RETURNS bytea
    LANGUAGE SQL
	RETURN gen_random_bytes($1/8);
```

Call the function:

```sql
select zid(128);
```

Output:

```sql
                zid                 
------------------------------------
 \xf7532ae3a3dfec14959a58434a6ac350
(1 row)
```


### Create an example table

Create an example table with a byte array as the primary key:

```sql
CREATE TABLE t (
   id bytea DEFAULT zid(128) PRIMARY KEY
);
```


### Insert

Insert an example row:

```sql
insert into t values (default);
```


### Read

Read results:

```sql
select * from t;
```


## Create zid_as_uuid function

Create a function name `zid_as_uuid` that generates a zid and converts it into a uuid type:

```sql
CREATE FUNCTION zid_as_uuid() RETURNS uuid
    LANGUAGE SQL
	RETURN CAST(substring(CAST(gen_random_bytes(16) AS text) from 3) AS uuid);
```


### Create an example table

Create an example table with a UUID data type as the primary key:

```sql
CREATE TABLE  (
   id uuid DEFAULT zid_as_uuid() PRIMARY KEY
);
```


### Insert

Insert an example row:

```sql
insert into t values (default);
```


### Read

Read results:

```sql
select * from 2;
```


### Caveat

Note that the output UUID will likely be invalid because the zid is completely random and deliberately isn't compatible with the UUID specification.



## Compare uuid-ossp

Load the uuid-ossp extension:

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```


### Create an example table

Create an example table with a UUID data type as the primary key:

```sql
CREATE TABLE t (
   id uuid DEFAULT uuid_generate_v4() PRIMARY KEY
);
```


### Insert

Insert an example row:

```sql
insert into t values (default);
```


### Read

Read results:

```sql
select * from t;
```


## Benchmarks


### Create tables

Create tables with the various kinds of primary keys:

```sql
CREATE TABLE benchmark_zid_128 (
   id bytea DEFAULT zid(128) PRIMARY KEY
);

CREATE TABLE benchmark_zid_as_uuid (
   id uuid DEFAULT zid_as_uuid() PRIMARY KEY
);

CREATE TABLE benchmark_uuid_generate_v4 (
   id uuid DEFAULT uuid_generate_v4() PRIMARY KEY
);
````


### Configure

Turn on timing:

```psql
\timing
```

Turn off pager:

```psql
\pset pager 0
```


### Benchmark insert

Benchmark a million insert statements for each table:

```psql
do $$
begin
for i in 1..1000000 loop
insert into benchmark_zid_128 values(default);
end loop;
end;
$$;
-- Time: 4549.356 ms (00:04.549)

do $$
begin
for i in 1..1000000 loop
insert into benchmark_zid_as_uuid values(default);
end loop;
end;
$$;
-- Time: 5349.053 ms (00:05.349)

do $$
begin
for i in 1..1000000 loop
insert into benchmark_uuid_generate_v4 values(default);
end loop;
end;
$$;
-- Time: 5897.088 ms (00:05.897)
```

| Algorithm   | Time |
|-------------|------|
| zid_128     | 4549 |
| zid_as_uuid | 5349 |
| uuid_v4     | 5897 |

Summary: benchmarks are similar.


### Benchmark inner join

Benchmark select statements for all million randomly-ordered ids:

```psql
DROP TABLE IF EXISTS ids;
SELECT id INTO ids FROM benchmark_zid_128 order by random();
SELECT count(*)  FROM benchmark_zid_128 INNER JOIN ids ON benchmark_zid_128.id = ids.id;
-- Time: 251.617 ms

DROP TABLE IF EXISTS ids;
SELECT id INTO ids FROM benchmark_zid_as_uuid order by random();
SELECT count(*)  FROM benchmark_zid_as_uuid INNER JOIN ids ON benchmark_zid_as_uuid.id = ids.id;
-- Time: 244.740 ms

DROP TABLE IF EXISTS ids;
SELECT id INTO ids FROM benchmark_uuid_generate_v4 order by random();
SELECT count(*)  FROM benchmark_uuid_generate_v4 INNER JOIN ids ON benchmark_uuid_generate_v4.id = ids.id;
-- Time: 236.810 ms
```

| Algorithm   | Time |
|-------------|------|
| zid_128     | 251  |
| zid_as_uuid | 244  |
| uuid_v4     | 236  |

Summary: benchmarks are similar.
