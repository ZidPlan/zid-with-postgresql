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
CREATE TABLE t1 (
   id bytea DEFAULT zid(128) PRIMARY KEY,
   name text
);
```

### Insert

Insert an example row:

```sql
insert into t1 values (default, 'alice');
```

### Read

Read results:

```sql
select * from t1;
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
CREATE TABLE t2 (
   id uuid DEFAULT zid_as_uuid() PRIMARY KEY,
   name text
);
```

### Insert

Insert an example row:

```sql
insert into t2 values (default, 'alice');
```

### Read

Read results:

```sql
select * from t2;
```

### Caveat

Note that the output UUID will likely be invalid because the zid is completely random and deliberately isn't compatible with the UUID specification.
