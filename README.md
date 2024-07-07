# ZID with PostgreSQL

Zero-Information-Data with PostgreSQL database.


## Steps

Start a typical PostgreSQL server and connect to it.

Example:

```sh
psql
```

Use PostgreSQL cryptography:

```psql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
```

Generate random bytes such as for 16 bytes a.k.a. 128 bits:

```psql
select gen_random_bytes(16);
```

Create a function with a parameter:


```psql
CREATE FUNCTION zid(int) RETURNS bytea
    LANGUAGE SQL
	RETURN gen_random_bytes($1/8);
```

Call the function:

```psql
select zid(128);
```

Output:

```psql
                zid                 
------------------------------------
 \xf7532ae3a3dfec14959a58434a6ac350
(1 row)
```
