oem@asus:~$ ssh -i ~/key admin@158.160.136.35
admin@avdonina-vm:~$ sudo -u postgres psql
postgres=# CREATE DATABASE hw1;
CREATE DATABASE
postgres=# \c hw1
hw1=# CREATE TABLE persons(id serial, first_name text, second_name text);
CREATE TABLE
hw1=# INSERT into persons(first_name,second_name)values('ivan','ivanov');
INSERT 0 1
hw1=# insert into persons (first_name,second_name)values('petr','petrov');
INSERT 0 1
hw1=# commit;
hw1=# \set AUTOCOMMIT OFF
hw1=# show transaction isolation level;
 transaction_isolation 
-----------------------
 read committed
hw1=# BEGIN;
BEGIN
hw1=*# insert into persons (first_name,second_name)values('sergey','sergeev');
INSERT 0 1
Вторая консоль:
hw1=# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 rows)
Новую строку не вижу, т.к. она не зафиксирована.
Зафиксировала в первой консоли
теперь вижу во второй:
hw1=# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  4 | sergey     | sergeev
(3 rows)
т.к. зафиксировано.

В первой консоли:
hw1=# BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN
hw1=*# insert into persons (first_name,second_name)values('sveta','svetova');
INSERT 0 1
hw1=*# show transaction isolation level;
 transaction_isolation 
-----------------------
 repeatable read
(1 row)
hw1=# BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN
hw1=*# insert into persons (first_name,second_name)values('sveta','svetova');
INSERT 0 1
hw1=*# show transaction isolation level;
 transaction_isolation 
-----------------------
 repeatable read
(1 row)
Вторая консоль:
hw1=# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  4 | sergey     | sergeev
(3 rows)
Новвой строки нет, т.к. не зафиксировано.
Фиксирую в первой консоли.
Теперь во второй консоли видно новую строку:
hw1=# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  4 | sergey     | sergeev
  5 | sveta      | svetova




