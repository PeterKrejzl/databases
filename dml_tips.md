As I'm coming from Microsoft SQL Server environment, working with MySQL is constantly surprising me in many ways, both good and bad :) In contrast to MS SQL we need to focus significantly more on the performance of our queries and be prepared that everything is not as easy as in MS SQL world :)

Here are just a couple of interesting MySQL features I found useful for my work. This time they are all related to inserting or updating data.

##### INSERT IGNORE
A row that duplicates an existing unique index or primary key is ignored and no warning generated.

**!!!!!** Do not use when inserting into table with auto generated primary key column (either single or composite). It will still generate an id but never insert it.

##### INSERT ON DUPLICATE UPDATE
Similar to INSERT IGNORE but moreover it you can specify an update clause like:

`insert into User (col1, col2, col3) values (v1, v2, v3) on duplicate update col3 = current_timestamp()`

##### REPLACE (= insert or delete + insert)
If there is no such row in the table it is inserted. If it already exists, it is deleted first and then reinserted.

##### INSERT DELAYED
Useful for clients that cannot wait until insert is done. Server confirms the query and then it is put into the queue until a table is not used by other processes. It is obviously slower than a normal INSERT and also these rows until inserted are stored in the memory only. So, there is a higher chance of losing them.

##### INSERT LOW_PRIORITY
Low_priority insert execution is delayed until no other clients are reading from the table. It is possible that such an insert will wait for a long time (theoretically forever :) )

##### Examples
```
create table if not exists tmp_test_peter 
(
ID int unsigned not null primary key auto_increment,
Record varchar(100) character set utf8,
ModifiedDate datetime not null default current_timestamp(),
unique index un_Record (Record)
);
```


##### INSERT IGNORE
```
insert into tmp_test_peter (Record) values ('record 1');
insert into tmp_test_peter (Record) values ('record 1');
```

As expected we will get:

`Error Code: 1062. Duplicate entry 'record 1' for key 'un_Record'`

One more try with **IGNORE** now. 
`insert IGNORE into tmp_test_peter (Record) values ('record 1');`
`0 row(s) affected`

No records inserted but also no errors.

Let's run it a couple of more times:
```
insert IGNORE into tmp_test_peter (Record) values ('record 1');
insert IGNORE into tmp_test_peter (Record) values ('record 1');
insert IGNORE into tmp_test_peter (Record) values ('record 1');
insert IGNORE into tmp_test_peter (Record) values ('record 1');
```

Still ok :) But what if we run another insert with different value:

```
insert IGNORE into tmp_test_peter (Record) values ('record 2');
select * from tmp_test_peter
```

No errors, a row inserted but let's select from the table. 

You see it? Newly inserted record has an ID = 11 meaning all these ignored inserts incremented identity column in the table!
`11 record 2 2016-02-02 07:54:17`

##### INSERT ON DUPLICATE UPDATE
```
insert into tmp_test_peter (Record)
values ('record 1')
on duplicate key update ModifiedDate = current_timestamp()

select * from tmp_test_peter
```
##### REPLACE

```
replace into tmp_test_peter (Record) values ('record 3');

select * from tmp_test_peter
```
