# note
### note for stored procedure postgresql

## describe table

```
                    Table "public.satuan_kerja"
 Column  |          Type          | Collation | Nullable | Default 
---------+------------------------+-----------+----------+---------
 id      | integer                |           |          | 
 name    | character varying(255) |           |          | 
 catatan | character varying(255) |           |          | 
```


## block structure
block structure start with <<first_block>> end first_block like this :

query input:
```
do $$
-- label
<<first_block>> 
declare
satker_count integer := 0;
begin
-- get the number of satker
select count(*)
into satker_count
from satuan_kerja;
-- display a message
raise notice 'The number of satker is %',satker_count;
-- end of lavel
end first_block $$;
```

result:
```
NOTICE:  The number of satker is 4
DO

Query returned successfully in 39 msec.

```
## %type

The %type provides the data type of a table column or another variable. Typically, you use the %type to declare a variable that holds a value from the database or another variable.

query input:

```
do $$
declare
	name_ satuan_kerja.name%type;
begin
	select name
	from satuan_kerja
	into name_
	where id=1;
	
	raise notice 'total satker adalah %', name_;
end $$;
```

result:


```
NOTICE:  total satker adalah satker 5
DO

Query returned successfully in 90 msec.
```

## %rowtype

To store the whole row of a result set returned by the select into statement, you use the row-type variable or row variable.

You can declare a variable that has the same datatype as the datatype of the row in a table by using the following syntax:

query input:
```
do $$
declare 
	selected_name satuan_kerja%rowtype;
begin
	select *
	from satuan_kerja
	into selected_name
	where id=1;
	
	raise notice 'The name % and catatan %',
	selected_name.name,
	selected_name.catatan;
end; $$

```

result:
```
NOTICE:  The name satker 5 and catatan bagian data
DO

Query returned successfully in 47 msec.
```

## Record Types

in this tutorial, you will learn about the PL/pgSQL record types that allow you to define variables that can hold a single row from a result set.
Let’s take some examples of using the record variables.

- Using record with the select into statement
    
    query input:
    ```do
    $$
    declare
        rec record;
    begin 
        select id,name,catatan
        into rec
        from satuan_kerja
        where id=1;
        
        raise notice '% % %', rec.id,rec.name,rec.catatan;
    end;
    $$
    language plpgsql;
    ```
    result:
    ``` 
        NOTICE:  1 satker 5 bagian data
        DO

        Query returned successfully in 35 msec.
    ```

- Using record variables in the for loop statement

    query input
    ```
    do
    $$
    declare
        rec record;
    begin
        for rec in select name
                        from satuan_kerja
                        where id > 0
                        order by id asc
        loop
            raise notice '%', rec.name;
        end loop;
    end;
    $$
    ```
    result:
    ```
    NOTICE:  satker 5
    NOTICE:  test kesekian kalinya
    NOTICE:  maluku
    NOTICE:  satker x
    DO

    Query returned successfully in 63 msec.
    ```
### summary
- A record is a placeholder that can hold a single row of a result set.
- A record has not predefined structure like a row variable. Its structure is determined when you assign a row to it.

## Errors and Messages


PostgreSQL provides the following levels:

- debug
- log
- notice
- info
- warning
- exception

query input:

```
do $$ 
begin 
  raise info 'information message %', now() ;
  raise log 'log message %', now();
  raise debug 'debug message %', now();
  raise warning 'warning message %', now();
  raise notice 'notice message %', now();
end $$;
```
result:

```
INFO:  information message 2022-07-27 08:24:27.928013+00
WARNING:  warning message 2022-07-27 08:24:27.928013+00
NOTICE:  notice message 2022-07-27 08:24:27.928013+00
DO

Query returned successfully in 37 msec.
```

### exception
query input:
```
do $$
declare
	email varchar(255) := 'rizkydarmadi@gmail.com';
begin
	-- check email for duplicate
	raise exception 'duplicate email: %',email
			using hint = 'check email again';

end $$;
```
result:
```
ERROR:  duplicate email: rizkydarmadi@gmail.com
HINT:  check email again
CONTEXT:  PL/pgSQL function inline_code_block line 6 at RAISE
SQL state: P0001
```
## Assert
The assert statement is a useful shorthand for inserting debugging checks into PL/pgSQL code.

The following illustrates the syntax of the assert statement:

`assert condition [, message];`

query input:
```
do $$
declare 
	satker_count integer;
begin
	select count(*)
	into satker_count
	from satuan_kerja;

	assert satker_count > 0, 'satker not found';
end$$;
```

result:
```
DO

Query returned successfully in 48 msec.
```

## if-else statement

in this tutorial, you will learn how to use the PL/pgSQL if statements to execute a command based on a specific condition.

The if statement determines which statements to execute based on the result of a boolean expression.


The following illustrates the simplest form of the if statement:

- PL/pgSQL if-then statement

    ```
    if condition then
    statements;
    end if;
    ```
    query_input:
    ```
    do $$
    declare
        selected_satker satuan_kerja%rowtype;
        satker_id satuan_kerja.id%type := 0;
    begin

        select * from satuan_kerja
        into selected_satker
        where id = satker_id;
        
        if not found then
            raise notice 'satker id % not found',
                satker_id;
        end if;
    end $$
    ```
    result:
    ```
    NOTICE:  satker id 0 not found
    DO

    Query returned successfully in 71 msec.
    ```
- PL/pgSQL if-then-else statement

    ```
    if condition then
    statements;
    else
    alternative-statements;
    END if;
    ```

    query_input:
    ```
    do $$
    declare
        selected_satker satuan_kerja%rowtype;
        input_satker_id satuan_kerja.id%type := 1;
    begin
        select * from satuan_kerja
        into selected_satker
        where id = input_satker_id;
        
        if not found then
            raise notice 'The satker % could not be found',
                input_satker_id;
        else
            raise notice 'The satker name is %', selected_satker.name;
        end if;
    end $$
    ```
    result:
    ```
    NOTICE:  The satker name is satker 5
    DO

    Query returned successfully in 62 msec.
    ```
